<!-- omit from toc -->
# Chapter Five: Developing Search Applications

- [Introduction to Developing Search Applications](#introduction-to-developing-search-applications)
- [Highlighting Search Terms](#highlighting-search-terms)
  - [Examples Highlighting Search Terms](#examples-highlighting-search-terms)
    - [Emphasize Tags (Default)](#emphasize-tags-default)
      - [Result](#result)
    - [Bold Tags](#bold-tags)
      - [Result](#result-1)
- [Sorting Search Results](#sorting-search-results)
  - [Examples Sorting Search Results](#examples-sorting-search-results)
- [Paginating Search Results](#paginating-search-results)
  - [Example Paginating Search Results](#example-paginating-search-results)
- [**(LAB)** Highlighting, Sorting, and Paginating Search Results in Elasticsearch 7.13](#lab-highlighting-sorting-and-paginating-search-results-in-elasticsearch-713)
  - [Objectives](#objectives)
  - [Solution](#solution)
    - [Query 1:](#query-1)
    - [Query 2:](#query-2)
    - [Query 3:](#query-3)
- [Defining Index Aliases](#defining-index-aliases)
  - [Managing Aliases through \_aliases API](#managing-aliases-through-_aliases-api)
    - [Creating an Alias](#creating-an-alias)
    - [Removing an Alias](#removing-an-alias)
  - [Managing Aliases through Templates](#managing-aliases-through-templates)
    - [Create Alias in Component Template](#create-alias-in-component-template)
    - [Create Index Template composed of Component Template](#create-index-template-composed-of-component-template)
- [**(LAB)** Leveraging Index Aliases in Elasticsearch 7.13](#lab-leveraging-index-aliases-in-elasticsearch-713)
  - [Objectives](#objectives-1)
  - [Solution](#solution-1)
    - [Alias 1](#alias-1)
    - [Alias 2](#alias-2)
    - [Alias 3](#alias-3)
      - [Component Template](#component-template)
      - [Index Template](#index-template)
- [Defining Search Templates](#defining-search-templates)
  - [Creating a Search Template](#creating-a-search-template)
  - [Testing a Search Template](#testing-a-search-template)
  - [Using a Search Template](#using-a-search-template)
- [**(LAB)** Using Search Templates in Elasticsearch 7.13](#lab-using-search-templates-in-elasticsearch-713)
  - [Objectives](#objectives-2)
  - [Solution](#solution-2)
    - [Create 'flights\_template'](#create-flights_template)
    - [Testing 'flights\_template'](#testing-flights_template)
    - [Using 'flights\_template'](#using-flights_template)
- [**(QUIZ)** Developing Search Applications in Elasticsearch 7.13](#quiz-developing-search-applications-in-elasticsearch-713)

## Introduction to Developing Search Applications

* Highlighting Search Terms
* Sorting Search Results
* Paginating Search Results
* Defining Index Aliases
* Defining Search Templates

## Highlighting Search Terms

_Emphasize the search term(s)._

Customize the way Elasticsearch highlights matched search term(s).

### Examples Highlighting Search Terms

#### Emphasize Tags (Default)

        GET ecommerce/_search
        {
            "query": {
                "match": {
                "products.product_name": "sweatshirt"
                }
            },
            "highlight": {
                "fields": {
                "products.product_name": {}
                }
            }
        }

##### Result
        ...,
        "highlight" : {
            "products.product_name" : [
                "<em>Sweatshirt</em> - black",
                "<em>Sweatshirt</em> - oliv"
            ]
        }

_This query will return a "highlight" section after each "hit", and wrap the queried content in "`<em></em>`" tags._

#### Bold Tags

        GET ecommerce/_search
        {
            "query": {
                "match": {
                "products.product_name": "sweatshirt"
                }
            },
            "highlight": {
                "pre_tags": "<b>",
                "post_tags": "</b>", 
                "fields": {
                "products.product_name": {}
                }
            }
        }

##### Result

        ...,
        "highlight" : {
          "products.product_name" : [
            "<b>Sweatshirt</b> - black",
            "<b>Sweatshirt</b> - oliv"
          ]
        }

_This query will return a "highlight" section after each "hit", and wrap the queried content in "`<b></b>`" tags._

## Sorting Search Results

_Meaningfully organize search results._

* Sort Values
  * Sort on the values of a hierarchyof specified fields.
* Sort Order
  * Determine whether it should be ascending or descending.
* Sort Mode
  * For multi-value fields, sort by min, max, sum, average, or median.

### Examples Sorting Search Results

        GET ecommerce/_search
        {
            "query": {
                "match": {
                    "products.product_name": "sweatshirt"
                }
            },
            "sort": [
                {
                    "customer_last_name.keyword": {}
                },
                {
                    "customer_first_name.keyword": {}
                },
                {
                    "products.price": {
                        "mode": "sum"
                    }
                }
            ]
        }

_This query will sort the results **Ascending** (Default) for the Lastname & Firstname, aswell as sorting them (**Descending -> default for numeric fields**) by the sum of the product prices._

_If you want to sort using the meta fields (_'variable'), just use the name of the field, no need for the brackets._

## Paginating Search Results

_Split search results into discrete pages._

Having too many search results displayed at once can be overwhelming. More often than not, what you actually care about is on the first page.

### Example Paginating Search Results

        GET ecommerce/_search
        {
            "from": 20, 
            "size": 20, 
            "query": {
                "match": {
                "products.product_name": "sweatshirt"
                }
            }
        }

_This query will return the 'second page' of results, it is starting 'from' the 20th hit until 20 more. Keep in mind, the "from" tag should be a multiple of the "size" tag!_

## **(LAB)** Highlighting, Sorting, and Paginating Search Results in Elasticsearch 7.13

### Objectives

The following sample queries should be crafted on the **sharespeare** dataset:

**Query 1:**
* Matches all documents where the word "**life**" appears in the **text_entry** field.
* Paginates the search results with **25** results per page.
* Highlights the matched term(s) with the HTML **`<em>`** and **`</em>`** tags.
* Sorts the results by relevancy score in decending order.


**Query 2:**
* Matches all documents where the word "**death**" appears in the **text_entry** field.
* Paginates the search results with **25** results per page.
* Highlights the matched term(s) with the HTML **`<b>`** and **`</b>`** tags.
* Sorts the results first by **play_name.keyword** then **speaker.keyword** in ascending order, followed by relevancy **_score** in descending order.

**Query 3:**
* Matches all documents where the **speaker.keyword** is "**THESEUS**" and the word "**death**" or "**life**" appears in the **text_entry** field.
* Paginates the search results with **10** results per page.
* Highlights the matched term(s) with the HTML **`<b>`** and **`</b>`** tags and the HTML **`<mark>`** and **`</mark>`** tags.
* Sorts the results by **line_id** in ascending order.

### Solution

#### Query 1:

    GET shakespeare/_search
    {
        "from": 0, 
        "size": 25, 
        "query": {
            "match": {
                "text_entry": "life"
            }
        },
        "highlight": {
            "fields": {
                "text_entry": {}
            }
        },
        "sort": [
            "_score"
        ]
    }

#### Query 2:

    GET shakespeare/_search
    {
        "from": 0,
        "size": 25,
        "query": {
            "match": {
                "text_entry": "death"
            }
        },
        "highlight": {
            "pre_tags": "<b>",
            "post_tags": "</b>", 
            "fields": {
                "text_entry": {}
            }
        },
        "sort": [
            {
            "play_name.keyword": {
                "order": "asc"
            }
            },
            {
            "speaker.keyword": {
                "order": "asc"
            }
            },
            "_score"
        ]
    }

#### Query 3:

    GET shakespeare/_search
    {
        "from": 0,
        "size": 10,
        "query": {
            "bool": {
                "must": [
                    {
                        "term": {
                            "speaker.keyword": {
                                "value": "THESEUS"
                            }
                        }
                    }
                ],
                "should": [
                    {
                        "match": {
                            "text_entry": "life"
                        }
                    },
                    {
                        "match": {
                            "text_entry": "death"
                        }
                    }
                ],
                "minimum_should_match": 1
            }
        },
        "highlight": {
            "pre_tags": "<b><mark>", 
            "post_tags": "</mark></b>", 
            "fields": {
                "speaker.keyword": {},
                "text_entry": {}
            }
        },
        "sort": [
            {
                "line_id": {
                    "order": "asc"
                }
            }
        ]
    }

## Defining Index Aliases

_Simplify data referencing._

Search across multiple indices with ease by assigning meaningful aliases to indices manully or automatically using index or component templates.

### Managing Aliases through _aliases API

#### Creating an Alias

        POST _aliases
        {
            "actions": [
                {
                    "add": {
                        "index": "kibana_sample_data_ecommerce",
                        "alias": "sample_data"
                    }
                }
            ]
        }

#### Removing an Alias

        POST _aliases
        {
            "actions": [
                {
                    "remove": {
                        "index": "kibana_sample_data_ecommerce",
                        "alias": "sample_data"
                    }
                }
            ]
        }

### Managing Aliases through Templates

#### Create Alias in Component Template

        PUT _component_template/courses
        {
            "template": {
                "aliases": {
                    "courses": {}
                }
            }
        }

#### Create Index Template composed of Component Template

        PUT _index_template/elastic_stack
        {
            "index_patterns": ["es-*"],
            "composed_of": ["courses"],
            "template": {
                "aliases": {
                    "elastic_stack": {},
                    "elasticsearch_101": {
                        "filter": {
                            "term": {
                                "course_name.keyword": "Elasticsearch 101"
                            }
                        }
                    }
                }
            }
        }

_This will add all indices with "es-" to the 'elastic_stack' alias, and add the documents where 'course_name.keyword' is 'Elasticsearch 101' to the alias 'elasticsearch_101'._

## **(LAB)** Leveraging Index Aliases in Elasticsearch 7.13

### Objectives

**Alias 1:**
The alias **sample_data** should be applied to the **kibana_sample_data_ecommerce**, **kibana_sample_data_flights**, and **kibana_sample_data_logs** indices.

**Alias 2:**
The alias **othello** should filter documents where the **play_name.keyword** field is "**Othello**" and should be applied to the **shakespeare** index.

**Alias 3:**
The component template **sample_data** should be configured to apply the **sample_data** alias.

The **sample_data_bank** index should:
* Match indices that start with "**accounts-**".
* Be composed of the **sample_data** component template.
* Include the **bank** alias.
* Be configured to create indices with **1** primary and **0** replica shards.

### Solution

#### Alias 1

        POST _aliases
        {
            "actions": [
                {
                    "add": {
                        "index": "kibana_sample_data_ecommerce",
                        "alias": "sample_data"
                    }
                },
                {
                    "add": {
                        "index": "kibana_sample_data_flights",
                        "alias": "sample_data"
                    }
                },
                {
                    "add": {
                        "index": "kibana_sample_data_logs",
                        "alias": "sample_data"
                    }
                }
            ]
        }

#### Alias 2

        POST _aliases
        {
            "actions": [
                {
                    "add": {
                        "index": "shakespeare",
                        "alias": "othello",
                        "filter": {
                            "term": {
                                "play_name.keyword": "Othello"
                            }
                        }
                    }
                }
            ]
        }

#### Alias 3

##### Component Template

        PUT _component_template/sample_data
        {
            "template": {
                "aliases": {
                    "sample_data": {}
                }
            }
        }

##### Index Template

        PUT _index_template/sample_data_bank
        {
            "index_patterns": ["accounts-*"],
            "composed_of": ["sample_data"],
            "template": {
                "aliases": {
                    "bank": {}
                },
                "settings": {
                    "number_of_shards": 1,
                    "number_of_replicas": 0
                }
            }
        }

## Defining Search Templates

_Reusable parameterized queries._

When using Elasticsearch as a backend for a search application, search templates enable you to pass user inputs to your search without exposing the whole query.

### Creating a Search Template

        PUT _scripts/ecommerce_template
        {
            "script": {
                "lang": "mustache",
                "source": {
                    "from": "{{from}}{{^from}}0{{/from}}",
                    "size": "{{size}}{{^size}}15{{/size}}",
                    "query": {
                        "match": {
                            "products.product_name": "{{product_name}}"
                        }
                    },
                    "highlight": {
                        "pre_tags": "<b>",
                        "post_tags": "</b>",
                        "fields": {
                            "products.product_name": {}
                        }
                    },
                    "sort": [
                        {
                            "customer_last_name.keyword": {}
                        },
                        {
                            "customer_first_name.keyword": {}
                        },
                        {
                            "products.price": {
                                "mode": "avg",
                                "order": "asc"
                            }
                        }
                    ]
                }
            }
        }

_The '{{}}' indicates a parameter which can later be filled in with user inputs, the '{{^from}}0{{/from}}' indicates the default value for given parameter._

### Testing a Search Template

        POST _render/template
        {
            "id": "ecommerce_template",
            "params": {
                "from": 0,
                "size": 10,
                "product_name": "boots"
            }
        }

_This will show the query how you would normally use queries, here you can check if the parameters are used correctly._

### Using a Search Template

        GET ecommerce/_search/template
        {
            "id": "ecommerce_template",
            "params": {
                "product_name": "belt"
            }
        }

_**Note:** There was no 'from' nor 'size' parameter declared so the set default values will be used._

## **(LAB)** Using Search Templates in Elasticsearch 7.13

### Objectives

Create the **flights_template** search template as follows:

* Paginate the search results with a default page sizeof **25** and display the first page by default.
* Perform a case-insensitive search on the **Carrier** field with the **carrier** parameter that is defaulted to search for all carriers.
* Perform a case-insensitive search on the **OriginCityName** field with the **origin** parameter that is defaulted to search for all origin cities.
* Perform a case-insensitive search on the **DestCityName** field with the **destination** parameter that is defaulted to search for all destination cities.
* Sort the results by the **AvgTicketPrice** in **descending** order.

Render the template without any parameter to test the query structure and default values. Then, perform some test searches using the template.

For example, try searching for flights by **Logstash Airways** from **Berlin** to **New York** with a results size of **10** per page.

### Solution

#### Create 'flights_template'

        PUT _scripts/flights_template
        {
            "script": {
                "lang": "mustache",
                "source": {
                    "from": "{{from}}{{^from}}0{{/from}}",
                    "size": "{{size}}{{^size}}25{{/size}}",
                    "query": {
                        "bool": {
                            "must": [
                                {
                                    "wildcard": {
                                        "Carrier": {
                                            "value": "{{carrier}}{{^carrier}}*{{/carrier}}",
                                            "case_insensitive": true
                                        }
                                    }
                                },
                                {
                                    "wildcard": {
                                        "OriginCityName": {
                                            "value": "{{origin}}{{^origin}}*{{/origin}}",
                                            "case_insensitive": true
                                        }
                                    }
                                },
                                {
                                    "wildcard": {
                                        "DestCityName": {
                                            "value": "{{destination}}{{^destination}}*{{/destination}}",
                                            "case_insensitive": true
                                        }
                                    }
                                }
                            ]
                        }
                    },
                    "sort": [
                        {
                            "AvgTicketPrice": {
                                "order": "desc"
                            }
                        }
                    ]
                }
            }
        }

#### Testing 'flights_template'

        GET _render/template
        {
            "id": "flights_template"
        }

#### Using 'flights_template'

        GET flights/_search/template
        {
            "id": "flights_template",
            "params": {
                "size": 10,
                "carrier": "Logstash Airways",
                "origin": "Berlin",
                "destination": "New York"
            }
        }

## **(QUIZ)** Developing Search Applications in Elasticsearch 7.13

* **Which mechanism would you use to emphasize a search term in search results?**
  * Highlighting
* **Which of the following enables the reuse of a parameterized search query?**
  * Search template
* **Which mechanism would you use to limit the set of search results that are returned at one time?**
  * Pagination
* **Which mechanism would you use to reorder the search results of a query?**
  * Sorting
* **Which of the following would you use to reference one or more indices with a single custom name?**
  * Alias