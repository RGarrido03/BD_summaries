# Índices e otimização
## Sintaxe
```sql
CREATE INDEX [name] ON [table_name]([attribute_name]);
CREATE INDEX [name] ON [table_name]([attribute1], [attribute2], ...);
CREATE INDEX [name] ON [table_name]([attribute1], [attribute2], ...) WHERE [attribute]=[value];
CREATE INDEX [name] ON [table_name]([attribute1], [attribute2], ...) INCLUDE ([attribute3]);

-- Clustered indexes
CREATE CLUSTERED INDEX [name] ON [table_name]([attribute_name]);
CREATE CLUSTERED INDEX [name] ON [table_name]([attribute1], [attribute2], ...);
CREATE UNIQUE CLUSTERED INDEX [name] ON [table_name]([attribute_name]);
CREATE UNIQUE CLUSTERED INDEX [name] ON [table_name]([attribute1], [attribute2], ...);

-- Índices com parâmetros não default
CREATE NONCLUSTERED INDEX [name] ON [table_name]([attribute_name]) WITH (FILLFACTOR = [int], PAD_INDEX = [ON/OFF]);

-- Eliminar um índice
DROP INDEX [name];

-- Desfragmentar índices
ALTER INDEX [name] ON [table_name] REORGANIZE
ALTER INDEX ALL ON Frag REBUILD WITH (FILLFACTOR=[int])
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
Apesar de um índice aumentar significativamente a velocidade de acesso a um valor de um atributo (ou a um `JOIN` com este), a verdade é que **os tempos das instruções `INSERT`, `UPDATE` e `DELETE` de tuplos tornam-se superiores**. Para além disso, o volume de dados tende a ser maior, à medida que se vão adicionando índices, já que o número de estruturas de dados torna-se maior.

Assim, **é necessário encontrar um equilíbrio** entre a eficiência da pesquisa e a alteração das relações, e perceber em quais atributos é vantajoso adicionar um índice.

### Organização física dos dados
A pesquisa de um tuplo obriga a que seja carregada na memória toda a página onde este está contido. O mesmo acontece aos índices (ver [definição](#contexto-e-definição)), pelo que as suas operações de acesso e modificação têm [custos temporais](#custos).

Deve-se tentar encontrar índices que obriguem a carregar poucas páginas.

### Chaves da relação
É uma boa prática indexar os atributos chave da relação, já que, sendo únicos, ou existe um tuplo ou não.

Contudo, caso um atributo não tenha muitos valores repetidos, torna-se um candidato a índice.

## Custos
### *Queries*
A existência de índices beneficia amplamente as *queries*. Sem estes, e como explicado na secção [Contexto e definição](#contexto-e-definição), o SGBD é obrigado a fazer uma pesquisa tuplo a tuplo, o que tem uma complexidade computacional $O(n)$. Assim, **a existência de um índice reduz o número de páginas visitadas** e, consequentemente, o tempo da *query* (quando esta contempla uma pesquisa pelo atributo com índice).

### Inserção ou modificação dos dados
O acesso ao disco envolve a leitura e a escrita de páginas. Por exemplo, ao adicionar um tuplo numa relação em que um dos atributos tem um índice associado, existem estas operações no disco:
- Leitura das páginas dos tuplos
- Escrita dessas páginas (com o novo tuplo)
- Leitura das páginas dos índices relativos ao atributo
- Escrita dessas páginas (com o novo índice)

Ou seja, rapidamente se percebe que, **à medida que o número de índices aumenta, o tempo de inserção aumenta** igualmente.

### Exemplo
Para cada filme existem 3 atores e para cada ator existem 3 filmes. A relação entre filme e ator ocupa 10 páginas.

| Ação | Sem índices | Índice do ator | Índice do filme | Ambos os índices |
| --- | --- | --- | --- | --- |
| *Query* 1 | 10 | **4**<sup>1</sup> | 10 | **4**<sup>1</sup> |
| *Query* 2 | 10 | 10 | **4**<sup>1</sup> | **4**<sup>1</sup> |
| Inserção<sup>2</sup> | **2** | 4 | 4 | 6 |
| **Custo total** | $2 + 8p_1 + 8p_2$ | $4 + 6p_2$ | $4 + 6p_1$ | $6 - 2p_1 - 2p_2$ |

**<sup>1</sup>** 4 acessos: 1 ao índice + 3 às páginas da relação.\
**<sup>2</sup>** Fórmula: $2 (n+1)$, com $n$ sendo o número de índices. Ver a [secção anterior](#inserção-ou-modificação-dos-dados) para mais detalhes.

Com base nestes dados, e consoante o tempo em que as *queries* 1 e 2 ocorrem ($p_1$ e $p_2$, respetivamente), deve-se fazer uma avaliação sobre qual o melhor cenário.

## Tipos de índices no SQL Server
Existem dois tipos de índices, **ambos implementados em B-trees**:
- ***Clustered index***:
    - Os vários nós contêm os dados da relação, numa estrutura 2-em-1 (índice + dados).
    - A tabela está ordenada pelo índice, ou seja, a B-tree está ordenada pelo *cluster key index*.
    - Só existe um por relação. 
- ***Non-clustered index***:
    - Os índices apontam para a tabela base, que pode ser *clustered* ou *heap* (ver secção [Tabelas *heap* vs *clustered*](#tabelas-heap-vs-clustered)). No caso de uma tabela *non-clustered*, o tuplo é identificado pelo seu *RowID*, no formato `[extent]:[página]:[row_offset]`.
    - Podem existir vários por relação.

## *B-Tree Page Split*
Os índices da *B-Tree* devem mater-se organizados pelo *key index*. Deste modo, quando se efetua uma alteração dos dados, como é que o SGBD lida com esta ação?

### Inserção
Quando se acrescenta um tuplo numa página cheia, o SGBD divide essa página em duas, num processo denominado ***page split***, através dos seguintes passos:

1. Cria uma nova página
2. Copia parte dos índices para esta
3. Reflete esta alteração nos nós superiores
4. Insere o novo índice

Este processo tem custos temporais significativos.

## Opções de especialização de índices
### *Unique*
Índice onde não existem valores duplicados.

Por defeito, **a *primary key* é um *unique clustered index***. No entanto, é possível criar um *non-clustered*.

### *Composite*
Índice composto por mais do que um atributo, onde a ordem destes atributos importa.

Este tipo de índice só é usado numa *query* caso o primeiro atributo do índice faça parte desta. Assim, pode ser vantajoso ter vários *composite indexes*, com os mesmos atributos, mas com ordem diferente.

### *Filtered*
Índice *non-clustered* que permite usar a cláusula `WHERE` na sintaxe `CREATE INDEX`.

Só são indexadas partes dos tuplos.

### Inclusão de atributos não chave
É possível incluir atributos que não são chave num índice *non-clustered*. As *queries* que utilizam este tipo de índice denominam-se *queries* cobertas.

## Tabelas *heap* vs *clustered*
As *heap tables* vê **os seus novos tuplos serem inseridos no final desta**, sem ordem. Assim, o índice (do tipo *non-clustered*) contém, em cada nó folha, um `RowID` da tabela.

As *clustered tables* inserem os novos tuplos na respetiva B-tree, **consoante a ordem da *clustered index key***. Durante a inserção, podem ocorrer [*page splits*](#b-tree-page-split), que dependem das características da *primary key* e do facto de os novos tuplos terem uma ordem natural ou não.

## Escolha do tipo de índices
Um **índice *clustered*** deve ser usado em:
- Chaves que não criam muitos [*page splits*](#b-tree-page-split).
- Chaves pequenas.

Os **índices *non-clustered*** são usados para melhorar os tempos de *queries*, sobretudo nas que possuem a cláusula `WHERE`. Devem ser utilizados em:
- Atributos que são chaves estrangeiras de outras relações, de modo a obter um maior desempenho em `JOIN`s.
- Atributos sobre os quais são efetuadas consultas ordenadas.

## Otimização de *B-trees*
Com o objetivo de minimizar o número de [*page splits*](#b-tree-page-split), surgem os seguintes parâmetros:
- ***Fill factor***: Define a percentagem de espaço livre em cada página.
- ***Pad index***: Define se o *fill factor* é aplicado nos nós folha.

Deve-se tentar **encontrar um equilíbrio entre o desperdício de espaço e a frequência da ocorrência de *page splits***. Por exemplo, caso as inserções sejam ordenadas, o *fill factor* pode ser próximo de 100%, ao passo que em inserções sem ordem o *fill factor* deve rondar os 65% e os 85%.

### Exemplo
```sql
-- Índice com 15% de espaço livre e intermediário
CREATE NONCLUSTERED INDEX IxOrderNumber ON dbo.[Order](OrderNumber) WITH (FILLFACTOR = 85, PAD_INDEX = ON);
```

## Desfragmentação de índices
Quando existem [*page splits*](#b-tree-page-split) ou remoções de tuplos, surgem espaços vazios nas páginas, o que acaba por ter algumas implicações não só no espaço ocupado desnecessariamente, como também na performance do SGBD.

**Verificar regularmente a fragmentação dos índices** torna-se, portanto, uma tarefa importante.

Caso o índice esteja bastante fragmentado, deve-se reconstruí-lo. Existem duas formas de o fazer, cada uma com as suas vantagens e desvantagens:

```sql
ALTER INDEX [name] ON [table_name] REORGANIZE
```
- Desfragmenta as folhas, de acordo com o *fill factor* definido (ver secção [Otimização de *B-trees*](#otimização-de-b-trees)).
- É efetuado através de um conjunto pequeno de transações.
- Não tem impacto nas instruções `INSERT`, `UPDATE` e `DELETE`.

```sql
ALTER INDEX ALL ON Frag REBUILD WITH (FILLFACTOR=[int])
```
- Reconstrói todos os índices completamente
    - É equivalente a fazer `DROP` e depois `CREATE`
- Permite alterar as características do índice previamente definidas (como, por exemplo, o *fill factor*).

## Ferramentas disponíveis no SQL Server
### *Execution plan*
Permite ver, para uma *query*, quais os passos efetuados "internamente", incluindo consulta de índices, páginas, etc. Mostra o tempo de execução de cada passo, bem como o total.

### SQL Server Profiler
Provavelmente das aplicações mais feias que já vi na minha vida (e olhem que já vi muita merda), confusa *as fuck*, e, na verdade, eu nem sei bem como é que aquilo funciona.

No entanto, parece que é útil (fonte: confia) capturar eventos e *queries* com isto, para poder analisar os resultados na ferramenta *Database Engine Tuning Advisor* e, assim, otimizar de forma consciente a base de dados.

O SQL Server Management Studio é uma merda, periodt.