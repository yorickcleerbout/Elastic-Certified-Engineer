<!-- omit from toc -->
# Chapter Four: Aggregating Data

- [Introduction to Aggregating Data](#introduction-to-aggregating-data)
- [Writing Metrics Aggregations](#writing-metrics-aggregations)
  - [Examples Metrics Aggregations](#examples-metrics-aggregations)
    - [Average](#average)
    - [Max](#max)
    - [Stats](#stats)
    - [Cardinality](#cardinality)
- [Writing Bucket Aggregations](#writing-bucket-aggregations)
  - [Examples Bucket Aggregations](#examples-bucket-aggregations)
    - [Date Histogram](#date-histogram)
    - [Terms](#terms)
- [Writing Sub-Aggregations](#writing-sub-aggregations)
  - [Examples Sub-Aggregations](#examples-sub-aggregations)
    - [Date Histogram \& Average](#date-histogram--average)
    - [Date Histogram \& Terms](#date-histogram--terms)
- [**(LAB)** Performing Metrics and Bucket Aggregations in Elasticsearch 7.13](#lab-performing-metrics-and-bucket-aggregations-in-elasticsearch-713)
  - [Objectives](#objectives)
  - [Solution](#solution)
    - [Question 1:](#question-1)
    - [Question 2:](#question-2)
    - [Question 3:](#question-3)
- [Writing Pipeline Aggregations](#writing-pipeline-aggregations)
  - [Parent Pipeline Aggregations](#parent-pipeline-aggregations)
  - [Sibling Pipeline Aggregations](#sibling-pipeline-aggregations)
  - [Examples Parent \& Sibling](#examples-parent--sibling)
- [**(LAB)** Performing Pipeline Aggregations in Elasticsearch 7.13](#lab-performing-pipeline-aggregations-in-elasticsearch-713)
  - [Objectives](#objectives-1)
  - [Solution](#solution-1)
    - [Question 1:](#question-1-1)
    - [Question 2:](#question-2-1)
    - [Question 3:](#question-3-1)
- [**(QUIZ)** Aggregating Data in Elasticsearch 7.13](#quiz-aggregating-data-in-elasticsearch-713)

## Introduction to Aggregating Data

* Writing Metrics Aggregations
* Writing Bucket Aggregations
* Writing Sub-Aggregations
* Writing Pipeline Aggregations

## Writing Metrics Aggregations

_Computes numeric values._

Metrics aggregations are either single or multi-value aggregations that can operate on a variety of **non-analyzed** fields to produce a numerical value.

### Examples Metrics Aggregations

#### Average

        GET earthquakes/_search
        {
            "size": 0, 
            "aggs": {
                "average_magnitude": {
                "avg": {
                    "field": "Magnitude"
                }
                }
            }
        }

_**Note:** the 'size' is set to '0' for a more usefull output, it will only show the aggregations and not a few documents that are used._

#### Max

        GET earthquakes/_search
        {
            "size": 0, 
            "aggs": {
                "max_magnitude": {
                "max": {
                    "field": "Magnitude"
                }
                }
            }
        }

#### Stats

        GET earthquakes/_search
            {
            "size": 0, 
            "aggs": {
                "stats_magnitude": {
                "stats": {
                    "field": "Magnitude"
                }
                }
            }
        }

_This aggregation gives information about the metrics, 'count', 'min', 'max', and so on._

#### Cardinality

        GET earthquakes/_search
        {
            "size": 0, 
            "aggs": {
                "types": {
                "cardinality": {
                    "field": "Type"
                }
                }
            }
        }

_This aggregation will count the unique values for the given field._

## Writing Bucket Aggregations

_Creates buckets of documents._

Establish a criterion to categorize documents into groups or buckets.

### Examples Bucket Aggregations

#### Date Histogram

        GET earthquakes/_search
        {
            "size": 0,
            "aggs": {
                "per_month": {
                "date_histogram": {
                    "field": "Date",
                    "calendar_interval": "month"
                }
                }
            }
        }

_This aggregation will show how many documents are available **grouped** by month._

#### Terms

        GET earthquakes/_search
        {
            "size": 0,
            "aggs": {
                "types": {
                "terms": {
                    "field": "Type",
                    "size": 10
                }
                }
            }
        }

_This aggregation will show how many documents are available **grouped** by a 'term'._

## Writing Sub-Aggregations

_Aggregates per bucket._

Each bucket of a parent pipeline aggregation can have sub-aggregations performed on it.

### Examples Sub-Aggregations

#### Date Histogram & Average

        GET earthquakes/_search
        {
            "size": 0,
            "aggs": {
                "per_month": {
                "date_histogram": {
                    "field": "Date",
                    "calendar_interval": "month"
                },
                "aggs": {
                    "avg_magnitude": {
                    "avg": {
                        "field": "Magnitude"
                    }
                    }
                }
                }
            }
        }

_This aggregation has all the documents **grouped** by month, aswell as the average calculated for such month._

#### Date Histogram & Terms

        GET earthquakes/_search
        {
            "size": 0,
            "aggs": {
                "per_year": {
                "date_histogram": {
                    "field": "Date",
                    "calendar_interval": "year"
                },
                "aggs": {
                    "types": {
                    "terms": {
                        "field": "Type",
                        "size": 10
                    }
                    }
                }
                }
            }
        }

_This aggregation displays all documents **grouped** by year, and also displays which type of earthquake occured and how many._

## **(LAB)** Performing Metrics and Bucket Aggregations in Elasticsearch 7.13

### Objectives

The following questions should be answered using aggregations performed against the **ecommerce** dataset:

* What are the total lifetime sales before tax?
* How many orders per day are there for each calendar day?
* What are the total sales before tax per day for each calendar day?

Set the hits array **size** to **0** to simplify the aggregation output.

### Solution

#### Question 1:

_**Answer:** 350884.12890625_

_**Query:**_

        GET ecommerce/_search
        {
            "size": 0, 
            "aggs": {
                "total_lifetime_sales_before_tax": {
                    "sum": {
                        "field": "taxful_total_price"
                    }
                }
            }
        }

#### Question 2:

_**Answer:** To Many to write down._

_**Query:**_

        GET ecommerce/_search
            {
            "size": 0,
            "aggs": {
                "order_per_day": {
                    "date_histogram": {
                        "field": "order_date",
                        "calendar_interval": "day"
                    }
                }
            }
        }

#### Question 3:

_**Answer:** To Many to write down._

_**Query:**_

        GET ecommerce/_search
        {
            "size": 0,
            "aggs": {
                "order_per_day": {
                "date_histogram": {
                    "field": "order_date",
                    "calendar_interval": "day"
                },
                "aggs": {
                    "total_sales": {
                        "sum": {
                            "field": "taxless_total_price"
                        }
                    }
                }
                }
            }
        }

## Writing Pipeline Aggregations

### Parent Pipeline Aggregations

_Takes the output of a parent aggregations._

Using the output of a parent aggregation, parent pipeline aggregations create new buckets or new values for existing buckets.

### Sibling Pipeline Aggregations

_Takes the output of a sibling aggregation._

Using the output of a sibling aggregation, sibling pipeline aggregations create new outputs at the same level as the sibling aggregations.

### Examples Parent & Sibling

        GET earthquakes/_search
        {
            "size": 0,
            "query": {
                "term": {
                    "Type": {
                        "value": "Nuclear Explosion"
                    }
                }
            },
            "aggs": {
                "per_year": {
                    "date_histogram": {
                        "field": "Date",
                        "calendar_interval": "year"
                    },
                    "aggs": {
                        "max_magnitude": {
                            "max": {
                                "field": "Magnitude"
                            }
                        },
                        "change_in_max_magnitude": {
                            "derivative": {
                                "buckets_path": "max_magnitude"
                            }
                        }
                    }
                },
                "stats_change_in_max_magnitude": {
                    "stats_bucket": {
                        "buckets_path": "per_year>change_in_max_magnitude"
                    }
                }
            }
        }

_This aggregation will first **group** the documents by year and calculate the 'max_magnitude' (sub-agg). After that it will calculate a 'derivative' on the **grouped** buckets (parent aggs), and at the end, it will calculate all the stats on the buckets as well as the parent-aggregation (sibling aggs)._

## **(LAB)** Performing Pipeline Aggregations in Elasticsearch 7.13

### Objectives

The following questions should be answered using pipeline aggregations performed against the **flights** dataset:

* What is the total distance travelled per day and on which day was the most distance travelled?
* What is the cumulative sum of flight delay time in minutes per day?
* What is the change in average ticket price per day and what is the min, max, average, and sum of that change?

Set the hits array **size** to **0** to simplify the aggregation output.

### Solution

#### Question 1:

_**Answer:** total distance --> to much to write, which day was the most distance --> 2023-07-01_

_**Query:** _

        GET flights/_search
        {
            "size": 0,
            "aggs": {
                "per_day": {
                    "date_histogram": {
                        "field": "timestamp",
                        "calendar_interval": "day"
                    },
                    "aggs": {
                        "total_distance": {
                            "sum": {
                                "field": "DistanceKilometers"
                            }
                        }
                    }
                },
                "max_distance_per_day": {
                    "max_bucket": {
                        "buckets_path": "per_day>total_distance"
                    }
                }
            }
        }

#### Question 2:

_**Answer:** 618150.0_

_**Query:**_

        GET flights/_search
        {
            "size": 0,
            "aggs": {
                "per_day": {
                    "date_histogram": {
                        "field": "timestamp",
                        "calendar_interval": "day"
                    },
                    "aggs": {
                        "sum_of_delays": {
                            "sum": {
                                "field": "FlightDelayMin"
                            }
                        },
                        "cumalitive_sum_of_delays": {
                            "cumulative_sum": {
                                "buckets_path": "sum_of_delays"
                            }
                        }
                    }
                }
            }
        }

#### Question 3:

_**Answer:** Change in avg --> To much to write down, min = -240.62153345191405, max = 225.19714620467596, avg = 5.123142866056011, sum = 210.04885750829646_

_**Query:**_

        GET flights/_search
        {
            "size": 0,
            "aggs": {
                "per_day": {
                    "date_histogram": {
                        "field": "timestamp",
                        "calendar_interval": "day"
                    },
                    "aggs": {
                        "avg_ticket_price": {
                            "avg": {
                                "field": "AvgTicketPrice"
                            }
                        },
                        "change_in_ticket_price": {
                            "derivative": {
                                "buckets_path": "avg_ticket_price"
                            }
                        }
                    }
                },
                "stats_of_change_per_day": {
                    "stats_bucket": {
                        "buckets_path": "per_day>change_in_ticket_price"
                    }
                }
            }
        }

## **(QUIZ)** Aggregating Data in Elasticsearch 7.13

* **What API would you use to execute an aggregation?**
  * _search
* **Which aggregation would you use to determine the unique count of a field's values?**
  * cardinality
* **Which aggregation would you use to divide a time series into daily buckets?**
  * date_histogram
* **Which aggregation would you use to calculate the rate of change of a numeric time series?**
  * derivative