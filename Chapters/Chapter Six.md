<!-- omit from toc -->
# Chapter Six: Data Processing

- [Introduction to Data Processing](#introduction-to-data-processing)
- [Explicitly Mapping Fields](#explicitly-mapping-fields)
  - [Examples](#examples)
    - [Create Explicit Mapping](#create-explicit-mapping)
- [Dynamically Mapping Fields](#dynamically-mapping-fields)
  - [Examples](#examples-1)
    - [Create Dynamic Mapping](#create-dynamic-mapping)
- [Defining a Custom Analyzer](#defining-a-custom-analyzer)
  - [Analyzer Anatomy](#analyzer-anatomy)
  - [Examples](#examples-2)
    - [Create Custom Analizer](#create-custom-analizer)
- [Defining Multi-Fields](#defining-multi-fields)
  - [Examples](#examples-3)
- [**(LAB)** Defining Custom Analyzers, Multi-Fields, and Mappings in Elasticsearch 7.13](#lab-defining-custom-analyzers-multi-fields-and-mappings-in-elasticsearch-713)
  - [Objectives](#objectives)
  - [Solution](#solution)
    - [string\_as\_keywords](#string_as_keywords)
    - [social\_media\_analyzer](#social_media_analyzer)
    - [twitter\_template](#twitter_template)
- [Reindexing Documents](#reindexing-documents)
  - [Examples](#examples-4)
    - [Local To Local](#local-to-local)
    - [Local To Local with query](#local-to-local-with-query)
    - [Remote To Local](#remote-to-local)
      - [Whitelist remote source](#whitelist-remote-source)
- [Updating Documents](#updating-documents)
  - [Examples](#examples-5)
    - [Update an individual document](#update-an-individual-document)
    - [Update documents with query](#update-documents-with-query)
- [**(LAB)** Reindexing and Updating Documents in Elasticsearch 7.13](#lab-reindexing-and-updating-documents-in-elasticsearch-713)
  - [Objectives](#objectives-1)
  - [Solution](#solution-1)
    - [Fixing the Typo](#fixing-the-typo)
    - [Copying Index](#copying-index)
- [Defining Ingest Pipelines](#defining-ingest-pipelines)
  - [Examples](#examples-6)
    - [Create Ingest Pipeline](#create-ingest-pipeline)
    - [Use Ingest Pipeline](#use-ingest-pipeline)
- [**(LAB)** Leveraging Ingest Pipelines in Elasticsearch 7.13](#lab-leveraging-ingest-pipelines-in-elasticsearch-713)
  - [Objectives](#objectives-2)
  - [Solution](#solution-2)
    - [migrate\_accounts](#migrate_accounts)
    - [Copying Index](#copying-index-1)
- [Handling Nested Arrays of Objects](#handling-nested-arrays-of-objects)
  - [Examples](#examples-7)
    - [Creating Index Template](#creating-index-template)
    - [Modify earlier created Component Template 'product\_review\_mappings'](#modify-earlier-created-component-template-product_review_mappings)
    - [Post new document with Array of Objects](#post-new-document-with-array-of-objects)
    - [Search using the 'nested' query type](#search-using-the-nested-query-type)
- [**(LAB)** Maintaining Nested Arrays of Objects in Elasticsearch 7.13](#lab-maintaining-nested-arrays-of-objects-in-elasticsearch-713)
  - [Objectives](#objectives-3)
  - [Solution](#solution-3)
    - [Create 'ecommerce\_fixed'](#create-ecommerce_fixed)
    - [Reindex Data](#reindex-data)
    - [Nested Search](#nested-search)
- [**(QUIZ)** Data Processing in Elasticsearch 7.13](#quiz-data-processing-in-elasticsearch-713)

## Introduction to Data Processing

* Explicitly Mapping Fields
* Dynamically Mapping Fields
* Defining a Custom Analyzer
* Defining Multi-Fields
* Reindexing Documents
* Updating Documents
* Defining Ingest Pipelines
* Handling Nested Arrays of Objects

## Explicitly Mapping Fields

_Manually define the fields and data types._

* Determine which string fields are analyzed.
* Define which numbers should be integers, floats, percents, etc.
* Customize the date format for date fields.

### Examples

#### Create Explicit Mapping

        PUT _component_template/product_review_mappings
        {
            "template": {
                "mappings": {
                    "properties": {
                        "product_id": {
                            "type": "long"
                        },
                        "reviews": {
                            "properties": {
                                "first_name": {
                                    "type": "keyword"
                                },
                                "last_name": {
                                    "type": "keyword"
                                },
                                "review": {
                                    "type": "text",
                                    "analyzer": "english"
                                }
                            }
                        }
                    }
                }
            }
        }

_This can also be done directly on the index, but for ease of use, create a template._

## Dynamically Mapping Fields

_Automatically add new fields and data types._

* New fields not already in the index will be added automatically (Default).
* Data detectors will automatically determine data types for new fields.
* Dynamic templates allow for the customization of dynamic mapping behavior.

### Examples

#### Create Dynamic Mapping

        PUT _component_template/integers_as_floats
        {
            "template": {
                "mappings": {
                    "dynamic_templates": [
                        {
                            "integers_as_floats": {
                                "match_mapping_type": "long",
                                "mapping": {
                                    "type": "double"
                                }
                            }
                        }
                    ]
                }
            }
        }

_This will overwrite the default behavior of Elastic in which they want to define values as integers (long), this will make them a float ('double'). **BUT** an Explicit mapping will **always** overwrite a Dynamic mapping!_

## Defining a Custom Analyzer

### Analyzer Anatomy

Analyzers allow Elasticsearch to return relevant results rather than exclusively exact matches.

* Character Filter
  * Transforms a string of text by adding, removing, or changing characters.
* Tokenizer
  * Converts a string of text into an array of individual tokens.
* Token Filters
  * Transforms a token by adding, removing, or changing it.

### Examples

#### Create Custom Analizer

        ...,
        "settings": {
            "analysis": {
                "analyzer": {
                    "user_reviews": {
                        "type": "custom",
                        "tokenizer": "classic",
                        "char_filter": ["social_acronyms"],
                        "filters": ["lowercase", "english_stop"]
                    }
                },
                "char_filter": {
                    "social_acronyms": {
                        "type": "mapping",
                        "mappings": [
                            "imo => in my opinion",
                            "b2b => business to business",
                            "b2c => business to consumer",
                            "roi => return on investment",
                            "afaik => as far as I know"
                        ]
                    }
                },
                "filter": {
                    "english_stop": {
                        "type": "stop",
                        "stopwords": ["_english_"],
                        "ignore_case": true
                    }
                }
            }
        }

_You can add this body to any PUT or POST request, most commonly used in component templates._

## Defining Multi-Fields

_Index the same field in multiple ways._

Multi-Fields allow you to index fields multiple times. This way, you can have one field mapped in different ways depending on the potential use cases.

* Map a string as both an analyzed **text** field and non analyzed **keyword** field.
* Map an analyzed **text** field with multiple different analyzers.

### Examples

        PUT _component_template/strings
        {
            "template": {
                "mappings": {
                    "dynamic_templates": [
                        {
                            "strings": {
                                "match_mapping_type": "string",
                                "mapping": {
                                    "type": "text",
                                    "analyzer": "standard",
                                    "fields": {
                                        "keyword": {
                                            "type": "keyword"
                                        }
                                    }
                                }
                            }
                        } 
                    ]
                }
            }
        }

_To define a Multi-Field, it is the same process as a 'Dynamic Mapping' but with the addition of 'fields'. This example will map normal strings using the standard analyzer, but will also create a keyword variant with the exact value in it._

## **(LAB)** Defining Custom Analyzers, Multi-Fields, and Mappings in Elasticsearch 7.13

### Objectives

**string_as_keywords** component template:
* Dynamically maps **string** fields as the **keyword** datatype.
* Only indexes the first **256** characters.

**social_media_analyzer** component template:
* Creates a custom analyzer called **social_media** that uses the **classic** tokenizer.
* Uses the **lowercase** filter and a custom **english_stop** filter. The **english_stop** filter should remove English stop words.
* Uses a custom **emoticons** mapping character filter.

**twitter_template** index template:
* Matches all indices that start with "**twitter-**".
* Composed of **strings_as_keywords** and **social_media_analyzer**.
* Maps the **tweet** field as an analyzed string field with the **social_media** analyzer.
* Maps a **tweet.keyword** multi-field as type **keyword** and only indexes the first **280** characters.
* Sets the number of shards to **1** and replicas to **0**.

### Solution

#### string_as_keywords

        PUT _component_template/strings_as_keywords
        {
            "template": {
                "mappings": {
                    "dynamic_templates": [
                        {
                            "strings_as_keywords": {
                                "match_mapping_type": "string",
                                "mapping": {
                                    "type": "keyword",
                                    "ignore_above": 256
                                }
                            }
                        }
                    ]
                }
            }
        }


#### social_media_analyzer

        PUT _component_template/social_media_analyzer
        {
            "template": {
                "settings": {
                    "analysis": {
                        "analyzer": {
                            "social_media": {
                                "type": "custom",
                                "tokenizer": "classic",
                                "char_filter": ["emoticons"],
                                "filter": ["lowercase", "english_stop"]
                            }
                        },
                        "char_filter": {
                            "emoticons": {
                                "type": "mapping",
                                "mappings": [
                                ":) => happy",
                                ":D => laughing",
                                ":( => sad",
                                ":') => crying",
                                ":O => surprised",
                                ";) => winking"
                                ]
                            }
                        },
                        "filter": {
                            "english_stop": {
                                "type": "stop",
                                "stopwords": ["_english_"]
                            }
                        }
                    }
                }
            }
        }

#### twitter_template

        PUT _index_template/twitter_template
        {
            "index_patterns": ["twitter-*"],
            "composed_of": ["social_media_analyzer", "strings_as_keywords"],
            "template": {
                "mappings": {
                    "properties": {
                        "tweet": {
                            "type": "text",
                            "analyzer": "social_media",
                            "fields": {
                                "keyword": {
                                "type": "keyword",
                                "ignore_above": 280
                                }
                            }
                        }
                    }
                },
                "settings": {
                    "number_of_shards": 1,
                    "number_of_replicas": 0
                }
            }
        }

## Reindexing Documents

* Source
  * Specify the local or remote source index and filter the source with a query.
* Destination
  * Specify the local index to reindex into.
* Pipeline
  * Specify an ingest pipeline to modify the data in flight before it's written to the destination index.

### Examples

#### Local To Local

    POST _reindex
        {
        "source": {
            "index": "flights"
        },
        "dest": {
            "index": "flights_reindexed"
        }
    }

#### Local To Local with query

        POST _reindex
        {
            "source": {
                "index": "flights",
                "query": {
                    "term": {
                        "Carrier": {
                            "value": "Kibana Airlines"
                        }
                    }
                }
            },
            "dest": {
                "index": "kibana_airlines"
            }
        }

_This will only reindex documents where the query result matches._

#### Remote To Local

        POST _reindex
        {
            "source": {
                "remote": {
                    "host": "http://172.31.107.70:9200",
                    "username": "elastic",
                    "password": "elastic_acg"
                },
                "index": "flights",
                "query": {
                    "term": {
                        "Carrier": {
                            "value": "Logstash Airways"
                        }
                    }
                }
            },
            "dest": {
                "index": "logstash_airways"
            }
        }

_This before you can run this, the remote source IP address has to be whitelisted on the Elastic node!_

##### Whitelist remote source

1) SSH into the machine (`ssh <username>@<publicIP>`)
2) Become root (`sudo su -`)
3) Navigate to Elasticsearch directory (`cd /etc/elasticsearch/`)
4) Open elasticsearch.yml (`vim elasticsearch.yml`)
5) Add the following variable somewhere in this file: `reindex.remote.whitelist: <remotePrivateIPHere>:9200`
6) Restart Elasticsearch (`systemctl restart elasticsearch`)

## Updating Documents

_Update by document ID or query._

The **update** and **update_by_query** APIs enable the modification of documents after they have already been indexed.

* Update documents to pick up mapping changes.
* Update documents with a script to change their source values.
* Update documents with an ingest pipeline.

### Examples

#### Update an individual document

        POST earthquakes/_update/<docID>
        {
            "doc": {
                "Type": "Natural"
            }
        }

#### Update documents with query

        POST earthquakes/_update_by_query
        {
            "script": {
                "source": "ctx._source.Type = 'Natural'",
                "lang": "painless"
            },
            "query": {
                "term": {
                    "Type": {
                        "value": "Earthquake"
                    }
                }
            }
        }

## **(LAB)** Reindexing and Updating Documents in Elasticsearch 7.13

### Objectives

* Fix the Typo for the King Lear Play
  * On the **shakespeare** dataset, the **play_name** 'King Lear' is misspelled as 'King Leer'
* Copy the **Shakespeare Index** from the **es1** Node the the **Shakespeare** Node.

### Solution

#### Fixing the Typo

        POST shakespeare/_update_by_query
        {
            "script": {
                "source": "ctx._source.play_name = 'King Lear'",
                "lang": "painless"
            },
            "query": {
                "term": {
                    "play_name.keyword": {
                        "value": "King Leer"
                    }
                }
            }
        }

#### Copying Index

        POST _reindex
        {
            "source": {
                "remote": {
                    "host": "http://<privateIP>:9200",
                    "username": "<username>",
                    "password": "<password>"
                },
                "index": "shakespeare"
            },
            "dest": {
                "index": "shakespeare"
            }
        }

## Defining Ingest Pipelines

_Process and enrich your data._

* Pipelines use processors to perform some action on a document.
* Processors are executed in order.
* Ingest pipelines can be used in the **update_by_query** and **reindex** APIs.

### Examples

#### Create Ingest Pipeline

        PUT _ingest/pipeline/earthquakes_refactor
        {
            "description": "refactoring the earthquakes dataset",
            "processors": [
                {
                    "remove": {
                        "field": ["Latitude", "Longitude"]
                    }
                },
                {
                    "set": {
                        "field": "timestamp",
                        "value": "{{_source.Date}} {{_source.Time}}"
                    }
                },
                {
                    "date": {
                        "field": "timestamp",
                        "formats": ["MM/dd/yyyy HH:mm:ss"]
                    }
                },
                {
                    "remove": {
                        "field": ["Date", "Time", "timestamp"]
                    }
                }
            ]
        }

_This pipeline will remove the 'Longitude' & 'Latitude' field, set a temperary timestamp field, format the new timestamp field to '@timestamp', and remove the temparary 'timestamp' field as well as 'Date' & 'Time'._

#### Use Ingest Pipeline

        POST _reindex
        {
            "source": {
                "index": "earthquakes"
            },
            "dest": {
                "index": "earthquakes_refactored",
                "pipeline": "earthquakes_refactor"
            }
        }


## **(LAB)** Leveraging Ingest Pipelines in Elasticsearch 7.13

### Objectives

* Create the **migrate_accounts** Ingest Pipeline
  * Remove the **account_number** field.
  * Create the **fullname** field, which is a concatenation of the **firstname** field, a space, and the **lastname** field.
  * Add a **5% bonus** to each account's balance and keep track of applied bonuses with the **bonus_pct** field.
* Copy the accounts Index from the **accounts1** Node to the **accounts2** Node with the **migrate_accounts** Ingest Pipeline

### Solution

#### migrate_accounts

        PUT _ingest/pipeline/migrate_accounts
        {
            "description": "refactoring accounts dataset and adding bonus of 5%",
            "processors": [
                {
                    "remove": {
                        "field": "account_number"
                    }
                },
                {
                    "set": {
                        "field": "fullname",
                        "value": "{{_source.firstname}} {{_source.lastname}}"
                    }
                },
                {
                    "script": {
                        "lang": "painless",
                        "source": """
                        ctx.balance += ctx.balance * 0.05;
                        if (ctx.bonus_pct == null) {
                        ctx.bonus_pct = 5;
                        } else {
                        ctx.bonus_pct += 5;
                        }
                        """
                    }
                }
            ]
        }

#### Copying Index

        POST _reindex
        {
            "source": {
                "remote": {
                    "host": "http://10.0.1.101:9200",
                    "username": "elastic",
                    "password": "elastic_acg"
                },
                "index": "accounts"
            },
            "dest": {
                "index": "accounts",
                "pipeline": "migrate_accounts"
            }
        }

## Handling Nested Arrays of Objects

_Maintain the independence of objects in an array._

* Arrays of objects are flattened, thereby losing the relationships which values belong to which object.
* The **nested** data type maintains the independence of each object in an array.
* The **nested** query searches object arrays as if each object were a seperate document.

### Examples

#### Creating Index Template

        PUT _index_template/product_reviews
        {
            "index_patterns": ["furniture", "outdoors", "kitchen"],
            "composed_of": ["integers_as_floats", "product_review_mappings"],
            "template": {
                "settings": {
                    "number_of_shards": 1,
                    "number_of_replicas": 0
                }
            }
        }

_Created for easier creation of documents._

#### Modify earlier created Component Template 'product_review_mappings'

        PUT _component_template/product_review_mappings
        {
            "template": {
                "mappings": {
                    "properties": {
                        "product_id": {
                            "type": "long"
                        },
                        "reviews": {
                            "type": "nested", 
                            "properties": {
                                "first_name": {
                                    "type": "keyword"
                                },
                                "last_name": {
                                "type": "keyword"
                                },
                                "review": {
                                    "type": "text",
                                    "analyzer": "user_reviews",
                                    "fields": {
                                        "standard": {
                                            "type": "text",
                                            "analyzer": "standard"
                                        }
                                    }
                                }
                            }
                        }
                    }
                },
                "settings": {
                    "analysis": {
                        "analyzer": {
                            "user_reviews": {
                                "type": "custom",
                                "tokenizer": "classic",
                                "char_filter": ["social_acronyms"],
                                "filters": ["lowercase", "stop"]
                            }
                        },
                        "char_filter": {
                            "social_acronyms": {
                                "type": "mapping",
                                "mappings": [
                                    "imo => in my opinion",
                                    "b2b => business to business",
                                    "b2c => business to consumer",
                                    "roi => return on investment",
                                    "afaik => as far as I know"
                                ]
                            }
                        }
                    }
                }
            }
        }

_The only thing that changed is under 'reviews', there is a 'type' added called 'nested'._

#### Post new document with Array of Objects

        POST furniture/_doc
        {
            "product_id": 1,
            "product_name": "Walnut Coffee Table",
            "review": [
                {
                    "first_name": "Myles",
                    "last_name": "Young",
                    "rating": 5,
                    "review": "Great mid-century-modern coffee table. Well built imo. Should last a few generations."
                },
                {
                    "first_name": "Matthew",
                    "last_name": "Pearson",
                    "rating": 4,
                    "review": "b2b customer here. Shipping took longer than expected. I'm no expert but afaik the table seems well built."
                }
            ]
        }

#### Search using the 'nested' query type

        GET furniture/_search
        {
            "query": {
                "nested": {
                "path": "reviews",
                    "query": {
                        "bool": {
                            "must": [
                                {
                                    "term": {
                                        "reviews.first_name": {
                                            "value": "Myles"
                                        }
                                    }
                                },
                                {
                                    "term": {
                                        "reviews.last_name": {
                                            "value": "Young"
                                        }
                                    }
                                }
                            ]
                        }
                    }
                }
            }
        }

_This 'path' tag is the tag that contains the list of objects, in this case 'reviews'._

## **(LAB)** Maintaining Nested Arrays of Objects in Elasticsearch 7.13

### Objectives

The **ecommerce_fixed** index should be created as follows:

* Contains all the same mappings as the **ecommerce** index with the expectation that the **products** object must be configured to maintain the relationships of the nested array of objects.
* Allocates with **1** primary and **0** replica shards.
* Contains a copy of the **ecommerce** index's data.

Then, perform a **nested** search on the **products** object of the **ecommerce_fixed** index.

A nested query that spans multiple products should no longer match, while a nested query that does not span multiple products should match.

### Solution

#### Create 'ecommerce_fixed'

        PUT ecommerce_fixed
        {
            "mappings" : {
                "properties" : {
                    "category" : {
                        "type" : "text",
                        "fields" : {
                            "keyword" : {
                                "type" : "keyword"
                            }
                        }
                    },
                    "currency" : {
                        "type" : "keyword"
                    },
                    "customer_birth_date" : {
                        "type" : "date"
                    },
                    "customer_first_name" : {
                        "type" : "text",
                        "fields" : {
                            "keyword" : {
                                "type" : "keyword",
                                "ignore_above" : 256
                            }
                        }
                    },
                    "customer_full_name" : {
                        "type" : "text",
                        "fields" : {
                            "keyword" : {
                                "type" : "keyword",
                                "ignore_above" : 256
                            }
                        }
                    },
                    "customer_gender" : {
                        "type" : "keyword"
                    },
                    "customer_id" : {
                        "type" : "keyword"
                    },
                    "customer_last_name" : {
                        "type" : "text",
                        "fields" : {
                            "keyword" : {
                                "type" : "keyword",
                                "ignore_above" : 256
                            }
                        }
                    },
                    "customer_phone" : {
                        "type" : "keyword"
                    },
                    "day_of_week" : {
                        "type" : "keyword"
                    },
                    "day_of_week_i" : {
                        "type" : "integer"
                    },
                    "email" : {
                        "type" : "keyword"
                    },
                    "event" : {
                        "properties" : {
                            "dataset" : {
                                "type" : "keyword"
                            }
                        }
                    },
                    "geoip" : {
                        "properties" : {
                            "city_name" : {
                                "type" : "keyword"
                            },
                            "continent_name" : {
                                "type" : "keyword"
                            },
                            "country_iso_code" : {
                                "type" : "keyword"
                            },
                            "location" : {
                                "type" : "geo_point"
                            },
                            "region_name" : {
                                "type" : "keyword"
                            }
                        }
                    },
                    "manufacturer" : {
                        "type" : "text",
                        "fields" : {
                            "keyword" : {
                                "type" : "keyword"
                            }
                        }
                    },
                    "order_date" : {
                        "type" : "date"
                    },
                    "order_id" : {
                        "type" : "keyword"
                    },
                    "products" : {
                    "type": "nested", 
                        "properties" : {
                            "_id" : {
                                "type" : "text",
                                "fields" : {
                                        "keyword" : {
                                            "type" : "keyword",
                                            "ignore_above" : 256
                                        }
                                    }
                            },
                            "base_price" : {
                                "type" : "half_float"
                            },
                            "base_unit_price" : {
                                "type" : "half_float"
                            },
                            "category" : {
                                "type" : "text",
                                "fields" : {
                                    "keyword" : {
                                        "type" : "keyword"
                                    }
                                }
                            },
                            "created_on" : {
                                "type" : "date"
                            },
                            "discount_amount" : {
                                "type" : "half_float"
                            },
                            "discount_percentage" : {
                                "type" : "half_float"
                            },
                            "manufacturer" : {
                                "type" : "text",
                                "fields" : {
                                    "keyword" : {
                                    "type" : "keyword"
                                    }
                                }
                            },
                            "min_price" : {
                                "type" : "half_float"
                            },
                            "price" : {
                                "type" : "half_float"
                            },
                            "product_id" : {
                                "type" : "long"
                            },
                            "product_name" : {
                                "type" : "text",
                                "fields" : {
                                    "keyword" : {
                                        "type" : "keyword"
                                    }
                                },
                                "analyzer" : "english"
                            },
                            "quantity" : {
                                "type" : "integer"
                            },
                            "sku" : {
                                "type" : "keyword"
                            },
                            "tax_amount" : {
                                "type" : "half_float"
                            },
                            "taxful_price" : {
                                "type" : "half_float"
                            },
                            "taxless_price" : {
                                "type" : "half_float"
                            },
                            "unit_discount_amount" : {
                                "type" : "half_float"
                            }
                        }
                    },
                    "sku" : {
                        "type" : "keyword"
                    },
                    "taxful_total_price" : {
                        "type" : "half_float"
                    },
                    "taxless_total_price" : {
                        "type" : "half_float"
                    },
                    "total_quantity" : {
                        "type" : "integer"
                    },
                    "total_unique_products" : {
                        "type" : "integer"
                    },
                    "type" : {
                        "type" : "keyword"
                    },
                    "user" : {
                        "type" : "keyword"
                    }
                }
            },
            "settings": {
                "number_of_shards": 1,
                "number_of_replicas": 0
            }        
        }

#### Reindex Data

        POST _reindex
        {
            "source": {
                "index": "ecommerce"
            },
            "dest": {
                "index": "ecommerce_fixed"
            }
        }

#### Nested Search

        GET ecommerce/_search
        {
            "query": {
                "bool": {
                    "must": [
                        {
                            "term": {
                                "products.sku": {
                                    "value": "ZO0549605496"
                                }
                            }
                        },
                        {
                            "term": {
                                "products._id": {
                                    "value": "sold_product_584677_19400"
                                }
                            }
                        }
                    ]
                }
            }
        }

_This query **SHOULD** match, which is not good when working with arrays of objects._

        GET ecommerce_fixed/_search
        {
            "query": {
                "nested": {
                "path": "products",
                    "query": {
                        "bool": {
                            "must": [
                                {
                                    "term": {
                                        "products.sku": {
                                            "value": "ZO0549605496"
                                        }
                                    }
                                },
                                {
                                    "term": {
                                        "products._id": {
                                            "value": "sold_product_584677_19400"
                                        }
                                    }
                                }
                            ]
                        }
                    }
                }
            }
        }

_This query **SHOULD NOT** match, which is good when working with arrays of objects._

        GET ecommerce_fixed/_search
        {
            "query": {
                "nested": {
                "path": "products",
                    "query": {
                        "bool": {
                            "must": [
                                {
                                    "term": {
                                        "products.sku": {
                                            "value": "ZO0549605496"
                                        }
                                    }
                                },
                                {
                                    "term": {
                                        "products._id": {
                                            "value": "sold_product_584677_6283"
                                        }
                                    }
                                }
                            ]
                        }
                    }
                }
            }
        }

_This query **SHOULD** match, because it is using the sku & product id of the same product / object._

## **(QUIZ)** Data Processing in Elasticsearch 7.13

* Which of the following is not true about ingest pipelines?
  * Ingest pipelines can only have up to 5 processors.
* What would you use to override the dynamic mapping behavior of Elasticsearch?
  * Dynamic template
* Which datatype must be used in order to maintain the relationships between nested arrays of objects?
  * nested
* What property must be set in the elasticsearch.yml file to enable reindexing from a remote cluster?
  * reindex.remote.whitelist
* What are the 3 primary components of an analyzer?
  * Tokenizer, character filter, token filter
* Which API would you use to update multiple documents at the same time?
  * _update_by_query
* Where in an index would you explicitly define fields and their datatypes?
  * mappings
* What would you use to index an analyzed text field with multiple analyzers?
  * Multi-field