---
title: "Gerenciamento de estatísticas em tabelas no SQL Data Warehouse | Microsoft Docs"
description: "Introdução às estatísticas em tabelas no Azure SQL Data Warehouse."
services: sql-data-warehouse
documentationcenter: NA
author: barbkess
manager: jenniehubbard
editor: 
ms.assetid: faa1034d-314c-4f9d-af81-f5a9aedf33e4
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.custom: tables
ms.date: 11/06/2017
ms.author: barbkess
ms.openlocfilehash: b007e1894f163d50dbf31e3c09b4b5ff329adb59
ms.sourcegitcommit: 5ac112c0950d406251551d5fd66806dc22a63b01
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2018
---
# <a name="managing-statistics-on-tables-in-sql-data-warehouse"></a>Gerenciamento de estatísticas em tabelas no SQL Data Warehouse
> [!div class="op_single_selector"]
> * [Visão geral][Overview]
> * [Tipos de Dados][Data Types]
> * [Distribuir][Distribute]
> * [Índice][Index]
> * [Partição][Partition]
> * [Estatísticas][Statistics]
> * [Temporário][Temporary]
> 
> 

Quanto mais o SQL Data Warehouse do Azure souber sobre seus dados, mais rápido ele poderá executar as consultas. Coletar estatísticas em seus dados e, em seguida, carregá-los no SQL Data Warehouse é uma das coisas mais importantes que você pode fazer para otimizar as consultas. Isso porque o otimizador de consulta do SQL Data Warehouse é um otimizador baseado em custo. Ele compara o custo de vários planos de consulta e, em seguida, escolhe o plano com o menor custo que, na maioria dos casos, é o plano que executa mais rápido. Por exemplo, se o otimizador estimar que a data usada para filtrar sua consulta retornará uma linha, ele poderá escolher um plano diferente do que se estimar que a data selecionada retornará 1 milhão de linhas.

Atualmente, o processo de criação e atualização de estatísticas é um processo manual, mas é simples de fazer.  Em breve você será capaz de criar e atualizar as estatísticas em colunas únicas e índices automaticamente.  Ao usar as seguintes informações, você poderá automatizar bastante o gerenciamento das estatísticas em seus dados. 

## <a name="getting-started-with-statistics"></a>Introdução às estatísticas
Criar estatísticas de exemplo em cada coluna é uma maneira fácil de começar. Estatísticas desatualizadas resultam em um desempenho abaixo do ideal. No entanto, atualizar estatísticas em todas as colunas à medida que seus dados crescem pode consumir memória. 

As seguintes são recomendações para diferentes cenários:
| **Cenário** | Recomendações |
|:--- |:--- |
| **Introdução** | Atualizar todas as colunas após a migração para SQL Data Warehouse |
| **Coluna mais importante para estatísticas** | Chave de distribuição de hash |
| **Segunda coluna mais importante para estatísticas** | Chave de partição |
| **Outras colunas importantes para estatísticas** | Data, JOINs frequentes, GROUP BY, HAVING e WHERE |
| **Frequência de atualizações de estatísticas**  | Conservadora: diária <br></br> Depois de carregar ou transformar os dados |
| **Amostragem** |  Menos de 1 bilhão de linhas, usar a amostragem padrão (20%) <br></br> Com mais de 1 bilhão de linhas, as estatísticas em uma faixa de 2% são boas |

## <a name="updating-statistics"></a>Atualização de estatísticas

Uma prática recomendada é atualizar as estatísticas em colunas de data por dia à medida que novas datas são adicionadas. Sempre que há um carregamento de novas linhas no data warehouse, novas datas de carga ou datas de transação são adicionadas. Isso altera a distribuição de dados e torna as estatísticas desatualizadas. Por outro lado, as estatísticas de uma coluna de país em uma tabela de clientes talvez nunca precisem ser atualizadas, porque a distribuição de valores geralmente não se altera. Supondo que a distribuição seja constante entre os clientes, adicionar novas linhas à variação de tabela não alterará a distribuição dos dados. No entanto, se seu data warehouse apenas contiver um país e você trouxer dados de um novo país, resultando em dados de vários países sendo armazenados, então, será necessário atualizar estatísticas na coluna do país.

Uma das primeiras perguntas a serem feitas quando você estiver solucionando problemas em uma consulta é, **"As estatísticas estão atualizadas?"**

