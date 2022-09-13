<h1>..:: Directory Tranversal ::..</h1>

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
