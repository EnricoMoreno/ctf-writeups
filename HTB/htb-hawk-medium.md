# Report — Máquina: `Hawk` — `10.129.95.193`

- **Fonte:** Hack The Box (HTB)
    
- **Data:** 2025-12-18
    
- **Autor:** Enrico Moreno
    
- **Nível:** Medium
    
---

## Executive Summary

A máquina **Hawk** foi comprometida através de uma cadeia de falhas envolvendo **exposição de credenciais via FTP**, **execução de código em Drupal** e **execução remota de comandos no H2 Database Console**. O ataque resultou inicialmente em acesso como `www-data`, posterior movimentação lateral via SSH e escalonamento final para **root** explorando um serviço H2 executando com privilégios elevados.

---

## Planning & Scoping

- **Alvo:** `10.129.95.193`
    
- **Sistema operacional:** Linux
    
- **Ferramentas utilizadas:** nmap, ftp, OpenSSL, bruteforce-salted-openssl, Drupal, SSH, Metasploit / Exploit-DB
    
- **Limitações:** sem brute force externo, sem pivoting externo
    

---

## Discovery / Recon

### Nmap — Varredura completa

```bash
sudo nmap -sS -T4 -p- -sV -Pn 10.129.95.193 -oN Nmap_Hawk.txt
```

Serviços identificados:

- FTP (login anônimo habilitado)
    
- SSH
    
- HTTP (Drupal)
    
- H2 Database Console (porta 8082 — acesso local)
    
- Output completo: `artifacts/Nmap_Hawk.txt`

![](Nmap_Hawk.txt)

---

## Enumeration & Analysis

### FTP — Credenciais Expostas

Acesso anônimo ao FTP revelou um arquivo oculto:

```
.drupal.txt.enc
```

O arquivo estava codificado em **Base64** e criptografado com **OpenSSL salted**.

Foi utilizado o **bruteforce-salted-openssl** para identificar a senha:

```
friends
```

Após descriptografia, o arquivo revelou credenciais válidas para o portal **Drupal**.

- Arquivo descriptografado: `artifacts/drupal.txt`

![](drupal.txt)

---

## Attack / Exploitation

### Drupal — Execução de Código

Com acesso administrativo ao Drupal:

- O módulo **PHP Filter** foi habilitado
    
- Código PHP arbitrário pôde ser executado
    

Isso permitiu a obtenção de uma **reverse shell** como:

```
www-data
```

---

## Post-Exploitation

### Credenciais em Configurações do Drupal

Durante a enumeração, o arquivo:

```
settings.php
```

revelou credenciais do banco MySQL:

```
drupal : drupal4hawk
```

As credenciais foram reutilizadas para autenticação via **SSH** como o usuário:

```
daniel
```

### User Flag

```
/home/daniel/user.txt
```

---

## Privilege Escalation

### H2 Database Console (Local)

Foi identificado que o **H2 Database Console** estava rodando localmente na porta **8082**.

Utilizando **SSH port forwarding**, o serviço foi exposto localmente:

```bash
ssh -L 8082:127.0.0.1:8082 daniel@10.129.95.193
```

O serviço foi explorado utilizando o **Exploit-DB #45506**, abusando da funcionalidade:

```
CREATE ALIAS
```

Isso permitiu execução remota de comandos.

Como o serviço estava sendo executado como **root**, foi possível obter uma shell privilegiada.

---

## Impact & Root Access

- **Impacto:** Comprometimento total do sistema
    
- **Nível de severidade:** Crítico
    
- **Motivo:** Cadeia de credenciais expostas + RCE em serviços administrativos
    

---

## Flags Obtidas

### User Flag

```
/home/daniel/user.txt
```

### Root Flag

```
/root/root.txt
```

---

## Remediation Recommendations

- Desabilitar login anônimo no FTP
    
- Nunca armazenar credenciais em arquivos acessíveis
    
- Restringir módulos perigosos no Drupal (ex.: PHP Filter)
    
- Não executar serviços administrativos como root
    
- Restringir acesso ao H2 Database Console
    

---

## Lessons Learned

- Serviços auxiliares frequentemente expõem dados sensíveis
    
- Reutilização de credenciais facilita movimentação lateral
    
- Port forwarding é essencial para explorar serviços locais
    
- Consoles administrativos expostos representam alto risco
    

---

## Summary

A máquina **Hawk** demonstra um ataque em camadas envolvendo enumeração cuidadosa, reutilização de credenciais e exploração de serviços administrativos mal configurados. O laboratório reforça a importância de hardening em serviços auxiliares e controle rigoroso de privilégios.
