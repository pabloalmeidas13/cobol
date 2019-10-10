
# Início
--------------------------------------

Quando usamos nossos programas COBOL, as vezes precisamos executar funções que o COBOL não possui ou não tem um bom suporte. Essas limitações do COBOL devem-se as vezes ao objetivo da criação da linguagem. Só como comentário, a linguagem COBOL foi criada com o objetivo do desenvolvimento de aplicações comerciais, com alto processamento e pouca exigência de hardware, o que nos dá argumentos para justificar a não-criação de complexas interfaces com o sistema operacional ou com periféricos de hardware.

Para resolver problemas que o COBOL não consegue resolver sozinho, precisamos usar através do COBOL uma outra linguagem, que possa nos dar os resultados que precisamos. Geralmente, a linguagem C é uma das maiores companheiras do COBOL(e de outras linguagens) pois, além de ser a linguagem de programação mais conhecida no mundo da informática(depois do assembler), é a linguagem que possui suporte a praticamente "tudo" o que pode ser feito no mundo da informática. Como o C é uma linguagem que tem um comitê de desenvolvimento(Como o ANSI e o ISO/IEC), ela é uma linguagem muito portável, de forma que o COBOL não tem nada a perder "conversando" com ela; só a ganhar.

Um grande detalhe neste assunto é que o COBOL e o C interpretam as variáveis de maneira diferente e para fazermos programas COBOL comunicarem-se com programas C, é necessário seguirmos algumas regras. Estas regras são o objetivo desse estudo.

Passagem de parâmetros

Passagem de parâmetros é o nome dado a técnica de se chamar um programa a partir de um programa já carregado na memória sendo que, nesta chamada, é permitido que o programa chamado troque informações com o programa que o chamou, embora a troca de informações depende unicamente de como a aplicação(isto inclui o programa chamado e o programa chamador) está sendo projetada.

Passagem

Há dois métodos que podem ser usados para se passar um parâmetro: Por valor e Por referência. Vamos explicá-los: 
Por valor: Consiste em passar os parâmetros em valor numérico. Estes valores deverão ser passados em formato binário(O que no COBOL é determinado como variáveis COMP. No C, isso depende do tipo de modelador de dados que você usa).
Por referência: Consiste em passar os parâmetros em valor alfanumérico. Como o COBOL e o C tem diferentes maneiras de interpretar os dados(Toda variável C que é declarada possui um zero binário), a variável declarada no COBOL deve ter no seu final um zero binário, para que o C saiba identificar corretamente o seu tamanho). A passagem de parâmetros por referência é comumente usada para armazenar na memória valores que possam ser vistos pelo programa chamador e pelo programa chamado.

Recebimento

Para recebermos um parâmetro de um programa chamado, devemos verificar se o protótipo da interface deste programa permite que os parâmetros sejam retornados. Embora a expressão "protótipo da interface" não seja a melhor dependendo da linguagem(devido a nomenclatura para o desenvolvimento desta), o objetivo é explicar que será possível retornar parâmetros somente se a área de ligação entre o programa chamador e o programa chamado permitir isso.

Através do C, podemos retornar para o COBOL um valor ou vários valores, isso só dependendo do tipo de dado a ser usado na interface da função chamada. Para retornarmos do C um único valor numérico(de formato binário) para o COBOL, podemos usar tipos int, char, float, double, long e short. Para retornarmos vários valores, devemos usar um tipo void.
A explicação disso se dá por que a função void trabalha com ponteiros de memória. Então, o programador C pode pegar o valor destes ponteiros que foram passados pelo programa COBOL e modificá-los na mesma área de memória que foram passados, de forma que o programa COBOL irá trazer estes valores de volta sem saber que eles foram modificados, pois quem modificou estes valores foi o programa C e não o programa COBOL. Isso acontece por que o run-time do compilador COBOL não faz uma ligação específica entre um programa COBOL e um programa nao-COBOL. A única coisa que o run-time do compilador COBOL determina nestes casos é que está sendo chamado um programa externo e que deve ser habilitado uma nova run-unit para ele. 
Na verdade, quando fazemos este tipo de interfaceamento, usamos uma técnica chamada de "falsificação" de valores, pois o que acontece ao recebermos valores por referência do C para o COBOL é um pseudo-recebimento de parâmetros. 

No COBOL, não nos referimos desta maneira(protótipo da interface) aos programas chamados, pois COBOL não tem, em nomenclatura, suporte a funções. 
Quando vamos falar sobre passagem ou recebimento de parâmetros, geralmente explicamos sobre a declaração de dados da LINKAGE SECTION e da necessidade da sua similariedade(quanto as variáveis) para o funcionamento da passagem ou recebimento de parâmetros. No COBOL é necessário apenas verificar se há no programa chamador e no programa chamado as variáveis declaradas, de mesmo tamanho, de mesmo nome e de mesmo nível. Também, deve-se verificar a cláusula PROGRAM-ID do programa COBOL, para que o run-time do compilador COBOL possa encontrar o objeto chamado. 
No COBOL nota-se também a facilidade da passagem de parâmetros, uma vez que entre programas COBOL não é necessário determinar se os parâmetros estão sendo passados ou recebidos por valor ou referência. O próprio run-time do compilador COBOL se responsabiliza por isso, transformando as variáveis conforme a necessidade.

