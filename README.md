# Compilador MiniLang

*Trabalho A2 da disciplina de Compiladores*

Este projeto é um compilador completo para uma linguagem de *script* personalizada (chamada **"MiniLang"**), construído em *Python*. Ele implementa todo o *pipeline* de compilação, desde a análise de texto até a execução.

O *front-end* (analisador léxico e sintático) foi gerado automaticamente usando o *ANTLR4*, e o *back-end* (análise semântica, intérprete e gerador de código) foi implementado manualmente em *Python*.

## Integrantes
* **Vitor Valente Rodrigues de Figueiredo - Matrícula: 1210201659**
 
## Funcionalidades

### Linguagem
* **Tipos de Dados:** `int`, `bool`.
* **Controle de Fluxo:** `if/else`, `while`, `do-while`.
* **Arrays:** Suporte para declaração (`int[] a`), alocação dinâmica (`new int[10]`) e acesso indexado (`a[i]`).
* **Constantes:** Suporte à palavra-chave `const` (variáveis imutáveis).
* **Comentários:** Suporte a comentários de linha (`// ...`) e de bloco (`/* ... */`) ignorados pelo parser.

### Compilador

O *pipeline* do compilador inclui as seguintes fases:

1.  **Análise Léxica e Sintática:** Gera a árvore de análise concreta (CST) utilizando ANTLR4.
2.  **Construção da AST:** Converte a Parse Tree do ANTLR em uma Árvore Sintática Abstrata (AST) limpa usando o padrão Visitor.
3.  **Análise Semântica:** Percorre a AST para verificar erros de tipo (ex: `int + bool`), declarações de variáveis, uso de constantes não inicializadas e constrói a Tabela de Símbolos.
4.  **Execução Dupla:** O código pode ser executado de duas formas:
    * Intérprete: A AST é executada diretamente.
    * Gerador de Código (Codegen): A AST é traduzida para código Python puro (utilizando lambdas para simular atribuições como expressão) e depois executada.
5.  **Tratamento de Erros :**
    * Erros Sintáticos: Captura erros de sintaxe (`ex: faltando ;`) e interrompe a compilação imediatamente com mensagens claras.
    * Erros Semânticos: Reporta erros lógicos antes da execução.
    * Erros de Runtime: Captura acesso inválido a arrays (IndexOutOfBounds) durante a execução.

O projeto está organizado em módulos independentes, separando claramente as definições da linguagem, o código-fonte do compilador e os casos de teste.

### 1. Definições e Ferramentas (Raiz)

| Arquivo / Pasta | Descrição |
| :--- | :--- |
| `grammar/MiniLang.g4` | **O Coração da Linguagem.** Arquivo ANTLR que define as regras léxicas (tokens) e sintáticas (gramática). |
| `app.py` | **Interface Gráfica.** Aplicação Web (Streamlit) para testar o compilador visualmente. |
| `tests/` | **Casos de Teste.** Contém scripts `.min` para validação |

### 2. Módulos do Compilador (`src/`)

O código-fonte (`src`) é o motor do compilador. Abaixo, a responsabilidade de cada módulo:

| Módulo | Arquivo | Responsabilidade |
| :--- | :--- | :--- |
| **Pacote Python.**| `__init__.py` | Arquivo que permite que a pasta `src` seja importada como um módulo. |
| **Driver (CLI)** | `main.py` | Ponto de entrada. Gerencia argumentos, leitura de arquivos e orquestra o pipeline. |
| **Modelagem** | `ast_nodes.py` | Define as classes Python (Dataclasses) que compõem a Árvore Sintática Abstrata (AST). |
| **Integração** | `visitor.py` | Implementa o padrão *Visitor* para percorrer a árvore do ANTLR e construir a nossa AST limpa. |
| **Análise** | `semantic.py` | Realiza a análise semântica: verificação de tipos, controle de escopo e regras de constantes. |
| **Execução** | `interp.py` | Interpreta e executa a AST diretamente (simulação de máquina). |
| **Transpilação** | `codegen.py` | Traduz a AST da MiniLang para código Python executável. |
| **Utilitários** | `pretty.py` | Gera a representação visual (ASCII) da árvore sintática. |
| **Erros** | `error_listener.py` | Intercepta erros do ANTLR para fornecer mensagens amigáveis e interromper a compilação. |
| **Gerados** | `generated/` | Contém o Lexer, Parser e Visitor base criados automaticamente pelo ANTLR4. |

---

## 🔤 Gramática (ANTLR4)

