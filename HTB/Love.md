# Report — Máquina: `Love` — `10.129.48.103`

- **Fonte:** Hack The Box (HTB)
    
- **Data:** 2025-12-18
    
- **Autor:** Enrico Moreno
    
- **Nível:** Easy

---

## Executive Summary

A máquina **Love** foi comprometida explorando uma falha de **SSRF** em um serviço de _staging_ interno, que expôs credenciais administrativas. Com essas credenciais, foi possível obter uma **shell reversa** via aplicação web vulnerável e, posteriormente, escalar privilégios até **NT AUTHORITY\SYSTEM** por meio da política **AlwaysInstallElevated**, resultando na obtenção das flags de usuário e root.

---

## Planning & Scoping

- **Alvo:** `10.129.48.103`
    
- **Sistema operacional:** Windows
    
- **Ferramentas utilizadas:** nmap, gobuster, Metasploit, certutil
    
- **Limitações:** sem brute force externo, sem pivoting
    

---

## Discovery / Recon

### Nmap — Varredura completa

```bash
sudo nmap -sS -T4 -p- -sV -Pn 10.129.48.103 -oN ~/Desktop/Nmap_Love.txt
```

**Resultado relevante:**

```
80/tcp    open  http   Apache httpd 2.4.46 (PHP/7.3.27)
443/tcp   open  https  Apache httpd 2.4.46
445/tcp   open  smb    Microsoft Windows
3306/tcp  open  mysql  MariaDB 10.3.x
5000/tcp  open  http   Apache httpd 2.4.46
```

O Nmap também revelou _virtual hosts_ configurados:

```
Service Info: Hosts: www.example.com, LOVE, www.love.htb
```


**Arquivo:**

![](Nmap_Love.txt)

---

## Enumeration & Analysis

### Enumeração Web

- O serviço na porta **80** utiliza **MariaDB** para validação de login
    
- A porta **443** retornava _Forbidden_, mesmo após ajuste no `/etc/hosts`
    

### Gobuster

Foram testadas as wordlists:

- `common.txt` (dirb)
    
- `directory-list-2.3-medium.txt`
    

Nenhuma rota relevante foi encontrada.

### Descoberta de Staging Interno

Com base no Nmap, foi inferido a existência de um ambiente interno:

```text
staging.love.htb
```

Após adicionar ao `/etc/hosts`, o host respondeu a **ping**, confirmando sua existência.


**Arquivos:**

![](GobusterCommon_Love.txt)

![](GobusterMedium_Love.txt)

---

## Attack / Exploitation

### SSRF no Staging

No ambiente `staging.love.htb`, a opção **Demo** permite submeter uma URL para escaneamento.

- Requisições para `127.0.0.1` foram bloqueadas
    
- Ao testar `127.0.0.1:5000`, o serviço retornou credenciais administrativas:
    

```
@LoveIsInTheAir!!!!
```

### Exploração da Aplicação Web

Com a senha obtida, foi utilizado o exploit do **Exploit‑DB – Voting System Exploit 1.0**.

Após ajustar os parâmetros do script e executá‑lo, foi obtida uma **shell reversa** no alvo.

---

## Post-Exploitation

### User Flag

```
C:\Users\Phoebe\Desktop\user.txt
```

O usuário **Administrator** não era acessível diretamente neste estágio.

---

## Privilege Escalation

### AlwaysInstallElevated

Durante a enumeração, foi identificado que a política **AlwaysInstallElevated** estava habilitada.

Foi criado um **payload malicioso (.msi)** e transferido para o alvo usando `certutil`, no diretório:

```
C:\Administration
```

A execução do instalador resultou em uma shell como **NT AUTHORITY\SYSTEM**.

---

## Impact & Root Access

- **Impacto:** Comprometimento total do sistema
    
- **Nível de severidade:** Crítico
    
- **Motivo:** Cadeia de SSRF + credenciais expostas + configuração insegura de privilégio
    

---

## Flags Obtidas

### User Flag

```
C:\Users\Phoebe\Desktop\user.txt
```

### Root Flag

```
C:\Users\Administrator\Desktop\root.txt
```

---

## Remediation Recommendations

- Restringir acesso a serviços internos (evitar SSRF)
    
- Nunca expor credenciais em respostas de aplicações
    
- Desabilitar **AlwaysInstallElevated**
    
- Segmentar ambientes de staging e produção
    

---

## Lessons Learned

- Informação em banners e _Service Info_ pode revelar caminhos críticos
    
- SSRF é um vetor poderoso para acesso a serviços internos
    
- Políticas inseguras de instalação facilitam escalonamento total
    

---

## Summary

A máquina **Love** apresenta uma cadeia realista de ataque combinando falha lógica (SSRF), vazamento de credenciais e má configuração de segurança no Windows. O laboratório reforça a importância de enumeração cuidadosa e validação de configurações internas.