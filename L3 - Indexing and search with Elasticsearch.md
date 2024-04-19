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

PS : Les informations ont été récupérées via l'interface graphique. Toutefois, il est possible d'effectuer la commande GET /"index1" "index2" "indexn"/_stats/store afin de récupérer les données.



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

|Rank|Key word| Count |Frequency|
|:---|:-------|:--------|----|
|1|_of_|1138|36.44 %|
|2|_algorithm_|975|31.22 %|
|3|_a_|895|28.66 %|
|4|_for_|714|22.86 %|
|5|_the_|645|20.65 %|
|6|_and_|434|13.9 %|
|7|_in_|416|13.32 %|
|8|_on_|340|10.89 %|
|9|_an_|275|8.81 %|



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

Un _whitespace analyzer_ dans Elasticsearch est un type d'analyseur qui est utilisé pour indexer et rechercher du texte en fonction des espaces entre les mots. Contrairement à d'autres analyseurs qui divisent le texte en mots en utilisant des règles de tokenization plus complexes, le _whitespace analyzer_ divise le texte uniquement là où il rencontre un espace. Cela signifie que chaque mot est considéré comme un seul token.

L'avantage d'utiliser un _whitespace analyzer_ est qu'il ne modifie pas la structure du texte d'entrée. Cela peut être utile dans certains cas où la tokenisation plus complexe d'autres analyseurs pourrait modifier le sens du texte.



**Explanation of _English_ analyzer**

Un "English analyzer" dans Elasticsearch est un outil spécialement conçu pour analyser et indexer du texte en anglais de manière efficace. Il comprend plusieurs étapes telles que la tokenization, la conversion en minuscules, le filtrage des mots vides (stopwords), le stemming et éventuellement la lemmatisation. Ces processus permettent de normaliser le texte, de réduire les mots à leur forme de base et d'améliorer la pertinence des résultats de recherche. En résumé, l'English analyzer optimise la recherche de texte anglais dans Elasticsearch en traitant le langage de manière appropriée.



**Explanation of custom analyzer (lowercase filter and output shingles of size 1 to 2)**

L'analyseur custom que nous avons créé utilise un analyseur standard et applique un filtre permettant de créer différentes combinaisons de token adjacents de taille 1 et 2. De plus, ce dernier utilise un filtre afin de détecter les caractères minuscules.

L'avantage principal de cet analyseur et qu'il peut facilement détecter les mots "isolés" ainsi que leur fréquence d'apparition en fonction des mots adjacents. Cela permet de grandement améliorer la capacité de recherche des phrases ainsi que des suites de mots courants sans devoir donner une phrase exacte.



**Explanation of custom analyzer (shingles of size 3) **

Cet analyseur custom est très similaire au précédent. Toutefois, il génère des shingles de taille 3 uniquement. De plus, le filtre permettant de convertir un caractère en minuscule a été supprimé. En outre, il fonctionne de façon très similaire à l'analyseur précédent. La génération de shingles de taille 3 permet d'élargir davantage la compréhenseion du contexte dans lequel se situent les mots. Cet analyseur peut donc aisément récupérer des textes dans lesquels des phrases de 3 mots sont importantes.



**Explanation of custom _stop list_ analyzer**

Cet analyseur utilise également l'analyseur standard ainsi qu'un filtre utilisant une liste de "stopwords". Il s'agit de mots clés anglais. Comme précédemment, l'analyseur standard décompose le texte et ensuite filtre les mots qui apparaissent dans la liste de "stopwords". Cela a pour but d'effectuer un contrôle sur des mots très précis. En outre, cela va permettre d'optimiser les résultats obtenus à la suite d'une recherche tout en ignorant les mots les plus fréquents qui sont dépourvus d'importance dans un contexte de recherche.



