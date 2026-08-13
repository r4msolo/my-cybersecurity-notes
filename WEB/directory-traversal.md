<h1>..:: Directory Tranversal / Path Tranversal ::..</h1>

É uma vulnerabilidade web que permite com que o hacker faça leitura de arquivos arbitrários no servidor que está rodando a aplicação, isso também inclui o código da aplicação, dados, credenciais para o sistema backend, em alguns casos o atacante pode escrever arquivos arbitrários no servidor, permitindo modificar dados e em ultimos casos tomar controle do servidor (Remote Code Execution).

Alguns dos payloads comuns:

<h4><li>Lendo Arquivos Arbitrários</li></h4>
<br>
Considere que a aplicação mostre imagens de itens para venda. Imagens são carregadas via tags HTML como por exemplo:

    <img src=”/loadImage?filename=218.png”>

O loadImage URL recebe o parâmetro filename e retorna o conteúdo do arquivo especificado. Os arquivos de imagens são armazenados no disco em /var/www/images/. Para retornar uma imagem, a aplicação adiciona o nome de arquivo solicitado a esse diretório base e usa a API filesystem para ler conteúdos do arquivo. No caso abaixo a aplicação lê o arquivo do seguinte caminho.

    /var/www/images/218.png

Quando a aplicação não implementa defesas contra directory traversal o atacante pode pedir por um arquivo de sistema do servidor.

    https://insecure-website.com/loadImage?filename=../../../etc/passwd

Isso causa que a aplicação tentará ler o arquivo da seguinte forma:

    /var/www/images/../../../etc/passwd

A sequencia de ../ significa que iremos subir um level na estrutura dos diretórios, a ideia seria voltar de diretório até conseguirmos ler por exemplo o arquivo /etc/passwd.

Nos sistemas baseado em UNIX o passwd mostra detalhes sobre os usuários que estão registrados no servidor.

Em sistema Windows temos a diferença que para retornar um diretório usamos a barra invertida ..\\..\\..\\windows\win.ini, temos aqui o equivalente ao passwd no sitema de janelas :)

<h4><li>Obstáculos comuns para explorar vulnerabilidades de Path Tranversal</li></h4>

Muitas aplicações implementam algum tipo de defesa contra ataques path tranversal no input do usuario, e isso geralmente pode ser contornado.

Se um aplicativo remover ou bloquear sequências de path tranversal do nome de arquivo fornecido pelo usuário, talvez seja possível contornar a defesa usando uma variedade de técnicas.

Você pode usar um caminho absoluto da raiz do sistema de arquivos, como filename=/etc/passwd, para referenciar diretamente um arquivo sem usar nenhuma sequência de travessia.

Talvez seja preciso utilizar sequencias de path tranversal com caracteres duplicados como por exemplo ....//....//....//etc//passwd, dependendo da forma como foi escrito o filtro pode simplesmente deletar uma das sequencias "../" por exemplo ....//....//....//etc//passwd na hora que a aplicação deletar a sequencia do payload ele juntara formando um simples ../../../etc/passwd.

Em alguns contextos, como em um caminho de URL ou no parâmetro de nome de arquivo de uma solicitação multipart/form-data, os servidores da Web podem remover qualquer sequência de directory tranversal antes de passar sua entrada para a aplicação. Às vezes, você pode ignorar esse tipo de higienização por codificação de URL ou até mesmo codificação de URL dupla, os caracteres ../, resultando em %2e%2e%2f ou %252e%252e%252f, respectivamente. Várias codificações não padrão, como ..%c0%af ou ..%ef%bc%8f, também podem funcionar.

Se um aplicativo exigir que o nome do arquivo fornecido pelo usuário comece com a pasta base esperada, como /var/www/images, talvez seja necessario incluir a pasta base seguida por sequências de passagem adequadas. Por exemplo:

    filename=/var/www/images/../../../etc/passwd
    
Se um aplicativo exigir que o nome de arquivo fornecido pelo usuário termine com uma extensão de arquivo esperada, como .png, talvez seja possível usar um byte nulo para encerrar efetivamente o caminho do arquivo antes da extensão necessária. Por exemplo:

    filename=../../../etc/passwd%00.png
    
 <h4>Como prevenir ataques de Directory Tranversal</h4>
 
 A maneira mais efetiva de previnir esse ataque é evitar passar completamento o caminho informado pelo usuario para a API Filesystem. Muitas funções de aplicações podem ser reescritas para entregar o mesmo resultado de maneira mais segura.
 
<hr>
Confira nesse link mais alguns exemplos e payloads utilizados em path tranversal:<br>
https://book.hacktricks.xyz/pentesting-web/file-inclusion
<br>
Referencia:<br><br>
https://portswigger.net/web-security/file-path-traversal
