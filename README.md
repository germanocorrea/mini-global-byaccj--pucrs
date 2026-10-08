# Verificador semântico da Mini simplificada (JFlex + BYACC/J)

Versão simplificada do exemplo da Unidade 4.3–4.4 (Construção de Compiladores, PUCRS):

- apenas **variáveis globais** — escalares (`int`, `double`, `bool`) e **arranjos de uma ou
  mais dimensões** (`int v[10]; double m[3][4];`);
- uma única função, **`void main()`**, sem parâmetros e sem variáveis locais;
- comandos: atribuição (inclusive `m[i][j] = ...`), `if/else`, `while`, `break`, `return`,
  `print` e blocos `{ }`.

As estruturas da versão completa foram **mantidas para futuras implementações**:
pilha de escopos em `TabSimb`, classes `VAR_LOCAL`, `PARAM` e `FUNCAO` em `Simbolo`,
`funcAtual`, `abreFuncao`/`fechaFuncao`, abertura de escopo em blocos, e os métodos
`declaraParam()` e `chama()` (na seção "RESERVADO PARA FUTURAS IMPLEMENTAÇÕES" de `mini.y`).

## Arquivos
| Arquivo | Conteúdo |
|---|---|
| `mini.flex` | analisador léxico (JFlex, modo `%byaccj`) |
| `mini.y` | gramática + ações semânticas + `main` (BYACC/J) |
| `Tipo.java` | tipos, arranjos multidimensionais (`array(3, array(4, int))`), relação ≤, tipo ERRO (⊥) |
| `Simbolo.java` | entrada da tabela de símbolos |
| `TabSimb.java` | tabela de símbolos: pilha de escopos (`LinkedHashMap` por escopo), trace e impressão |
| `DeclId.java` | record auxiliar: nome + lista de dimensões de cada item declarado |
| `exemplos/ok.mini` | programa correto (matriz preenchida em laços aninhados) |
| `exemplos/erros.mini` | programa com 15 erros semânticos |
| `exemplos/tabela.mini` | demonstra `$MOSTRA_TS` e `$TRACE_ON`/`$TRACE_OFF` |
| `Makefile` | geração, execução, teste e limpeza |

## Como gerar e executar
```sh
make              # jflex + byaccj -J -Jnorun + javac
make ok           # exemplos/ok.mini com -v
make erros        # exemplos/erros.mini
make tabela       # exemplos/tabela.mini
make trace        # exemplos/ok.mini com -t
make test         # confere os três exemplos
make conflitos    # gera y.output e mostra conflitos
make clean
```
Ferramentas com outro nome: `make JFLEX="java -jar jflex-full-1.9.1.jar" BYACCJ=./yacc.linux`.
Testado com JFlex 1.7, BYACC/J 1.15 e Java 21; `byaccj -v` não reporta conflitos.

## Visualizando a tabela de símbolos e os escopos
| Recurso | Onde | Efeito |
|---|---|---|
| `-v` | linha de comando | imprime cada escopo no momento em que é fechado |
| `-t` | linha de comando | trace: abre/fecha escopo, insere, resolve (`m → m@5`) |
| `$MOSTRA_TS` | no fonte (entre as declarações, nos comandos ou após o main) | imprime a pilha de escopos abertos |
| `$TRACE_ON` / `$TRACE_OFF` | no fonte | liga/desliga o trace em um trecho |

## Arranjos multidimensionais
- `double m[3][4]` tem tipo `array(3, array(4, double))`, impresso como `double[3][4]`.
- Cada índice "descasca" uma dimensão: `m[i]` tem tipo `double[4]`; `m[i][j]` tem tipo `double`.
- Todos os índices devem ser `int`; mais índices do que dimensões é erro.
- Indexação parcial é permitida em expressões, mas um arranjo não pode ser alvo de atribuição
  nem operando aritmético.

## O que é verificado
- redeclaração de variável global; dimensões positivas na declaração
- uso de identificador não declarado; uso de `main` como variável
- tipos em expressões aritméticas, relacionais, lógicas e de igualdade (coerção int → double)
- indexação: variável deve ser arranjo, índices `int`, número de índices ≤ dimensões
- atribuição (T2 ≤ T1); alvo não pode ser arranjo
- condição de `if`/`while` do tipo bool; `break` só dentro de laço
- `return` sem valor (main é `void`); só a função `main` é aceita
- recuperação: o tipo ERRO (⊥) evita mensagens em cascata