Desenvolvendo

COBOL

Para desenvolvermos programas COBOL que possam se comunicar com C, devemos conhecer bem os verbos COBOL que iremos usar. Para tanto, eles serão explicados a medida que nós precisarmos deles.

Primeiro Programa

Nosso primeiro programa será um simples programa, que irá passar uma mensagem para uma função. O que precisaremos saber aqui é usar o verbo CALL. Vejamos o programa desenvolvido:
```
       identification division.
       program-id. tp1.
       author. Hudson Reis.
 
       data division.
       working-storage section.
       77 ws77-test-var          pic x(014) value "Hello C World!".

       procedure division.
           call "tp1_c" using ws77-test-var.
           stop run.
```

Veja que o verbo CALL irá chamar a função C chamada "tp1_c". Eis o código da mesma:

```
int tp1_c(char test_var[15]) {
        printf("%s\n", test_var);
}
```

Observação 1: Note que na função tp1_c(C), eu declarei a variável test_var com 15 bytes enquanto no programa tp1(COBOL), eu declarei a variável ws77-test-var com 14 bytes. Isto é necessário pois toda variável C tem um zero binário no seu final, você passando este zero binário na variável COBOL ou não. Resumindo: 14 bytes para os dados e 1 byte para o zero binário.
Observação 2:Usei o tipo de dado int para mostrar o protótipo desta função, mas não se preocupe com os protótipos das funções agora(esqueça para ler estes itens iniciais, mas lembre-se na hora de desenvolver suas rotinas), pois teremos exemplos específicos para cada tipo de dado do C.

Para compilar este programa, devemos gerar objetos de ambos e depois linkeditá-los com o GCC.

hudson@mynotebook:~$ htcobol -c tp1.cob
hudson@mynotebook:~$ gcc -o cfunc.o -c cfunc.c
hudson@mynotebook:~$ gcc -o tp1 tp1.o cfunc.o -lhtcobol

Para executar, basta digitar ./tp1 ou tp1 em sua linha de comando.

hudson@mynotebook:~$ ./tp1
Hello C World!
hudson@mynotebook:~$ 

Segundo Programa

Após visualizarmos como passar parâmetros de um programa COBOL para um programa C, veremos agora como passar diversos parâmetros de um programa COBOL para um programa C. Passaremos os três valores derivados da frase "Hello C World!" e ele mostrará esta frase de maneiras diferentes de posicionamento. Eis o programa abaixo: 

       identification division.
       program-id. tp2.
       author. Hudson Reis.
 
       data division.
       working-storage section.
       77 ws77-test-var-1        pic x(014) value "Hello C World!".
       77 ws77-test-var-2        pic x(014) value "World C Hello!".
       77 ws77-test-var-3        pic x(014) value "C Hello World!".

       procedure division.
           call "tp2_c" using ws77-test-var-1
                              ws77-test-var-2
                              ws77-test-var-3
           end-call
           stop run.

Este programa faz a mesma coisa do primeiro programa, com a exceção de que ele passa ao invés de um valor, três valores. Outra diferença é que ele está chamando a função "tp2_c". Eis o código da mesma:

int tp2_c(char test_var_1[15], char test_var_2[15], char test_var_3[15]) {
        printf("First variable.: %s\n", test_var_1);
        printf("Second variable: %s\n", test_var_2);
        printf("Third variable.: %s\n", test_var_3);
}

Faça a compilação do mesmo e execute o programa tp2, como abaixo:

Compilação:

hudson@mynotebook:~$ htcobol -c tp2.cob
hudson@mynotebook:~$ gcc -o cfunc.o -c cfunc.c
hudson@mynotebook:~$ gcc -o tp2 tp2.o cfunc.o -lhtcobol

Execução:

hudson@mynotebook:~$ ./tp2
First variable.: Hello C World!World C Hello!C Hello World!
Second variable: World C Hello!C Hello World!
Third variable.: C Hello World!
hudson@mynotebook:~$ 

Note que na primeira variável está sendo mostrado os três valores, na segunda variável os dois valores e na terceira variável um único valor. Note também que os valores das variáveis estão sendo mostrados a partir da variável a ser mostrada. 

Abaixo, veja um quadro explicando como o C recebeu as três variáveis passadas pelo programa COBOL. Como o quadro ficou muito grande, role sua barra de rolagem horizontal para ver o final, que é a parte mais importante deste quadro:


0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 H e l l o 

C 

W o r l d ! W o r l d 

C 

H e l l o ! C 

H e l l o 

W o r l d ! \0 

Devemos tirar algumas considerações a partir deste quadro:
* Todo variável C começa do 0, não do 1, como é no COBOL. 
* Quando o C recebe uma(ou várias) variável(is) COBOL, ele insere um zero binário apenas no final da última das variáveis passadas, ou seja, ele insere um zero binário apenas no final da área de memória preenchida pelas três variáveis. 