#### D10
| Analyseur                                     | Whitespace                                                   | English                                                      | Lowercase and shingles 1                                     | Shingles 3                                                   | Stop words                                                   |
| :-------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Nb documents indexés                          | 3202                                                         | 3202                                                         | 3202                                                         | 3202                                                         | 3202                                                         |
| Nb de termes indexés dans le champ "sumarry"  | 103'275                                                      | 72'298                                                       | 237'189                                                      | 144'719                                                      | 66'555                                                       |
| Top 10 termes indexés dans le champ "sumarry" | 01. _of_ (1534)<br />02. _the_ (1501)<br />03. _is_ (1382)<br />04. _and_ (1369)<br />05. _a_ (1321)<br />06. _to_ (1293)<br />07. _in_ (1188)<br />08. _for_ (1167)<br />09. _the_ (1072)<br />10. _are_ (1022) | 01. _which_ (781)<br />02. _us_ (778)<br />03. _comput_ (663)<br />04. _program_ (635)<br />05. _system_ (586)<br />06. _present_ (514)<br />07. _describ_ (505)<br />08. _paper_ (428)<br />09. _can_ (421)<br />10. _gener_ (411) | 01. _the_ (1541)<br />02. _of_ (1534)<br />03. _a_ (1426)<br />04. _is_ (1384)<br />05. _and_ (1376)<br />06. _to_ (1301)<br />07. _in_ (1234)<br />08. _for_ (1182)<br />09. _are_ (1025)<br />10. _of the_ (938) | 01. _the number of_ (97)<br />02. _the use of_ (86)<br />03. _in terms of_ (82)<br />04. _a set of_ (74)<br />05. _is shown that_ (71)<br />06. _It is shown_ (64)<br />07. _In this paper_ (63)<br />08. _as well as_ (63)<br />09. _This paper describes_ (55)<br />10. _shown to be_ (53) | 01. _The_ (1072)<br />02. _A_ (690)<br />03. _This_ (465)<br />04. _computer_ (441)<br />05. _system_ (429)<br />06. _paper_ (421)<br />07. _presented_ (372)<br />08. _time_ (354)<br />09. _program_ (339)<br />10. _data_ (309) |
| Taille de l'index                             | 1.9 mb                                                       | 1.5 mb                                                       | 3.59 mb                                                      | 3.98 mb                                                      | 1.52 mb                                                      |
| Temps d'indexation                            | 359 ms                                                       | 214 ms                                                       | 401 ms                                                       | 381 ms                                                       | 193 ms                                                       |



#### D11

**1. Efficacité de l'indexation :**
   - L'utilisation de l'analyseur _Lowercase and shingles 1_ permet de générer un nombre considérable de termes indexés dans le champ "summary" (237'189). Cela suggère que l'application de shingles et la mise en minuscule des mots peuvent accroître la granularité de l'indexation, ce qui est bénéfique pour une recherche plus précise sur les données.
   - En revanche, l'analyseur _English_ produit un nombre moins élevé de termes indexés (72'298), ce qui peut indiquer une indexation plus concise mais potentiellement plus pertinente pour des documents en anglais technique.

**2. Performance et stockage :**

   - Bien que l'analyseur "Lowercase and shingles 1" génère un index plus volumineux (3.59 mb), il offre une granularité accrue pour la recherche, ce qui peut être avantageux lorsque la précision est essentielle.
   - L'analyseur "Whitespace" se distingue par son temps d'indexation le plus court (193 ms), ce qui en fait un choix optimal pour une balance entre efficacité de l'indexation et vitesse de traitement, car il se contente de diviser le texte en fonction des espaces sans autres transformations.

