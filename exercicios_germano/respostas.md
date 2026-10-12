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
