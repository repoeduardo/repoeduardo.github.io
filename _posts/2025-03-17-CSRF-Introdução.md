---
title: "CSRF: Introdução"
author: "Eduardo Araujo"
categories: [Segurança]
tags: [web, js, javascript, csrf, bugbounty, lab, ngrok, html, get]
render_with_liquid: false
image:
  path:  /images/seguranca/csrf/csrf.svg
---

Cross-Site Request Forgery (CSRF) é uma vulnerabilidade que permite que um atacante force um usuário autenticado a executar ações indesejadas em um site sem que ele perceba. O ataque explora a confiança que o site tem na sessão autenticada do usuário (geralmente via cookies), enviando requisições maliciosas em nome dele

**Em resumo**: O atacante "engana" o navegador da vítima para que ele faça uma requisição (como mudar a senha ou transferir dinheiro) sem o consentimento dela.

## PRA QUE SERVE

CSRF é útil para atacar sistemas que:

- Não validam adequadamente a origem das requisições.

- Confiam apenas na autenticação via cookies ou tokens automáticos sem proteção adicional.

- Executam ações sensíveis (como alterar dados, fazer transações ou configurações) via métodos como GET ou POST sem verificação.

- Alteração de senha ou e-mail

- Transferência de dinheiro

- Publicação de conteúdo indesejado

- Exclusão de dados

    Tudo isso sem que a vítima perceba

**Exemplo prático:** Imagine um banco online onde o endpoint `POST /transferir` aceita uma requisição para transferir dinheiro sem verificar se a ação foi intencional. Um atacante pode explorar isso.


## QUANDO USAR

Você deve testar CSRF sempre que encontrar endpoints que realizam ações críticas (exemplo: formulários de alteração de dados, botões de exclusão, etc.). **O foco principal está em requisições que alteram o estado do sistema** **(HTTP POST, PUT, DELETE)**.

Você deve procurar por CSRF quando:

- A autenticação do usuário é baseada apenas em cookies persistentes.
- O site permite requisições de terceiros sem verificar a origem.
- O site realiza ações sensíveis (ex.: alterar senha, excluir conta, enviar mensagens) via requisições HTTP (GET ou POST).
- Não há uso de tokens CSRF (um código único por sessão para validar a requisição).
- O site não verifica cabeçalhos como Referer ou Origin.

    **Dica de Bug Bounty:** Teste endpoints críticos em plataformas como bancos, redes sociais ou e-commerce.


## COMO EXPLORAR

Para explorar CSRF, você precisa:

1. Identificar uma ação sensível no site, por exemplo: mudar email, transferir fundos;
2. Verificar se a requisição pode ser replicada sem validação de token ou origem.
3. Criar "iscas" (ex: página HTML maliciosa) que envie a requisição automaticamente quando a vítima acessá-la (é possível fazer isso através de funções JavaScript).

Passos gerais:

- Capture a requisição legítima com uma ferramenta proxy (Burp Suite, Caido, etc).
- Veja se ela funciona sem modificação em outra aba/sessão.
- Construa um PoC que envie essa requisição sem interação do usuário.

## PoC 

Vamos criar um exemplo básico. Suponha que o site `vulneravel.com` permite mudar o email da conta com esta requisição:

~~~http
POST /alterar-email HTTP/1.1
Host: vulneravel.com
Cookie: session=abc123
Content-Type: application/x-www-form-urlencoded

novo_email=atacante@malicioso.com
~~~

Se o site não usa tokens CSRF, um atacante pode criar uma página HTML maliciosa como essa:

~~~html
<html>
  <body>
    <form action="http://vulneravel.com/alterar-email" method="POST" id="csrf-form">
      <input type="hidden" name="novo_email" value="atacante@malicioso.com" />
    </form>
    <script>
      document.getElementById("csrf-form").submit();
    </script>
  </body>
</html>
~~~

### Como funciona

1. A vítima, autenticada em `vulneravel.com`, acessa essa página (aqui entra algo importante, muitas vezes será preciso técnicas de phishing para induzir a pessoa a acessar a página)
2. O formulário é enviado automaticamente pelo JavaScript através da função `.submit();`
3. O navegador inclui o cookie de sessão da vítima, e o email é alterado.

    Teste no Bug Bounty: Hospede esse HTML em um servidor controlado ou use ferramentas como ngrok para simular o ataque.



### Exemplos Práticos

**Transferência de Dinheiro**

Requisição legítima: `POST /transferir?valor=1000&conta=12345`. <br>
PoC: Um `<img src="http://banco.com/transferir?valor=1000&conta=12345">` em um fórum. O navegador tenta carregar a "imagem" e executa a transferência.

**Mudança de Senha**

Requisição: `POST /mudar-senha` com nova_senha=123456. <br>
PoC: Formulário oculto como o do exemplo acima.

**Exclusão de Conta**

Requisição: `GET /deletar-conta` <br>
PoC: `<iframe src="http://site.com/deletar-conta" style="display:none"></iframe>`

### Dicas para Bug Bounty

**Onde procurar:** Teste formulários, endpoints de API e ações administrativas.
**Ferramentas**: Use Burp Suite para capturar e repetir requisições; curl ou Postman para testar manualmente; ferramenta devtools do próprio navegador
**Mitigação**: Se o site usa tokens CSRF (ex: `<input type="hidden" name="csrf_token" value="xyz">`), o ataque falha. Reporte apenas se conseguir bypass.
**Impacto**: Foque em ações críticas (ex: roubo de conta, perda de dados) para maximizar a recompensa.


## COMO MITIGAR O CSRF?

- Implementar tokens CSRF únicos para cada requisição.
- Utilizar SameSite Cookies para limitar o envio de cookies apenas ao próprio site.
- Exigir reautenticação para ações críticas.
- Configurar CORS corretamente para evitar requisições indesejadas.


## CONCLUSÃO GERAL

O objetivo do ataque CSRF é enganar um usuário autenticado para que ele execute uma ação sem perceber. O atacante precisa **induzir a vítima a carregar ou interagir com uma página maliciosa**. E o phishing é um dos principais métodos para isso.

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