**3. Fréquence des termes dans les index :**

   - L'examen des dix termes les plus fréquemment indexés dans le champ "summary" révèle des différences significatives entre les analyseurs. Par exemple, l'analyseur "Lowercase and shingles 1" produit une liste où certains termes apparaissent fréquemment ("of", "the", "is"), ce qui peut indiquer une granularité fine de l'indexation mais avec des termes communs.
   - En revanche, l'analyseur "English" présente une liste où des termes spécifiques au domaine informatique sont plus fréquents ("which", "comput", "program"), reflétant potentiellement une indexation plus spécialisée et pertinente pour ce domaine.
   - Cette différence dans la fréquence des termes suggère que le choix de l'analyseur peut influencer la pertinence des résultats de recherche en fonction du corpus de documents et des besoins spécifiques de l'application.



### 2.5 Searching

#### D12

**1. Publications containing “Information Retrieval”**

[INPUT]
```Json
GET cacm_english/_search
{
  "stored_fields": [
    "id"
  ],
  "query": {
    "query_string": {
      "query": "\"Information Retrieval\"",
      "fields": [
        "summary"
      ]
    }
  }
}
```

[OUTPUT]

```Json
{
  "took": 649,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 20,
      "relation": "eq"
    },
    "max_score": 8.097484,
    "hits": [
      {
        "_index": "cacm_english",
        "_id": "eWa0544BNHAuLhkC0xA4",
        "_score": 8.097484,
        "fields": {
          "id": [
            "891"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "7ma0544BNHAuLhkC0xU6",
        "_score": 7.363991,
        "fields": {
          "id": [
            "2288"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "r2a0544BNHAuLhkC0xI5",
        "_score": 7.089669,
        "fields": {
          "id": [
            "1457"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "jWa0544BNHAuLhkC0xQ6",
        "_score": 6.768644,
        "fields": {
          "id": [
            "1935"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "oWa0544BNHAuLhkC0xM5",
        "_score": 6.714481,
        "fields": {
          "id": [
            "1699"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "jWa0544BNHAuLhkC0w83",
        "_score": 6.425843,
        "fields": {
          "id": [
            "655"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "6Ga0544BNHAuLhkC0xI5",
        "_score": 5.8798933,
        "fields": {
          "id": [
            "1514"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "WWa0544BNHAuLhkC0xM5",
        "_score": 5.7904453,
        "fields": {
          "id": [
            "1627"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "1Wa0544BNHAuLhkC1BZd",
        "_score": 5.5377173,
        "fields": {
          "id": [
            "2519"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "WGa0544BNHAuLhkC1Bde",
        "_score": 5.5377173,
        "fields": {
          "id": [
            "2650"
          ]
        }
      }
    ]
  }
}
```



**2. Publications containing “Information” and “Retrieval”**
[INPUT]

```Json
GET cacm_english/_search
{
  "stored_fields": [
    "id"
  ],
  "query": {
    "query_string": {
      "query": "(Information) AND (Retrieval)",
      "fields": [
        "summary"
      ]
    }
  }
}
```

[OUTPUT]
```Json
{
  "took": 89,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 36,
      "relation": "eq"
    },
    "max_score": 8.710291,
    "hits": [
      {
        "_index": "cacm_english",
        "_id": "7ma0544BNHAuLhkC0xU6",
        "_score": 8.710291,
        "fields": {
          "id": [
            "2288"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "eWa0544BNHAuLhkC0xA4",
        "_score": 8.470867,
        "fields": {
          "id": [
            "891"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "r2a0544BNHAuLhkC0xI5",
        "_score": 8.343676,
        "fields": {
          "id": [
            "1457"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "Bma0544BNHAuLhkC0xE4",
        "_score": 8.070196,
        "fields": {
          "id": [
            "1032"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "PGa0544BNHAuLhkC1Blg",
        "_score": 7.984795,
        "fields": {
          "id": [
            "3134"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "oWa0544BNHAuLhkC0xM5",
        "_score": 7.7670193,
        "fields": {
          "id": [
            "1699"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "AWa0544BNHAuLhkC0xY7",
        "_score": 7.0211506,
        "fields": {
          "id": [
            "2307"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "6Ga0544BNHAuLhkC0xI5",
        "_score": 6.9998264,
        "fields": {
          "id": [
            "1514"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "jWa0544BNHAuLhkC0xQ6",
        "_score": 6.7686434,
        "fields": {
          "id": [
            "1935"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "1Wa0544BNHAuLhkC1BZd",
        "_score": 6.673133,
        "fields": {
          "id": [
            "2519"
          ]
        }
      }
    ]
  }
}
```



