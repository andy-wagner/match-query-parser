# Match Query Parser

Tightly control how Solr query parsing and execution by passing the query analyzer at query time. Read more in [this blog post](http://opensourceconnections.com/blog/2017/01/23/our-solution-to-solr-multiterm-synonyms/).

as an example:

q=sea biscuit likes to fish&**bq={!match analyze_as=text_synonym search_with=term qf=body v=$q}**

With this query, query processings goes through the following steps:

1. Analyze the query string using `text_synonym` field type, perhaps resulting in tokens `[seabiscuit][sea] [biscuit] [likes] [to] [fish]`
2. Treat the resulting tokens as term queries, with dismax for overlapping positions: `(sea biscuit | sea | biscuit) OR likes OR to OR fish`

Match QP gives you an extremely high level of control over the search. You control both query analysis and the resulting lucene queries. For example, if you repeat the above example with a shingle analyzer, you can run a bigram search (like pf2 in edismax):

1. Analyze the query string using `text_shingle` field type, perhaps resulting in `[sea biscuit] [biscuit likes] [likes to]` ...
2. Treat the resulting tokens as phrase queries by setting `search_with=phrase`: `("sea biscuit" OR "biscuit likes" OR "likes to" ...)`

Or with a synonym analysis that outputs full synonyms as individual tokens, but with `search_with=phrase`:

1. `[seabiscuit][sea biscuit] [likes] [to] [fish]`
2. `("sea biscuit" | seabiscuit) OR likes OR to OR fish`

Read more in [this tutorial](TUTORIAL.md) and [this blog post](http://opensourceconnections.com/blog/2017/01/23/our-solution-to-solr-multiterm-synonyms/)

# Download and Install

- Download plugin for [Solr 6.0](http://matchqp.labs.o19s.com/match-query-parser-0.1.0-solr6.0.0.jar) | [Solr 6.6](http://matchqp.labs.o19s.com/match-query-parser-0.1.0-solr6.6.0.jar)
- Place in a [suitable location for plugins](https://wiki.apache.org/solr/SolrPlugins#How_to_Load_Plugins)
- Add XML to your solrconfig.xml:

```
<queryParser name="match" class="com.o19s.solr.search.MatchQParserPlugin"></queryParser>
```


# Parameters

### qf

A single field to be searched.

### analyze_as

Use this field type for analysis. The field type's query-time analyzer is used to analyze the query string. When using match qp, I often create field types for the sole purpose of having different query-time analyzers at my disposal.

If omitted, uses the query analysis of qf.

### search_with

 Either `term` (default) or `phrase`.

 - `term` the tokens output from analysis from step (1) above are turned into term queries
 - `phrase` the tokens output  from analysis from step (1) are whitespace tokenized, and turned into phrase queries   


 An important note about position overlaps. In the above example, we pretended that seabiscuit was transformed into just `[sea biscuit]` when in reality, both tokens `[seabiscuit]` and `[sea biscuit]` would be omitted in the same position. In this case, tokens in the same position are wrapped in a DisjunctionMaximum (dismax) query. So the actual query would be, using | to show the dismax operation

 `("sea biscuit" | seabiscuit) OR likes OR to OR fish`

### mm

Min-should-match expression used to specify the mm of the outer boolean query.

### pslop

Phrase slop to use for phrase query type.

# Acknowledgements

 - This is somewhat inspired by Elasticsearch's [match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html)
 - Sponsored by [OpenSource Connections](http://opensourceconnections.com)
