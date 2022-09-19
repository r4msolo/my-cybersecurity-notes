<h1>Command Injection</h1>

<h2>O que é o Command OS injection? </h2>

A injeção de comando do SO (também conhecida como injeção de shell) é uma vulnerabilidade de segurança da Web que permite que um invasor execute comandos
arbitrários do sistema operacional (SO) no servidor que está executando um aplicativo e, normalmente, compromete totalmente o aplicativo e todos os seus 
dados. Muitas vezes, um invasor pode aproveitar uma vulnerabilidade de injeção de comando do SO para comprometer outras partes da infraestrutura 
de hospedagem, explorando relações de confiança para direcionar o ataque para outros sistemas dentro da organização.

<h2>Executando comandos arbitrarios</h2>

Considere que uma aplicação de compras permite que o usuario veja os itens em seu estoque particular. Essa informação é acessada conferme a seguinte URL:

    https://insecure-website.com/stockStatus?productID=381&storeID=29
    
Para fornecer essa informação, a aplicação precisa consultar diversos sistemas. Essa funcionalidade chama uma shell com o argumento do produto e ID da loja:

    stockreport.pl 381 29
    
Essa saida do comando de estoque ṕara determinado item é retornado para o usuario. Desde que a aplicação não tenha defesas contra OS Command Injection, o 
atacante pode submeter a seguinte entrada para executar o comando arbitrario:

    & echo aiwefwlguh &
