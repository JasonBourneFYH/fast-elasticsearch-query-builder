# fast-elasticsearch-query-builder
A fast way to build **ElasticSearch** query string, ***even without writing implement code.***

*Read this in other languages: [English](README.md), [简体中文](README.zh-cn.md)*

## Brief

You can integrate it in your project with these steps:

1. Complete the search criteria DTO **MySearchCirteria**（***add correct @annotation on the field***）

   ```java
   public class MySearchCirteria {
       @Must
       @Match
       private String name;
   }
   ```

2. Complete the query builder class **MyQueryBuilder** extending the  **BaseQueryBuilder**, with specifing **MySearchCirteria**  as the generics class

   ```java
   public class MyQueryBuilder extends BaseQueryBuilder<MySearchCriteria> {
   }
   ```

3. Invoke **MyQueryBuilder#build** to build the query string

   ```java
   public class TestQueryBuilder {
       public static void main(String[] args) {
           MyQueryBuilder myQueryBuilder = new MyQueryBuilder();
   		MySearchCriteria mySearchCriteria = new MySearchCriteria();
   		mySearchCriteria.setName("jack");
   		System.out.println(myQueryBuilder.build(mySearchCriteria));
       }
   }
   ```

   result

   ```json
   {
     "from": 0,
     "size": 20,
     "query": {
       "bool": {
         "must": [
           {
             "match": {
               "name": {
                 "query": "jack",
                 "operator": "OR",
                 "prefix_length": 0,
                 "max_expansions": 50,
                 "fuzzy_transpositions": true,
                 "lenient": false,
                 "zero_terms_query": "NONE",
                 "auto_generate_synonyms_phrase_query": true,
                 "boost": 1
               }
             }
           }
         ],
         "adjust_pure_negative": true,
         "boost": 1
       }
     }
   }
   ```

***fast-elasticsearch-query-builder*** generates dsl based on `org.elasticsearch-elasticsearch-6.3.0`

## Instruction

### Annotation Instruction

***fast-elasticsearch-query-builder*** privides two kinds of annotation to build query string: **Function Annotation** and **Query Annotation**.

#### Function Annotation

To set query properties.

| Annotation        | Field Type          | Function                                                     | Parameters                                                   |
| ----------------- | ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @Higilighters     | Collection\<String> | set ***highlight***                                          | **type**: the highlight type, default to "fvh"               |
| @PageNo           | Integer             | set  ***from***                                              | -                                                            |
| @PageSize         | Integer             | set **size**                                                 | -                                                            |
| @Sort             | List\<Sortable>     | set ***sort***, you should implement ***Sortable*** interface | -                                                            |
| @Source           | Collection\<String> | set ***_source.includes***                                   | -                                                            |
| @TermsAggregation | Integer             | set ***terms aggregation***                                  | **name**: aggregation name<br />**field**: aggregation field<br />**maxSize**: max value of the field<br />**order**: buckets' order<br />**executionHint**: mechanisms of aggregations execution |

> from + size <= max_result_window

#### Query Annotation

To build query clause. It includes **Query Context Annotation** and **Query Type Annotation**, and they ***must be used together***.

##### Query Context Annotation

Decides how  **Query Type Annotation** affect the hits (just filter the hits or affect the score).

| Annotation | Function                                                     |
| ---------- | ------------------------------------------------------------ |
| @Must      | set ***must query***                                         |
| @MustNot   | set ***must_not query***                                     |
| @Should    | set ***should query***                                       |
| @Filter    | set ***filter query***                                       |
| *@Or*      | will be converted to **should clause** in ***filter query*** , like **or** in **sql** |
| *@OrNot*   | will be converted to **should.mustNot clause** in  ***filter query*** 中的 **should.mustNot**, like **or not** in **sql** |

> @Or and @OrNot are not standard query context in Elasticsearch

##### Query Type Annotation

Decides the behavior of search.

| Annotation | Field Type    | Function                 | Parameters                                                   |
| ---------- | ------------- | ------------------------ | ------------------------------------------------------------ |
| @Match     | String        | set ***match query***    | **operator**: control boolean clause (or / and)              |
| @Term      | -             | set ***term query***     | -                                                            |
| @Terms     | Collection<?> | set ***terms query***    | -                                                            |
| @Range     | Number        | set ***range query***    | **type**: boundary type (from / to)<br />**includedBoundary**: whether includes boundary |
| @Exists    | Boolean       | set ***exists query***   | -                                                            |
| @Wildcard  | String        | set ***wildcard query*** | -                                                            |

> The annotations above contaion **fieldName** parameter, implicate the index field name using in building query, default to the field name that the annotation works on.

### Method Instruction

AbstractQueryBuilder#build flow chart like this:

![AbstractQueryBuilder#build flow chart](https://raw.githubusercontent.com/Thare-Lam/fast-elasticsearch-query-builder/master/query-builder-flow-chart.jpg)

You can custom your query in the abstract method.

For example，you can custon **filter query** like this:

```java
@Override
protected void customFilterQueries(List<QueryBuilder> filterQueries, 
                                   StuSearchCriteria stuSearchCriteria) {
    // custom query that generated not through annotation
    Optional.ofNullable(stuSearchCriteria.getCustomField())
        .ifPresent(t -> filterQueries.add(termQuery("customFilterField", t)));
}
```

**You can see more detail usage in the demo under the `test/` directory.**