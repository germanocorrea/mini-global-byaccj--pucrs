# Parte C
## C1
1. Caminho para gerar o parser:
  -  Primeiro, jflex procesa o arquivo mini.flex. Esse arquivo descreve o léxico da linguagem, e o jflex gera um scanner de tokens a partir desse léxico
  - Depois, byacc processa o arquivo mini.y. Esse arquivo representa a gramática da linguagem e suas ações semânticas, e o byacc produz um parser LALR(1) da linguagem a partir dela. O parser gerado utiliza o scanner gerado pelo jflex para ler os tokens. O byaccj usa a flag -J para indicar que o parser é em Java, e -Jnorun para indicar que o método main descrito em mini.y deve ser replicado nesse parser. Esse arquivo main é definido explicitamente no arquivo mini.y porque ele controla a execução inicial do parser - lẽ o arquivo de um código fonte em mini, processa flags de execução (verbosidade, trace, etc) e inputa o conteúdo deste fonte na classe Parser.java para ser parseado.
<!-- FIXME: acho que a parte a seguir não é ncessária no C1 -->
2. Caminho para executar o parser em dado código fonte:
  - Executa-se o Parser compilado, inputando um código fonte em mini. O parser inicializa o Yylex para fazer o scanner dos tokens do arquivo fonte e definir as configurações globais (verbosidade, trace, etc). O parser inicializa a pilha vazia e inicia um loop principal, obtendo os tokens do scanner e inicializando o ParserVal para o valor semântico do token atual.

## C2
O que declaram a seguir?
- %token <ival> NUM_INT
  Declara o token NUM_INT como valor numérico, atribuindo `ival` a ele (conecta esse token inteiro ao tipo ival do ParseVal) 
- %type <obj> expr dims_decl indices
  Semelhante, mas define o tipo das produções `tipo`, `expr`, `lista_ids`, `item_id`, `dims_decl` e `indices` como objetos no ParseVal, ao invés dos tipos "nativos" dele sval, ival ou dval.

- O que o léxico coloca em yylval ao reconhecer um ID e um NUM_INT?
  O léxico coloca um objeto ParserVal em ambos. No caso de ID, coloca um ParserVal cujo argumento é a string do texto do token atual. No caso de NUM_INT, o argumento é o resultado de Integer.parseInt dessa mesma string.

- Por que as ações escrevem (Tipo)$1 e (List<Tipo>)$2?
  (Tipo)$1 faz casting de $1 para Tipo, a classe que representa os tipos da linguagem.
  (List<Tipo>)$2 faz casting de $2 para uma lista de tipos. Cada item da lista pode, na realidade, ter um tipo diferente dentre os que a classe Tipo define (porque seu tipo é Tipo mesmo, e Tipo pode representar qualquer tipo que o define). Não diz que $2 é uma variável do tipo array ou lista, mas sim que $2 é para o parser uma lista (poderia ser uma lista de argumentos ou parametros por exemplo, onde cada tem seu tipo, ou lista de variáveis a serem declaradas).
  As ações semânticas que usam esses castings estão basicamente realizando esse casting para generalizar a ações semânticas que não dependem de tipos especificos. Exemplo: ao comparar dois valores com `==`, pouco importa saber o tipo específico a menos que seja para ter certeza que se tratam do mesmo tipo (esse é um exemplo de uso de (Tipo)$1, onde cada parametro de `igualdade` a ser testado é de Tipo). No caso da lista, (List<Tipo>)$2 (casting do segundo parametro da produção) é usado exclusivamente na produção de `indices`, pois esta pode ser uma lista de tamanho n de indices de acesso de array (como `[1]` ou `[2][3][1][0]`). Cada token `expr` é um elemento da lista List<Tipo>.

- Na produção dims_decl : dims_decl ‘[’ NUM_INT ‘]’, que valor tem $3 e de que campo de ParserVal ele vem?
  $3 possui o valor do token NUM_INT, vindo do campo ival do ParserVal

## C3

- Explique a estrutura de um programa imposta pelas produções programa, globais, decl_main e finais.
  A produção `programa` define a estrutura do programa em si, dizendo que  ela é composta de uma lista de declarações globais, da função main, e de diretivas finais. A lista de declaração de globais pode conter declaração de variáveis ou diretivas (sendo diretivas opções de debugging relacionadas ao trace do programa e a pilha de escopos). A função main é a computação do programa em si, onde as variáveis globais são utilizadas e onde o resto das estruturas do programa também são usadas. Por fim, a produção final apenas permite a declaração opcional das diretivas (as mesmas que podemos declarar antes da função main).

