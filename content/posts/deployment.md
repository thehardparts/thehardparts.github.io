---
title: "Deployment"
date: 2022-09-27T10:14:15+02:00
draft: true
---

# Why

The applications that we create need to be deployed somewhere in order to be useable. There are many different ways to do this, and different places to deploy on. This document aims to illustrate the type of deployments we currently do in our team and how to decide on the deployment target.

# How

As a general rule, everything we deploy runs containerized, with [docker]({{< ref "containerizing.md" >}})
 (or OCI images) being the usual technology to do this. And things are generally preferred to be deployed to [google cloud]({{< ref "gcp.md" >}}) (*cloud first*), where we prefer to use serverless technologies (see below).

# What

This article will first give a short introduction on which usual deployment targets we have. And give some tips as to which to pick.

{{< toc >}}

## Kubernetes (local)

Most melexis sites have a local [kubernetes](https://kubernetes.io/) cluster. This allows us to easily deploy linux containers on each site.

Getting access to the clusters the first time requires quite a bit of setup. See [this guide](https://cmdb.elex.be/products/kubernetes/User_manual.md#section-2) on CMDB. If you still get an accesdenied afterwards it might be that you need to be added to the CBS LDAP group.

Deploying from Gitlab CI to kubernetes is done using the `dsl.melexis.com:5000/k8s-kubectl:1.19.2` image, for an example see [this pipeline job](https://gitlab.melexis.com/cbs/eda/fileeventstream/-/blob/master/.gitlab-ci.yml#L29).

## GKE

Google Kubernetes Engine is a hosted kubernetes solution. We can deploy a GKE cluster inside a GCP project to run pods related to applications belonging to said project.

We try to avoid this in favour of serverless solutions, as there is much more maintenance overhead (managing CPU/mem, upgrading machines, upgrading kubernetes versions,...).

An example of this is the [GKE cluster](https://console.cloud.google.com/kubernetes/list/overview?project=com-melexis-wmdb) of [wafermap-database](https://gitlab.melexis.com/cbs/wafermap-database/wafermap-database).

Next to regular GKE in mlx-project-specific GCP projects there are 3 special projects:

- com-melexis-test-diegem
- com-melexis-uat-diegem
- com-melexis-prod-diegem

These are projects running a single GKE cluster each, running pods that do not have their own GCP projects. These are generally only older products.

Deploying to these projects from Gitlab-CI requires quite a bit of setup, see [this](https://cmdb.elex.be/products/kubernetes/faq/How_to_migrate_an_application_to_gke.md). For an example see [this pipeline job](https://gitlab.melexis.com/cbs/sensordesign/web_tool/-/blob/master/.gitlab-ci.yml#L19).

## App Engine

Appengine is the oldest serverless service that GCP provides. It allows us to either deploy some code ([appengine standard](https://cloud.google.com/appengine/docs/standard)), or some containers ([appengine Flexible](https://cloud.google.com/appengine/docs/flexible)). App Engine instances are created when needed to respond to incoming (HTTP) requests.

App Engine Flex is to be avoided, as deploying to it is very slow, the containers never scale to 0 and every use case seems covered by cloud run.

App Engine Standard is still ok, however most use cases are covered by cloud functions and cloud run, which have some other advantages, eg. their access can be restricted using IAM, cloud run isn't limited to a couple of languages,...

Examples MLX:
Appengine Standard: [Stack](https://gitlab.melexis.com/cbs/pppure/stack)
Appengine Flex: [Rip](https://gitlab.melexis.com/cbs/pppure/rip)

## Cloud Run

[Cloud Run](https://cloud.google.com/run/docs/overview/what-is-cloud-run) is very similar to appengine flex, allowing us to deploy linux containers, however it does scale to 0, deploying to it doesn't take forever, endpoints can be secured with IAM, the logs are less of a mess,...

We have only ever used 'Cloud Run Services', where we scale up to handle incoming HTTP requests, as of recently there is also 'Cloud Run Jobs', which is more of a cronjob kind of deal. This we normally do with Cloud Scheduler + HTTP Cloud Functions.

Examples MLX: [Dat-to-testevent](https://gitlab.melexis.com/cbs/datalake/dat-to-testevents), [Pat](https://gitlab.melexis.com/cbs/pppure/pat)

## Cloud functions

[Cloud functions](https://cloud.google.com/functions/docs/how-to) allow us to directly deploy code without having to containerize it first. Unlike Cloud Run it is limited to a small number of programming languages.

1st gen Cloud Functions can be split in 2 categories:

- HTTP functions respond to (any type of) HTTP requests
- Event-driven functions respond to [GCP events](https://cloud.google.com/functions/docs/calling#1st-gen-triggers)

As of recent there are also '2nd gen' Cloud Functions, which are a shortcut to deploying functions code to Cloud Run (without containerizing it ourselves first), and comes with out of the box CloudEvent and Eventarc support (which we don't use at MLX).

Examples MLX: [datalake events-to-cloud-storage](https://gitlab.melexis.com/cbs/datalake/test-event-to-bucket/-/blob/master/index.js), [PP Pure UI functions](https://gitlab.melexis.com/cbs/pppure/pure_ui/-/tree/master/functions)

## What to use

In general: use Cloud Run or Cloud Functions, unless this isn't feasible for one or more reasons.

Local kubernetes:

In general only if the service needs to be accessible within a site even when the internet line is dead, or latency needs to be as low as possible,...

Diegem GKE:

Generally not for new development, maybe if an application needs to be constantly running (ie not well suited for cloud run et al) and setting up a GKE cluster in a dedicated project for 1 pod is overkill.

GKE in dedicated project:

Applications that need longer running time than Cloud Run/functions provide (usually >60mins). Ie an application that needs to be running 24/7.

App Engine:

Probably look at Cloud Run or Cloud Functions instead.

Cloud Functions:

Generally simple, single-source-file, applications that do 1 thing in response to a request.

Cloud Run:

More complex applications that are doing something as response to a requests.