Essa questão não pode ser respondida pela idade dos dados. Um objeto de estatísticas atualizado pode ser antigo se não houver nenhuma alteração importante nos dados subjacentes. Quando o número de linhas mudar substancialmente, ou houver uma alteração material na distribuição de valores para uma coluna, *então*, significa que é hora de atualizar as estatísticas.

Como não há nenhuma exibição de gerenciamento dinâmico para determinar se os dados da tabela foram alterados, já que as estatísticas da última vez foram atualizadas, saber a idade das suas estatísticas poderá fornecer uma parte da imagem.  Você pode usar a seguinte consulta para determinar a última vez que suas estatísticas foram atualizadas em cada tabela.  

> [!NOTE]
> Lembre-se de que, se houver uma alteração importante na distribuição de valores para uma coluna, você deverá atualizar as estatísticas independentemente da última vez que foram atualizadas.  
> 
> 

```sql
SELECT
    sm.[name] AS [schema_name],
    tb.[name] AS [table_name],
    co.[name] AS [stats_column_name],
    st.[name] AS [stats_name],
    STATS_DATE(st.[object_id],st.[stats_id]) AS [stats_last_updated_date]
FROM
    sys.objects ob
    JOIN sys.stats st
        ON  ob.[object_id] = st.[object_id]
    JOIN sys.stats_columns sc    
        ON  st.[stats_id] = sc.[stats_id]
        AND st.[object_id] = sc.[object_id]
    JOIN sys.columns co    
        ON  sc.[column_id] = co.[column_id]
        AND sc.[object_id] = co.[object_id]
    JOIN sys.types  ty    
        ON  co.[user_type_id] = ty.[user_type_id]
    JOIN sys.tables tb    
        ON  co.[object_id] = tb.[object_id]
    JOIN sys.schemas sm    
        ON  tb.[schema_id] = sm.[schema_id]
WHERE
    st.[user_created] = 1;
```

As **colunas de data** em um data warehouse, por exemplo, normalmente precisam de atualizações frequentes de estatísticas. Sempre que há um carregamento de novas linhas no data warehouse, novas datas de carga ou datas de transação são adicionadas. Isso altera a distribuição de dados e torna as estatísticas desatualizadas.  Por outro lado, as estatísticas de uma coluna de gênero em uma tabela de clientes talvez nunca precisem ser atualizadas. Supondo que a distribuição seja constante entre os clientes, adicionar novas linhas à variação de tabela não alterará a distribuição dos dados. No entanto, se o seu data warehouse contiver apenas um gênero e um novo requisito resultar em gêneros múltiplos, então, será necessário atualizar estatísticas sobre a coluna de gênero.

Para obter mais explicações, veja [Estatísticas][Statistics] no MSDN.

## <a name="implementing-statistics-management"></a>Implementação do gerenciamento de estatísticas
Geralmente, convém estender os processos de carregamento de dados a fim de garantir que as estatísticas estejam atualizadas ao final do carregamento. É no carregamento de dados que as tabelas frequentemente mudam de tamanho e/ou distribuição de valores. Portanto, esse é um momento lógico para implementar alguns processos de gerenciamento.

Os seguintes princípios orientadores são fornecidos para atualizar suas estatísticas durante o processo de carregamento:

* Certifique-se de que cada tabela carregada tenha pelo menos um objeto de estatísticas atualizado. Isso atualiza as informações do tamanho da tabela (contagem de linhas e contagem de páginas) como parte da atualização de estatísticas.
* Concentre-se em colunas que participam de cláusulas JOIN, GROUP BY, ORDER BY e DISTINCT.
* Considere uma atualização mais frequentes das colunas de "chave crescente", por exemplo, datas de transação, porque esses valores não serão incluídos no histograma de estatísticas.
* Considere atualizar as colunas de distribuição estática com menos frequência.
* Lembre-se, cada objeto estatístico é atualizado em sequência. Simplesmente implementar `UPDATE STATISTICS <TABLE_NAME>` nem sempre é ideal, especialmente para tabelas amplas com muitos objetos de estatística.

Para obter mais explicações, veja [Estimativa de cardinalidade][Cardinality Estimation] no MSDN.

## <a name="examples-create-statistics"></a>Exemplos: criar estatísticas
Estes exemplos mostram como usar várias opções para a criação de estatísticas. As opções usadas para cada coluna dependem das características dos dados e de como a coluna será usada em consultas.