Depois desta explicação, vamos nos informar de como as três variáveis passadas se encaixam dentro da string mostrada no quadro:


Variável C Valor Ponto inicial Ponto final test_var_1 Hello C World! 0 13 test_var_2 World C Hello! 14 28 test_var_3 C Hello World! 29 41 

Finalizando a explicação, então quando o printf() vai mostrar a primeira variável, ele vai mostrar os valores até encontrar o zero binário. Para a segunda e a terceira variável ele segue o mesmo raciocício. Então: 
* First variable.: posição 0 a 41 
* Second variable: posição 14 a 41 
* Third variable.: posição 29 a 41 


Você deve ter percebido que para que o C entenda corretamente cada variável do COBOL, é necessário passar ao final da variável um zero binário, para que o C saiba aonde é o final da string. Bom, o uso da palavra string foi proposital, porque somente em variáveis deste tipo é necessário inserir o zero binário. 

Para corrigir este problema no programa tp2.cob, basta usar no COBOL o verbo STRING, para concatenar o zero binário e o valor da variável. Devemos lembrar que como vamos inserir o zero binário dentro da variável, devemos aumentar na picture das variáveis ws77-test-var-1, ws77-test-var-2 e ws77-test-var-3, um(1) byte.
Vejamos agora um programa usando o verbo STRING e concatenando as strings a serem passadas para o programa C:

       identification division.
       program-id. tp2a.
       author. Hudson Reis.
 
       data division.
       working-storage section.
       77 ws77-test-var-1        pic x(015) value spaces.
       77 ws77-test-var-2        pic x(015) value spaces.
       77 ws77-test-var-3        pic x(015) value spaces.

       procedure division.
           string "Hello C World!" x"00" into ws77-test-var-1
           string "World C Hello!" x"00" into ws77-test-var-2
           string "C Hello World!" x"00" into ws77-test-var-3
           call "tp2a_c" using ws77-test-var-1
                               ws77-test-var-2
                               ws77-test-var-3
           end-call
           stop run.

No programa acima, para concatenar os campos não é "indispensável" o uso do STRING, porém ele é o que dá mais flexibilidade ao programador COBOL. Se você quiser usar uma variável grupo, com uma variável para os dados e a outra variável como um FILLER valendo um zero binário, tudo bem, não há problema. Se você quiser usar o verbo MOVE com modificação de referência, não há problema também. Se você quiser usar o STRING com variáveis ao invés de literais, não tem problema algum também.
Quanto ao zero binário, ele pode ser representado por x"00" ou low-values. Também, você pode armazená-lo dentro de uma variável.

Neste programa, a função que vamos chamar é a "tp2a_c". A única diferença entre a "tp2a_c" e a "tp2_c" é que a segunda possui um byte a mais em suas variáveis. Veja o código da função:

int tp2a_c(char test_var_1[16], char test_var_2[16], char test_var_3[16]) {
        printf("First variable.: %s\n", test_var_1);
        printf("Second variable: %s\n", test_var_2);
        printf("Third variable.: %s\n", test_var_3);
}

Faça a compilação do mesmo e execute o programa tp2a, como abaixo:

Compilação:

hudson@mynotebook:~$ htcobol -c tp2a.cob
hudson@mynotebook:~$ gcc -o cfunc.o -c cfunc.c
hudson@mynotebook:~$ gcc -o tp2a tp2a.o cfunc.o -lhtcobol

Execução:

hudson@mynotebook:~$ ./tp2a
First variable.: Hello C World!
Second variable: World C Hello!
Third variable.: C Hello World!
hudson@mynotebook:~$ 

Note que agora o C recebeu os dados do COBOL corretamente e os mostrou corretamente.

Terceiro Programa

       identification division.
       program-id. tp3.
       author. Hudson Reis.
 
       data division.
       working-storage section.
       77 ws77-test-var          pic x(020) value "Hello C World!".
       77 ws77-test-var-s        pic x(030) value spaces.

       procedure division.
           string ws77-test-var x"00" into ws77-test-var-s
           call "tp3_c" using ws77-test-var-s
           stop run.

Antes de mais nada, devemos notar que no programa acima está sendo passado o zero binário ao final da variável, então o C conseguirá separar as variáveis corretamente.
Entretanto, vamos prestar atenção no quadro abaixo, explicando como a variável ws77-test-var-s estará sendo passada para a função "tp3_c":


1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 H e l l o 

C 

W o r l d ! 






\0 










Como o C sempre irá procurar pelo zero binário, se quisermos inspecionar esta variável, ela terá 20 bytes, e não 14, como foi preenchida. Isso acontece pois a função strlen() irá procurar do início da variável até zero binário, sem discriminar nada. Para resolver isto, teremos que re-alinhá-la para que ela fique desta maneira:


1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 H e l l o 

C 

W o r l d ! \0 
















Como comentado acima, a função "tp3_c" irá alinhar a variável passada pelo programa COBOL. Eis o código desta função:

