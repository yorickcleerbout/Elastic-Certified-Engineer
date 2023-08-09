<!-- omit from toc -->
# Chapter Seven: Cluster Management

- [Introduction to Cluster Management](#introduction-to-cluster-management)
- [Diagnose Shard Issues](#diagnose-shard-issues)
  - [Cluster Allocation Explain API](#cluster-allocation-explain-api)
  - [Examples](#examples)
    - [Create diagnosic index](#create-diagnosic-index)
    - [Check Shards](#check-shards)
    - [Explain Allocation](#explain-allocation)
    - [Set the number of replicas to 0](#set-the-number-of-replicas-to-0)
- [**(LAB)** Diagnosing and Repairing Shard Issues in Elasticsearch 7.13](#lab-diagnosing-and-repairing-shard-issues-in-elasticsearch-713)
  - [Objectives](#objectives)
  - [Solution](#solution)
  - [accounts-1 index \& accounts-2](#accounts-1-index--accounts-2)
    - [Check whats wrong](#check-whats-wrong)
    - [Change settings](#change-settings)
    - [Re-enable node 3](#re-enable-node-3)
    - [Check if the node is up again](#check-if-the-node-is-up-again)
  - [Prevent accounts-2 from going red if a node fails](#prevent-accounts-2-from-going-red-if-a-node-fails)
  - [Explain the issue](#explain-the-issue)
  - [Change this routing setting](#change-this-routing-setting)
- [Snapshotting Data](#snapshotting-data)
  - [Examples](#examples-1)
    - [Register a snapshot repository](#register-a-snapshot-repository)
      - [Create \& whitelist file location](#create--whitelist-file-location)
    - [Create a snapshot](#create-a-snapshot)
    - [Create Snapshot Lifecycle Management Policy](#create-snapshot-lifecycle-management-policy)
- [Configuring Searchable Snapshots](#configuring-searchable-snapshots)
  - [Examples](#examples-2)
    - [Manually using \_mount API](#manually-using-_mount-api)
    - [Using ILM Policy](#using-ilm-policy)
- [Restoring Data](#restoring-data)
  - [Examples](#examples-3)
- [**(LAB)** Snapshotting and Restoring Data in Elasticsearch 7.13](#lab-snapshotting-and-restoring-data-in-elasticsearch-713)
  - [Objectives](#objectives-1)
  - [Solution](#solution-1)
    - [Define snapshot repository](#define-snapshot-repository)
    - [Create snapshot](#create-snapshot)
    - [Restore index](#restore-index)
    - [Mount snapshot](#mount-snapshot)
- [Setting Up Cross-Cluster Replication](#setting-up-cross-cluster-replication)
  - [Examples](#examples-4)
    - [Configure Remote Cluster](#configure-remote-cluster)
    - [Configure Cross-Cluster Replication](#configure-cross-cluster-replication)
    - [Auto-Follow](#auto-follow)
- [**(LAB)** Implementing Cross-Cluster Replication in Elasticsearch 7.13](#lab-implementing-cross-cluster-replication-in-elasticsearch-713)
  - [Objectives](#objectives-2)
  - [Solution](#solution-2)
    - [Configure Remote Cluster](#configure-remote-cluster-1)
    - [Create Cross-Cluster Replica](#create-cross-cluster-replica)
    - [Enable Auto-Follow](#enable-auto-follow)
- [Defining Access Control](#defining-access-control)
  - [Authenticate](#authenticate)
    - [Build-In Realms](#build-in-realms)
    - [Custom Realm](#custom-realm)
  - [Authorize](#authorize)
    - [Cluster](#cluster)
    - [Index](#index)
  - [Examples](#examples-5)
    - [Create role with read access on specific field](#create-role-with-read-access-on-specific-field)
    - [Create a user with role](#create-a-user-with-role)
- [**(LAB)** Implementing Role-Based Access Control in Elasticsearch 7.13](#lab-implementing-role-based-access-control-in-elasticsearch-713)
  - [Objectives](#objectives-3)
  - [Solution](#solution-3)
    - [Create the Role](#create-the-role)
    - [Create the User](#create-the-user)
- [**(QUIZ)** Cluster Management in Elasticsearch 7.13](#quiz-cluster-management-in-elasticsearch-713)

## Introduction to Cluster Management

* Diagnose Shard Issues
* Snapshotting Data
* Configuring Searchable Snapshots
* Restoring Data
* Setting Up Cross-Cluster Replication
* Defining Access Control

## Diagnose Shard Issues

### Cluster Allocation Explain API

_Make the cluster explain itself._

This API will provide a detailed explanation as to why something is allocated or not allocated.

* Providing no parameters will explain the first **unassigned** shard.
* Providing an index and a shard will explain the allocation of that shard.

### Examples

#### Create diagnosic index

        PUT diagnostic

#### Check Shards

        GET _cat/shards/diagnostic?v

#### Explain Allocation

        GET _cluster/allocation/explain
        {
            "index": "diagnostic",
            "shard": 0,
            "primary": false
        }

#### Set the number of replicas to 0

        PUT diagnostic/_settings
        {
            "number_of_replicas": 0
        }

_This will put this index in a **green** state._

## **(LAB)** Diagnosing and Repairing Shard Issues in Elasticsearch 7.13

### Objectives

* The **accounts-1** and **accounts-2** indices must be allocated with a **green** state.
* The **accounts-1** and **accounts-2** indices must be configured with the best possible reduncandy and reliability.

### Solution

### accounts-1 index & accounts-2

#### Check whats wrong

        GET _cat/shards/accounts-1?v

        GET _cluster/allocation/explain

        GET _cat/indices/accounts-1?v

_Here we can see we have **3** primary and **3** replica shards, but we have only a **3 node cluster**, so we only can have a maximum of **2** replicas._

#### Change settings

        PUT accounts-1/_settings
            {
            "number_of_replicas": 2
            }

_After this setting change, we see that the state is still **yellow**. After checking the nodes we see that the 3rd node is down._

#### Re-enable node 3

        1) ssh cloud_user@<publicIP>
        2) sudo systemctl status elasticsearch
        3) sudo systemctl start elasticsearch

#### Check if the node is up again

        GET _cat/nodes?v

_This should fix the 'accounts-1' index to have a **green** state. This will also fix the **red** state of 'accounts-2' to be **green**._

### Prevent accounts-2 from going red if a node fails

        PUT accounts-2/_settings
        {
            "number_of_replicas": 2
        }

_After this setting change, we can see that the state is **yellow**._

### Explain the issue

        GET _cluster/allocation/explain
        {
            "index": "accounts-2",
            "shard": 0,
            "primary": false
        }

_We can see that there is a index routing issue 'index.routing.allocation.require'._

### Change this routing setting

        PUT accounts-2/_settings
        {
            "number_of_replicas": 2,
            "index.routing.allocation.require._name": null 
        }

## Snapshotting Data

1) Register a snapshot repository
2) Create a snapshot of 1 or more indices.
3) Use snapshot lifecycle management (SLM) (automation).
   
### Examples

#### Register a snapshot repository

        PUT _snapshot/cluster
            {
            "type": "fs",
            "settings": {
                "location": "/mnt/backups"
            }
        }

_'cluster' is the name I have given to the repository. If the file location does not exists, you need to create the folder and whitelist the path on the server it self._

##### Create & whitelist file location

1) ssh into machine
2) Create directory (`mkdir /mnt/backups`)
3) Change ownership (`chown elasticsearch:elasticsearch /mnt/backups`)
4) Whitelist the directory
   1) `vim /etc/elasticsearch/elasticsearch.yml`
   2) Add directory (`path.repo: /mnt/backups`
5) Restart Elasticsearch (`systemctl restart elasticsearch`)

#### Create a snapshot

        PUT _snapshot/cluster/backup_1
        {
            "indices": "*",
            "include_global_state": true
        }

_The 'global_state' includes aliases, index_templates, etc._

#### Create Snapshot Lifecycle Management Policy

        PUT _slm/policy/daily_snapshot
        {
            "schedule": "0 0 1 * * ?",
            "name": "<daily_snapshot_{now/d}>",
            "repository": "cluster",
            "config": {
                "indices": "*",
                "include_global_state": true
            },
            "retention": {
                "expire_after": "30d",
                "min_count": 7,
                "max_count": 30
            }
        }

_This SLM will create everyday a snapshot of all indices and will let it expire after 30d, but keep a minimum of 7 backups and maximum of 30 backups._

## Configuring Searchable Snapshots

_Offload data while still being able to search it._

* No replica shards by default. The snapshot repository is responsible for redundancy.
* Replica shards can be configured in order to increase search throughput.
* Can be configured through index lifecycle management (ILM) automatically or the _mount API manually.

### Examples

#### Manually using _mount API

        POST _snapshot/cluster/backup_1/_mount
        {
            "index": "shakespeare",
            "renamed_index": "shakespeare_searchable_snapshot"
        }

#### Using ILM Policy

        PUT _ilm/policy/my_policy
        {
            "policy": {
                "phases": {
                    "cold": {
                        "actions": {
                            "searchable_snapshot": {
                                "snapshot_repository": "my_repository"
                            }
                        }
                    }
                }
            }
        }

## Restoring Data

* Restore 1 or more indices from a snapshit.
* Restored indices can be renamed during the restore process.
* Restore a snapshot from different cluster.

### Examples

        POST _snapshot/cluster/backup_1/_restore
        {
            "indices": "shakespeare",
            "rename_pattern": "(.+)",
            "rename_replacement": "$1_restored"
        }

_This will restore one index from the snapshot and rename it to <index>_restored at restore time._

## **(LAB)** Snapshotting and Restoring Data in Elasticsearch 7.13

### Objectives

The following requirements should be met:

* The **accounts** repository should be created at **/mnt/backups/accounts**.
* The **accounts_test_backup** snapshot should be created to backup the **accounts** index with global state.
* The **accounts** index should be restored from the **accounts_test_backup** snapshot as the index **accounts_test_restore**.
* The **accounts** index should be mounted from the **accounts_test_backup** snapshot as the index **accounts_test_backup**.

In the end, you should have the original **accounts** index, the **accounts_test_restore** index, and the searchable snapshot **accounts_test_backup**.

### Solution

#### Define snapshot repository

        PUT _snapshot/accounts
        {
            "type": "fs",
            "settings": {
                "location": "/mnt/backups/accounts"
            }
        }

#### Create snapshot

        PUT _snapshot/accounts/accounts_test_backup
        {
            "indices": "accounts",
            "include_global_state": true
        }

#### Restore index

        POST _snapshot/accounts/accounts_test_backup/_restore
        {
            "indices": "accounts",
            "rename_pattern": "(.+)",
            "rename_replacement": "$1_test_restore"
        }

#### Mount snapshot

        POST _snapshot/accounts/accounts_test_backup/_mount
        {
            "index": "accounts",
            "renamed_index": "accounts_test_backup"
        }

## Setting Up Cross-Cluster Replication

_A leader indexes and a follower replicates._

Configure a remote cluster and follow 1 or more indices from it. Follower indices will replicate all actions performed on the remote leader index.

* Disaster recovery
* Data locality
* Centralized reporting

### Examples

#### Configure Remote Cluster

        PUT _cluster/settings
        {
        "persistent": {
                "cluster": {
                    "remote": {
                        "leader": {
                            "seeds": "<privateIP>:9300"
                            }
                    }
                }
            }
        }

#### Configure Cross-Cluster Replication

        PUT shakespear/_ccr/follow
        {
            "remote_cluster": "leader",
            "leader_index": "shakespeare"
        }

_Any changes done to the 'shakespeare' index on the 'leader' cluster, will automatically be replicated to the 'follower' cluster._

#### Auto-Follow

        PUT _ccr/auto_follow/shakespeare
        {
            "remote_cluster": "leader",
            "leader_index_patterns": ["shakespeare-*"]
        }

_Every index created with 'shakespeare-*' as pattern will be automatically followed and thus replicated._

## **(LAB)** Implementing Cross-Cluster Replication in Elasticsearch 7.13

### Objectives

Configure cross-cluster replication so that the **accounts-1** and all future ** accounts-* ** indices on the **es1** cluster are replicated to the **es2** cluster.

### Solution

#### Configure Remote Cluster

        PUT _cluster/settings
        {
            "persistent": {
                "cluster": {
                    "remote": {
                        "accounts_leader": {
                            "seeds": "10.0.1.101:9300"
                        }
                    }
                }
            }
        }

#### Create Cross-Cluster Replica

        PUT accounts-1/_ccr/follow
        {
            "remote_cluster": "accounts_leader",
            "leader_index": "accounts-1"
        }

#### Enable Auto-Follow

        PUT _ccr/auto_follow/accounts
        {
            "remote_cluster": "accounts_leader",
            "leader_index_patterns": ["accounts-*"]
        }

## Defining Access Control

_Authenticate and authorize._

Prevent access to your data with a variety of authentication realms. Limit access to your data with roles.

### Authenticate

#### Build-In Realms
* native
* ldap
* active_directory
* pki
* file
* saml
* oidc

#### Custom Realm
Using the token and API key services, you can build a custom realm.

### Authorize

#### Cluster

Define **cluster-level** privileges. Cluster privileges are typically reserved for administrators.

#### Index

Define the **index-level** privileges for one or more indices. This can optionally include **document-level** and **field-level** privileges.

### Examples

#### Create role with read access on specific field

        POST _security/role/products_read
        {
            "cluster": [],
            "indices": [
                {
                    "names": ["ecommerce"],
                    "privileges": ["read"],
                    "field_security": {
                        "grant": ["products.*"]
                    },
                    "query": {
                        "term": {
                            "geoip.continent_name": {
                                "value": "North America"
                            }
                        }
                    }
                }
            ]
        }

_This role will only give access to the products array of order from 'North America'._

#### Create a user with role

        POST _security/user/yorick
        {
            "full_name": "Yorick Cleerbout",
            "email": "yorick@company.com",
            "password": "test123",
            "roles": ["products_read", "kibana_user"]
        }

## **(LAB)** Implementing Role-Based Access Control in Elasticsearch 7.13

### Objectives

The following role and user should be created:

Role: **account_holders_read**
* Only has **read** access to the **accounts** index.
* Only has access to the documents where **mail_opt_out** is **false**.
* Only has access to the **firstname**, **lastname**, and **email** fields.

User: **accounts_mailer**
* Full Name: **Accounts Mailer**
* Email: **accounts_mailer@company.com**
* Password: **yUqS54J9d6nx**
* Roles: **account_holders_read**

### Solution

#### Create the Role

        POST _security/role/account_holders_read
        {
            "indices": [
                {
                    "names": ["accounts"],
                    "privileges": ["read"],
                    "field_security": {
                        "grant": ["firstname", "lastname", "email"]
                    },
                    "query": {
                        "term": {
                            "mail_opt_out": {
                                "value": false
                            }
                        }
                    }
                }
            ]
        }

#### Create the User

        POST _security/user/accounts_mailer
        {
            "full_name": "Accounts Mailer",
            "email": "accounts_mailer@company.com",
            "password": "yUqS54J9d6nx", 
            "roles": ["account_holders_read"]
        }

## **(QUIZ)** Cluster Management in Elasticsearch 7.13

* Which of the following is not true about user access control in Elasticsearch?
  * All users have default access to Kibana.
* Which of the following is not true about cross-cluster replication?
  * You can only replicate to one cluster at a time.
* Which of the following is not true about searchable snapshots?
  * They can be indexed to.
* What parameter must be set in elasticsearch.yml in order to create a filesystem snapshot repository?
  * path.repo
* Which of the following is not true about restoring indices in Elasticsearch?
  * You can only restore one index at a time.
* Which API would you use to troubleshoot shard allocation issues?
  * _cluster/allocation/explain