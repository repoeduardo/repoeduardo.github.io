---
title: "Splunk: Instalação e configuração"
author: "Eduardo Araujo"
categories: [SIEM]
tags: [siem, splunk, logs]
render_with_liquid: false
image:
  path: /images/seguranca/cors/cors.svg
---

## Como instalar e executar o Splunk

### Download na página oficial

Visite a página oficial da ferramenta [Splunk](https://www.splunk.com/en_us/download.html) > Selecione Splunk Enterprise Free Trial > faça seu cadastro na plataforma (é obrigatório) > Em seguida faça o Login

⚠️ no campo **Business Email** ele aceita @gmail.com


Escolha o pacote do Splunk de acordo com o sistema operacional. Nesse guia será instalado em um Linux

 Linux:

 ~~~bash
 wget -O splunk-9.4.3-237ebbd22314-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/9.4.3/linux/splunk-9.4.3-237ebbd22314-linux-amd64.deb"
 ~~~

Windows:

 ~~~bash
 wget -O splunk-9.4.3-237ebbd22314-windows-x64.msi "https://download.splunk.com/products/splunk/releases/9.4.3/ windows/splunk-9.4.3-237ebbd22314-windows-x64.msi"
 ~~~

 MacOS:

 ~~~bash
 wget -O splunk-9.4.3-237ebbd22314-darwin-intel.dmg "https://download.splunk.com/products/splunk/releases/9.4.3/ osx/splunk-9.4.3-237ebbd22314-darwin-intel.dmg"
 ~~~
 
<br>
<br>

Use o gerenciador de pacotes `DPKG` para instalar o pacote `.deb` baixado

~~~bash
sudo dpkg -i splunk-9.4.3-237ebbd22314-linux-amd64.deb
~~~

- Aguarde a instalação finalizar;

- Verifique o status da instalação com o seguinte comando:

~~~bash
dpkg --status splunk
~~~

![splunk-status](../images/seguranca/siem/splunk/installation-configuration/splunk-status.png)

- Outra maneira de verificar se deu certo é acessar o diretório `/opt`; Deve existir um diretório chamado `splunk`


### Iniciando o Splunk

- Acesse o diretório `/opt/splunk/bin`

- Execute o comando `./splunk start`

- Primeiro será mostrado um contrato de licenças e coisas similares. Aperte a tecla `Space` até o final do documento. Em seguida aceite os termos com `y` 

- Depois defina um nome de usuário administrador e uma senha 

    ⚠️ anote essas credenciais

- Após criar as credenciais você devera ver uma tela igual essa abaixo. Significa que o webserver está funcionando.

![webserver-splunk](../images/seguranca/siem/splunk/installation-configuration/webserver-splunk.png)

- Você pode acessar o servidor através do localhost, hostname (nome do servidor) ou endereço ip:
    - Se estiver na sua máquina local: http://127.0.0.1:8000
    - Se estiver remoto ou na sua rede local: http://ip ou hostname do servidor:8000

![webserver-login-pages](../images/seguranca/siem/splunk/installation-configuration/splunk-first-screen.png)

- Insira suas credenciais de acesso criadas anteriormente


## Como configurar

O próximo passo é adicionar uma fonte de dados ao Splunk. Existem muitas formas de fazer isso, nesse guia vou mostrar de duas maneiras: através de dados prontos e através de Universal Forwarders (pense nisso como "coletores" de dados)


### Dados prontos

A documentação oficial fornece dados "prontos" empacotados em arquivos .zip onde contém logs de acesso web, logs de arquivos formatados, logs de arquivos de vendas e listas de preços. Você pode baixar esses arquivos na [documentação oficial](https://help.splunk.com/en/splunk-enterprise/get-started/search-tutorial/9.2/part-1-getting-started/what-you-need-for-this-tutorial#id_00ebcad1_5243_445b_b1f1_e3a49fd8c759__Download_the_tutorial_data_files) 

Ou diretamente do meu repositório no Github

~~~bash
wget https://github.com/repoeduardo/splunk/blob/main/tutorialdata.zip && wget https://github.com/repoeduardo/splunk/blob/main/Prices.csv.zip
~~~


#### Importando os dados no Splunk

- Acesse: *Configurations > Add data > Upload > Select File > agora selecione o arquivo zip*

Após importar o arquivo .zip você poderá ver um alerta dizendo: *Preview is not supported for this archive file, but it can still be indexed*. Não se preocupe com essa mensagem.

*Next > Review > Submit > Start Searching*

- Agora que os dados foram "ingeridos" e indexados pelo Splunk podemos testar a busca de dados

- Acesse a aba de buscas *Search*. Você deve ver essa tela

![search-screen](../images/seguranca/siem/splunk/installation-configuration/search-screen.png)

Vamos testar buscando dados que tem origem do arquivo de `prices.csv.zip` com a seguinte consulta:

~~~txt
source="Prices.csv.zip:*"
~~~

OBS: filtre o tempo por `All Time`

![search-result](../images/seguranca/siem/splunk/installation-configuration/splunk-search-result.png)


Depois teste novamente com o outro arquivo: `source="tutorialdata.zip:*"`


### Universal Forwarders

Os Universal Forwarders são como "agentes" coletadores de dados, ideal para coletar dados de sistemas remotos e enviar ao seu Splunk.


#### Instalação em Linux

~~~bash
wget -O splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.4.3/linux/splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb"
~~~

- Após fazer o download do pacote .deb faça a instalação com o gerenciador de pacotes dpkg

~~~bash
sudo dpkg -i splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb
~~~

- Verifique o status da instalação: `dpkg --status splunkforwarder`

- De acordo com a [documentação oficial](https://help.splunk.com/en/splunk-enterprise/forward-and-process-data/universal-forwarder-manual/9.4/install-the-universal-forwarder/install-a-nix-universal-forwarder#about-installing-with-tar-files-0) é necessário criar um usuário com menos privilégios que o *root* chamado `splunkfwd` por boas práticas de segurança. Essa etapa é opcional, para fazer isso certifique de fazer login como root do sistema e criar um usuário e grupo executando os comandos a seguir:

~~~bash
useradd -m splunkfwd
groupadd splunkfwd
export SPLUNK_HOME="/opt/splunkforwarder"
chown -R splunkfwd:splunkfwd $SPLUNK_HOME
~~~

- Execute o SplunkForwarder

~~~bash
sudo $SPLUNK_HOME/bin/splunk start --accept-license
~~~


#### Configurando Universal Forwarder e o Splunk

- O primeiro passo a se fazer é abrir um porta de escuta do host que irá receber os dados

- Existem diferentes maneiras de fazer isso: através do painel web ou linha de comando. Nesse exemplo irei usar o painel web. Em caso de dúvidas consulte esse [documento oficial](https://help.splunk.com/en/splunk-enterprise/forward-and-process-data/universal-forwarder-manual/9.4/configure-the-universal-forwarder/enable-a-receiver-for-splunk-enterprise#id_8dd83488_23ef_4bc4_94ee_d4ca8aa9cfeb__Enable_a_receiver_for_Splunk_Enterprise)

- No seu Splunk acesse: *Settings > Forwarding and receiving > Configure receiving > New Receiving Port > Adicione a porta e salve (a padrão é 9997)*


- Agora acesse o host que tem o SplunkForwarder instalado e configure para enviar os dados para o *receiver*, o Splunk Enterprise

~~~bash
./splunk add forward-server <ip do Splunk>:<Porta de escuta>
~~~

- Agora você pode monitorar alguns logs, por exemplo para monitorar a pasta `/var/logs` do seu forwarder

~~~bash
./splunk add monitor /var/log
~~~

- Vá até a barra de busca de seu Splunk Enterprise e busque por `*`

![forwarder-ok](../images/seguranca/siem/splunk/installation-configuration/forwarder-ok.png)



## O que fazer agora?

A partir daqui você define quais tipos de dados pretende monitorar. Essa [página](https://help.splunk.com/en/splunk-enterprise/get-started/get-data-in/9.4/introduction/what-data-can-i-index) te auxiliar em como fazer isso. Por exemplo, você pode subir um software vulnerável onde o Forwarder está instalado para monitorar em tempo real possíveis ataques; você pode montar dashboards de acessos ssh; criar alertas e reports; criar dashboard de máquinas Windows (pra isso você precisa instalar o Forwarder em um Windows).

A partir daqui você pode criar muitos tipos de laboratórios que te ajudam em seu objetivo.


## Referências

- https://docs.splunk.com/Documentation/Splunk/9.4.2/Installation/StartSplunkforthefirsttime

- https://www.splunk.com/en_us/download.html

- https://help.splunk.com/en/splunk-enterprise/get-started/search-tutorial/9.2/part-1-getting-started/what-you-need-for-this-tutorial#id_00ebcad1_5243_445b_b1f1_e3a49fd8c759__Download_the_tutorial_data_files

- https://help.splunk.com/en/splunk-enterprise/forward-and-process-data/universal-forwarder-manual/9.4/install-the-universal-forwarder/install-a-nix-universal-forwarder#bfa92018_7238_476c_8351_2dd1ee65ef8c__Install_the_universal_forwarder_on_Linux

- https://help.splunk.com/en/splunk-enterprise/forward-and-process-data/universal-forwarder-manual/9.4/configure-the-universal-forwarder/enable-a-receiver-for-splunk-enterprise#id_8dd83488_23ef_4bc4_94ee_d4ca8aa9cfeb__Enable_a_receiver_for_Splunk_Enterprise

- https://help.splunk.com/en/splunk-enterprise/forward-and-process-data/universal-forwarder-manual/9.4/configure-the-universal-forwarder/configure-the-universal-forwarder-using-configuration-files

- https://help.splunk.com/en/splunk-enterprise/get-started/get-data-in/9.4/introduction/what-data-can-i-index



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