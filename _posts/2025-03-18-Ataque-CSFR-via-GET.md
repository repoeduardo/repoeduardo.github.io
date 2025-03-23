---
title: "CSRF: Ataque via GET"
author: "Eduardo Araujo"
categories: [Segurança]
tags: [web, js, javascript, csrf, bugbounty, ngrok]
render_with_liquid: false
image:
  path: /images/seguranca/csrf/csrf_via_get.svg
---


É conveniente você procurar por CSRF quando for executar alguma ação de mudança de estado importante no sistema. Como exclusão, edição, efetuar pagamentos e coisas similares a formulários de mudança de informação. 

Então, suponha que um usuário tenha logado em um website. Os usuários autenticados possuem algumas ações em seu perfil como mostrado abaixo

![csrf_lab](/images/seguranca/csrf/lab_notifications/csrf_lab.png)


**Apenas para fins didáticos**, assuma que o nome de dominio desse laboratório seja `https://rosadoces.net`

## CASO HIPOTÉTICO: ativar/desativando notificações

Veremos agora um caso hipotético de um usuário que está logado em um website, onde a ação é simplesmente ativar ou desativar notificações. O impacto do CSRF depende muito do contexto, nesse exemplo de notificações a severidade não deve ser alta se comparado com mudar um e-mail ou senha. O grau de severidade sempre depende do tipo de informação que o atacante pode/cosegue alterar.

![csrf_not](/images/seguranca/csrf/lab_notifications/csrf_not.png)

Nesse momento estamos em: `https:rosadoces.net/notifications`

### ANALISANDO O COMPORTAMENTO

A primeira tarefa é analisar o comportamento da aplicação no momento em que é engatilhado algum evento de mudança de estado - nesse caso o botão de `enabled Notifications`

Para analisar as requisições e repostas, use alguma ferramenta de proxy - Caido, CharlesProxy, BurpSuite. Para esse caso vou utilizar somente o *DevTools do Chrome > Aba Network*

![csrf_not_reqres](/images/seguranca/csrf/lab_notifications/csrf_not_reqresponse.png)


Vamos analisar com cuidado essa imagem

Após clicar no botão ficamos no mesmo site, porém veja como indica a **seta azul** que foi alterado alguns textos. Analisando a requisição com as **setas vermelhas**, repare que ele bateu no endpoint `/notifications?enabled=true`

Então é como se ele tivesse feito assim na url: `https://rosadoces.net/notifications?enabled=true`
Após fazer isso, é retornado um `Status Code 302 FOUND` e em seguida através do campo `Location: /notifications` fomos retornados a página que estávamos anteriormente.

### PREPARANDO O ATAQUE

A primeira coisa que podemos fazer é tentar criar um formulário em HTML que contenha os mesmos parâmetros e valores encontrados acima. Isso possível graças a tag `<form>` com seus atributos `method` e `action` <br>
Dentro do formulário podemos colocar entradas invisiveis com valores pré-definidos através da tag `<input>` e dos atributos `type`, `name` e `value`

É possível testar da mesma forma com a tag `<a>` que ao ser clicado redireciona o usuário para a url especificada no atributo `href`

**Mas porque usar essas tags?**

Na verdade você pode usar outras tags HTML, o objetivo é de alguma forma "acionar" o endpoint da requisição com os mesmos parâmetros via método GET, nesse caso: `https://rosadoces.net/notifications?enabled=true`; É possível fazer isso com `<form>`, `<a>`, `<img>` e talvez até de outras formas. O atacante escolho o modo de fazer.


Usando formulário HTML

~~~html
<form action="https://rosadoces.net/notifications" method="GET">
    <input type="hidden" name="enabled" value="true">
    <input type="submit" value="enviar">
</form>
~~~

A idéia é que ao clicar no botão o formulário vai acionar examente a mesma requisição que analisamos anteriormente `https://rosadoces.net/notifications?enabled=true`

Usando um hiperlink com `<a>`

~~~html
<a href="https://rosadoces.net/notifications?enabled=true">clique aqui</a>
~~~

A idéia é que ao clicar no hiperlink será acionado examente a mesma requisição que analisamos anteriormente `https://rosadoces.net/notifications?enabled=true`


**Contudo, aqui cabe uma observação importante:** devido a algumas atualizações recentes do Chrome esse método pode não funcionar se a origem não for de um HTTPS. Em outras palavras, caso você rode esse HTML direto da sua máquina o navegador pode não aceitar. Nesse caso, você pode contornar esse possível problema colocando esse arquivo HTML em um domínio próprio (caso você tenha um), mas vou assumir que você não tenha domínio próprio. 
<br>

**Deve estar se perguntando: OK, tudo que preciso é de um arquivo HTML que chame a URL por meio de um formulário, hiperlink ou qualquer outro método parecido, partindo de um HTTPS. Mas como posso fazer isso?**

Ao invés de comprar um domínio, podemos fazer isso através de um DDNS. Porém uma solução mais fácil seria usar o serviço do [NGROK](https://ngrok.com/) 

    O Ngrok não serve arquivos diretamente; ele cria um túnel para um servidor que já está rodando na sua máquina

**Funciona da seguinte maneira:** instale o ngrok e faça a autenticação com sua conta criada em [https://ngrok.com](https://ngrok.com) utilizando o Token que será fornecido em seu perfil. Em seguida, crie um servidor HTTP local (isso pode ser feito com Python ou Node) depois execute o ngrok na mesma porta do servidor HTTP local

    Para instalar o ngrok visite o site e faça o passo a passo recomendado de acordo com seu sistema

Após instalar o ngrok, crie o arquivo HTML

~~~html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
</head>
<body>
    <h1>Formulário HTML</h1>
    <form action="https://rosadoces.net/notifications" method="GET" target="frm">
        <input type="hidden" name="enabled" value="true">
        <input type="submit" value="enviar">
    </form>

    <h2>Hiperlink</h2>
    <a href="https://rosadoces.net/notifications?enabled=true">ativar</a>
    
</body>
</html>
~~~

Agora acesse a pasta onde está esse arquivo e crie um servidor HTTP simples com Python

~~~python
python3 -m http.server 8080
~~~

Abra outro terminal e execute o ngrok na mesma porta do servidor local

~~~bash
ngrok http 8080
~~~

Em seguida o NGROK vai fornecer uma URL aleatória HTTPS. Basta acessar e você deverá ser capaz de visualizar seu arquivo HTML

![csrf_not_ngrok](/images/seguranca/csrf/lab_notifications/csrf_not_ngrok.png)

Após clicar no botão ou no link será acionado o mesmo evento que vimos anteriormente. O resultado:

![csrf_not_ngrok_enabled](/images/seguranca/csrf/lab_notifications/csrf_not_ngrok_enabled.png)

Deu certo! Essa página esta vúlneravel a CSRF pois aceita requsições de qualquer origem HTTPS
