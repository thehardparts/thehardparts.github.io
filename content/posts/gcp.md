---
title: "Google Cloud Platform (GCP)"
date: 2022-09-26T16:28:30+02:00
draft: true
---

# Why

Applications in Melexis are developed with a cloud-first architecture in mind. This means we aim to deploy applications in a google cloud project, unless there is a good business reason for why it should be deployed locally.

# How

New applications are either deployed to an existing google cloud project, or to a new google cloud project related to the application. In both cases there are 3 google cloud projects: one for test, one for uat and one for production. This is reflected in the names of the projects.

eg. for the project datalake:

- com-melexis-test-datalake
- com-melexis-uat-datalake
- com-melexis-prod-datalake

All melexis projects follow the `com-melexis-\<env>-\<project_name>` naming convention. This is enforced by only allowing projects to be created using a specific rundeck job.

# What

It is required to install [gcloud](https://cloud.google.com/sdk/gcloud), the GCP command line interface. Most actions can also be performed using the [cloud console](https://console.cloud.google.com) UI.

Any form of infrastructure inside GCP projects is to be managed exclusively by terraform. For an overview of how we use terraform see the [relevant article]({{< ref "terraform.md" >}}))

Within melexis we generally only use a small subset of the available services of GCP, this article aims to be a quick guideline on which services you will probably need to use and what for. However any other GCP service can also be considered for use in projects, if there is deemed to be a good enough use case for it. Nobody should be afraid of proposing the use of alternate services not mentioned in this document.

{{< toc >}}

## Pub/Sub

Pub/Sub provides messaging functionality within the google cloud platform. We mainly use it to asynchronously (for synchronous communication we usually use HTTP) connect different (micro)services.

The basic components that we need to know of are topics and subscriptions. Each subscription is attached to a single topic, each topic can have many subscriptions. When a message is published to a topic, every subscription of that topic will receive a copy of the message.

![topics/subscriptions](/thehardparts/pubsub.png)