- Execute os dois programas abaixo e explique por que um gera erro sintático e o outro erro semântico.
```bash
λ ~/org/pucrs/CC/mini-global-byaccj/exercicios_germano/ main* java Parser exercicios/C3_a.mini
Picked up _JAVA_OPTIONS: -Dsun.java2d.uiScale=2
Linha 3: erro sintático (syntax error)
Linha 3: erro sintático (stack underflow. aborting...)
2 erro(s) encontrado(s).
```
Este gera erro sitnático pois há uma declaração de variável global depois da main, e como vimos, as declarações devem ser feitas antes. O erro de sintaxe existe porque depois de `decl_main`, não há produção que suporte `decl_var` mais.

```bash
λ ~/org/pucrs/CC/mini-global-byaccj/exercicios_germano/ main* java Parser exercicios/C3_b.mini
Picked up _JAVA_OPTIONS: -Dsun.java2d.uiScale=2
Linha 2: erro: nesta versão só é permitida a função 'main' (encontrado 'f')
Linha 3: erro: programa sem função 'main'
2 erro(s) encontrado(s).
```

O erro semântico aqui é porque, apesar de sintaticamente correto, estamos declarando uma função `f` quando o programa suporta apenas declaração da função `main`. O segundo erro semântico é justamente a falta de uma função `main`.

- Qual dos dois tipos de mensagem é mais útil para o programador? Como a gramática poderia ser alterada para que o programa (a) recebesse uma mensagem semântica clara?
  O segundo contém mais informações sobre o erro. O primeiro simplesmente não informa que existe uma estrutura não suportada depois da declaração da main.
  Uma mudança seria uma produção nova em finais que aceita declaração de variáveis, mas larga um erro semântico. Assim, o parsing aceita a declaração mas emite um erro melhor.
```yacc
finais : /* vazio */
       | finais diretiva
       | finais decl_var   { erro("declarações globais devem vir antes de 'main'"); }
       ;
```
## C4
- Explique a diferença entre busca(), buscaLocal() e resolve() e onde cada uma é usada em mini.y.
  buscaLocal busca o identificador na tabela de simbolos apenas dentro do escopo atual, ao contrário de busca que itera os escopos do mais interno ao mais externo para encontrar esse identificador (retorna o mais proximo). resolve realiza a busca e registra no trace, utilizada exclusivamente ao usar o identificador, diferente de busca e buscaLocal que são usados ao declarar valores.
- Nesta versão só existem variáveis globais: quais escopos são abertos durante a análise de ok.mini e quantos podem estar abertos ao mesmo tempo? Por que, ainda assim, a pilha de escopos foi mantida?
  Existem 6 escopos diferentes no programa ok.mini:
    1. escopo global, no nivel 0
    2. escopo da função main, no nivel 1
    3. escopo do primeiro while, nivel 2 (todos de nivel 2 pra cima estao dentro do escopo de main)
    4. escopo do segundo while, nivel 3 dentro do while anterior
    5. escopo do terceiro while, nivel 2
    6. escopo do if, nivel 2
  Na execução de ok.mini apenas 4 escopos ficam abertos ao mesmo tempo (global -> main -> while -> while), porém a linguagem permite N escopos. A pilha de escopos foi mantida para futuramente implementarmos declarações locais.

## C5
- Em que situação declara() emite erro e em que situação emite aviso de ocultação?
  Ao tentar redeclarar uma variável já declarada no escopo atual

- É possível obter um aviso de ocultação nesta versão da linguagem? Justifique com o código.
  Não, pois a versão atual não suporta declarações fora de escopo global. As evidências podem ser vistas tanto num programa exemplo (abaixo), quanto na própria gramática que não suporta listas de declaração dentro de escopos não globais, o que justifica o erro sintático e não semântico do exemplo abaixo:
```bash
λ ~/org/pucrs/CC/mini-global-byaccj/ main* cat exercicios_germano/exercicios/C5.mini
int val;

void main() {
    int val = 0;
}
λ ~/org/pucrs/CC/mini-global-byaccj/ main* java Parser exercicios_germano/exercicios/C5.mini
Picked up _JAVA_OPTIONS: -Dsun.java2d.uiScale=2
Linha 4: erro sintático (syntax error)
Linha 4: erro sintático (stack underflow. aborting...)
2 erro(s) encontrado(s).
λ ~/org/pucrs/CC/mini-global-byaccj/ main*
```

