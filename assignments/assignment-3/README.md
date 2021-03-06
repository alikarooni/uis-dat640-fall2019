# Assignment 3

**UPDATE 18/11**: Since indexing can be resource intensive, you may choose to work with a smaller subset of the collection by choosing between two options:
  * Index the following 3 files in full: `labels_en`, `long_abstracts_en`, and `page_links_en`.
  * Index only 10% of each file listed below.

Should you decide to choose any of these options, please state in your report explicitly which option you have chosen, as well as any relevant implementation details.

----

Your task is to implement three entity retrieval models on top of Elasticsearch and evaluate them on a standard test collection.

## Models

You need to implement three models and report evaluation results on those. All models should be implemented as a re-ranking mechanism over top-100 (first-pass) retrieval results that you obtain using the default retrieval model (BM25) in Elasticsearch.

  1. **MLM**: The mixture of language models approach with two fields, title and content, with weights 0.2 and 0.8, respectively. Content should be the "catch-all" field. Use Dirichlet smoothing with the smoothing parameter set to 2000.
  1. **SDM+ELR**: Sequential dependence model with the ELR extension.
      - This is a single-field variant of SDM, so you need to use the "catch-all" field for term-based scoring.
      - Use standard weights, that is, 0.8 for unigram matches, 0.05 for ordered bigram matches, 0.05 for unordered bigram matches, and 0.1 for entity matches.
      - Use Dirichlet smoothing with the smoothing parameter set to 2000.
      - Note: you'll need to build a positional index.
      - The entity annotations for queries are provided as part of the input.
  1. **Your model**: This may be any known extension/variant of existing models or your own invention. You may use any technique or combination of techniques introduced in the lectures, e.g., multiple fields, query expansion, word embeddings, etc. A standard option is FSDM+ELR, which should give you 10% relative improvement over SDM+ELR.

