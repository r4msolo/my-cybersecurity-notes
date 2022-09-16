# XXE Injection

<h1>O que seria o XML External Entity Injection? (XXE)</h1>

É uma falha de segurança WEB que permite o atacante interferir em uma aplicação que processe dados de XML. Isso permite que um atacante visualize arquivos
no filesystem do servidor da aplicação. Em alguns casos o atacante pode escalar um ataque XXE para comprometer os servidores adjacentes ou outra estrutura
de backend, aproveitando a vulnerabilidade para tentar ataques de falsificação de solicitação do lado do servidor SSRF (Server-side Request Forgery).

<h2>Como surgem as vulnerabilidades de XXE?</h2>
Algumas aplicações usam o formato XML para transmitir dados entre o navegador e o servidor. Aplicações que fazem isso quase sempre utilizam uma biblioteca
padrão ou de plataforma para processar os dados XML no servidor. As vulnerabilidades XXE surgem porque a especificação XML contém vários recursos potencialmente perigosos, e os analisadores padrão suportam esses recursos, mesmo que normalmente não sejam usados pela aplicação.

<h2> Quais são os tipos de ataque XXE? </h2>

<ul>
  <li><b>Explorando XXE para retornar arquivos:</b>Uma entidade externa é definida contendo o conteúdo de um arquivo e retorna na resposta do aplicativo.</b></li>
  <li><b>Explorando XXE para ataques SSRF:</b> Uma entidade externa é definida com base em um URL em um sistema de back-end.</li>
  <li><b>Explorando blind XXE exfiltrando dados out-of-band:</b> Onde dados confidenciais são transmitidos do servidor de aplicativos para um sistema que o invasor controla.</li>
  <li><b>Explorando blind XXE para retornar dados via mensagens de erro:</b> Onde o invasor pode acionar uma mensagem de erro de análise contendo dados confidenciais.</li>
</ul>

<h4>Exploiting XXE to retrieve files</h4>

Para executar um ataque de injeção XXE que recupera um arquivo arbitrário do sistema de arquivos do servidor, você precisa modificar o XML enviado de duas maneiras:

<li>Introduzir (ou editar) um elemento doctype que define uma entidade externa que contém o caminho para o arquivo.</li>
<li>Edite um valor de dados no XML que é retornado na resposta do aplicativo, para usar a entidade externa definida.</li>

Por exemplo, suponha que um aplicativo de compras verifique a quantidade em estoque de um produto enviando o seguinte XML para o servidor:

    <?xml version="1.0" encoding="UTF-8"?>
    <stockCheck><productId>381</productId></stockCheck>
   
A aplicação não tem uma defesa em particular contra ataques XXE, então você pode explorar essa vilnerabilidade por exemplo trazendo o arquivo /etc/passwd enviando o seguinte payload XXE:

      <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
      <stockCheck><productId>&xxe;</productId></stockCheck>

Esse payload define um entidade externa como &xxe; que contem o valor de /etc/passwd e usa o valor da entidade productId. Desse modo aplicação retorna o conteudo do arquivo:

      Invalid product ID: root:x:0:0:root:/root:/bin/bash
      daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
      bin:x:2:2:bin:/bin:/usr/sbin/nologin
      ...

<h4>Exploiting XXE to perform SSRF Attacks</h4>

Além da recuperação de dados sensíveis, o outro impacto principal dos ataques XXE é que eles podem ser usados para executar a falsificação de solicitação do servidor (SSRF). Essa é uma vulnerabilidade potencialmente séria, na qual o aplicativo do lado do servidor pode ser induzido para fazer solicitações HTTP a qualquer URL que o servidor possa acessar.

Para explorar uma vulnerabilidade XXE para executar um ataque SSRF, você precisa definir uma entidade XML externa usando a URL que deseja segmentar e usar a entidade definida dentro de um valor de dados. Se você puder usar a entidade definida em um valor de dados que for retornado na resposta do aplicativo, poderá visualizar a resposta da URL na resposta da aplicação e, portanto, obterá interação bidirecional com o sistema de backend. Caso contrário, você só poderá realizar ataques cegos de SSRF (que ainda podem ter consequências críticas).
