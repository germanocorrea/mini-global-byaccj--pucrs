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
  <!-- WIP: nao sei :( -->
```bison

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


## C7
Desenhe a árvore da expressão de tipo construída para “double m[3][4]” e para “int c[2][3][4]”. Explique por que o laço de declaraVars() percorre as dimensões da última para a primeira. Quais são os valores de dimensoes(), base() e toString() para c, c.elem e c.elem.elem?
Referência: Tipo.java (array, dimensoes, base, toString); mini.y — declaraVars()
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