int tp3_c(char *test_var) {
        int i=0;
        printf("Received string....: %s\n", test_var);
        printf("String size........: %d\n", (strlen(test_var)));
        if (*test_var) {
                for(i=(strlen(test_var));i>=0;i--) {
                        if(test_var[i] != ' ' && test_var[i] != '\0') {
                                break;
                        }
                }
                test_var[i+1]='\0';
        }
        printf("Aligned string.....: %s\n", test_var);
        printf("Aligned string size: %d\n", (strlen(test_var)));
}

Neste caso, o importante nesta rotina é descobrir aonde é o final da string e não aonde é o final da variável. Se você for procurar pelo zero binário, provavelmente você o encontrará, entretanto saberá aonde é o final da variável mas não saberá aonde é o final da string. Além disso, se você tiver alguma string que tenha espaços entre os nomes, você não saberá alinhá-la corretamente. O ideal para resolver isto é verificando a string de trás para frente, subententendendo-se que o primeiro caracter encontrado a não ser um zero binário ou um espaço, é o final da string.

Faça a compilação do mesmo e execute o programa tp3, como abaixo:

Compilação:

hudson@mynotebook:~$ htcobol -c tp3.cob
hudson@mynotebook:~$ gcc -o cfunc.o -c cfunc.c
hudson@mynotebook:~$ gcc -o tp3 tp3.o cfunc.o -lhtcobol

Execução:

hudson@mynotebook:~$ ./tp3
Received string....: Hello C World!      ^M
String size........: 20
Aligned string.....: Hello C World!^M
Aligned string size: 14
hudson@mynotebook:~$ 

Note que agora o C recebeu os dados do COBOL corretamente, alinhou-os e os mostrou corretamente.
Quando usei o aplicativo script para gerar o log de execução do programa tp3, notei que ele jogou o caracter "^M" exatamente aonde deveria estar o zero binário, então para comprovar mais uma vez o alinhamento da variável, decidi não remover o "^M" do log, para que você veja melhor o que aconteceu.

Quarto Programa

       identification division.
       program-id. tp4.
       author. Hudson Reis.
 
       data division.
       working-storage section.
       77 ws77-test-var          pic x(020) value "Hello C World!".
       77 ws77-test-var-s        pic x(030) value spaces.

       procedure division.
           string ws77-test-var x"00" into ws77-test-var-s
           display "Initial String: " ws77-test-var-s
           call "tp4_c" using ws77-test-var-s
           display "Final String..: " ws77-test-var-s
           stop run.

Antes de mais nada, nota-se que este programa está seguindo todas as regras para se chamar um programa não-COBOL.
Ele tem o zero binário no final da variável. A variável que será passada no CALL tem tamanho suficiente para suportar o tamanho da variável e o zero binário, enfim, tudo o que é preciso.
Nota-se também que ele está usando o verbo DISPLAY para mostrar duas vezes a mesma variável. Pela lógica, e pelo título que os dois verbos DISPLAY estão mostrando, provavelmente esta variável quando for retornada da função C terá seu valor modificado.
Neste caso chegamos a um impasse: Como fazer para trazer da função C uma variável alfanumérica e modificada pela função C? Estudando mais a frente, notaremos que o COBOL não oferece uma maneira natural de receber parâmetros alfanuméricos do C e chegaremos a conclusão que para resolvermos este problema, teremos que passar estes valores por referência, pois assim os valores estariam armazenados na memória(como ponteiros de memória), e desta forma o COBOL e o C podem "ver" estes valores. No C, devemos declarar uma função void(que não retorna parâmetro algum) e que trabalha com ponteiros, que é aonde o programa COBOL deve armazenar suas variáveis(em ponteiros de memória).
Devemos nos lembrar também que não haverá uma comunicação direta entre o COBOL e o C neste caso. Cada rotina funcionará sozinha, sem saber o que a outra fez. O programa COBOL jogará os valores na memória, a função em C modificará estes valores na memória e o programa COBOL recolherá estes valores na memória novamente. Costuma-se chamar esta técnica de "falsificação de valores".

No código da função "tp4_c", você vai notar que não me preocupo com alinhamento de variáveis. Não o faço, pois não irei usar a variável test_var para nada especial, só para modificar o seu valor.

int tp4_c(char *test_var) {
        char *temp;
        temp = "!World C Hello";
        while(*temp) {
                *test_var++ = *temp++;
        }
}

Note que para se modificar o valor da variável test_var, você deve criar uma variável temporária, nesta variável temporária, determinar o novo valor da variável e fazer um laço de repetição onde você transfere byte a byte os valores da variável temporária para a variável original.

Faça a compilação do mesmo e execute o programa tp4, como abaixo:

Compilação:

hudson@mynotebook:~$ htcobol -c tp4.cob
hudson@mynotebook:~$ gcc -o cfunc.o -c cfunc.c
hudson@mynotebook:~$ gcc -o tp4 tp4.o cfunc.o -lhtcobol

Execução:

hudson@mynotebook:~$ ./tp4
Initial String: Hello C World!      ^@         ^M
Final String..: !World C Hello      ^@         ^M
hudson@mynotebook:~$ 

