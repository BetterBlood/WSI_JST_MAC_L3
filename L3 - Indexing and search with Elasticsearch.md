# MAC L3 - Indexing and search with Elasticsearch
## Authors : Slimani Walid & Steiner Jeremiah
### 2.2 Indexing
#### D1
**Create the index**
[INPUT]

```Json
PUT cacm_standard
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "standard"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword",
        "index": false,
        "store": true
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets"
      }
    }
  }
}
```

[OUTPUT]
```Json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "cacm_standard"
}
```
**Using reindex command**
[INPUT]
```Json
POST _reindex
{
  "source": {
    "index": "cacm_dynamic"
  },
  "dest": {
    "index": "cacm_standard"
  }
}
```

[OUTPUT]
```Json
{
  "took": 260,
  "timed_out": false,
  "total": 3202,
  "updated": 0,
  "created": 3202,
  "deleted": 0,
  "batches": 4,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```



#### D2

**Create the index**
[INPUT]
```Json
PUT cacm_termvector
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword",
        "index": false,
        "store": true
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets",
        "term_vector": "with_offsets"
      }
    }
  }
}
```

[OUTPUT]
```Json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "cacm_termvector"
}
```

**Using reindex command**
[INPUT]
```Json
POST _reindex
{
  "source": {
    "index": "cacm_dynamic"
  },
  "dest": {
    "index": "cacm_termvector"
  }
}
```

[OUTPUT]
```Json
{
  "took": 502,
  "timed_out": false,
  "total": 3202,
  "updated": 0,
  "created": 3202,
  "deleted": 0,
  "batches": 4,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```



#### D3

**Check the presence of the term vector**
[INPUT]
```Json
GET /cacm_termvector/_termvectors/AGa0544BNHAuLhkC0w00?fields=summary
```

[OUTPUT]
```Json
{
  "_index": "cacm_termvector",
  "_id": "AGa0544BNHAuLhkC0w00",
  "_version": 1,
  "found": true,
  "took": 2,
  "term_vectors": {}
}
```



#### D4

**Explanation of what a termvector is**
Un _termvector_ est une structure de données utilisée dans _Elasticsearch_ pour représenter les termes et leurs statistiques associées dans un document. Essentiellement, c'est une manière organisée de stocker des informations sur les termes qui apparaissent dans un champ textuel spécifique d'un document. Ils fournissent des détails sur la fréquence, la position et d'autres caractéristiques des termes dans un champ donné. Ils sont utilisés pour des tâches telles que l'analyse de la pertinence des résultats de recherche, la génération de rapports sur la distribution des termes, ou encore la mise en évidence des termes de recherche dans les résultats.
En ce qui concerne la commande effectuée, Elle permet de demander à _ElasticSearch_ de fournir des informations sur les termes du champ "summary" pour un document spécifique identifié par l'ID "AGa0544BNHAuLhkC0w00" dans l'index "cacm_termvector". En incluant le paramètre "fields=summary", nous demandons uniquement les informations sur les termes du champ "summary".



#### D5

**summary table of index sizes**
|Index|size|
|:-------|:------------|
|_cacm_standard_|1.69 mb|
|_cacm_termvector_|2.36 mb|

**Calculating the increase in size**
La formule pour calculer l'augmentation de taille d'un index avec term_vector with_offsets par rapport à un index standard est la suivante (La soustraction de 1 est utilisée pour normaliser la valeur de l'augmentation de taille par rapport à la taille de l'index standard) :

Augmentation de la taille = ((2.36 / 1.69) - 1) * 100 = 39,64 %

L'index utilisant un _term_vector with_offsets_ sur le champ "summary" affiche une augmentation de taille d'environ 40%, soit 0.67 mb de plus que l'index _cacm_standard_. Cette croissance est justifiable en examinant la quantité d'informations supplémentaires obtenues. En optant pour d'autres configurations telles que _with_position_offsets_, nous enrichissons davantage chaque terme, ce qui entraîne une augmentation plus significative de la taille de l'index.



### 2.2 Reading Index

#### D6
**Author with the highest number of publications**

[INPUT]
```Json
GET cacm_standard/_search
{
  "size": 0,
  "aggs": {
    "authors": {
      "terms": {
        "field": "author",
        "size": 1
      }
    }
  }
}
```

[OUTPUT]
```Json
{
  "took": 14,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3202,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "authors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 4268,
      "buckets": [
        {
          "key": "Thacher Jr., H. C.",
          "doc_count": 38
        }
      ]
    }
  }
}
```
**"Thacher Jr., H. C."** est l'auteur qui à le plsu grand nombre de publications avec une valeur de **38 publications**.



#### D7

**Top 10 terms in the title field with their frequency**

[INPUT]
```Json
GET cacm_standard/_search
{
  "size": 0,
  "aggs": {
    "titles": {
      "terms": {
        "field": "title",
        "size": 10
      }
    }
  }
}
```