## C6
Para a declaração “int a, t[2][5], b;”, mostre o valor do atributo de dims_decl e de lista_ids após cada redução e diga em que momento os três símbolos são inseridos na tabela. Esses atributos são herdados ou sintetizados?

- dims_decl: 
- lista_ids:


## C7
Desenhe a árvore da expressão de tipo construída para “double m[3][4]” e para “int c[2][3][4]”. Explique por que o laço de declaraVars() percorre as dimensões da última para a primeira. Quais são os valores de dimensoes(), base() e toString() para c, c.elem e c.elem.elem?
Referência: Tipo.java (array, dimensoes, base, toString); mini.y — declaraVars()

array(3)                    array(2)
   |                            |
array(4)                    array(3)
   |                            |
double                     array(4)
                                |                               
                               int
O laço percorre da última pra primeira pois a função é feita de dentro pra fora, o array tem o elemento que precisa aparecer antes dele preservando a ordem correta das dimensões
Para c os valores de dimensoes(), base() e toString() são respectivamente 3, int e "int[2][3][4]"
Para c.elem os valores de dimensoes(), base() e toString() são respectivamente 2, int e "int[3][4]"
Para c.elem.elem os valores de dimensoes(), base() e toString() são respectivamente 1, int e "int[4]"

## C8
Acompanhe indexa() para m[i][j], m[1] e m[1][2][0] (com double m[3][4]) e para v[d] (com int v[5] e double d), indicando o tipo devolvido ou a mensagem. Em seguida, explique as mensagens das linhas 14 a 19 de exemplos/erros.mini. Por que a indexação parcial m[1] é aceita em uma expressão, mas não como alvo de atribuição?
Referência: mini.y — indexa(), atribui(); erros.mini linhas 14–19

## C9
Calcule o resultado de igual() para os pares (int[3][4], int[5][4]), (int[3][4], int[3][5]) e (int[3], int[3][4]). Que regra de equivalência isso representa? Em que métodos igual() é usado nesta versão e por que, na prática, ele quase nunca é chamado com arranjos?
Referência: Tipo.igual(), Tipo.atribuivelA()

## C10
Para a condição “m[2][3] > 10.5 && !achou” de ok.mini, liste em ordem os métodos de verificação chamados durante o parsing e o tipo devolvido por cada um, até checaCond().
Referência: mini.y — regras de expr; ok.mini

## C11
No trace de tabela.mini, a linha 11 (“m[1][n] = v[n] * 1.5;”) produz resolve n, resolve n, resolve v e só então resolve m. Explique por que m, que aparece primeiro no texto, é resolvido por último.
Referência: make tabela; exemplos/tabela.mini linha 11

## C12
Por que laco é um contador e não um boolean? Como checaReturn() sabe que “return 1;” é inválido em main? Que parte desse código só fará diferença quando a linguagem tiver outras funções?
Referência: mini.y — cmd (WHILE, BREAK, RETURN), checaReturn(), funcAtual

## C13
A produção bloco abre um escopo novo, mas nesta versão nada pode ser declarado dentro dele. Isso é um erro de projeto? Compare bloco com corpo e diga o que acontece na tabela, segundo o trace, ao entrar e sair de um bloco.
Referência: mini.y — bloco, corpo

## C14
Identifique no código os métodos, campos e valores de enumeração que existem mas não são usados (ou nunca recebem dados) nesta versão. Para cada um, explique que papel teria em uma versão com funções, parâmetros e variáveis locais.
Referência: mini.y — seção “RESERVADO PARA FUTURAS IMPLEMENTAÇÕES”; Simbolo.java

Classe.VAR_LOCAL : o campo desse operador nunca é executado no declaraVars, pois nessa versão enxuta, só temos declaracoes globais. Em uma versão completa esse operador funciona para marcar variáveis declaradas dentro de funcõess e blocos específicos, elas são registradas na memória de maneiras diferentes, as globais ficam em enderecos estáticos e as locais em pilhas.
PARAM: só é referênciado dentro de declaraParam, que foi guardado para implementacões futuras. Ou seja, nunca é chamado. Em uma versão completa, esse PARAM funciona para classificar parametros formais, ele permite com que o código reconheca se o parâmetro foi redeclarado como local.
List<Tipo> params: mais uma vez esse é só usado por declaraParam, fazendo params.add(). Portanto, se declaraParam nao é usada a lista fica sempre vazia. Para uma versão completa essa lista guardaria a assinatura das funcões.
DeclaraParam: nada na gramática chama ele. Ele tem o papel de completar a assinatura das funcões e coloca o parametro como símbulo no escopo da funcão.
Chama: é uma funcão que também não temos nada existente como producão em expr para esse metodo. Ele verifica as chamadas analizando se é funcão, os argumentos etc, funcionando para evitar erros.

