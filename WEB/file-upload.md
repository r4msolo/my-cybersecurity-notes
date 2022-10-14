<h1>O que é a vulnerabilidade de File Upload?</h1>

A vulnerabilidade de file upload acontece quando o servidor permite o usuario fazer upload de arquivo para o sistema sem fazer as validações do 
conteudo, nome de arquivo, tipo ou tamanho. Deixar de aplicar as restrições adequadamente pode significar que até mesmo uma função básica de 
upload de imagem pode ser usada para fazer upload de arquivos arbitrários e potencialmente perigosos. Isso inclui até mesmo arquivos de scripts
no lado do servidor que permitira uma execução de codigo remoto (RCE - Remote Code Execution).

<h2>Como surgem as vulnerabilidades de File Upload?</h2>

É raro os sites que não possuem nenhuma restrição sobre os arquivos que os usuarios podem carregar. Normalmente os desenvovedores implementam o que eles
acreditam ser uma validação mais robusta, mas que em muitos casos podem ser facilmente contornado.
Por exemplo, eles podem criar uma blacklist contendo as extensões de arquivos nos quais não serão permitidos pois são considerados perigosos, é possivel explorar algumas variações dessas extensões como por exemplo a aplicação pode bloquear arquivos .php mas acabar deixando passar um .phtml que faria a interpretação de forma igual ou ainda utilizar nullbyte no nome do arquivo como por exemplo arquivo.php%00 ou alguma variação do tipo. Para mais exemplos leia esse artigo do hacktricks <a href='https://book.hacktricks.xyz/pentesting-web/file-upload#bypass-file-extensions-checks' target='_blank'>Bypass File Extension</a>.