[OUTPUT]
```Json
{
  "took": 73,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3202,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "titles": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 17307,
      "buckets": [
        {
          "key": "of",
          "doc_count": 1138
        },
        {
          "key": "algorithm",
          "doc_count": 975
        },
        {
          "key": "a",
          "doc_count": 895
        },
        {
          "key": "for",
          "doc_count": 714
        },
        {
          "key": "the",
          "doc_count": 645
        },
        {
          "key": "and",
          "doc_count": 434
        },
        {
          "key": "in",
          "doc_count": 416
        },
        {
          "key": "on",
          "doc_count": 340
        },
        {
          "key": "an",
          "doc_count": 275
        },
        {
          "key": "computer",
          "doc_count": 275
        }
      ]
    }
  }
}
```

|Rank|Key word|frequency|
|:---|:-------|:--------|
|1|_of_|1138|
|2|_algorithm_|975|
|3|_a_|895|
|4|_for_|714|
|5|_the_|645|
|6|_and_|434|
|7|_in_|416|
|8|_on_|340|
|9|_an_|275|
|10|_computer_|275|



###  2.4 Using different Analyzers

#### D8
**Create index with whitespace analyzer**

[INPUT]
```Json
PUT cacm_whitespace
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "whitespace"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword",
        "index": false,
        "store": true
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets"
      }
    }
  }
}
```

[OUTPUT]
```Json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "cacm_whitespace"
}
```

**Using reindex function for _cacm_whitespace_**

[INPUT]
```Json
POST _reindex
{
  "source": {
    "index": "cacm_dynamic"
  },
  "dest": {
    "index": "cacm_whitespace"
  }
}
```

[OUTPUT]
```Json
{
  "took": 359,
  "timed_out": false,
  "total": 3202,
  "updated": 0,
  "created": 3202,
  "deleted": 0,
  "batches": 4,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

**Create index with english analyzer**

[INPUT]
```Json
PUT cacm_english
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "english"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword",
        "index": false,
        "store": true
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets"
      }
    }
  }
}
```

[OUTPUT]
```Json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "cacm_english"
}
```

**Using reindex function for cacm_english**

[INPUT]
```Json
POST _reindex
{
  "source": {
    "index": "cacm_dynamic"
  },
  "dest": {
    "index": "cacm_english"
  }
}
```

[OUTPUT]
```Json
{
  "took": 214,
  "timed_out": false,
  "total": 3202,
  "updated": 0,
  "created": 3202,
  "deleted": 0,
  "batches": 4,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

**Create index with custom analyzer (lowercase filter and output shingles)**

[INPUT]
```Json
PUT cacm_lowercase_shingles1
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "shingles"
          ]
        }
      },
      "filter": {
        "shingles": {
          "type": "shingle",
          "max_shingle_size": 2,
          "output_unigrams": true
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword",
        "index": false,
        "store": true
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets"
      }
    }
  }
}
```

[OUTPUT]
``` Json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "cacm_lowercase_shingles1"
}
```

**Using reindex function for cacm_lowercase_shingles1**

[INPUT]
```Json
POST _reindex
{
  "source": {
    "index": "cacm_dynamic"
  },
  "dest": {
    "index": "cacm_lowercase_shingles1"
  }
}
```

[OUTPUT]
```Json
{
  "took": 401,
  "timed_out": false,
  "total": 3202,
  "updated": 0,
  "created": 3202,
  "deleted": 0,
  "batches": 4,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

**Create index with shingles of size 3**

[INPUT]
```Json
PUT cacm_shingles3
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "shingles"
          ]
        }
      },
      "filter": {
        "shingles": {
          "type": "shingle",
          "max_shingle_size": 3,
          "min_shingle_size": 3,
          "output_unigrams": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword",
        "index": false,
        "store": true
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets"
      }
    }
  }
}
```

[OUTPUT]
```Json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "cacm_shingles3"
}
```

**Using reindex function for cacm_shingles3**

[INPUT]
```Json
POST _reindex
{
  "source": {
    "index": "cacm_dynamic"
  },
  "dest": {
    "index": "cacm_shingles3"
  }
}
```

[OUTPUT]
```Json
{
  "took": 381,
  "timed_out": false,
  "total": 3202,
  "updated": 0,
  "created": 3202,
  "deleted": 0,
  "batches": 4,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

**Create index using a stop filter with common words**

[INPUT]
```Json
PUT cacm_stop
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "stop_filter"
          ]
        }
      },
      "filter": {
        "stop_filter": {
          "type": "stop",
          "stopwords_path": "data/common_words.txt"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword",
        "index": false,
        "store": true
      },
      "author": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
        "fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets"
      }
    }
  }
}
```

[OUTPUT]
```Json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "cacm_stop"
}
```

**Using reindex function for cacm_stop**

[INPUT]
```Json
POST _reindex
{
  "source": {
    "index": "cacm_dynamic"
  },
  "dest": {
    "index": "cacm_stop"
  }
}
```

[Output]
```Json
{
  "took": 193,
  "timed_out": false,
  "total": 3202,
  "updated": 0,
  "created": 3202,
  "deleted": 0,
  "batches": 4,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

#### D9
**Explanation of _Whitespace_ analyzer**

**Explanation of _English_ analyzer**

**Explanation of custom analyzer (lowercase filter and output shingles of size 1 to 2)**

**Explanation of custom analyzer (shingles of size 3) **

**Explanation of custom _stop list_ analyzer**