## Deliverable

  * The code solutions must be submitted as edits in the Jupyter notebooks provided:
    - *1_Indexing*: Code used for building the index.
    - *2_Ranking_Model_1*: Implementation of the MLM model.
    - *2_Ranking_Model_2*: Implementation of the SDM+ELR model.
    - *2_Ranking_Model_3*: Implementation of Your model.
    - *3_Evaluation*: Evaluation code is provided.
  * The ranking output files corresponding to the three models.
  * You need to complete the [REPORT.md](REPORT.md) file in your private repository.
  * You must submit your best ranking file to the [Kaggle competition](https://www.kaggle.com/t/b409c375ddd644b986874d54d8c8972a), and your grade will take your performance results into account.

### Assignment scoring

The specific criteria for scoring the assignment is as follows:

  * Indexing (2 points)
  * Implementation of MLM (2 points)
  * Implementation of SDM+ELR (2 points)
  * Implementation of Your model (2 points)
  * Performance of Your model (6 points)
    - Points are mainly based on the relative improvement made in percentage points (using rules of rounding) compared to the SDM+ELR model in terms of NDCG@10 score (on the training query set, i.e., `queries.txt`):

    | Improvement | Points |
    | -- | -- |
    | <2% | 0 |
    | >=2 and <4% | 1 |
    | >=4 and <6% | 2 |
    | >=6 and <8% | 3 |
    | >=8 and <10% | 4 |
    | >=10% | 5 |

  * Report (6 points)
  * Code readability and hygiene (5 points)


### Submission deadline

The **deadline** for submitting your code on GitHub as well as your final ranking on Kaggle is **27/11 17:00**.


## Data

### Knowledge base

The knowledge base we use as our dataset is DBpedia, specifically [version 2015-10](http://wiki.dbpedia.org/Downloads2015-10). You need to download and index this collection using Elasticsearch on your local machine.  Keep only entities that have both title and abstract (i.e., `rdfs:label` and `rdfs:comment` predicates).

DBpedia is distributed as a set of sub-collections. The following files are to be used (extract from the linked bzip2 files):

  * [anchor_text_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/anchor_text_en.ttl.bz2)
  * [article_categories_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/article_categories_en.ttl.bz2)
  * [disambiguations_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/disambiguations_en.ttl.bz2)*
  * [infobox_properties_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/infobox_properties_en.ttl.bz2)
  * [instance_types_transitive_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/instance_types_transitive_en.ttl.bz2)
  * [labels_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/labels_en.ttl.bz2)
  * [long_abstracts_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/long_abstracts_en.ttl.bz2)
  * [mappingbased_literals_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/mappingbased_literals_en.ttl.bz2)
  * [mappingbased_objects_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/mappingbased_objects_en.ttl.bz2)
  * [page_links_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/page_links_en.ttl.bz2)
  * [persondata_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/persondata_en.ttl.bz2)
  * [short_abstracts_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/short_abstracts_en.ttl.bz2)
  * [transitive_redirects_en.ttl](http://downloads.dbpedia.org/2015-10/core-i18n/en/transitive_redirects_en.ttl.bz2)*

The files marked with * contain reverse relations, meaning that in the subject-predicate-object triple the entity stands as object. Thus, it's the subject component of the triple that needs to be indexed (i.e., "reverse" the direction of the triple).

### Indexing

  * The choice of fields and preprocessing applied is up to you. In previous work, the following fields have been used successfully for text-based retrieval:

  | Field | Description | Predicates | Notes |
  | --- | --- | --- | --- |
  | Names | Names of the entity | `<foaf:name>`, `<dbp:name>`, `<foaf:givenName>`, `<foaf:surname>`, `<dbp:officialName>`, `<dbp:fullname>`, `<dbp:nativeName>`, `<dbp:birthName>`, `<dbo:birthName>`, `<dbp:nickname>`, `<dbp:showName>`, `<dbp:shipName>`, `<dbp:clubname>`, `<dbp:unitName>`, `<dbp:otherName>`, `<dbo:formerName>`, `<dbp:birthname>`, `<dbp:alternativeNames>`, `<dbp:otherNames>`, `<dbp:names>`, `<rdfs:label>` | |
  | Categories | Entity types | `<dcterms:subject>` | |
  | Similar entity names | Entity  name variants | `!<dbo:wikiPageRedirects>`, `!<dbo:wikiPageDisambiguates>`, `<dbo:wikiPageWikiLinkText>` | `!` denotes reverse direction (i.e. `<o, p, s>`) |
  | Attributes | Literal attributes of entity | All `<s, p, o>`, where *"o"* is a literal and *"p"* is not in *Names*, *Categories* or *Similar entity names*. For each `<s, p, o>` triple, if `p matches <dbp:.*>` both *p* and *o* are stored (i.e. *"p o"* is indexed). | |
  | Related entity names | URI relations of entity|  Similar to *Attributes* field, but *"o"* should be a URI. | |  

  * You'll also need a "catch-all" field (to be used as the "content" field for MLM and as the single field for SDM).
  * For entity-based retrieval, the ELR model assumes that each predicate is indexed as a separate field.  In practice, it is reasonable to consider only the top-1000 most frequent predicates as fields, and ignore predicates outside this set. You may mix the term-based and entity-based representations in a single index (using different field types) or create a separate entity-based index.
  * For subject-predicate-object (SPO) triples where the object is an entity, you want to make the object value searchable both as text (by performing URI resolution, i.e., replacing the URI with the canonical name of the entity) and as an entity. That is, you’ll need to have two different types of fields, text and entity. Then all relevant SPO triples with entity objects must contribute to both types of fields.
    - Category URIs may be resolved using `category_labels_en.ttl` file.
    - Predicate URIs may resolved using `infobox_property_definitions_en.ttl` file, should you decide that you also want to include the name of the attribute in the index in the Attributes field.
  * Remember that indexing may take some time, so make sure you leave enough time for it.
  * Use a single shard to make sure you're getting the right term statistics.


### Queries

The [queries.txt](data/queries.txt) file contains 234 queries in total.  Each line starts with a queryID, followed by the query string (separated by a tab).  E.g.,

```
INEX_LD-2009022	Szechwan dish food cuisine
INEX_LD-2009053	finland car industry manufacturer saab sisu
INEX_LD-2009062	social network group selection
...
```

You are provided with the relevance judgments for these queries (see below).

The [queries2.txt](data/queries2.txt) file contains additional 233 queries. These are "unseen" queries, for which you'll have to generate entity rankings, but you don't get to see the corresponding relevance judgments.


### Query entity annotations

Entity annotations for the queries are provided in the [entity_annotations.json](data/entity_annotations.json) file.  For each query (in `queries.txt` and `queries.txt`), there is an entry containing a dictionary of mentions along with the linked entity and corresponding confidence score.  For example, for query "Airlines that currently use Boeing 747 planes" (ID: TREC_Entity-7), annotations look as follows:

```
"TREC_Entity-7": {
    "Airlines": {
        "score": 0.27762,
        "uri": "<dbpedia:Airline>"
    },
    "Boeing 747": {
        "score": 0.73878,
        "uri": "<dbpedia:Boeing_747>"
    },
    "planes": {
        "score": 0.26357,
        "uri": "<dbpedia:Fixed-wing_aircraft>"
    }
}
```

Note that these annotations are generated automatically by some entity annotation system, therefore are imperfect.


### Relevance judgments

The [qrels.csv](data/qrels.csv) file contains the relevance judgments for the queries in `data/queries.txt`. Each line contains the relevance label for a query-entity pair.  Relevance ranges from 0 to 2, where 0 is non-relevant, 1 is (somewhat) relevant, and 2 is highly relevant.

Since EntityId-s may contain commas, they are enclosed in double quotes.

```
QueryId,EntityId,Relevance
INEX_LD-2009022,"<dbpedia:Afghan_cuisine>",0
INEX_LD-2009022,"<dbpedia:Akan_cuisine>",0
INEX_LD-2009022,"<dbpedia:Ambuyat>",0
INEX_LD-2009022,"<dbpedia:American_Chinese_cuisine>",1
...
```


### Output file format

The output file should contain two columns: QueryId and EntityId. For each query, up to 100 entities may be returned, in decreasing order of relevance (i.e., more relevant first).

Since EntityId-s may contain commas, they need to be enclosed in double quotes.

The file should contain a header and have the following format:

```
QueryId,EntityId
INEX_LD-2009022,"<dbpedia:American_Chinese_cuisine>"
INEX_LD-2009022,"<dbpedia:Chinese_cuisine>"
INEX_LD-2009022,"<dbpedia:Hot_and_sour_soup>"
...
INEX_LD-2009053,"<dbpedia:Sisu_A-45>"
INEX_LD-2009053,"<dbpedia:Sisu_A2045>"
INEX_LD-2009053,"<dbpedia:Sisu_Auto>"
...
```