# Parte D

## D1
- Execute make tabela e explique cada uma das três impressões produzidas por $MOSTRA_TS: por que a primeira não contém fim nem main, por que a segunda mostra três escopos (dois vazios) e o que a terceira representa. Relacione com as linhas “abre escopo” e “fecha escopo” do trace.
  1. A primeira não contém fim nem main pois ambos foram declarados apenas depois do primeiro MOSTRA_TS. Apenas n, v e m foram declaradas antes.
  2. Sobre os escopos da segunda MOSTRA_TS: o nivel 2 é o escopo mais interno, de um bloco iniciado na linha 10. Ele é vazio pois não há nenhuma declaração de variável nele. O nivel 1 é da função main, ela também não possui nenhuma declaração de variável e por isso é vazia. O último é o escopo global, nivel 0, onde todas declarações foram feitas. Nenhum escopo foi fechado no segundo MOSTRA_TS, apenas aberto: há print de 2, o de nível 1 e de nível 2. O nível 0 não exibe abertura de escopo, pois ele nunca é "aberto" de fato, ele simplesmente já existe.
  3. A terceira MOSTRA_TS representa o estado final da tabela de simbolos quando o programa esta terminando de executar. Só possui o escopo global pois é chamada fora da main. A tabela de simbolos do escopo global é igual a tabela de simbolos do escopo da segunda MOSTRA_TS porque não há mudança, não se remove declarações globais. Vale lembrar que essa versão da mini só suporta declarações globais.

## D2
- Antes de executar, preveja o tipo de cada expressão ou a mensagem de erro do programa abaixo. Depois execute e compare. Explique qualquer diferença entre a previsão e o resultado.
  ```mini
int c[2][3][4];
int x;
double d;
void main() {
  x = c[1][2][3];
  d = c[1][2][3] / 2;
  x = c[1][2];
  x = c[1][2][3][0];
  c[0][0][0] = d;
  x = c[x][x][x] + c[0][0];
}
```
  Previsao:
```mini
int c[2][3][4]; // matriz de 3 dimensoes de inteiros
int x; // valor inteiro
double d; // valor double
void main() { // funcao main com retorno void
  x = c[1][2][3]; // obtem o inteiro salvo na matriz c no indice [1][2][3] e salva na variavel inteira x
  d = c[1][2][3] / 2; // divide o mesmo inteiro acima por 2, o resultado é um double e é salvo em uma variável double
  x = c[1][2]; // aqui daria erro, pois c[1][2] é um array de inteiros, não um inteiro, e esta tentando-se salvar esse valor em uma variável inteira
  x = c[1][2][3][0]; // aqui da erro também, porque a matriz só tem 3 dimensoes e aqui tenta-se acessar 4
  c[0][0][0] = d; // aqui da erro pois tenta-se atribuir um double d em uma posicao de inteiro na matriz c
  x = c[x][x][x] + c[0][0]; // as últimas tentativas de atribuir x falharam, então provavelmente não deve chegar aqui - se chega aqui, x sendo inteiro é válido para acessar indices, mas em runtime pode ser um inteiro maior que o suportado para c e falhar com isso. indice [0][0] para c nao existe, o que também da erro. A soma entao nem existiria por conta dos erros pois estariamos somando dois valores invalidos que nao sao numericos necessariamente.
}
```

