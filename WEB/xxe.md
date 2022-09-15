# XXE Injection

<h1>O que seria o XML External Entity Injection? (XXE)</h1>

É uma falha de segurança WEB que permite o atacante interferir em uma aplicação que processe dados de XML. Isso permite que um atacante visualize arquivos
no filesystem do servidor da aplicação. Em alguns casos o atacante pode escalar um ataque XXE para comprometer os servidores adjacentes ou outra estrutura
de backend, aproveitando a vulnerabilidade para tentar ataques de falsificação de solicitação do lado do servidor SSRF (Server-side Request Forgery).

<h2>Como surgem as vulnerabilidades de XXE?</h2>
Algumas aplicações usam o formato XML para transmitir dados entre o navegador e o servidor. Aplicações que fazem isso quase sempre utilizam uma biblioteca
padrão ou de plataforma para processar os dados XML no servidor.

<h2> Quais são os tipos de ataque XXE? </h2>

<ul>
  <li>Explorando XXE para retornar arquivos</li>
  <li>Explorando XXE para ataques SSRF</li>
  <li>Explorando blind XXE exfiltrando dados out-of-band</li>
  <li>Explorando blind XXE para retornar daodos via mensagens de erro</li>
</ul>
