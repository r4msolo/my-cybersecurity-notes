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
  <li><b>Explorando XXE para retornar arquivos:</b> Uma entidade externa é definida contendo o conteúdo de um arquivo e retorna na resposta da aplicação. </b></li>
  <li><b>Explorando XXE para ataques SSRF:</b> Uma entidade externa é definida com base em uma URL em um sistema de backend.</li>
  <li><b>Explorando blind XXE exfiltrando dados out-of-band:</b> Onde dados confidenciais são transmitidos do servidor da aplicação para um sistema que o invasor controla.</li>
  <li><b>Explorando blind XXE para retornar dados via mensagens de erro:</b> Onde o invasor pode acionar uma mensagem de erro de análise contendo dados confidenciais.</li>
</ul>

<h4>1) Exploiting XXE to retrieve files</h4>

Para executar um ataque de injeção XXE que recupera um arquivo arbitrário do sistema de arquivos do servidor, você precisa modificar o XML enviado de duas maneiras:

<li>Introduzir (ou editar) um elemento doctype que define uma entidade externa que contém o caminho para o arquivo.</li>
<li>Edite um valor de dados no XML que é retornado na resposta da aplicação, para usar a entidade externa definida.</li>

Por exemplo, suponha que uma aplicação de compras verifique a quantidade em estoque de um produto enviando o seguinte XML para o servidor:

    <?xml version="1.0" encoding="UTF-8"?>
    <stockCheck><productId>381</productId></stockCheck>
   
A aplicação não tem uma defesa em particular contra ataques XXE, então você pode explorar essa vulnerabilidade por exemplo trazendo o arquivo /etc/passwd enviando o seguinte payload XXE:

      <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
      <stockCheck><productId>&xxe;</productId></stockCheck>

Esse payload define um entidade externa como &xxe; que contem o valor de /etc/passwd e usa o valor da entidade productId. Desse modo aplicação retorna o conteudo do arquivo:

      Invalid product ID: root:x:0:0:root:/root:/bin/bash
      daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
      bin:x:2:2:bin:/bin:/usr/sbin/nologin
      ...

<h4>2) Exploiting XXE to perform SSRF Attacks</h4>

Além da recuperação de dados sensíveis, o outro impacto principal dos ataques XXE é que eles podem ser usados para executar a falsificação de solicitação do servidor (SSRF). Essa é uma vulnerabilidade potencialmente séria, na qual a aplicação do lado do servidor pode ser induzido para fazer solicitações HTTP a qualquer URL que o servidor possa acessar, isso inclui serviços internos no quais não estão expostos diretamente a internet.

Para explorar uma vulnerabilidade XXE para executar um ataque SSRF, você precisa definir uma entidade XML externa usando a URL que deseja segmentar e usar a entidade definida dentro de um valor de dados. Se você puder usar a entidade definida em um valor de dados que for retornado na resposta do aplicativo, poderá visualizar a resposta da URL na resposta da aplicação e, portanto, obterá interação bidirecional com o sistema de backend. Caso contrário, você só poderá realizar ataques cegos de SSRF (que ainda podem ter consequências críticas).

No exemplo XXE a seguir, a entidade externa fará com que o servidor faça uma solicitação HTTP de backend a um sistema interno dentro da infraestrutura da organização:

      <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>
 
<h4>3) Blind XXE vulnerabilities</h4>

Muitas instâncias de vulnerabilidade XXE são blind (cegas). Isso significa que o aplicativo não retorna os valores de entidades externas definidas em suas respostas e, portanto, a recuperação direta dos arquivos do lado do servidor não é possível.

As vulnerabilidades Blind XXE ainda podem ser detectadas e exploradas, mas são necessárias técnicas mais avançadas. Às vezes, você pode usar técnicas out-of-band para encontrar vulnerabilidades e explorá-las para exfiltrar os dados. E às vezes você pode acionar erros de análise XML que levam à divulgação de dados confidenciais nas mensagens de erro.

<h4>Encontrando a superfície de ataque oculta para injeção de XXE</h4>

A superfície de ataque para vulnerabilidades de injeção XXE é óbvia em muitos casos, porque o tráfego HTTP normal da aplicação inclui solicitações que contêm dados no formato XML. Em outros casos, a superfície de ataque é menos visível. No entanto, se você olhar nos lugares certos, encontrará a superfície de ataque XXE em solicitações que não contêm nenhum XML.

