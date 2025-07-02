---
title: "Laboratório: Detecção de ataques Web, criação de alertas e remediação"
author: "Eduardo Araujo"
categories: [Portfolio]
tags: [mitre_att&ck, lab, splunk, detection, ssh, directories, hydra, dirb, apache, DVWA]
render_with_liquid: false
image:
  path: /images/seguranca/siem/splunk/installation-configuration/splunk.svg
---

## Overview do laboratório 

O objetivo desse laboratório é aprender a configurar e entender como funciona a detecção de ataques em tempo real utilizando o SIEM [Splunk](https://www.splunk.com/), assim como entender quais procedimentos são adequados para remediar esses ataques. A idéia desse exercício foi colocado em prática após eu compreender conceitos importantes do Framework MITRE ATT&CK, sendo que serão abordadas as técnicas de Bruteforce denominadas [T1110](https://attack.mitre.org/techniques/T1110/) de acordo com o Framework, específicamente *Fuzzing* de diretórios com a ferramenta *Dirb* e bruteforce de senhas contra serviço de SSH utilizando a ferramenta *Hydra*


## Identificando as jóias da coroa

A idéia central é parecer ao máximo com o mundo real, portanto, vamos partir do pressuposto que você foi contratado por um cliente que deseja monitar a rede da empresa. O primeiro passo é conhecer a topologia e identificar os ativos mais importantes da empresa. 

💡 Pense em ativos que caso fossem "hackeados" traria muitas consequências negativas a empresa, seja financeira, de confiança, perda de clientes ou perda de reputação 

Alguns exemplos são: Controlador de domínio, propriedade intelectual, sistema de RH, banco de dados, aplicação exposta a internet, fileservers


## Topologia da rede

Após uma reunião com o cliente você constatou que se trata de uma rede simples, onde o principal sistema usado é apenas uma aplicação web usada pelos colaboradores em uma rede local. Para simular essa rede podemos montar o laboratório dessa maneira:

![topologia-da-rede](../images/seguranca/lab/ataques-web/topologia-rede.png)


O [DVWA](https://github.com/digininja/DVWA) é uma aplicação feita exclusivamente para ser explorada. Em ambientes empresariais é comum encontrar aplicações Web usadas por colaboradores, algumas inclusives dentro de DMZ e expostas a internet, portanto a função do DVWA aqui é simular uma aplicação web usada pelos colaboradores da empresa. O DVWA será instalado em uma VM Ubuntu Server e receberá IP da minha rede local via modo Bridge. 

Splunk Enterprise para análise de logs e criação de alertas de segurança. Nesse laboratório ele estará hospedado em uma VPS mas fique vontade para instalar o Splunk em sua máquina ou você pode instalar até mesmo em outra VM


⚠️ **Aqui vale ressaltar algo importante:** pode ser interessante monitorar os logs do Firewall, porque apesar de proteger a rede, ele contém informações e visibilidade de todas as conexões que acontece na rede interna. Mas nesse guia vamos pular essa parte


## Configurando o ambiente

Antes de tudo instale as máquinas virtuais que você vai utilizar.

### Instalando o Splunk

Faça a instalação do Splunk Enterprise em um host de sua preferência; em seguida configure os Universal Forwarders nas máquinas alvos para alimentar esse Splunk com dados. Em caso de dúvidas nesse [artigo](https://repoeduardo.github.io/posts/Splunk-Instalacao-Configuracao/) eu ensino como instalar e configurar o Splunk

### Instalando o DVWA

⚠️ Instale o software DVWA em uma máquina virtual, porque é **propositalmente** vulnerável a ataques. Não é recomendável instalar em máquinas reais ou de produção


![stack](../images/seguranca/lab/ataques-web/stack-dvwa.png)

A aplicação necessita de três componentes essenciais para funcionar: **php, servidor web e um banco de dados**. Vamos utilizar o Apache2 e o MySQL como servidor web e banco de dados respectivamente.


Instalando o php, mysql e apache2; Habilitando o serviço do apache2:

~~~bash
sudo apt install -y apache2 php php-mysql mysql-server
sudo systemctl enable apache2
~~~

Verifique se o apache está instalado corretamente acessando a porta 80 do servidor, ou seja, simplesmente jogue o IP no browser

![apache2-on](../images/seguranca/lab/ataques-web/apache2-running.png)

Ou verifique o status do serviço com esse comando: `sudo systemctl status apache2`


Baixe o DVWA do repositório oficial (precisa ter o git instalado); Aplique permissões totais aos diretórios web

~~~bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git
sudo chown -R www-data:www-data DVWA
sudo chmod -R 777 DVWA
~~~

Acesse o MySQL como root: `sudo mysql` e execute os seguintes comandos:

~~~sql
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'p@ssw0rd';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EXIT;
~~~

Agora edite o arquivo de configuração localizado em `/var/www/html/DVWA/config/config.inc.php.dist`

Seguindo essas valores:

~~~php
$_DVWA['db_server'] = 'localhost';
$_DVWA['db_database'] = 'dvwa';
$_DVWA['db_user'] = 'dvwa';
$_DVWA['db_password'] = 'p@ssw0rd';
~~~

Para funcionar a aplicação precisa ler um arquivo de configuração ``.php`` e não ``.dist``; portanto apenas faça uma cópia do arquivo dist para outro com extensão php

~~~bash
sudo cp /var/www/html/DVWA/config/config.inc.php.dist /var/www/html/DVWA/config/config.inc.php
~~~

Você já deve ser capaz de acessar a aplicação em `http://<IP do host>/DVWA`

![dvwa-running](../images/seguranca/lab/ataques-web/dvwa-running.png)

- Credenciais de acesso padrão: `admin / password`

- Clique em ``Create / Reset Database`` na página inicial para reconfigurar o banco.

![dvwa-home](../images/seguranca/lab/ataques-web/dvwa-home.png)


Pronto, temos uma aplicação Web no ambiente 👍

### Enviando logs ao Splunk

O próximo passo é começar a enviar os logs ao Splunk. Um dos requisitos é detectar tentativas de acesso SSH, portanto precisamos ler logs de autenticação do linux que fica nesse diretório: `/var/log/auth.log`; Outro requisito é detectar um possível fuzzing de diretórios na aplicação web, nesse caso os logs do apache2 vai nos ajudar: `/var/log/apache2/access.log`

Adicione esses arquivos ao monitoramento do Splunk:

~~~bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/apache2/access.log -sourcetype apache:access:combined
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -sourcetype linux:audit
~~~

## Referências

- https://www.splunk.com/
- https://attack.mitre.org/
- https://attack.mitre.org/techniques/T1110/
- https://github.com/digininja/DVWA
- https://repoeduardo.github.io/posts/Splunk-Instalacao-Configuracao/



<style>
.center img {
  display:block;
  margin-left:auto;
  margin-right:auto;
}
.wrap pre{
    white-space: pre-wrap;
}
</style>