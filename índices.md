# Índices e otimização
## Sintaxe
```sql
CREATE INDEX [name] ON [table_name]([attribute_name]);
CREATE INDEX [name] ON [table_name]([attribute1], [attribute2], ...);
CREATE CLUSTERED INDEX [name] ON [table_name]([attribute_name]);
CREATE CLUSTERED INDEX [name] ON [table_name]([attribute1], [attribute2], ...);
DROP INDEX [name];
```

## Contexto e definição
À medida que a base de dados aumenta, a procura de informações torna-se mais lenta, já que **o SGBD tem de fazer uma procura com complexidade computacional $O(n)$**. Para além disso, os dados não se encontram ordenados nas tabelas.

Assim, surgiu a necessidade de criar índices, i.e., **estruturas de dados independentes que armazenam, para cada tuplo, um certo valor ordenado de um atributo, e um ponteiro que apontam o dito tuplo**. O ponteiro pode ser denso ou não.

Os tuplos, bem como os índices, encontram-se armazenados em páginas, cada uma com um determinado armazenamento em *bytes*.

É possível criar múltiplos índices, com um ou mais atributos.

## Índices *single-level ordered*
Estruturas que indexam um atributo, com definição semelhante à da secção anterior. A complexidade computacional é $log_2(n)$.

Existem três tipos de índices *single-level*:
- ***Primary index***: indexa um atributo chave, que não se repete.
- ***Clustered index***: indexa um atributo que pode ter valores duplicados, onde os atributos estão agrupados.
- ***Secondary index***: indexa outros atributos, sejam eles chave candidata ou não chave.

Só pode existir um índice *primary* ou *clustered*.

## Índices *multi-level*
Índice com amadas adicionais de indexação que permitem alterar a complexidade computacional para $log_{f_o}(n)$. $f_o$ significa *fan out* e geralmente é $>2$. A cada etapa do algoritmo, reduz-se o espaço de procura num fator $f_o$.

Estes índices encontram-se implementados em *binary trees* balanceadas, conhecidas pelos SGBDs como *B-Trees*, onde a distância de cada nó folha à raiz é sempre a mesma.

## Critérios de seleção de índices
Apesar de um índice aumentar significativamente a velocidade de acesso a um valor de um atributo (ou a um `JOIN` com este), a verdade é que os tempos das instruções `INSERT`, `UPDATE` e `DELETE` de tuplos são superiores. Para além disso, o volume de dados tende a ser maior, à medida que se vão adicionando índices, já que o número de estruturas de dados torna-se maior.

Assim, é necessário encontrar um equilíbrio entre a eficiência da pesquisa e a alteração das relações, e perceber em quais atributos é vantajoso adicionar um índice.

### Organização física dos dados
A pesquisa de um tuplo obriga a que seja carregada na memória toda a página onde este está contido. O mesmo acontece aos índices (ver [definição](#contexto-e-definição)), pelo que as suas operações de acesso e modificação têm custos temporais.

Deve-se tentar encontrar índices que obriguem a carregar poucas páginas.

### Chaves da relação
É uma boa prática indexar os atributos chave da relação, já que, sendo únicos, ou existe um tuplo ou não.

Contudo, caso um atributo não tenha muitos valores repetidos, torna-se um candidato a índice.

## Custos
[continua]