### <a name="create-single-column-statistics-with-default-options"></a>Criar estatísticas de coluna única com opções padrão
Para criar estatísticas em uma coluna, basta fornecer um nome para o objeto de estatísticas e o nome da coluna.

Esta sintaxe usa todas as opções padrão. Por padrão, o SQL Data Warehouse utiliza uma **amostragem de 20%** da tabela ao criar estatísticas.

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]);
```

Por exemplo: 

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1);
```

### <a name="create-single-column-statistics-by-examining-every-row"></a>Criar estatísticas de coluna única examinando cada linha
A taxa de amostragem padrão de 20 por cento é suficiente para a maioria das situações. No entanto, você pode ajustar essa taxa de amostragem.

Para usar toda a tabela como amostragem, use a seguinte sintaxe:

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]) WITH FULLSCAN;
```

Por exemplo: 

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH FULLSCAN;
```

### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Criar estatísticas de coluna única, especificando o tamanho da amostra
Como alternativa, você pode especificar o tamanho da amostra como uma porcentagem:

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH SAMPLE = 50 PERCENT;
```

### <a name="create-single-column-statistics-on-only-some-of-the-rows"></a>Criar estatísticas de coluna única em apenas algumas das linhas
Também é possível criar estatísticas em uma parte das linhas na tabela. Isso é chamado de estatística filtrada.

Por exemplo, é possível usar estatísticas filtradas quando você planeja consultar uma partição específica de uma tabela particionada grande. Ao criar estatísticas apenas nos valores de partição, a precisão das estatísticas melhora e, portanto, o desempenho da consulta também.

Este exemplo cria estatísticas em um intervalo de valores. Os valores podem ser facilmente definidos para corresponder ao intervalo de valores em uma partição.

```sql
CREATE STATISTICS stats_col1 ON table1(col1) WHERE col1 > '2000101' AND col1 < '20001231';
```

> [!NOTE]
> Para que o otimizador de consulta considere usar estatísticas filtradas ao escolher o plano de consulta distribuída, a consulta deve ser adequada à definição do objeto de estatísticas. Usando o exemplo anterior, a cláusula WHERE da consulta precisa especificar valores col1 entre 2000101 e 20001231.
> 
> 

### <a name="create-single-column-statistics-with-all-the-options"></a>Criar estatísticas de coluna única com todas as opções
Também é possível combinar as opções juntas. O exemplo a seguir cria um objeto estatístico filtrado com um tamanho de amostra personalizado:

```sql
CREATE STATISTICS stats_col1 ON table1 (col1) WHERE col1 > '2000101' AND col1 < '20001231' WITH SAMPLE = 50 PERCENT;
```

Para obter a referência completa, veja [CREATE STATISTICS][CREATE STATISTICS] no MSDN.

### <a name="create-multi-column-statistics"></a>Criar estatísticas de várias colunas
Para criar um objeto estatístico de várias colunas, use os exemplos anteriores, mas especifique mais colunas.

> [!NOTE]
> O histograma, que é usado para estimar o número de linhas no resultado da consulta, está disponível apenas para a primeira coluna listada na definição do objeto estatístico.
> 
> 

Neste exemplo, o histograma está em *product\_category*. As estatísticas entre colunas são calculadas em *product\_category* e *product\_sub_category*:

```sql
CREATE STATISTICS stats_2cols ON table1 (product_category, product_sub_category) WHERE product_category > '2000101' AND product_category < '20001231' WITH SAMPLE = 50 PERCENT;
```

Como há uma correlação entre a *categoria do\_produto* e a *sub\_categoria do\_produto*, um objeto de estatística de colunas múltiplas poderá ser útil se essas colunas forem acessadas ao mesmo tempo.

### <a name="create-statistics-on-all-columns-in-a-table"></a>Criar estatísticas em todas as coluna em uma tabela
É uma maneira de criar estatísticas é emitir comandos CREATE STATISTICS depois de criar a tabela:

```sql
CREATE TABLE dbo.table1
(
   col1 int
,  col2 int
,  col3 int
)
WITH
  (
    CLUSTERED COLUMNSTORE INDEX
  )
;