<h4> XInclude Attacks</h4>
Algumas aplicações recebem dados submitidos do cliente, incorporam-os no lado do servidor em um documento XML e analisam o documento. Um exemplo disso ocorre quando os dados substituídos pelo cliente são colocados em uma solicitação de SOAP de backend, que é processada pelo serviço SOAP (<a href="https://www.w3schools.com/xml/xml_soap.asp" target="_blank">Simple Object Access Protocol</a>) de backend.
<br>
Nessa situação, você não pode realizar um ataque XXE clássico, porque você não controla todo o documento XML e, portanto, não pode definir ou modificar um elemento DOCTYPE. No entanto, você pode usar XInclude em vez disso. XInclude é uma parte da especificação XML que permite que um documento XML seja construído a partir de subdocumentos. Você pode colocar um ataque de XInclude em qualquer valor de dados em um documento XML, para que o ataque possa ser executado em situações em que você controla apenas um único item de dados que é colocado em um documento XML do lado do servidor.
<br>
Para realizar um ataque XInclude, você precisa fazer referência ao XInclude namespace e fornecer o caminho para o arquivo que deseja incluir. Por exemplo:

      <foo xmlns:xi="http://www.w3.org/2001/XInclude">
      <xi:include parse="text" href="file:///etc/passwd"/></foo>

<h4>XXE attacks via file upload</h4>

Algumas aplicações permitem que os usuários enviem arquivos que são então processados no lado do servidor. Alguns formatos de arquivo comuns usam XML ou contêm subcomponentes XML. Exemplos de formatos baseados em XML são formatos de documentos do Office, como DOCX e formatos de imagem como SVG.

Por exemplo, um aplicativo pode permitir que os usuários enviem imagens e processem ou validem-as no servidor após o upload. Mesmo que o aplicativo espere receber um formato como PNG ou JPEG, a biblioteca de processamento de imagens que está sendo usada pode suportar imagens SVG. Como o formato SVG usa XML, um invasor pode enviar uma imagem SVG maliciosa e, portanto, alcançar a superfície de ataque oculta para as vulnerabilidades XXE.
<h4>XXE attacks via modified content type</h4>
A maioria das solicitações de postagem usa o tipo de conteúdo padrão gerado pelos formulários HTML, como Application/X-Www-Form-Urlancoded. Alguns sites da Web esperam receber solicitações neste formato, mas toleram outros tipos de conteúdo, incluindo XML.

Exemplo, se a solicitação normal contiver o seguinte:

        POST /action HTTP/1.0
        Content-Type: application/x-www-form-urlencoded
        Content-Length: 7
        
        foo=bar
        
 Então você poderá enviar a seguinte solicitação, com o mesmo resultado:

      POST /action HTTP/1.0
      Content-Type: text/xml
      Content-Length: 52

      <?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>

Se a aplicação tolera solicitações contendo XML no corpo da mensagem e analisar o conteúdo do corpo como XML, você poderá atingir a superfície de ataque XXE oculta simplesmente reformatando solicitações para usar o formato XML. 

<h4>Como encontrar e testar vulnerabilidades XXE</h4>
A grande maioria das vulnerabilidades XXE pode ser encontrada de maneira rápida e confiável, usando o scanner de vulnerabilidade da Burp Suite. Testar manualmente as vulnerabilidades XXE geralmente envolve:
<ul>
  <li>Teste para recuperação de arquivos Definindo uma entidade externa com base em um arquivo de sistema operacional bem conhecido e usando essa entidade em dados que são retornados na resposta do aplicativo.</li>
  <li>Testando as vulnerabilidades Blind XXE, definindo uma entidade externa baseada em um URL para um sistema que você controla e monitorando as interações com esse sistema. O Burp Collaborator client é perfeito para esse fim.</li>
  <li>Testando a inclusão vulnerável de dados não xml fornecidos pelo usuário em um documento XML do lado do servidor usando um ataque Xinclude para tentar recuperar um arquivo de sistema operacional bem conhecido.</li>
</ul>

<h4>Como prevenir vulnerabilidades XXE</h4>
Praticamente todas as vulnerabilidades XXE surgem porque a biblioteca de análise XML do aplicativo suporta recursos XML potencialmente perigosos que o aplicativo não precisa ou pretende usar. A maneira mais fácil e eficaz de prevenir ataques XXE é desativar esses recursos.
<br>
Geralmente, é suficiente desativar a resolução de entidades externas e desativar o apoio ao Xinclude. Isso geralmente pode ser feito por meio de opções de configuração ou substituindo programaticamente o comportamento padrão. Consulte a documentação para sua biblioteca de análise XML ou API para obter detalhes sobre como desativar recursos desnecessários.
<hr>
<h2>References:</h2>
https://portswigger.net/web-security/xxe
