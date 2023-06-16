# Views
## Sintaxe
```sql
CREATE VIEW [name] AS [query];
ALTER VIEW [name] AS [new_query];
DROP VIEW [name];
```
Utilizar `SELECT`, `INSERT`, `UPDATE` e `DELETE` como numa tabela normal.

## Exemplos
```sql
CREATE VIEW EMPLOYEE_PROJECTS AS
    SELECT Fname, Lname, Pname, Hours
    FROM EMPLOYEE
        JOIN WORKS_ON ON Ssn=Essn
        JOIN PROJECT ON Pno=Pnumber;
```
```sql
CREATE VIEW DEPT_INFO(Dept_name, No_of_emps, Total_sal) AS
    SELECT Dname, Count(*), Sum(Salary)
    FROM EMPLOYEE JOIN DEPARTMENT ON Dno=Dnumber
    GROUP BY Dname;
```

## Modo de atuação
Dependendo do SGBD, existem duas formas:
- *Query modification*
- *View materialization*: criação de uma tabela temporária com os resultados da View, onde são efetuadas as *queries*.

### *Query modification*
A *query* é modificada de forma a assemelhar-se a uma *query* sobre a(s) tabela(s) da View.

Por exemplo, o SGBD processa a *query*
```sql
SELECT Fname, Lname
FROM EMPLOYEE_PROJECTS
WHERE Pname=‘GalaxyS’;
```
como sendo
```sql
SELECT Fname, Lname
FROM EMPLOYEE, PROJECT, WORKS_ON
WHERE Ssn=Essn AND Pno=Pnumber AND Pname=‘GalaxyS’;
```

## Check option
Ao incluir `WITH CHECK OPTION` na criação da view, os argumentos da condição `WHERE` são verificados.

## Update de dados
Só é updatable se incluir todas as chaves primárias e todos os atributos `NOT NULL`, e se não tiver várias tabelas (i.e., um ou mais `JOIN`s).