Resultado de rodar:
```bash
λ ~/org/pucrs/CC/mini-global-byaccj/ main* java Parser -v -t  exercicios_germano/exercicios/D2.mini
Picked up _JAVA_OPTIONS: -Dsun.java2d.uiScale=2
[ts] linha   1  abre escopo 'global' (nível 0)
[ts] linha   1  insere c : VAR_GLOBAL int[2][3][4] no escopo 'global'
[ts] linha   2  insere x : VAR_GLOBAL int no escopo 'global'
[ts] linha   3  insere d : VAR_GLOBAL double no escopo 'global'
[ts] linha   4  insere main : FUNCAO (retorno void) no escopo 'global'
[ts] linha   4    abre escopo 'função main' (nível 1)
[ts] linha   5    resolve c → c@1  (VAR_GLOBAL int[2][3][4], nível 0)
[ts] linha   5    resolve x → x@2  (VAR_GLOBAL int, nível 0)
[ts] linha   6    resolve c → c@1  (VAR_GLOBAL int[2][3][4], nível 0)
[ts] linha   6    resolve d → d@3  (VAR_GLOBAL double, nível 0)
[ts] linha   7    resolve c → c@1  (VAR_GLOBAL int[2][3][4], nível 0)
[ts] linha   7    resolve x → x@2  (VAR_GLOBAL int, nível 0)
Linha 7: erro: tipos incompatíveis na atribuição a 'x': esperado int, encontrado int[4]
[ts] linha   8    resolve c → c@1  (VAR_GLOBAL int[2][3][4], nível 0)
Linha 8: erro: 'c' tem 3 dimensão(ões), mas foi indexado com 4 índice(s)
[ts] linha   8    resolve x → x@2  (VAR_GLOBAL int, nível 0)
[ts] linha   9    resolve d → d@3  (VAR_GLOBAL double, nível 0)
[ts] linha   9    resolve c → c@1  (VAR_GLOBAL int[2][3][4], nível 0)
Linha 9: erro: tipos incompatíveis na atribuição a 'c': esperado int, encontrado double
[ts] linha  10    resolve x → x@2  (VAR_GLOBAL int, nível 0)
[ts] linha  10    resolve x → x@2  (VAR_GLOBAL int, nível 0)
[ts] linha  10    resolve x → x@2  (VAR_GLOBAL int, nível 0)
[ts] linha  10    resolve c → c@1  (VAR_GLOBAL int[2][3][4], nível 0)
[ts] linha  10    resolve c → c@1  (VAR_GLOBAL int[2][3][4], nível 0)
Linha 10: erro: operador '+' não se aplica a int e int[4]
[ts] linha  10    resolve x → x@2  (VAR_GLOBAL int, nível 0)
  ┌─ fechando escopo 'função main' (nível 1)
  └─
[ts] linha  11    fecha escopo 'função main' (nível 1) — 0 símbolo(s) descartado(s)
  ┌─ fechando escopo 'global' (nível 0)
  │ c        VAR_GLOBAL int[2][3][4]           nível 0  (linha 1)
  │ x        VAR_GLOBAL int                    nível 0  (linha 2)
  │ d        VAR_GLOBAL double                 nível 0  (linha 3)
  │ main     FUNCAO     [] -> void             nível 0  (linha 4)
  └─
[ts] linha  12  fecha escopo 'global' (nível 0) — 4 símbolo(s) descartado(s)
4 erro(s) encontrado(s).
```

A previsão foi certeira, mas acabo se aprofundando muito na linha 10, ao passo que o erro era simplesmente que o operador + não podia se aplicar a int e int[4], acabamos analisando erros que seriam de runtime também.

## D3
Permita a declaração de variáveis locais no início de main e dentro de blocos, alterando apenas a gramática. Mostre a alteração, execute make conflitos e teste com o programa abaixo, explicando cada aviso e a tabela mostrada pelo $MOSTRA_TS.
Que estruturas mantidas “para futuras implementações” tornaram essa mudança tão pequena?
int a, m[2][2];void main() {int a;a = 1;{ double a; int k; a = 2.5; k = m[1][0]; $MOSTRA_TS }int m;}

Alteração apenas em itens, passando a valer em todas estruturas solicitadas:
```mini
itens      : /* vazio */
           | itens cmd
           | itens diretiva
           | itens decl_var
           ;
```