Como aconteceu no Terceiro programa, o aplicativo script quando gerou o log, gravou também a coluna aonde está o zero binário(^M) da variável C. Como nós não alinhamos a variável, retornou alguma sujeira(^@) e o zero binário(^M) retornou, mas longe dos caracteres, dando um tamanho a string que ela não possui realmente.

Fim de desenvolvimento

Todas as regras para a conversação entre um programa COBOL e uma função C foram explicadas aqui. Agora iremos partir para a conversação entre os tipos de variáveis C e os tipos de passagens de parâmetros do COBOL, o que podemos considerar como a parte avançada deste tipo de interfaceamento.

Variáveis COBOL x Variáveis C

Por serem linguagens diferentes(e terem tido métodos de desenvolvimento e de análise diferentes, o C e o COBOL tem uma série de diferenças na declaração de variáveis e iremos citá-las agora:

* O C possui um caracter nulo no final de cada variável e o COBOL não. 
* Dependendo do modelador de dados, o C pode mostrar as variáveis em formato binário, hexadecimal e numérico enquanto que o COBOL mostra somente em formato numérico. No COBOL, pode-se usar a cláusula COMP para determinar a variável a característica de armazenar os valores em formato nativo de máquina 
* No C, podemos determinar ponteiros de memória, no COBOL não. Para determinarmos o tamanho de uma variável no C, devemos criar um vetor, enquanto que no COBOL, na própria declaração da variável você já faz isso(determinar o tamanho), entretanto no COBOL não é exigido ao programador ter que fazer o uso de vetores desta maneira. 
Para interfacearmos estas linguagens devemos saber bem cada tipo de variável, dentre ambas. Quando estava desenvolvendo esta documentação, criei um quadro básico, dentro do meu conhecimento, explicando como as variáveis do COBOL e do C se comunicavam. Então o David Essex quando estava me ajudando na passagem de parametros entre uma variável grupo COBOL e uma struct C, melhorou este quadro, veja:


Variavel COBOLTipo CTamanhopic 9(n) comp-5 n=1-2unsigned char1pic s9(n) comp-5 n=1-2signed char1pic 9(n) comp-5 n=3-4unsigned short2pic s9(n) comp-5 n=3-4signed short2pic 9(n) comp-5 n=5-9unsigned int4pic s9(n) comp-5 n=5-9signed int4pic 9(n) comp-5 n=10-18unsigned long long8pic s9(n) comp-5 n=10-18signed long long8pic s9(n) comp-1float4pic s9(n) comp-2double8pic x(n)char nngrupochar*soma dos valores elementares

Passando parâmetros entre o COBOL e o C

Para passarmos parâmetros entre o COBOL e o C, haverá duas maneiras, retornando um valor ou retornando vários valores:

Retornando um valor

Quando falamos de retornar um valor, queremos dizer que o COBOL irá chamar uma função C que irá retornar um valor para ela(função C) e depois esta função C irá devolver este valor para o programa COBOL. Funções C que retornam valores possuem tipos C equivalentes a char, short, int, long, float e double e estas retornam um valor através da função return(), sendo que o COBOL recebe estes valores através da cláusula RETURNING. Este valor de retorno geralmente é int(sinalizado ou não, dependendo do compilador COBOL). Como exemplo deste interfaceamento iremos mostrar um exemplo de um interfaceamento entre um programa COBOL e uma função C de tipo INT:

       identification division.
       program-id. int.
       author. Hudson Reis.

       data division.
       working-storage section.
       77 ws77-unsigned-int-var pic  9(005) comp value 1.
       77 ws77-return-code-sign pic s9(005) comp.

       procedure division.
          display "== Begin =="
          display " "
          display "Note that the return-code is unsigned int,"
          display "pic 9(005) comp, while ws77-return-code-sign"
          display "is signed int(or just int), pic s9(005) comp."
          display " "
           
          display "Test results:"
          display "COBOL message: ws77-unsigned-int-var before: "
                  ws77-unsigned-int-var
          display "COBOL message: return-code before..........: "
                  return-code
          display "COBOL message: function cFuncInt(int) called..."
          call "cFuncInt" using by value ws77-unsigned-int-var
                               returning return-code
          end-call
          display "COBOL message: return-code after...........: "
                  return-code
          display " "
          
          add 1                 to ws77-unsigned-int-var             
          move zeros            to ws77-return-code-sign
          display "COBOL message: ws77-unsigned-int-var before: "
                  ws77-unsigned-int-var
          display "COBOL message: ws77-return-code-sign before: " 
                  ws77-return-code-sign
          display "COBOL message: function cFuncInt(int) called..."
          call "cFuncInt" using by value ws77-unsigned-int-var
                               returning ws77-return-code-sign
          end-call
          display "COBOL message: ws77-return-code-sign after.: "
                  ws77-return-code-sign
          
          display " "
          display "== End =="
           
          stop run.

Note que no programa acima, para fazer o cobol interpretar uma variável numérica como int, ela deve ter o tamanho de 5 a 9 posições e ter o atributo COMP, que é a maneira de armazenar os dados da maneira nativa que o pc guarda. Note também que ele faz duas chamadas a função C "cFuncInt"(Note que o C é case sensitive) e esta retorna dois valores diferentes. Se o valor passado for 1 ela retorna 1, se o valor passado for 2 ela retorna -1. Vamos dar uma olhada na função cFuncInt.

int cFuncInt(unsigned int uint_var) {
        printf("C message....: Value received..............: %d\n", uint_var);
        if (uint_var == 1) {
                return 1;
        } else if (uint_var == 2) {
                return -1;
        }
}

Note que na função C acima ela recebe o valor como unsigned int, pois sempre no programa ele irá passar valores numéricos não sinalizados. Note também que dependendo do valor passado, ele irá fazer um return diferente. Como o Tipo C int, para interfaceamentos com tipos C char short, long, float e double é a mesma coisa. Deve-se observar que a única diferença entre eles é o tipo C mesmo, então:
* Para variáveis numéricas, estas devem ser passadas usando a cláusula by value no CALL. 
* Para uma função C tipo short deve-se passar uma variável do tipo pic s9(04) comp-5, 
* Para uma função C tipo long deve-se passar uma variável do tipo pic s9(10) comp-5, 
* Para uma função C tipo float deve-se passar uma variável do tipo pic s9(n) comp-1, 
* Para uma função C tipo double deve-se passar uma variável do tipo pic s9(n) comp-2. 
* Para variáveis alfanuméricas, estas devem ser passadas usando a cláusula by reference no CALL, 
* Para tipos C char, deve-se declarar variáveis do tipo x(n). 


Retornando vários valores

Quando falamos de retornar vários valores é apenas uma boa maneira de dizer que o programa COBOL está recebendo os valores, mas na verdade não está havendo nenhuma passagem de parâmetros. O que acontece é uma técnica de armazenar os dados em uma área de memória em que o programa COBOL e a função C "conhecem", de modo que a função C possa modificar estes valores para quando o programa COBOL voltar da função C, trazer os valores modificados. Esta técnica é chamada de "falsificação" de valores. Desta maneira podemos passar todas as variáveis como alfanuméricas ou então como variáveis grupo, pois a função C(que deve ser obrigatoriamente void, pois não retorna parametro algum e trabalha com ponteiros) deverá recebê-las como ponteiros de memória para que ele possa modificá-las e devolvê-las para o programa COBOL.

Passando uma variável numérica

Partindo do pressuposto que uma variável numérica pode ser considerada um tipo int, short, float, double ou long, todas estas variáveis devem ser passadas como ponteiros e não é necessário passá-las através da cláusula by value. Veja o código:

       identification division.
       program-id. voidint.
       author. Hudson Reis.

       data division.
       working-storage section.
       77 ws77-int-var             pic s9(005) comp value -5.

       procedure division.
           display "== Begin =="
           display " "
           display "Int test results:"
           display "COBOL message: ws77-int-var before: " ws77-int-var
           display "COBOL message: function cFuncVoidInt(void) called.."
           call "cFuncVoidInt" using ws77-int-var
           display "COBOL message: ws77-int-var after.: " ws77-int-var
           display " "
           display "== End == "
           . 

Veja que no programa acima ele está mostrando o valor antes da chamada, depois ele chama a função cFuncVoidInt(que é void), e por final ele mostra o valor depois da chamada. Veja o código da função cFuncVoidInt: 

void cFuncVoidInt(int *int_var) {
        printf("C message....: Value received....: %d\n", *int_var);
        *int_var = -10;
        printf("C message....: Value sended......: %d\n", *int_var);
}

Devemos notar que a variável int em uma função void deve ser declada como ponteiro. Depois, veja que o valor dela está sendo mudando, de -5 para -10, então o valor de ws77-int-var mostrado depois da função chamada é -10. 

Passando uma variável alfanumérica

Partindo do pressuposto que uma variável alfanumérica pode ser considerada um tipo char, este tipo de variável deve ser passada como ponteiro e não é necessário passá-las através da cláusula by reference. Veja o código:

       identification division.
       program-id. voidchar.
       author. Hudson Reis.

       data division.
       working-storage section.
       77 ws77-char-var            pic  x(030)      value "Char Value".
 
       procedure division.
           display " "
           display "== Begin =="
           display "Char test results:"
           string "Char Value" x"00" into ws77-char-var
           display "COBOL message: ws77-char-var before: " 
                   ws77-char-var
           display "COBOL message: function cFuncVoidChar(char) called."
           call "cFuncVoidChar" using ws77-char-var 
           display "COBOL message: ws77-char-var after.: " 
                   ws77-char-var
           display "== End == "
           display " "
           . 

Veja que no programa acima ele está mostrando o valor antes da chamada, depois ele chama a função cFuncVoidInt(que é void), e por final ele mostra o valor depois da chamada. Veja o código da função cFuncVoidInt: 

void cFuncVoidChar(char *char_var) {
        char *c=char_var;
        printf("C message...: Value received....: %s\n", c);
        c = "Value Char";
        printf("C message...: Value sended......: %s\n", c);
        if  (c != NULL) {
                while(*c) {
                        *char_var++ = *c++;
                }
        }
}

Devemos notar que a variável int em uma função void deve ser declada como ponteiro. Depois, veja que o valor dela está sendo mudando, de "Char Value" para "Value Char", então o valor de ws77-char-var mostrado depois da função chamada é "Char Value". 

Passando uma variável grupo

No C possuimos estruturas(structs), e estas permitem que seja passado a uma função C determinadas variáveis, que tem um determinado tamanho e estrutura. Um item a se observar é que no que irei explicar abaixo, consegui passar para a função C apenas valores definidos(nenhum ponteiro). Para evitar problemas de alinhamento de memória use a diretiva pragma pack(1). Procure mais informações a respeito dela em http://www.arnold.af.mil/hpc/hp/cpp/pragmas.htm". Vamos a um exemplo de código:

       identification division.
       program-id. voidstrc.
       author. Hudson Reis.

       environment division.
       configuration section.
       special-names.
           decimal-point is comma.
 
       data division.
       working-storage section.
       01 ws01-struct.
          02 ws02-struct-int-var    pic s9(005) comp value -5.
          02 ws02-struct-float-var  pic s9(005)v99 comp-1 value -1,23.
          02 ws02-struct-double-var pic s9(005)v99 comp-2 value -1,23.
          02 ws02-struct-longlong-var pic s9(018) comp value 18.
          02 ws02-struct-short-var  pic s9(003) comp value 4.
          02 ws02-struct-char-var.
             03 ws03-char-var       pic  x(030) value "Char Value".
             03 ws03-end-of-string  pic  x(001) value low-values.

       procedure division.
       begin.
           display "== Begin =="
           display "Struct test results:"
           display "COBOL message: ws02-struct-int-var before......: " 
                   ws02-struct-int-var
           display "COBOL message: ws02-struct-char-var before.....: " 
                   ws02-struct-char-var
           display "COBOL message: ws02-struct-float-var before....: " 
                   ws02-struct-float-var
           display "COBOL message: ws02-struct-double-var before...: " 
                   ws02-struct-double-var
           display "COBOL message: ws02-struct-longlong-var before.: "
                   ws02-struct-longlong-var
           display "COBOL message: ws77-struct-short-var before....: "
                   ws02-struct-short-var
           display 
                  "COBOL message: function cFuncVoidStruct(void) loaded"
           call "cFuncVoidStruct"   using ws01-struct
           display 
                "COBOL message: function cFuncVoidStruct(void) unloaded"
           display "COBOL message: ws02-struct-int-var after.......: " 
                   ws02-struct-int-var
           display "COBOL message: ws02-struct-char-var after......: " 
                   ws02-struct-char-var
           display "COBOL message: ws02-struct-float-var after.....: " 
                   ws02-struct-float-var
           display "COBOL message: ws02-struct-double-var before...: " 
                   ws02-struct-double-var
           display "COBOL message: ws02-struct-longlong-var after..: " 
                   ws02-struct-longlong-var
           display "COBOL message: ws02-struct-short-var after.....: " 
                   ws02-struct-short-var
           display "== End =="
           stop run
           . 

Olhando o exemplo abaixo, nota-se que O programa COBOL acima está passando todos os tipos de variáveis geralmente usadas no COBOL, através de uma variável grupo. Deve-se notar que a variável alfanumérica tem um caracter nulo também, o que mostra que isso deve ser sempre feito quando se for passar um caracter nulo. O programa COBOL acima está chamando a função "cFuncVoidStruct" e esta função receberá os dados em uma struct do C, que possui uma maneira semelhante ao COBOL para se criar estruturas de dados. Veja o código da "cFuncVoidStruct":

#pragma pack(1)
struct cStruct {
        int   int_var;
        float float_var;
        double double_var;
        long long  longlong_var;
        short short_var;
        char char_var[31];
}; 
#pragma pack()

void cFuncVoidStruct(char *g) {

 struct cStruct *group = (struct cStruct *)g;

 printf("C message....: Struct Int Value received.......: %d\n", group->int_var);
 group->int_var = -10;
 printf("C message....: Struct Int Value sended.........: %d\n", group->int_var);
 
 printf("C message....: Struct Char Value received......: %s\n", group->char_var);
 strcpy(group->char_var, "Value Char");
 printf("C message....: Struct Char Value sended........: %s\n", (group->char_var));
 
 printf("C message....: Struct Float Value received.....: %f\n", group->float_var); 
 group->float_var = -3.21;
 printf("C message....: Struct Float Value sended.......: %f\n", group->float_var);
 
 printf("C message....: Struct Double Value received....: %lf\n", group->double_var); 
 group->double_var = -3.21;
 printf("C message....: Struct Double Value Sended......: %lf\n", group->double_var);
 
 printf("C message....: Struct Long Long Value received.: %d\n", group->longlong_var);
 group->longlong_var = 81;
 printf("C message....: Struct Long Long Value Sended...: %d\n", group->longlong_var);
 
 printf("C message....: Struct Short Value received.....: %d\n", group->short_var);
 group->short_var = 40;
 printf("C message....: Struct Short Value Sended.......: %d\n", group->short_var);
}

É interessante notar que na variável alfanumérica, o valor "Value Char" é copiado para a variável char_var. E isso acaba simplificando bastante o código.

Fim
Nesta versão deste documento, aqui está o final. Porém em uma próxima versão dele terá explicações de como passar uma variável do tipo OCCURS para uma função C, correções das coisas que ficaram erradas(meu conhecimento não é tão poderoso assim AINDA), Melhoras no makefile e outros pequenos detalhes. Além disso, este texto será totalmente re-escrito com o objetivo de acabar com as redundâncias, colocando os assuntos em tópicos numerados e dando a você, leitor, uma melhor leitura. 


Verbos COBOL

CALL

CALL  PROGRAMA/"SYSTEM"
        [       { BY REFERENCE  }       VARIAVEL-ALFANUMERICA           ]
        [                                                               ]
        [ USING { BY CONTENT    }       VARIAVEL-ESTATICA               ]
        [                                                               ]
        [       { BY VALUE      }       VARIAVEL-NUMERICA(BINARIA)      ]

        [ RETURNING                     RETURN-CODE(NUMERICO)           ]

        [ ON EXCEPTION     ]
        [ NOT ON EXCEPTION ]
END-CALL        

Na linguagem COBOL, quando queremos chamar um objeto, executável, biblioteca dinâmica/estática, fazemos esta chamada usando os verbos CALL. Com o verbo CALL, o programa principal chama o programa receptor e depois que o programa receptor é finalizado ele volta para o programa principal.

Explicando o verbo CALL e suas cláusulas

CALL: Verbo.
ON EXCEPTION: Cláusula que dispara uma rotina/comando, se houver problema na chamada do programa. Facultativa.
NOT ON EXCEPTION: Cláusula que dispara uma rotina/comando, se não houver problema na chamada do programa. Facultativa.

Cláusulas específicas para passagem de parâmetros 

USING: Cláusula que informa que a chamada está passando parâmetros. Se for chamado um programa ou biblioteca, ela poderá retornar parâmetros, através das variáveis que foram passadas. Se for uma chamada "system"(Chamada de sistema), é permitido apenas passar a linha de comando, e não há retorno de parâmetros.

Cláusulas específicas para passagem de parâmetros - Cláusulas BY:

Indicam que tipo de dado deve ser passado. 
BY CONTENT: Indica que o dado a ser passado não sofrerá alteração alguma e da maneira que for passado, será retornado. 
BY REFERENCE: Indica que o dado a ser passado DEVE ser alfanumérico 
BY VALUE: Indica que o dado a ser passado DEVE ser numérico(binário) 

Obs: Se o tipo de dado a ser passado na hora de fazer o CALL não estiver de acordo com o que a cláusula adequada(Ex: by value, numérico. by reference, alfanumérico), este dado será convertido para o tipo de parâmetro que deve estar sendo passado. Então, se você passar em uma cláusula BY REFERENCE uma variável numérica, ela será passada como alfanumérica, se você passar em uma cláusula BY VALUE uma variável alfanumérica, ela será passada como numérica. Desta forma, poderá haver erros nas passagens de parâmetros se elas não forem bem especificadas(podendo haver até falhas de segmentação).

Cláusula para recebimento de parâmetros

Através da própria cláusula USING, pode-se receber parâmetros também, porém se você quiser determinar uma variável específica para retorno, voce pode indicar a cláusula RETURNING. A cláusula RETURNING retorna apenas valores numéricos(binários) e o tamanho desta variável é dependente do compilador. Existe uma constante, chamada RETURN-CODE, que pode ser usada como variável. Ela é uma constante(e não uma variável), pois não precisa ser declarada. A sua interpretação como variável é dependente do compilador

Tipos de chamada:

1. Chamada simples: Permite ao COBOL chamar um outro programa COBOL, passando e recebendo parâmetros.

CALL "cob_file_exist" USING nome-arquivo retorno
ON EXCEPTION
   DISPLAY "Não foi possível chamar o programa!"
NOT ON EXCEPTION
   DISPLAY "O programa foi chamado!"
END-CALL

ou

CALL "cob_file_exist USING nome-arquivo retorno
END-CALL

ou

CALL "cob_file_exist" USING nome-arquivo-retorno.

Só depende da escolha do programador COBOL.

2. Chamadas "system": Permite ao COBOL executar uma linha de comando no sistema operacional. Se você quiser abrir um prompt de dentro do seu programa COBOL, basta dar um call "system" passando como parâmetro o caminho do bash.

CALL "system" USING "/usr/bin/bash"
ON EXCEPTION
   DISPLAY "Não foi possível chamar o bash!"
NOT ON EXCEPTION
   DISPLAY "Chamou o bash!"
END-CALL

ou

CALL "system" USING "/usr/bin/bash"
