<h1>Server-side Request Forgery</h1>

<h2>O que é o SSRF?</h2>

Server-side request forgery (SSRF) é uma vulnerabilidade web que permite o atacante induzir o servidor a fazer requisições para um local não intencional.
Em um tipico ataque SSRF, o atacante poderia fazer o servidor criar uma conexão a serviços da rede interna, dentro da infraestrutura da organização. Em
outros casos, ele pode forçar o servidor a conectar em servidores arbitrarios externos, potencialmente vazando informações como credenciais de acesso.

<h2>Qual o imapacto de ataques SSRF?</h2>
Um ataque SSRF bem sucedido pode resultar em ações não autorizadas ou acesso a informações dentro da organização. Na propria aplicação vulneravel ou em
outros sistemas back-end que a aplicação possa se comunicar. Em algumas situaçõe, o SSRF pode permitir que o atacante execute comandos arbitrarios.

<h2>Ataques SSRF comuns</h2>

SSRF pode explorar relações de confiança para escalar um ataque do aplicativo vulneral e realizar ações não autorizadas. Essa relação de confiança pode
existir em relação ao proprio servidor, ou em relação a outros sistemas back-end dentro da mesma organização.

<h2>Ataque SSRF contra o proprio servidor</h2>

Em um ataque de SSRF contra o proprio servidor, o atacante pode induzir com que a aplicação faça uma requisição HTTP de volta ao servidor que está
hospedando a aplicação, via interface de loopback. Isso irá envolver usar o hostname na url como 127.0.0.1 (endereço reservado para o loopback) ou localhost.

Por exemplo, considerando que uma aplicação de compras deixa o usuario ver e o item está em estoque em uma determinada loja. Para prover informações de
estoque, a aplicação precisa fazer varias consultas a REST API no back-end, dependendo do produto e loja em questão. A função é implementada passando a URL
para o endpoint da API no back-end via HTTP request no front-end. Então quando o usuario ver o status de estoque para um item, o seu navegador irá fazer a
seguinte requisição:

    POST /product/stock HTTP/1.0
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 118

    stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
    
Isso irá fazer com que o server faça uma requisição para a URL especificada, retornando o status de estoque para o usuario.
Nessa situação, um atacante pode modificar a requisição para uma URL local para o proprio servidor. POr exemplo:

      POST /product/stock HTTP/1.0
      Content-Type: application/x-www-form-urlencoded
      Content-Length: 118

      stockApi=http://localhost/admin
      
 Claro que o atacante poderia acessar diretamente a URL administrativa, porem em muitos casos essa funcionalidade é permitida apenas para usuarios
 autenticados. Portando quando um atacante visitar diretamente essa pagina administrativa ele não verá nada interessante, mas quando a requisição vem da
 propria maquina local, o acesso normal é bypassado. Dessa forma a aplicação mostra o acesso completo a funcionalidade administrativa, porque a solicitação
 parece ter origem de um local confiável.