```bash
λ ~/org/pucrs/CC/mini-global-byaccj/ main* make conflitos
byaccj -v -J -Jnorun mini.y
Nenhum conflito na gramática.
λ ~/org/pucrs/CC/mini-global-byaccj/ main*
λ ~/org/pucrs/CC/mini-global-byaccj/ main* java Parser -v -t  exercicios_germano/exercicios/D3.mini
Picked up _JAVA_OPTIONS: -Dsun.java2d.uiScale=2
[ts] linha   1  abre escopo 'global' (nível 0)
[ts] linha   2  insere a : VAR_GLOBAL int no escopo 'global'
[ts] linha   2  insere m : VAR_GLOBAL int[2][2] no escopo 'global'
[ts] linha   3  insere main : FUNCAO (retorno void) no escopo 'global'
[ts] linha   3    abre escopo 'função main' (nível 1)
Linha 4: aviso: 'a' oculta a declaração da linha 2
[ts] linha   4    insere a : VAR_LOCAL int no escopo 'função main'
[ts] linha   5    resolve a → a@4  (VAR_LOCAL int, nível 1)
[ts] linha   6      abre escopo 'bloco' (nível 2)
Linha 7: aviso: 'a' oculta a declaração da linha 4
[ts] linha   7      insere a : VAR_LOCAL double no escopo 'bloco'
[ts] linha   8      insere k : VAR_LOCAL int no escopo 'bloco'
[ts] linha   9      resolve a → a@7  (VAR_LOCAL double, nível 2)
[ts] linha  10      resolve m → m@2  (VAR_GLOBAL int[2][2], nível 0)
[ts] linha  10      resolve k → k@8  (VAR_LOCAL int, nível 2)
╔══ Tabela de símbolos na linha 11 — 3 escopo(s) aberto(s)
║ [nível 2] bloco   ← topo
║     a        VAR_LOCAL  double                 nível 2  (linha 7)
║     k        VAR_LOCAL  int                    nível 2  (linha 8)
║ [nível 1] função main
║     a        VAR_LOCAL  int                    nível 1  (linha 4)
║ [nível 0] global
║     a        VAR_GLOBAL int                    nível 0  (linha 2)
║     m        VAR_GLOBAL int[2][2]              nível 0  (linha 2)
║     main     FUNCAO     [] -> void             nível 0  (linha 3)
╚══
  ┌─ fechando escopo 'bloco' (nível 2)
  │ a        VAR_LOCAL  double                 nível 2  (linha 7)
  │ k        VAR_LOCAL  int                    nível 2  (linha 8)
  └─
[ts] linha  12      fecha escopo 'bloco' (nível 2) — 2 símbolo(s) descartado(s)
Linha 13: aviso: 'm' oculta a declaração da linha 2
[ts] linha  13    insere m : VAR_LOCAL int no escopo 'função main'
  ┌─ fechando escopo 'função main' (nível 1)
  │ a        VAR_LOCAL  int                    nível 1  (linha 4)
  │ m        VAR_LOCAL  int                    nível 1  (linha 13)
  └─
[ts] linha  14    fecha escopo 'função main' (nível 1) — 2 símbolo(s) descartado(s)
  ┌─ fechando escopo 'global' (nível 0)
  │ a        VAR_GLOBAL int                    nível 0  (linha 2)
  │ m        VAR_GLOBAL int[2][2]              nível 0  (linha 2)
  │ main     FUNCAO     [] -> void             nível 0  (linha 3)
  └─
[ts] linha  15  fecha escopo 'global' (nível 0) — 3 símbolo(s) descartado(s)
Programa semanticamente correto.
λ ~/org/pucrs/CC/mini-global-byaccj/ main*```

Aviso da linha 4: a declaração da variável 'a' oculta uma variável 'a' declarada no escopo global.
Aviso da linha 7: o mesmo ocorre, a variável 'a' declarada nesse bloco oculta a variável declarada no escopo main.
Aviso da linha 13: a declaração de uma variável 'm' oculta uma variável 'm' declarada já no escopo global. Esse escopo é da função main, pois já se fechou o escopo do bloco que havia sido aberto.
A tabela de simbolos printada mostra as variáveis declaradas em cada escopo. No bloco, o nivel mais interno, há uma variável 'a' e outra 'k'. 'a' oculta 2 variáveis declaradas em escopos "mais altos". A função main declara apenas uma variável 'a' naquele momento, que oculta o escopo de uma 'a' declarada globalmente. Outras declarações da main não aparecem porque ainda não ocorreram. No escopo global, há a declaração de outras entidades: a, m e a função main.

As estruturas mantidas para futuras implementações são o controle de escopos da tabela de simbolos, principalmente. Assim, só alterar a gramática era necessário.



## D4
Desafio (opcional): faça o verificador detectar índices constantes fora dos limites, como v[7] com int v[5] ou m[1][4] com double m[3][4]. Hoje o atributo de expr é apenas um Tipo. Que informação precisaria ser acrescentada? Descreva (e, se possível, implemente) a solução e diga por que ela não resolve v[i] com i variável.
Referência: mini.y — expr, indexa(); slide “Responsabilidades da tabela de símbolos” (dobramento de constantes)
