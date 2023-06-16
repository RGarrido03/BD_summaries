# Normalização
Codd inventou três formas normais, i.e. "regras" e "testes" na criação e manutenção de bases de dados. Mais tarde, Boyce e Codd propuseram uma nova forma normal, chamada BCNF (*Boyce-Codd Normal Form*). Foram ainda propostas uma quarta e quinta formas normais, baseadas em dependências multivalor e de junção, respetivamente.

**O objetivo destas formas normais é reduzir a redundância**.

Cada forma normal extende a anterior. Por exemplo, a 3ª FN inclui, entre outras, as regras da 1ª e da 2ª FNs.

## Dependências funcionais
Um atributo depende de outro(s), ou seja, há uma dependência.

Por exemplo, num `JOIN` entre empregados e projetos, o nome d@ empregad@ depende do seu `ssn`, e a localização do projeto depende do seu `id`. Estas dependências escrevem-se da forma `ssn → name` e `id → location`.

Noutro exemplo, numa tabela N:M `WORKS_ON`, onde:

- as *primary keys* são o `ssn` d@ empregad@ e o `id` do projeto
- está presente o atributo `hours`

pode-se considerar que `hours` depende funcionalmente do `ssn` e do `id`. Esta dependência pode ser escrita da forma `{ssn, id} → hours`.

### Tipos de dependêcias funcionais
- **Dependência parcial**: atributo depende de parte dos atributos que compõem a chave da relação.
- **Dependência transitiva**: atributo que não faz parte da chave da relação depende de um atributo que também não faz parte da chave da relação.
- **Dependência total**: atributo depende de toda a chave da relação.

## 1ª forma normal
- **Não existem atributos multivalor**, e todos são atómicos (i.e., um atriuto não pode ser um tuplo).
- **Não existem *nested relations***, ou seja, não podem existir relações como valor de um atributo de um tuplo.

## 2ª forma normal
**Não existem dependências parciais**, ou seja, todos os atributos (que não sejam chaves candidatas) devem depender totalmente da(s) chave(s).

### Exemplo
| <ins>ssn</ins> | <ins>project_id</ins> | hours | ename | pname | plocation |
| -------------- | --------------------- | ----- | ----- | ----- | --------- |

passa a:
| <ins>ssn</ins> | <ins>project_id</ins> | hours |
| -------------- | --------------------- | ----- |

| <ins>ssn</ins> | ename |
| -------------- | ----- |

| <ins>project_id</ins> | pname | plocation |
| --------------------- | ----- | --------- |

## 3ª forma normal
**Não existem dependências transitivas**, ou seja, dependências entre atributos que não sejam chave.

### Exemplo
| ssn | <ins>ename</ins> | bdate | address | dnumber | dname | dmgr_ssn |
| --- | ---------------- | ----- | ------- | ------- | ----- | -------- |

passa a:
| ssn | <ins>ename</ins> | bdate | address | dnumber |
| --- | ---------------- | ----- | ------- | ------- |

| <ins>dnumber</ins> | dname | dmgr_ssn |
| ------------------ | ----- | -------- |

## BCNF (*Boyce-Codd Normal Form*)
Todos os atributos são **funcionalmente dependentes apenas da totalidade das chaves** da relação.

### Exemplo
| <ins>A</ins> | <ins>C</ins> | B | D |
| ------------ | ------------ | - | - |

onde:
- {A, C} → B
- {A, C} → D
- C -> B

passa a:
| <ins>A</ins> | <ins>C</ins> | D |
| ------------ | ------------ | - |

| <ins>C</ins> | B |
| ------------ | - |

onde:
- {A, C} → D
- C → B

## 4ª forma normal
**Não existem [dependências multivalor](#dependências-multivalor)**.

No entanto, esta forma normal é rara de existir.

## 5ª forma normal
**Não existem [dependências de junção](#dependências-de-junção)**.

Esta forma normal é ainda mais rara de reconhecer do que a 4ª forma normal. 

## Dependências multivalor
Garantir que, se existirem duas dependências `X → Y` e `X → Z` numa tabela `X | Y | Z`, então devem-se separar em duas relações.

| <ins>ename</ins> | <ins>pname</ins> | <ins>dname</ins> |
| ---------------- | ---------------- | ---------------- |

passa a:
| <ins>ename</ins> | <ins>pname</ins> |
| ---------------- | ---------------- |

| <ins>ename</ins> | <ins>dname</ins> |
| ---------------- | ---------------- |

## Dependências de junção

São pouco percetíveis, já que é difícil detetá-las.

| <ins>s_name</ins> | <ins>part_name</ins> | <ins>proj_name</ins> |
| ----------------- | -------------------- | -------------------- |

passa a:
| <ins>s_name</ins> | <ins>part_name</ins> |
| ----------------- | -------------------- |

| <ins>s_name</ins> | <ins>proj_name</ins> |
| ----------------- | -------------------- |

| <ins>part_name</ins> | <ins>proj_name</ins> |
| -------------------- | -------------------- |