CREATE STATISTICS stats_col1 on dbo.table1 (col1);
CREATE STATISTICS stats_col2 on dbo.table2 (col2);
CREATE STATISTICS stats_col3 on dbo.table3 (col3);
```

### <a name="use-a-stored-procedure-to-create-statistics-on-all-columns-in-a-database"></a>Use um procedimento armazenado para criar estatísticas em todas as colunas em um banco de dados
O SQL Data Warehouse não tem um procedimento armazenado no sistema equivalente ao sp_create_stats no SQL Server. Esse procedimento armazenado cria um objeto de estatísticas de coluna única em todas as colunas do banco de dados que ainda não tenham estatísticas.

O exemplo a seguir ajudará você a começar o projeto do banco de dados. Fique à vontade para adaptá-lo às suas necessidades:

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_create_stats]
(   @create_type    tinyint -- 1 default 2 Fullscan 3 Sample
,   @sample_pct     tinyint
)
AS

IF @create_type NOT IN (1,2,3)
BEGIN
    THROW 151000,'Invalid value for @stats_type parameter. Valid range 1 (default), 2 (fullscan) or 3 (sample).',1;
END;

IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN;
    DROP TABLE #stats_ddl;
END;

CREATE TABLE #stats_ddl
WITH    (   DISTRIBUTION    = HASH([seq_nmbr])
        ,   LOCATION        = USER_DB
        )
AS
WITH T
AS
(
SELECT      t.[name]                        AS [table_name]
,           s.[name]                        AS [table_schema_name]
,           c.[name]                        AS [column_name]
,           c.[column_id]                   AS [column_id]
,           t.[object_id]                   AS [object_id]
,           ROW_NUMBER()
            OVER(ORDER BY (SELECT NULL))    AS [seq_nmbr]
FROM        sys.[tables] t
JOIN        sys.[schemas] s         ON  t.[schema_id]       = s.[schema_id]
JOIN        sys.[columns] c         ON  t.[object_id]       = c.[object_id]
LEFT JOIN   sys.[stats_columns] l   ON  l.[object_id]       = c.[object_id]
                                    AND l.[column_id]       = c.[column_id]
                                    AND l.[stats_column_id] = 1
LEFT JOIN    sys.[external_tables] e    ON    e.[object_id]        = t.[object_id]
WHERE       l.[object_id] IS NULL
AND            e.[object_id] IS NULL -- not an external table
)
SELECT  [table_schema_name]
,       [table_name]
,       [column_name]
,       [column_id]
,       [object_id]
,       [seq_nmbr]
,       CASE @create_type
        WHEN 1
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+')' AS VARCHAR(8000))
        WHEN 2
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH FULLSCAN' AS VARCHAR(8000))
        WHEN 3
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH SAMPLE '+@sample_pct+'PERCENT' AS VARCHAR(8000))
        END AS create_stat_ddl
FROM T
;

DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''
;

WHILE @i <= @t
BEGIN
    SET @s=(SELECT create_stat_ddl FROM #stats_ddl WHERE seq_nmbr = @i);

    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END

DROP TABLE #stats_ddl;
```

Para criar estatísticas em todas as colunas na tabela com esse procedimento, simplesmente chame o procedimento.

```sql
prc_sqldw_create_stats;
```

## <a name="examples-update-statistics"></a>Exemplos: atualizar as estatísticas
Para atualizar as estatísticas, você pode:

- Atualizar um objeto de estatísticas. Especifique o nome do objeto de estatísticas que você deseja atualizar.
- Atualizar todos os objetos de estatísticas em uma tabela. Especifique o nome da tabela em vez de um objeto de estatísticas específico.

### <a name="update-one-specific-statistics-object"></a>Atualizar um objeto de estatísticas específico
Use a sintaxe a seguir para atualizar um objeto de estatísticas específico:

```sql
UPDATE STATISTICS [schema_name].[table_name]([stat_name]);
```

Por exemplo: 

```sql
UPDATE STATISTICS [dbo].[table1] ([stats_col1]);
```

Ao atualizar objetos de estatísticas específicos, você pode minimizar o tempo e os recursos necessários para o gerenciamento de estatísticas. Isso requer algum planejamento para escolher os melhores objetos de estatísticas para atualizar.

### <a name="update-all-statistics-on-a-table"></a>Atualizar todas as estatísticas em uma tabela
Isso mostra um método simples para atualizar todos os objetos de estatísticas em uma tabela:

```sql
UPDATE STATISTICS [schema_name].[table_name];
```

Por exemplo: 

```sql
UPDATE STATISTICS dbo.table1;
```

Esta instrução é fácil de usar. Lembre-se de que isso atualizará *todas* as estatísticas na tabela e, portanto, poderá executar mais trabalho do que o necessário. Se o desempenho não for um problema, essa é a maneira mais fácil e completa de garantir que as estatísticas sejam atualizadas.

