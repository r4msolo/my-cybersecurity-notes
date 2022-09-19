<h1>Command Injection</h1>

<h2>O que é o Command Injection? </h2>

A injeção de comando do SO (também conhecida como injeção de shell) é uma vulnerabilidade de segurança da Web que permite que um invasor execute comandos
arbitrários do sistema operacional (SO) no servidor que está executando uma aplicação e, normalmente, compromete totalmente a aplicação e todos os seus 
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

Se essa entrada for enviada no parametro productID, então temos a execução do comando pela aplicação da seguinte forma:

        stockreport.pl & echo aiwefwlguh & 29

O comando echo exibe a string que foi passada como entrada ecoando isso na saida, acaba sendo uma forma útil de testar tipos de command injection. O caracter & é um comando separador do shell, ele consegue concatenar comandos executando um após o outro. O resultado disso é retornado para o usuario como no exemplo da seguinte forma:

    Error - productID was not provided
    aiwefwlguh
    29: command not found

As 3 linhas de saida demostram que:

<li>O comando stockreport.pl original foi executado sem seus argumentos esperados e, portanto, retornou uma mensagem de erro.</li>
<li>O comando injetado echo foi executado e a string fornecida foi ecoada na saída.</li>
<li>O argumento original 29 foi executado como um comando, o que causou um erro.</li>

Colocar o separador de comando adicional & após o comando injetado geralmente é útil porque separa o comando injetado do que segue o ponto de injeção. Isso reduz a probabilidade de que o que se segue impeça a execução do comando injetado.

<h2>Blind OS command injection vulnerabilities</h2>

Muitas vezes instancias de OS command injection são do tipo blind (cega). Isso significa que a aplicação não retorna saida do comando em uma resposta HTTP. Vulnerabilidades do tipo blind ainda podem ser exploradas, porem usando técnicas diferentes.
Considere que o site permite o usuario enviar um feedback. O usuario entra com o email dele e deixa uma mensagem de feedback. O lado do servidor gera um email para o administrador do site contendo a mensagem. Para fazer isso ele chama fora do programa mail com os seguintes detalhes. por exemplo:

        mail -s "This site is great" -aFrom:peter@normal-user.net feedback@vulnerable-website.com
        
A saida do programa mail (se tiver) não é retornada na resposta da aplicação, e então usamos o echo payload não seria efetivo, Nessa situação, você pode usar uma variedade de técnicas para detectar e explorar uma vulnerabilidade.

<h3>Detectando um Blind OS Commmand Injection usando tempo de espera</h3>

Você pode usar um comando injetado que irá executar um delay, permitindo assim você confirmar que o comando foi executado com base no tempos de espera informado. O ping é um comando efetiivo para fazer isso, ele permite especificar o numero de pacotes ICMP para enviar, e portanto ele leva o tempo necessario para o comando rodar.

        & ping -c 10 127.0.0.1 &
        
Esse comando irá causar na aplicação um loopback no adaptador de rede por 10 segundos. 
