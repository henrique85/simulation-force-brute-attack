Simulação de Ataque de Força Bruta, utilizando a ferramenta Medusa no Kali Linux.

### Requisitos

- Software de Máquina Virtual (no caso será utilizado o VirtualBox, que você pode baixar clicando [aqui](https://www.virtualbox.org/wiki/Downloads)).
- VM Kali Linux (link para download [aqui](https://www.kali.org/get-kali/#kali-virtual-machines)).
- VM Metasploitable (link para download [aqui](https://sourceforge.net/projects/metasploitable/files/)).

### ⚠️ Observações

- Kali Linux: login e senha padrão → **kali**
- Metasploitable: login e senha padrão → **msfadmin**

### Técnicas utilizadas

- Password Spraying e Credential Stuffing

### Etapas

**1. Enumeração**

A primeira parte desta simulação é a enumeração, que consiste em descobrir quais serviços estão disponíveis no sistema alvo. Para isso é utilizado o comando **nmap**, que vai escanear as portas dos principais protocolos de rede como FTP, SSH, HTTP, HTTPS e SMB. O parâmetro **-sV** serve para identificar a versão do serviço que está rodando em cada porta:

nmap -sV -p 21,22,80,445,139 192.168.56.102

`nmap -sV -p 21,22,80,445,139 192.168.56.102`

```bash
nmap -sV -p 21,22,80,445,139 192.168.56.102
```

Este comando vai mostrar qual o estado das portas como "**open**".

![Varredura Nmap](./images/01_varredura_nmap.png)

Se quiser validar se o protocolo FTP realmente está realmente aberto, basta executar o comando:

ftp 192.168.56.102

**2. Wordlist**

Conforme a imagem anterior, sabemos que o serviço está aberto mas não temos como saber qual login e senha correto. Diante disso, vamos criar duas Wordlists, uma contendo nomes de usuários e outra contendo senhas. 
Wordlists são arquivos com usuários e senhas possíveis. Para isso vamos utilizar os comandos abaixo:

- Comando para criar lista de usuários:

echo -e “user\nmsfadmin\nadmin\nroot” > users.txt

- Comando para criar lista de senhas:

echo -e “123456\npassword\nqwerty\nmsfadmin” > pass.txt








