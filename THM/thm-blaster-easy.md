# 1 — Visão geral da metodologia

1. **Objetivo:** Comprometer a máquina Windows expondo falhas de autenticação fraca e vulnerabilidade local para obtenção de privilégio de administrador.
    
2. **Estrutura:** Recon → Enumeração HTTP → Validação de credenciais → Acesso RDP → Privesc → Flags → Pós-exploração.
    
3. **Princípios:** Documentação reprodutível com coleta de evidências.
    

---

# Report — Máquina: `Blaster` — `10.64.XXX.XXX` _(THM IP dinâmico)_

- **Fonte:** THM
    
- **Data:** 2025-11-29 e 2025-11-30
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Easy
    
- **Início:** 07:30
    
- **Pausa:** 20:15
    
- **Retorno:** 11:41 (dia seguinte)
    
- **Finalizado:** 12:00
    
- **Tempo gasto efetivo:** ~1h
    

---

## 1) Executive summary

IIS expôs diretório oculto com pistas de credenciais. Autenticação fraca possibilitou login RDP direto como usuário local. Elevação para Administrator usando CVE-2019-1388, garantindo acesso completo ao host e captura das flags.

---

## 2) Escopo e limitações

- **Alvo:** 10.64.x.x (Rede interna THM)
    
- **Serviços:** IIS / RDP
    
- **Ferramentas:** Nmap, ffuf, RDP client, CVE-2019–1388 exploit
    
- **Limitações:** Nenhuma especificada pelo lab
    

---

## 3) Metodologia aplicada

- Identificação de IIS + diretório oculto
    
- Coleta de informações textuais (OSINT interno)
    
- Tentativa de login por credenciais sugeridas
    
- Acesso interativo Windows via RDP
    
- Privesc usando UAC bypass (CVE-2019-1388)
    
- Obtenção das flags
    

---

## 4) Achado principal

**Vulnerabilidade:** Credenciais fracas + binário vulnerável com UAC bypass  
**Local:** Autenticação no RDP + exploit local  
**PoC (resumo):**  
→ Login credenciais:

`Username: wade Password: parzival`

→ Execução de binário `hhupd` para exploração CVE-2019-1388

---

## 5) Evidências / PoC (resumo)

### Recon

`sudo nmap -Pn -sS -sV -T4 -p- <IP>`

→ serviços: HTTP (IIS) + RDP

### Enumeração Web

`ffuf -u http://<IP>/FUZZ -w /usr/share/seclists/.../directory-list-2.3-medium.txt`

→ descoberto `/retro`

**Conteúdo do blog:**  
Senha sugerida pelo próprio usuário → "parzival"

### Acesso ao Desktop remoto

Após tentativas na VM local, acesso foi obtido via AttackBox da TryHackMe.

**Flag user encontrada:**

### Privesc

Binário `hhupd` encontrado na área de trabalho + pesquisa sobre CVE-2019-1388 → UAC bypass

Execução do exploit → Administrator

**Flag root encontrada:**

### Pós-exploração

Persistência implementada via Meterpreter:

`exploit/multi/script/web_delivery run persistence -X`

---

## 6) Impacto & Severidade

**Severidade:** Alta

- Senhas triviais → acesso total via RDP
    
- Execução local vulnerável → privilege escalation
    
- Integridade e confidencialidade completamente comprometidas
    

---

## 7) Reproduzir (passo-a-passo)

Scan e fingerprint:

`nmap -Pn -sS -sV -T4 -p- <IP>`

Enumeração HTTP:

`ffuf -u http://<IP>/FUZZ -w directory-list-2.3-medium.txt`

Acessar `/retro` → ler as pistas  
Tentar login RDP:

`xfreerdp /u:wade /p:parzival /v:<IP>`

Encontrar `hhupd` → executar CVE-2019-1388  
Obter Administrator  
Coletar flags

---

## 8) Recomendações (gestores)

- Implementar política forte de senhas
    
- Bloquear exposição pública de RDP
    
- Corrigir binários vulneráveis (UAC bypass)
    
- Habilitar monitoramento e MFA para acesso remoto
    

---

## 9) Recomendações técnicas

- Desabilitar RDP quando não necessário:
    

`reg add "HKLM\System\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 1 /f`

- Gestão de credenciais + reforço em MFA
    
- Patching urgente do CVE-2019-1388
    
- Logging centralizado + alertas de brute-force
    

---

## 10) Apêndice técnico

**Arquivos a anexar:**

#### Nmap
![](Nmap_Blaster.txt)
#### Ffuf
![](FfufDirList_Retro.txt)

---

## 11) Lições aprendidas / Follow-up

- Sempre validar conteúdo de diretórios expostos
    
- Informações em posts e comentários podem ser credenciais
    
- Procura rápida por executáveis suspeitos → privesc fácil
    
- Em Windows: **CVE-2019-1388** costuma ser um caminho direto
    

---

## Conclusão

✔ Acesso total ao sistema Windows via RDP  
✔ Privesc local totalmente reprodutível  
✔ Flags obtidas com sucesso
