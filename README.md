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
5.  **Tratamento de Erros (Bônus):**
    * Erros Sintáticos: Captura erros de sintaxe (`ex: faltando ;`) e interrompe a compilação imediatamente com mensagens claras.
    * Erros Semânticos: Reporta erros lógicos antes da execução.
    * Erros de Runtime: Captura acesso inválido a arrays (IndexOutOfBounds) durante a execução.

## Estrutura do Projeto
