<h1>SQL Injection</h1>

<h2>O que é SQL Injection (SQLi)?</h2>

SQL Injection é uma vulnerabilidade web que permite o atacante interferir em consultas que a aplicação faz ao banco de dados. Isso geralmente permite o
atacante visualizar dados que normalmente ele não consegueria recuperar. Isso pode incluir dados pertencentes a outros usuarios ou qualquer outros dados que
o aplicativo possa acessar. Em muitos casos, um atacante pode modificar ou deletar esses dados, causando mudanças persistentes no conteudo ou comportamento 
da aplicação.
Em algumas situações, um invasor pode escalar um ataque de injeção de SQL para comprometer o servidor subjacente ou outra infraestrutura de backend ou 
realizar um ataque de negação de serviço.

<h2>Exemplos de SQL Injection</h2>

<li>Retrieving hidden data, quando você modifica o SQL para retornar resultados adicionais</li>
<li>Subverting application logic, quando você muda a consulta para interferir na logica da aplicação</li>
<li>UNION Attacks, Quando você recupera dados de uma tabela de banco de dados diferente</li>
<li>Examining the database,quando você pode extrair a informação sobre a versão de estrutura do banco de dados</li>
<li>Blind SQL Injection, quando o resultado da consulta não é retornado na resposta da aplicação</li>

<h2>Recuperando dado oculto</h2>

Considere que a aplicação que mostra produtos em categorias diferentes. Quando o usuario clicla na categoria Gifts, o seu navegador pede pela seguinte URL:

    https://insecure-website.com/products?category=Gifts
    
Isso faz com que a aplicação faça uma consulta SQL para retornar detalhes de produtos relevantes do banco de dados:

    SELECT * FROM products WHERE category = 'Gifts' AND released = 1
    
Essa consulta SQL solicita que o banco de dados retorne:

<li>Todos detalhes (*)</li>
<li>da tabela de produtos</li>
<li>onde a categoria é Gifts</li>
<li>e released é 1</li>

A restrição released = 1 é usada para esconder produtos que não foram lançados, para produtos ainda não lançados presumisse release = 0.
A aplicação não implementa nenhuma defesa contra ataques SQL injection, então um atacante pode construir um ataque como:

    https://insecure-website.com/products?category=Gifts'--
    
Isso resulta em uma consulta SQL:
     
    SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
    
A sequencia de caracteres -- indica um comentario em SQL, e significa que o resto da consulta não sera interpretada. Isso remove efetivamente o resto da consulta de forma que não existe mais o AND released = 1. Isso significa que todos produtos serão mostrados, incluindo os produtos não lançados.

Indo além, um invasor pode fazer com que a aplicação exiba todos os produtos em qualquer categoria, incluindo categorias que ele não conhece:

    https://insecure-website.com/products?category=Gifts'+OR+1=1--
    
O resultado dessa consulta SQL:

    SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
    
A consulta modificada retornará todos os itens em que a categoria seja Gifts ou 1 seja igual a 1. Como 1=1 é sempre verdadeiro, a consulta retornará todos os itens.

<h2>Subvertendo a lógica da aplicação</h2>

Considere que a aplicação deixa os usuarios logar com o username e password. Se o usuario enviar um username wiener e a senha bluecheese, a aplicação irá verificar as credenciais por meio da sequinte consulta SQL:

        SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
        
Se a consulta retornar detalhes do usuario então o login é efetuado, senão, ela será rejeitada.
Aqui o atacante pode logar em qualquer usuario sem a senha simplesmente usando o comentario SQL -- para remover a senha verificada pela clausula WHERE na consulta. Por exemplo, enviando um usuario administrator'-- e uma senha em branca resultará na seguinte query:

        SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
        
A consulta irá retorna o usuario que o username é administrator e irá logar o atacante nessa conta.

<h2>Retornando dados de outra tabela no banco de dados</h2>

No caso onde o resultado da consulta SQL é retornado na resposta da aplicação, o atacante pode aproveitar a vulnerabilidade de SQL Injection para retornar dados de uma outra tabela no banco de dados. Isso pode ser feito usando a clausula UNION, que deixa você executar um SELECT adicional na consulta e acrescenta no final da consulta original.

Por exemplo, se a aplicação executa a seguinte consulta contendo a entrada do usuario 'Gifts'.

        SELECT name, description FROM products WHERE category = 'Gifts'
        
Então o atacante pode enviar a entrada:

        ' UNION SELECT username, password FROM users--
        
Isso fará a aplicação retornar todos usernames e senhas, juntamente com os nomes e descrições dos produtos.

<h2>Examinando o banco de dados</h2>

Após a identificação de um SQL Injection, geralmente é útil obter algumas informações sobre o banco de dados. Essas informações podem abrir caminhos para uma maior exploração.

Você pode consultar por detalhes de versão do banco de dados. A maneira como isso é feito depende do tipo de banco de dados. Por exemplo, no Oracle você pode executar:

        SELECT * FROM v$version
        
Você também pode determinar tabelas existentes, e quais colunas ela contem. Por exemplo, na maioria dos bancos de dados você pode executar a seguinte consulta para listar as tabelas:

        SELECT * FROM information_schema.tables
        