> [!NOTE]
> Ao atualizar todas as estatísticas em uma tabela, o SQL Data Warehouse realiza um exame para coletar amostras da tabela para cada objeto de estatística. Se a tabela for grande e tiver muitas colunas e muitas estatísticas, talvez seja mais eficiente atualizar estatísticas individuais com base na necessidade.
> 
> 

Para uma implementação de um `UPDATE STATISTICS` procedimento, consulte [Tabelas Temporárias][Temporary]. O método de implementação é ligeiramente diferente do procedimento anterior `CREATE STATISTICS`, mas o resultado é o mesmo.

Para obter a sintaxe completa, veja [Atualizar estatísticas][Update Statistics] no MSDN.

## <a name="statistics-metadata"></a>Metadados de estatísticas
Há várias exibições e funções do sistema que podem ser utilizadas para localizar informações sobre estatísticas. Por exemplo, você pode ver se um objeto de estatísticas está desatualizado usando a função stats-date para ver quando as estatísticas foram criadas ou atualizadas pela última vez.

### <a name="catalog-views-for-statistics"></a>Exibições de catálogo para as estatísticas
Essas exibições do sistema fornecem informações sobre estatísticas:

| Exibição do catálogo | DESCRIÇÃO |
|:--- |:--- |
| [sys.columns][sys.columns] |Uma linha para cada coluna. |
| [sys.objects][sys.objects] |Uma linha para cada objeto no banco de dados. |
| [sys.schemas][sys.schemas] |Uma linha para cada esquema no banco de dados. |
| [sys.stats][sys.stats] |Uma linha para cada objeto de estatísticas. |
| [sys.stats_columns][sys.stats_columns] |Uma linha para cada coluna no objeto de estatísticas. Conecta novamente a sys.columns. |
| [sys.tables][sys.tables] |Uma linha para cada tabela (inclui tabelas externas). |
| [sys.table_types][sys.table_types] |Uma linha para cada tipo de dados. |

### <a name="system-functions-for-statistics"></a>Funções de sistema para estatísticas
Essas funções de sistema são úteis para trabalhar com estatísticas:

| Função do sistema | DESCRIÇÃO |
|:--- |:--- |
| [STATS_DATE][STATS_DATE] |Data da última atualização do objeto de estatísticas. |
| [DBCC SHOW_STATISTICS][DBCC SHOW_STATISTICS] |Nível de resumo e informações detalhadas sobre a distribuição de valores conforme entendido pelo objeto de estatísticas. |

### <a name="combine-statistics-columns-and-functions-into-one-view"></a>Combinar colunas de estatísticas e funções em uma exibição
Essa exibição une as colunas relacionadas às estatísticas e os resultados da função STATS_DATE() em conjunto.

```sql
CREATE VIEW dbo.vstats_columns
AS
SELECT
        sm.[name]                           AS [schema_name]
,       tb.[name]                           AS [table_name]
,       st.[name]                           AS [stats_name]
,       st.[filter_definition]              AS [stats_filter_defiinition]
,       st.[has_filter]                     AS [stats_is_filtered]
,       STATS_DATE(st.[object_id],st.[stats_id])
                                            AS [stats_last_updated_date]
,       co.[name]                           AS [stats_column_name]
,       ty.[name]                           AS [column_type]
,       co.[max_length]                     AS [column_max_length]
,       co.[precision]                      AS [column_precision]
,       co.[scale]                          AS [column_scale]
,       co.[is_nullable]                    AS [column_is_nullable]
,       co.[collation_name]                 AS [column_collation_name]
,       QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS two_part_name
,       QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS three_part_name
FROM    sys.objects                         AS ob
JOIN    sys.stats           AS st ON    ob.[object_id]      = st.[object_id]
JOIN    sys.stats_columns   AS sc ON    st.[stats_id]       = sc.[stats_id]
                            AND         st.[object_id]      = sc.[object_id]
JOIN    sys.columns         AS co ON    sc.[column_id]      = co.[column_id]
                            AND         sc.[object_id]      = co.[object_id]
JOIN    sys.types           AS ty ON    co.[user_type_id]   = ty.[user_type_id]
JOIN    sys.tables          AS tb ON  co.[object_id]        = tb.[object_id]
JOIN    sys.schemas         AS sm ON  tb.[schema_id]        = sm.[schema_id]
WHERE   1=1
AND     st.[user_created] = 1
;
```