**3. Publications containing at least the term “Retrieval” and, possibly “Information” but not “Database”**
[INPUT]

```Json
GET cacm_english/_search
{
  "stored_fields": [
    "id"
  ],
  "query": {
    "query_string": {
      "query": "+Retrieval Information -Database",
      "fields": [
        "summary"
      ]
    }
  }
}
```

[OUTPUT]
```Json
{
  "took": 133,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 69,
      "relation": "eq"
    },
    "max_score": 8.710291,
    "hits": [
      {
        "_index": "cacm_english",
        "_id": "7ma0544BNHAuLhkC0xU6",
        "_score": 8.710291,
        "fields": {
          "id": [
            "2288"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "eWa0544BNHAuLhkC0xA4",
        "_score": 8.470867,
        "fields": {
          "id": [
            "891"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "r2a0544BNHAuLhkC0xI5",
        "_score": 8.343676,
        "fields": {
          "id": [
            "1457"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "Bma0544BNHAuLhkC0xE4",
        "_score": 8.070196,
        "fields": {
          "id": [
            "1032"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "PGa0544BNHAuLhkC1Blg",
        "_score": 7.984795,
        "fields": {
          "id": [
            "3134"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "oWa0544BNHAuLhkC0xM5",
        "_score": 7.7670193,
        "fields": {
          "id": [
            "1699"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "AWa0544BNHAuLhkC0xY7",
        "_score": 7.0211506,
        "fields": {
          "id": [
            "2307"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "6Ga0544BNHAuLhkC0xI5",
        "_score": 6.9998264,
        "fields": {
          "id": [
            "1514"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "jWa0544BNHAuLhkC0xQ6",
        "_score": 6.7686434,
        "fields": {
          "id": [
            "1935"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "1Wa0544BNHAuLhkC1BZd",
        "_score": 6.673133,
        "fields": {
          "id": [
            "2519"
          ]
        }
      }
    ]
  }
}
```



**4. Publications containing a term starting with “Info”**
[INPUT]

```Json
GET cacm_english/_search
{
  "stored_fields": [
    "id"
  ],
  "query": {
    "query_string": {
      "query": "Info*",
      "fields": [
        "summary"
      ]
    }
  }
}
```

[OUTPUT]
```Json
{
  "took": 41,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 205,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "cacm_english",
        "_id": "3Ga0544BNHAuLhkC0w01",
        "_score": 1,
        "fields": {
          "id": [
            "222"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "Dma0544BNHAuLhkC0w41",
        "_score": 1,
        "fields": {
          "id": [
            "272"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "ima0544BNHAuLhkC0w42",
        "_score": 1,
        "fields": {
          "id": [
            "396"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "i2a0544BNHAuLhkC0w42",
        "_score": 1,
        "fields": {
          "id": [
            "397"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "l2a0544BNHAuLhkC0w42",
        "_score": 1,
        "fields": {
          "id": [
            "409"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "tma0544BNHAuLhkC0w42",
        "_score": 1,
        "fields": {
          "id": [
            "440"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "4Wa0544BNHAuLhkC0w42",
        "_score": 1,
        "fields": {
          "id": [
            "483"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "Zma0544BNHAuLhkC0w83",
        "_score": 1,
        "fields": {
          "id": [
            "616"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "gma0544BNHAuLhkC0w83",
        "_score": 1,
        "fields": {
          "id": [
            "644"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "jWa0544BNHAuLhkC0w83",
        "_score": 1,
        "fields": {
          "id": [
            "655"
          ]
        }
      }
    ]
  }
}
```