A linguagem foi definida formalmente no arquivo `grammar/MiniLang.g4`. A especificação abaixo utiliza a notação do ANTLR (EBNF), destacando o uso de **recursão à esquerda direta** para expressões matemáticas e **rótulos (#)** para a geração do padrão Visitor.

<details>
<summary><strong> Clique para expandir o código completo da Gramática</strong></summary>

```antlr
grammar MiniLang;

program : block EOF;

block   : '{' stmt* '}';

stmt
    : block                                         # StmtBlock
    | decl                                          # StmtDecl
    | 'print' '(' expr (',' expr)* ')' ';'          # StmtPrint
    | 'if' '(' expr ')' stmt ('else' stmt)?         # StmtIf
    | 'while' '(' expr ')' stmt                     # StmtWhile
    | 'do' stmt 'while' '(' expr ')' ';'            # StmtDoWhile
    | expr ';'                                      # StmtExpr
    ;

decl    : CONST? type (LBRACK RBRACK)? ID ('=' expr)? ';' ;

type    : 'int' | 'bool';

expr
    : expr op=('*'|'/') expr                        # MulDiv
    | expr op=('+'|'-') expr                        # AddSub
    | expr op=('<'|'<='|'>'|'>=') expr              # Relational
    | expr op=('=='|'!=') expr                      # Equality
    | expr op='&&' expr                             # LogicAnd
    | expr op='||' expr                             # LogicOr
    | <assoc=right> expr op='=' expr                # Assignment
    | '!' expr                                      # UnaryNot
    | '-' expr                                      # UnaryMinus
    | atom                                          # AtomExpr
    ;

atom
    : '(' expr ')'                                  # Paren
    | NUM                                           # IntLit
    | 'true'                                        # BoolLit
    | 'false'                                       # BoolLit
    | 'new' type '[' expr ']'                       # NewArray
    | ID '[' expr ']'                               # ArrayAccess
    | ID                                            # VarRef
    ;

CONST: 'const';
INT: 'int';   BOOL: 'bool';
PRINT: 'print';
IF: 'if';     ELSE: 'else';
WHILE: 'while'; DO: 'do';
TRUE: 'true'; FALSE: 'false';
NEW: 'new';

LBRACK: '[';  RBRACK: ']';

ID: [a-zA-Z_] [a-zA-Z0-9_]*;
NUM: [0-9]+;

WS: [ \t\r\n]+ -> skip;
LINE_COMMENT: '//' ~[\r\n]* -> channel(HIDDEN);
BLOCK_COMMENT: '/*' .*? '*/' -> channel(HIDDEN);
```
</details>

## Como Rodar o Projeto

### 1. Instalação e Dependências

Para que o compilador funcione, seu ambiente precisa de **Java** (para a ferramenta geradora do ANTLR) e **Python 3**.

**Passo 1: Instalar Java (JRE/JDK)**
Abra seu terminal e digite `java -version`.
* Se aparecer a versão (ex: `1.8`, `11`, `17`...), pule este passo.
* Se der erro, instale o Java em [java.com/download](https://www.java.com/download/).

**Passo 2: Instalar Bibliotecas Python.**
Instale o runtime do ANTLR, para isso abra o CMD (Prompt de comando) e digite o seguinte comando:

```bash
pip install antlr4-python3-runtime
```
**Passo 3: Instalar as ferramentas do ANTLR**
* Acesse o link: https://www.antlr.org/download.html.
* Clique em Complete ANTLR 4.13.2 Java binaries jar, e salve na pasta grammar.
* O link para baixar é esse:[ https://www.antlr.org/download.html#:~:text=Complete%20ANTLR%204.13.2%20Java%20binaries%20jar](https://www.antlr.org/download/antlr-4.13.2-complete.jar)
* Rodar o seguinte comando no CMD da pasta grammar do seu progama
```bash
java -jar antlr-4.13.2-complete.jar -Dlanguage=Python3 -visitor -no-listener -o ../src/generated MiniLang.g4
```
**Passo 4: Rodar o comando de teste**
* Para o nosso exemplo usaremos a pasta de teste chamada `ok_fibonacci.min`.
* Para realizar o teste vá para o CMD da pasta Raiz e rode o comando:
```bash
python -m src.main tests/ok_fibonacci.min
```
## Testando o ok_fibonacci.min

### 1. AST (Arvore Sintatica Abstrata)
A AST representa a estrutura hierárquica do código, ignorando detalhes de pontuação (;, {).

<details>
 
 <summary><strong> Clique para expandir a AST gerada</strong></summary>

  ```
=== AST ASCII ===
Program
└── [block] Block
    └── [stmts] Seq
        ├── [first] Seq
        │   ├── [first] Seq
        │   │   ├── [first] Seq
        │   │   │   ├── [first] Seq
        │   │   │   │   ├── [first] Seq
        │   │   │   │   │   ├── [first] Seq
        │   │   │   │   │   │   ├── [first] None
        │   │   │   │   │   │   └── [second] Decl
        │   │   │   │   │   │       ├── [type] Id(int)
        │   │   │   │   │   │       ├── [id] Id(max)
        │   │   │   │   │   │       └── [init] Num(10)
        │   │   │   │   │   └── [second] Decl
        │   │   │   │   │       ├── [type] Id(int)
        │   │   │   │   │       ├── [id] Id(n1)
        │   │   │   │   │       └── [init] Num(0)
        │   │   │   │   └── [second] Decl
        │   │   │   │       ├── [type] Id(int)
        │   │   │   │       ├── [id] Id(n2)
        │   │   │   │       └── [init] Num(1)
        │   │   │   └── [second] Decl
        │   │   │       ├── [type] Id(int)
        │   │   │       ├── [id] Id(next)
        │   │   │       └── [init] None
        │   │   └── [second] Decl
        │   │       ├── [type] Id(int)
        │   │       ├── [id] Id(count)
        │   │       └── [init] Num(0)
        │   └── [second] Print
        │       ├── [arg[0]] Id(n1)
        │       └── [arg[1]] Id(n2)
        └── [second] While
            ├── [cond] Rel
            │   ├── [op='<'] Id(count)
            │   └── [right] Id(max)
            └── [body] Block
                └── [stmts] Seq
                    ├── [first] Seq
                    │   ├── [first] Seq
                    │   │   ├── [first] Seq
                    │   │   │   ├── [first] Seq
                    │   │   │   │   ├── [first] None
                    │   │   │   │   └── [second] Eval
                    │   │   │   │       └── [expr] Assign
                    │   │   │   │           ├── [left] Id(next)
                    │   │   │   │           └── [right] Ari
                    │   │   │   │               ├── [op='+'] Id(n1)
                    │   │   │   │               └── [right] Id(n2)
                    │   │   │   └── [second] Print
                    │   │   │       └── [arg[0]] Id(next)
                    │   │   └── [second] Eval
                    │   │       └── [expr] Assign
                    │   │           ├── [left] Id(n1)
                    │   │           └── [right] Id(n2)
                    │   └── [second] Eval
                    │       └── [expr] Assign
                    │           ├── [left] Id(n2)
                    │           └── [right] Id(next)
                    └── [second] Eval
                        └── [expr] Assign
                            ├── [left] Id(count)
                            └── [right] Ari
                                ├── [op='+'] Id(count)
                                └── [right] Num(1)
```
            
</details>

           
### 2. Tabela de Símbolos
A análise semântica gera uma tabela para rastrear identificadores, tipos e escopos. No exemplo de Fibonacci, todas as variáveis residem no **escopo global (0)** e são do tipo inteiro.

<details>

 <summary><strong> Clique para expandir a Tabela de Simbolos gerada</strong></summary>
 
```
=== TABELA DE SÍMBOLOS ===
[escopo 0] count : int
[escopo 0] max   : int
[escopo 0] n1    : int
[escopo 0] n2    : int
[escopo 0] next  : int
```
</details>

### 3. Execução (Intérprete e Código Gerado)

O programa é executado tanto pelo intérprete

**Saída do Intérprete:**
<details> <summary style="color:#add8e6;font-weight:bold">Clique para ver a Saída do Intérprete do arquivo `ok_fibonacci.min`</summary>

```
=== EXECUÇÃO (INTÉRPRETE) ===
0 1
1
2
3
5
8
13
21
34
55
89
```
</details>

**Codigo gerado:**
<details> <summary style="color:#add8e6;font-weight:bold">Clique para ver o código Python gerado do arquivo `ok_fibonacci.min`</summary>

```python
=== PYTHON GERADO ===
def __ml_run(env=None):
    frames = [env if isinstance(env, dict) else {}]
    def push(): frames.append({})
    def pop(): frames.pop()
    def declare(name, val): frames[-1][name] = val
    def get(name):
        for d in reversed(frames):
            if name in d: return d[name]
        raise NameError(f'Variável não encontrada: {name}')
    def setvar(name, val):
        for d in reversed(frames):
            if name in d: d[name]=val; return val
        frames[-1][name]=val; return val
    push()
    declare('max', 0)
    setvar('max', 10)
    declare('n1', 0)
    setvar('n1', 0)
    declare('n2', 0)
    setvar('n2', 1)
    declare('next', 0)
    declare('count', 0)
    setvar('count', 0)
    print(get('n1'), get('n2'))
    while bool((get('count') < get('max'))):
        push()
        setvar('next', (get('n1') + get('n2')))  # eval
        print(get('next'))
        setvar('n1', get('n2'))  # eval
        setvar('n2', get('next'))  # eval
        setvar('count', (get('count') + 1))  # eval
        pop()
    pop()
    return frames[0]
```
</details>