## <a name="dbcc-showstatistics-examples"></a>Exemplos de DBCC SHOW_STATISTICS()
DBCC SHOW_STATISTICS() mostra os dados contidos em um objeto de estatísticas. Esses dados estão divididos em três partes:

- Cabeçalho
- Vetor de densidade
- Histograma

Os metadados de cabeçalho sobre as estatísticas. O histograma exibe a distribuição de valores na primeira coluna de chave do objeto de estatísticas. O vetor de densidade mede a correlação entre colunas. O SQL Data Warehouse calcula estimativas de cardinalidade com qualquer um dos dados no objeto de estatística.

### <a name="show-header-density-and-histogram"></a>Mostrar cabeçalho, densidade e histograma
Este exemplo simples mostra as três partes de um objeto de estatísticas:

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
```

Por exemplo: 

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1);
```

### <a name="show-one-or-more-parts-of-dbcc-showstatistics"></a>Mostrar uma ou mais partes de DBCC SHOW_STATISTICS()
Se você estiver interessado apenas em visualizar partes específicas, use a cláusula `WITH` e especifique quais partes deseja ver:

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>) WITH stat_header, histogram, density_vector
```

Por exemplo: 

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1) WITH histogram, density_vector
```

## <a name="dbcc-showstatistics-differences"></a>Diferenças do DBCC SHOW_STATISTICS()
DBCC SHOW_STATISTICS() é implementado mais estritamente no SQL Data Warehouse comparado ao SQL Server:

- Não há suporte para recursos não documentados.
- Não é possível usar Stats_stream.
- Não é possível unir resultados para subconjuntos específicos de dados estatísticos. Por exemplo, (STAT_HEADER JOIN DENSITY_VECTOR).
- NO_INFOMSGS não pode ser definido para a supressão de mensagem.
- Não é possível usar colchetes em nomes de estatísticas.
- Não é possível usar nomes de coluna para identificar objetos de estatísticas.
- Não há suporte para o erro personalizado 2767.

## <a name="next-steps"></a>Próximas etapas
Para obter mais detalhes, veja [DBCC SHOW_STATISTICS][DBCC SHOW_STATISTICS] no MSDN.

  Para saber mais, consulte os artigos sobre [Visão geral da tabela][Overview], [Tipos de dados da tabela][Data Types], [Distribuição de uma tabela][Distribute], [Indexação de uma tabela][Index], [Particionamento de uma tabela][Partition] e [Tabelas temporárias][Temporary].
  
   Para saber mais sobre as práticas recomendadas, consulte [Práticas Recomendadas do SQL Data Warehouse][SQL Data Warehouse Best Practices].  

<!--Image references-->

<!--Article references-->
[Overview]: ./sql-data-warehouse-tables-overview.md
[Data Types]: ./sql-data-warehouse-tables-data-types.md
[Distribute]: ./sql-data-warehouse-tables-distribute.md
[Index]: ./sql-data-warehouse-tables-index.md
[Partition]: ./sql-data-warehouse-tables-partition.md
[Statistics]: ./sql-data-warehouse-tables-statistics.md
[Temporary]: ./sql-data-warehouse-tables-temporary.md
[SQL Data Warehouse Best Practices]: ./sql-data-warehouse-best-practices.md

<!--MSDN references-->  
[Cardinality Estimation]: https://msdn.microsoft.com/library/dn600374.aspx
[CREATE STATISTICS]: https://msdn.microsoft.com/library/ms188038.aspx
[DBCC SHOW_STATISTICS]:https://msdn.microsoft.com/library/ms174384.aspx
[Statistics]: https://msdn.microsoft.com/library/ms190397.aspx
[STATS_DATE]: https://msdn.microsoft.com/library/ms190330.aspx
[sys.columns]: https://msdn.microsoft.com/library/ms176106.aspx
[sys.objects]: https://msdn.microsoft.com/library/ms190324.aspx
[sys.schemas]: https://msdn.microsoft.com/library/ms190324.aspx
[sys.stats]: https://msdn.microsoft.com/library/ms177623.aspx
[sys.stats_columns]: https://msdn.microsoft.com/library/ms187340.aspx
[sys.tables]: https://msdn.microsoft.com/library/ms187406.aspx
[sys.table_types]: https://msdn.microsoft.com/library/bb510623.aspx
[UPDATE STATISTICS]: https://msdn.microsoft.com/library/ms187348.aspx

<!--Other Web references-->  