**5. Publications containing the term “Information” close to “Retrieval” (max distance 5)**
[INPUT]

```Json
GET cacm_english/_search
{
  "stored_fields": [
    "id"
  ],
  "query": {
    "query_string": {
      "query": "\"Information Retrieval\"~5",
      "fields": [
        "summary"
      ]
    }
  }
}
```

[OUTPUT]
```Json
{
  "took": 147,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 30,
      "relation": "eq"
    },
    "max_score": 8.097484,
    "hits": [
      {
        "_index": "cacm_english",
        "_id": "eWa0544BNHAuLhkC0xA4",
        "_score": 8.097484,
        "fields": {
          "id": [
            "891"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "7ma0544BNHAuLhkC0xU6",
        "_score": 7.363991,
        "fields": {
          "id": [
            "2288"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "r2a0544BNHAuLhkC0xI5",
        "_score": 7.089669,
        "fields": {
          "id": [
            "1457"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "jWa0544BNHAuLhkC0xQ6",
        "_score": 6.768644,
        "fields": {
          "id": [
            "1935"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "oWa0544BNHAuLhkC0xM5",
        "_score": 6.714481,
        "fields": {
          "id": [
            "1699"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "jWa0544BNHAuLhkC0w83",
        "_score": 6.425843,
        "fields": {
          "id": [
            "655"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "6Ga0544BNHAuLhkC0xI5",
        "_score": 5.8798933,
        "fields": {
          "id": [
            "1514"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "AWa0544BNHAuLhkC0xY7",
        "_score": 5.7931695,
        "fields": {
          "id": [
            "2307"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "WWa0544BNHAuLhkC0xM5",
        "_score": 5.7904453,
        "fields": {
          "id": [
            "1627"
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "1Wa0544BNHAuLhkC1BZd",
        "_score": 5.5377173,
        "fields": {
          "id": [
            "2519"
          ]
        }
      }
    ]
  }
}
```



#### D13

**1. Result of publications containing “Information Retrieval”**

```Json
"hits": {
    "total": {
      "value": 20,
      "relation": "eq"
    }
```



**2. Result of publications containing “Information” and “Retrieval”**

```Json
"hits": {
    "total": {
      "value": 36,
      "relation": "eq"
    }
```



**3. Result of publications containing at least the term “Retrieval” and, possibly “Information” but not “Database”**

```Json
"hits": {
    "total": {
      "value": 69,
      "relation": "eq"
    }
```



**4. Result of publications containing a term starting with “Info”**

```Json
"hits": {
    "total": {
      "value": 205,
      "relation": "eq"
    }
```



**5. Result of publications containing the term “Information” close to “Retrieval” (max distance 5)**

```Json
"hits": {
    "total": {
      "value": 30,
      "relation": "eq"
    }
```



### 2.6 Tuning the Lucene Score
#### D14

[INPUT]

```Json
GET cacm_english/_search
{
  "query": {
    "function_score": {
      "query": {
        "query_string": {
          "query": "compiler program"
        }
      },
      "linear": {
        "date": {
          "origin": "1970-01",
          "scale": "90d",
          "decay": 0.5
        }
      }
    }
  }
}
```

[OUTPUT]