Subscriptions can either operate in a [push](https://cloud.google.com/pubsub/docs/push) or [pull](https://cloud.google.com/pubsub/docs/pull) configuration. With push the messages will be automatically posted to an http endpoint as they come in. With pull the messages stay on the subscription until a client explicitely asks for messages.

By default pub/sub operates with an unordered atleast-once delivery strategy. This means messages will come out of order and can be delivered multiple times. It is crucial that applications are designed to be able to handle this. It is possible to provide an [ordering key](https://cloud.google.com/pubsub/docs/ordering) to a message, causing messages with the same key to be delivered in-order, however this has quite some performance impact and should thus be avoided. [Exactly once delivery](https://cloud.google.com/pubsub/docs/exactly-once-delivery) exists in alpha, but also has performance impact.

Pubsub nowadays has some more advanced functionality, such as [schemas](https://cloud.google.com/pubsub/docs/schemas-valid) (enforcing message structure) and [snapshot/seek](https://cloud.google.com/pubsub/docs/replay-overview) (replay messages), but this is not commonly used in Melexis. Same goes for [pubsub lite](https://cloud.google.com/pubsub/docs/choosing-pubsub-or-lite).

Example projects using Pub/Sub:

- [Pat](https://gitlab.melexis.com/cbs/pppure/pat/-/blob/master/src/main/java/com/melexis/pat/Controller.kt#L71) (receiving push notifications)
- [dat-to-testevents](https://gitlab.melexis.com/cbs/datalake/dat-to-testevents/-/blob/master/terraform/pubsub.tf) (create topics and subcriptions with terraform)
- [aggregate](https://gitlab.melexis.com/cbs/datalake/aggregate-for-parquet/-/blob/master/aggregator/src/main/kotlin/com/melexis/aggregateparquet/EventPublisher.kt#L48) (publish to topic)
- [akaunto](https://gitlab.melexis.com/cbs/akaunto/-/blob/master/mailing_service_akaunto/mailing_service.py#L27) (get messages from pull subscription)


## Cloud storage

Google Cloud Storage (GCS) provides basic object storage within the google cloud platform. It is the main service that we use to store any form of file-like data and is used in every Melexis project.

Each project can have many cloud storage buckets, which in turn can contain an infinite number of objects.

Some important differences to realize when comparing GCS with regular file/block storage.

- There is no concept of folders, however the UI does show this, and we can emulate this by listing objects on prefixes
- It is essentially a key-value store, with the object name being a unique key in the bucket
- Objects are immutable, making changes will remove the old object and write a new one
- Depending on the [storage class](https://cloud.google.com/storage/docs/storage-classes), accessing objects might not be free
- Object names have different [limitations](https://cloud.google.com/storage/docs/naming-objects)
- We can maintain multiple [versions](https://cloud.google.com/storage/docs/object-versioning) of the same object to retain history, however we need to pay for all of the copies
- All objects within a bucket are replicated to at least two different zones (~data centers) even if the data is stored within a single region
- Multi-region storage is also available for latency and disaster protection reasons, but comes at a higher cost

Objects within a bucket can have individual storage classes each with their own pricing. The nice thing about this is that it allows us to do things like moving stale objects to cheaper storage classes by setting [lifecycle rules](https://cloud.google.com/storage/docs/control-data-lifecycles) on the bucket.

## Deployment of applications (cloud run, cloud functions, appengine, GKE)

See [Deployment]({{< ref "deployment.md" >}})

## Cloud SQL

[Cloud SQL](https://cloud.google.com/sql/docs/features) provides a hosted solution for traditional SQL databases: MySQL, PostgreSQL and SQL Server.

Within Melexis we usually use postgres databases locally, so in the cloud this is also the first choice. However a traditional database is not always the best choice for any architecture as it doesn't scale well and might be overly expensive if not much data is stored. So it should be checked to see if other services aren't more applicable. We don't tend to use much cloud SQL for new projects (although there are obviously still valid use cases), it is usually enabled for older projects that were migrated.

Connecting from a local pc to cloud sql databases is usually done using the [cloud sql proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy). Other connection options are listed in the same documentation, unix sockets for cloud sql dbs in the same project are provided for some services (appengine, cloud run,...). Also notable is that databases in the diegem and xservices projects are open for direct connections if you are on the melexis network.

## Bigquery

Bigquery is a serverless data warehousing product. It's mainly used to store non-relational columnar data.

One GCP project has one or more bigquery datasets, each of which can contain many tables. The datasets can be queried via the cloud console or CLI using an SQL dialect, making it easy to perform queries for people coming from regular databases.

You pay for both the storage costs and cpu-time of the queries.

Usage of bigquery within Melexis is quite limited, in the short term it will be used to store the events (allowing us to replay them) in the [datalake project](https://gitlab.melexis.com/cbs/datalake/test-events-to-bigquery).

## Firestore

Firestore is a no-sql database that stores data as documents in a hierarchical structure:
Root level collections contain documents, which contain values or collections, which contain documents, etc...

The big thing that it provides is real-time updates. A client can subscribe to a document and will be notified whenever it is updated, making it quite useful for things like updating user interfaces.

Within Melexis, Firestore is used to store the command and job statuses of postprocessing pure jobs. This is then used for visualisation in the [pure ui](https://ui-dot-com-melexis-pp.appspot.com/) ([code](https://gitlab.melexis.com/cbs/pppure/pure_ui)). The pppure2spc converter also [subscribes to the data](https://gitlab.melexis.com/TMNT/pppure2spc/pppure2spc/-/blob/masterV2/pppure2spc-common/src/main/java/com/melexis/pppure2spc/common/service/FireStoreService.java#L59) to track changes for specific jobs.

## Cloud logging

The cloud logging interface is where all logs from a google cloud project (and settings related to the logging) can be found. In general this is where you will end up when trying to debug things. The logs can be queried either by using the UI elements (clicking on the filters on the left menu) or by writing [the query](https://cloud.google.com/logging/docs/view/logging-query-language) yourself.

A special case for Melexis is the [com-melexis-prod-logging](https://console.cloud.google.com/logs/query?project=com-melexis-prod-logging) (and [-test]((https://console.cloud.google.com/logs/query?project=com-melexis-test-logging)), [-uat]((https://console.cloud.google.com/logs/query?project=com-melexis-uat-logging))) project. In this project all logs coming from the local kubernetes clusters (and some VMs) can be found.

Since this is a massive amount of data querying can be quite slow. When trying to debug a single deployment/statefulset/pod it is highly recommended to speedup your search by prepending your query with:

{{< highlight perl>}}
logName =~ <deployment_name>
{{< / highlight >}}
