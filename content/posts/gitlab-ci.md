---
title: "Gitlab Ci"
date: 2022-09-09T14:26:09+02:00
draft: false
---

# Why

When developing software we want to automate as many tasks as possible to reduce the time wasted and avoid the risk of making manual mistakes.

# How

Since we use Gitlab to store our source code, the logical solution to use is [Gitlab CI](https://docs.gitlab.com/ee/ci/).

# What

Whenever a new project is created we immediately set up a gitlab CI pipeline for it. This can usually be done before any code is written by just deploying a dummy application to the test env. Configuration of the pipeline is done using the [.gitlab-ci.yml](https://docs.gitlab.com/ee/ci/yaml/) file which is placed in the root of the project.

For a tutorial for creating a basic pipeline see [this](https://about.gitlab.com/blog/2020/12/10/basics-of-gitlab-ci-updated/).

{{< toc >}}

## Gitlab ci basics

## Default CI flow

We build and run all applicable tests on every push, regardless of the branch.

The deployment targets are the following:

- **feature branch**: deploy to test
- **main**: deploy to test and uat
- **tag**: deploy to test, uat and prod
  
Usually deployment comes after some preparation steps, eg:

- Promoting the image (see [Artifactory]({{< ref "artifactory.md" >}}))
- Running terraform
- ...

A minimal pipeline usually looks something like this:

{{< highlight yaml "linenos=false" >}}
stages:
  - build
  - infrastructure test
  - deploy test
  - promote & infrastructure uat
  - deploy uat
  - promote & infrastructure prod
  - deploy prod
{{< / highlight >}}

## gitlab-ci.yml quality rules

Secrets should be stored in the [group variables](https://docs.gitlab.com/ee/ci/variables/#add-a-cicd-variable-to-a-group) if they are shared between multiple repos of one project. Otherwise secrets can be stored in the [project variables](https://docs.gitlab.com/ee/ci/variables/#add-a-cicd-variable-to-a-project).

Use either the [extends](https://docs.gitlab.com/ee/ci/yaml/yaml_optimization.html#use-extends-to-reuse-configuration-sections) keyword or [yaml anchors](https://docs.gitlab.com/ee/ci/yaml/yaml_optimization.html#yaml-anchors-for-scripts) to prevent duplication of code.


## CD

In order to be able to automate the production deployment flow within melexis (see [CMDB]({{< ref "cmdb.md" >}}) and [Jira]({{< ref "jira.md" >}})), we have created a docker image (*dsl.melexis.com:5000/brh/create_release*) that allows us to perform all release steps automatically.

The release script also shepherds the new release to production by automatically updating the RFC ticket and closing it when done, or rejecting it in case a deployment fails. It also interacts with slack via [jralphio]({{< ref "jralphio.md" >}}), notifying the relevant slack channel with the deployment status.

To set this up for a new project:

1. Create the product in CMDB, and obtain the full path
2. Figure out the project in Jira where this product belongs to


Add the following variables to your gitlab-ci.yml:

{{< highlight yaml "linenos=false" >}}
variables:
- CI: "<path of cmdb ci>"
- JIRA_KEY: "<jira project key>"
{{< / highlight >}}
  
And add the jobs:

**create-release:**

  {{< highlight yaml "linenos=false" >}}
  only:
    - tags  
  script:
    - VERSION="$(git describe --tags)"
    - /usr/local/bin/python /app/create_release.py --key $JIRA_KEY --project $CI --version $VERSION --cd
  artifacts:
    paths:
      - rfc.txt
  {{< / highlight >}}

**update-rfc-uat:**

  {{< highlight yaml "linenos=false" >}}
  dependencies:
    - create-release
  only:
    - tags
  script:
    - RFC=$(cat rfc.txt)
    - /usr/local/bin/python /app/update_rfc.py --rfc $RFC --pipeline $CI_PIPELINE_URL --deployed-to uat
  {{< / highlight >}}
 
**update-rfc-prod:**

  {{< highlight yaml "linenos=false" >}}
  dependencies:
    - create-release
  only:
    - tags
  script:
    - RFC=$(cat rfc.txt)
    - /usr/local/bin/python /app/update_rfc.py --rfc $RFC --pipeline $CI_PIPELINE_URL --deployed-to prod
  {{< / highlight >}}


Tagging a commit will now start the release procedure and perform all steps without manual intervention.

For an example of a complex pipeline with CD set up, see [this](https://gitlab.melexis.com/cbs/datalake/dat-to-testevents/-/blob/master/.gitlab-ci.yml).

## Getting secrets from vault

There are 2 ways to get a secret (token, password, service account file,...) from vault from a gitlab-ci pipeline:

1. Using a vault token stored in the group/project variables, we get the secret using curl:
{{< highlight yaml "linenos=false" >}}
curl -s --header "X-Vault-Token: $VAULT_TOKEN" -X GET https://vault.colo.elex.be:8200/v1/secret/data/$SECRET_PATH
{{< / highlight >}}

2. If the gitlab project is registered in vault as an application with read rights to the secret, we first need to get a short-lived token and then execute the curl command:
{{< highlight yaml "linenos=false" >}}
VAULT_ADDR="https://vault.melexis.com"
APPLICATION=<application name>
VAULT_TOKEN=$(vault write -field=token auth/gitlab-jwt/login role=$APPLICATION jwt="$CI_JOB_JWT")
curl -s --header "X-Vault-Token: $VAULT_TOKEN" -X GET https://vault.colo.elex.be:8200/v1/secret/data/$SECRET_PATH
{{< / highlight >}}
  

## Other useful resources

[Which runner tag to use](https://cmdb.elex.be/products/gitlab-runner/user-manual#section-4)
