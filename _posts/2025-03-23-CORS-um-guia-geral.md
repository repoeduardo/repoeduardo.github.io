---
title: "CORS: Introdução e Exploração"
author: "Eduardo Araujo"
categories: [Segurança]
tags: [web, misconfiguration, cors, bugbounty]
render_with_liquid: false
image:
  path: /images/seguranca/cors/cors.svg
---


CORS (Cross-Origin Resource Sharing) é um mecanismo de segurança implementado nos navegadores para controlar quais domínios podem fazer requisições HTTP para um servidor diferente do seu próprio domínio de origem.

## PARA QUE SERVE

CORS existe para:

 - Permitir integração segura: APIs modernas precisam ser acessadas por frontends em domínios diferentes (ex.: app.exemplo.com acessando api.exemplo.com);

 - Proteger contra abusos: Impede que qualquer site aleatório (ex.: malicioso.com) leia dados sensíveis de outro domínio sem permissão explícita;

 - Flexibilidade com segurança: Dá ao servidor controle granular sobre quem pode acessar o quê.


## COMO FUNCIONA O CORS

Por padrão, navegadores bloqueiam requisições `AJAX/XHR/FETCH `de origens diferentes. Isso significa que um site em https://siteA.com não pode acessar a API de https://siteB.com, a menos que siteB.com permita explicitamente.

O controle do CORS acontece no cabeçalho de resposta HTTP do servidor, através do `Access-Control-Allow-Origin`.

Exemplo de resposta de um servidor que permite acesso somente de **example.com**:

~~~http
Access-Control-Allow-Origin: https://example.com
~~~
Se qualquer outro site tentar acessar a API, o navegador bloqueia a requisição.

Já um servidor totalmente aberto pode permitir requisições de qualquer origem com:

~~~http
Access-Control-Allow-Origin: *
~~~


### CABEÇALHOS

**Access-Control-Allow-Origin:** Define quais origens podem acessar o recurso (ex: * ou http://exemplo.com);

**Access-Control-Allow-Methods:** Métodos HTTP permitidos;

**Access-Control-Allow-Headers:** Cabeçalhos permitidos;

**Access-Control-Allow-Credentials:** Permite envio de cookies/sessões (valor: true ou false).



### REQUISIÇÕES COM PRÉ-VERIFICAÇÃO (Preflight Requests)

Para métodos complexos (ex: PUT, DELETE) ou cabeçalhos customizados, o navegador envia uma requisição OPTIONS antes da requisição real. Exemplo:

~~~http
OPTIONS /api/dados HTTP/1.1
Host: api.exemplo.com
Origin: http://exemplo.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
~~~

Resposta:

~~~http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://exemplo.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
~~~

Se o servidor aprovar, a requisição real (PUT) é enviada.


## COMO "VER" O CORS

Você pode verificar o CORS de um site pelo DevTools do navegador:

1. Abra o site que deseja testar

2. Abra o DevTools (F12 ou Ctrl+Shift+I no Chrome)

3. Vá para a aba *Network* → Filtre por *Fetch/XHR*

4. Clique na requisição e veja os Response Headers

    - Se houver Access-Control-Allow-Origin: *, o site está totalmente aberto

    - Se estiver null ou faltar esse cabeçalho, CORS pode estar bloqueado


5. Outra opção é testar diretamente com curl no terminal:

~~~sh
curl -I https://site.com/api -H "Origin: https://evil.com"
~~~

Se a resposta incluir `Access-Control-Allow-Origin: https://evil.com`, o site permite requisições de terceiros.


No código-fonte procure por chamadas AJAX/Fetch em JavaScript que acessem outros domínios.

~~~javascript
fetch("https://api.exemplo.com/dados", { credentials: "include" })
~~~


## COMO EXPLORAR

Geralmente falhas em CORS se devem a configurações mau feitas pelos desenvolvedores. Essas "Misconfigurations" no CORS podem expor dados sensíveis ou permitir ataques.

### Access-Control-Allow-Origin *

Ao analisar a resposta do servidor preste atenção ao Header (Cabeçalho) de resposta `Access-Control-Allow-Origin`, caso esteja marcado com um asterisco (*) significa que o servidor permite acesso de qualquer origem. Isso quer dizer que um `dominioB` pode fazer uma requisição ao `dominioA` sem ter problemas. A situação pode se agravar ainda mais se combinado com outro Header: `Access-Control-Allow-Credentials: true` pois isso permite envio de sessões e cookies


    Normalmente os navegadores modernos bloqueiam quando Allow-Credentials é true junto com Allow-Origin:*    


Se você existir esse contexto, para explorar crie um site malicioso que faça uma requisição ao endpoint vulnerável. Exemplo usando o `fetch` do JavaScript:

~~~html
<script>
  fetch("https://api.dominioA.com/usuarios")
    .then(response => response.json())
    .then(data => console.log(data));
</script>
~~~

Se o endpoint retornar dados sensíveis (ex.: tokens, informações do usuário), você os captura.



### Reflexão do Origin

A Reflexão do Origin acontece quando você intencionalmente define um Header `Origin: dominioqualquer.com` na requisição e no Header da resposta esteja com o mesmo valor, nesse caso: `Access-Control-Allow-Origin: dominioqualquer.com`.


Suponha que você fez login em um website e descobriu que o site fez uma chamada a uma API (fetch ou algo similar)

*Considere apenas para fins didáticos que o domínio do site seja `https://rosadoces.com`*

![cors_normal](/images/seguranca/cors/cors_normal.png)

Observe que o Header `Access-Control-Allow-Origin` tem como valor o domínio atual. Por vezes, os desenvolvedores configuram isso de forma errada. Permitindo que esse Header reflita o valor que está vindo do Header da request chamado `Origin`

Ao interceptar a requisição, podemos inserir manualmente o Header `Origin` e analisar o comportamento.

É muito improvável que se você colocar qualquer domínio aleatório funcione. O segredo é usar o domínio atual com alguma variação, porque o filtro pode estar sendo aplicado ao nome do domínio. Por exemplo, no cabeçalho da requisição:

`Origin:` https://evil.rosadoces.com/

![cors_reflected](/images/seguranca/cors/cors_reflected.png)

Deu certo! Essa aplicação está vulneravel ao Cors de reflexão Origin. Sendo assim, qualquer um que tenha um domínio que contenha *rosadoces* no nome poderá fazer chamadas a endpoints sensíveis.