```Json
{
  "took": 66,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 822,
      "relation": "eq"
    },
    "max_score": 4.0517297,
    "hits": [
      {
        "_index": "cacm_english",
        "_id": "DWa0544BNHAuLhkC0xQ6",
        "_score": 4.0517297,
        "_source": {
          "date": "1969-12",
          "summary": "A method of optimizing the computation of arithmetic and indexing expressions of a Fortran program is presented.  The method is based on a linear analysis of the definition points of the variables and the branching and DO loop structure of the program. The objectives of the processing are (1) to eliminate redundant calculations when references are made to common subexpression values, (2) to remove invariant calculations from DO loops, (3) to efficiently compute subscripts containing DO iteration variables, and (4) to provide efficient index register usage.  The method presented requires at least a three-pass compiler, the second of which is scanned backward.  It has been used in the development of several FORTRAN compilers that have proved to produce excellent object code without significantly reducing the compilation speed.",
          "id": "1807",
          "title": "Optimization of Expressions in Fortran",
          "author": [
            "Busam, V. A.",
            "England, D. E."
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "PWa0544BNHAuLhkC0xU6",
        "_score": 3.0391405,
        "_source": {
          "date": "1970-02",
          "summary": "Several specialized techniques are shown for efficiently incorporating spelling correction algorithms in to compilers and operating systems.  These include the use of syntax and semantics information, the organization of restricted keyword and symbol tables, and the consideration of a limited class of spelling errors.  Sample 360 coding for performing spelling correction is presented.  By using systems which perform spelling correction, the number of debugging runs per program has been decreased, saving both programmer and machine time.",
          "id": "2111",
          "title": "Spelling Correction in Systems Programs",
          "author": [
            "Morgan, H. L."
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "L2a0544BNHAuLhkC0xU6",
        "_score": 2.4397445,
        "_source": {
          "date": "1970-03",
          "summary": "The TEACH system was developed at MIT to ease the cost and improve the results of elementary instruction in programming.  To the student, TEACH offers loosely guided experience with a  conversational language which was designed with teaching in mind.  Faculty involvement is minimal.  A term of experience with TEACH is discussed.  Pedagogically, the system appears to be successful; straightforward reimplementation will make it economically successful as well. Similar programs of profound tutorial skill will appear only as the results of extended research.  The outlines of his research are beginning to become clear.",
          "id": "2097",
          "title": "A Program to Teach Programming",
          "author": [
            "Fenichel, R. R.",
            "Weizenbaum, J.",
            "Yochelson, J. C."
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "EWa0544BNHAuLhkC0xQ6",
        "_score": 1.9922912,
        "_source": {
          "date": "1969-12",
          "summary": "An affirmative partial answer is provided to the question of whether it is possible to program parallel-processor computing systems to efficiently decrease execution time for useful problems.  Parallel-processor systems are multiprocessor systems in which several of the processors can simultaneously execute separate tasks of a single job, thus cooperating to decrease the solution time of a computational problem. The processors have independent instruction counters, meaning that each processor executes its own task program relatively independently of the other processors.  Communication between cooperating processors is by means of data in storage shared by all processors.  A program for the determination of the distribution of current in an electrical network was written for a parallel-processor computing system, and execution of this program was simulated.  The data gathered from simulation runs demonstrate the efficient solution of this problem, typical of a large class of important problems.  It is shown that, with proper programming, solution time when N processors are applied approaches 1/N times the solution time for a single processor, while improper programming can actually lead to an increase of solution time with the number of processors. Stability of the method of solution was also investigated.",
          "id": "1811",
          "title": "A Case Study in Programming for Parallel-Processors",
          "author": [
            "Rosenfeld, J. L."
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "D2a0544BNHAuLhkC0xQ6",
        "_score": 1.8519735,
        "_source": {
          "date": "1969-12",
          "summary": """Numerical Analysis is the study of methods and procedures used to obtain "approximate solutions" to mathematical problems.  Much of the emphasis is on scientific calculation.  The difficulties of education in such a broad area center around the question of background and emphasis.  The Numerical Analysis program in the Computer Science Department should emphasize an awareness of the problems of computer implementation and experimental procedures.  Nevertheless, there is a need for a solid background in applied mathematics.""",
          "id": "1809",
          "title": "Numerical Analysis in a Ph.D. Computer Science Program",
          "author": [
            "Parter, S. V."
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "UGa0544BNHAuLhkC0xU6",
        "_score": 1.8470237,
        "_source": {
          "date": "1970-01",
          "summary": "Time-shared, multiprogrammed, and overlayed batch systems frequently require segmentation of computer programs into discrete portions. These program portions are transferred between executable and peripheral storage whenever necessary; segmentation of program s in a manner that  reduces the frequency of such transfers is the subject of this paper.  Segmentation techniques proposed by C. V. Ramamoorthy are subject to limitations that arise when the preferred segment size is not compatible with the physical restrictions imposed by the available computing equipment.  A generalization of Ramamoorthy's suggestions is made in order to allow their application when circumstances are other than ideal.",
          "id": "2130",
          "title": "Automatic Segmentation of Cyclic Program Structures Based on Connectivity and Processor Timing",
          "author": [
            "Lowe, T. C."
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "EGa0544BNHAuLhkC0xQ6",
        "_score": 1.7301204,
        "_source": {
          "date": "1969-12",
          "summary": """The operation of "folding" a program into the available memory is discussed.  Measurements by Brown et al. and by Nelson on an automatic folding mechanism of simple design, a demand paging unit built at the IBM Research Center by Belady, Nelson, O'Neil, and others, permitting its quality to be compared with that of manual folding, are discussed, and it is shown that given some care in use the unit performs satisfactorily under the conditions tested, even though it is operating across a memory-to-storage interface with a very large speed difference.  The disadvantages of prefolding, which is required when the folding is manual, are examined, and a number of the important troubles which beset computing today are shown to arise from, or be aggravated by, this source.  It is concluded that a folding mechanism will probably become a normal part of most computing systems.""",
          "id": "1810",
          "title": """Is Automatic "Folding" of Programs Efficient Enough To Displace Manual?""",
          "author": [
            "Sayre, D."
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "A2a0544BNHAuLhkC0xQ6",
        "_score": 1.6233125,
        "_source": {
          "date": "1969-12",
          "id": "1797",
          "title": "Solution of Linear programs in 0-1 (Algorithm 341 [H])",
          "author": [
            "Proll, L. G."
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "Lma0544BNHAuLhkC0xU6",
        "_score": 1.5039517,
        "_source": {
          "date": "1970-03",
          "summary": "The M & N procedure is an improvement to the mini-max backing-up procedure widely used in computer program for game-playing and other purposes.  It is based on the principle that it is desirable to have many options when making decisions in the face of uncertainty.  The mini-max procedure assigns to a MAX (MIN) node the value of the highest (lowest) valued successor to that node. The M & N procedure assigns to a MAX (MIN) node some function of the M (N) highest (lowest) valued successors.  An M & N procedure was written in LISP to play the game of kalah, and it was demonstrated that the M & N procedure is significantly superior to the mini-max procedure.  The statistical significance of important conclusions is given. Since information on statistical significance has often been lacking in papers on computer experiments in the artificial intelligence field, these experiments can perhaps serve as a model for future work.",
          "id": "2096",
          "title": "Experiments with the M & N Tree-Searching Program",
          "author": [
            "Slagle, J. R.",
            "Dixon, J. K."
          ]
        }
      },
      {
        "_index": "cacm_english",
        "_id": "Dma0544BNHAuLhkC0xU6",
        "_score": 1.4472508,
        "_source": {
          "date": "1970-05",
          "summary": "Operations on vectors, matrices, and higher dimensional storage arrays are standard features of most compilers today.  The elements of such structures are usually restricted to be scalars.  For many sophisticated applications this restriction can impose cumbersome data representations. An efficient system has been devised and implemented which allows the elements of multidimensional arrays to themselves be multidimensional arrays.  This system was developed from a storage structure in which the location, length, and content of each array is described by a codeword which can be interpreted by the system.  Code words may describe arrays containing more codewords, thus providing all needed descriptive information for hyperstructures of any form.",
          "id": "2064",
          "title": "Operations on Generalized Arrays with the Genie Compiler",
          "author": [
            "Sitton, G. A."
          ]
        }
      }
    ]
  }
}
```
