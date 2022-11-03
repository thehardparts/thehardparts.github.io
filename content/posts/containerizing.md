---
title: "Containerizing applications"
date: 2022-09-28T11:04:58+02:00
draft: true
---

In general most of our software runs on linux containers.

The most common way to package our software is to manually write a [Dockerfile](https://docs.docker.com/engine/reference/builder/) and add this to the project. After build we then generally just run:

{{< highlight bash>}}
docker build -t <image_name>:<version> .
docker push dsl.melexis.com:5000
{{< / highlight >}}

For a Dockerfile examples for java see: [dat-to-testevents](https://gitlab.melexis.com/cbs/datalake/dat-to-testevents/-/blob/master/Dockerfile). Note that if more is done than just copying the jar, you generally want to do this copy as the very last step to try to take as much advantage of [layer caching](https://stackoverflow.com/a/33836848/9936828) as you can.

A downside to this approach is that it requires the building machine (usually in [gitlab-ci]({{< ref "gitlab-ci.md" >}}), which requires the *dind-prod* tag) to be running docker (or podman,...). For JVM projects it is therefore worth contemplating using [Jib](https://github.com/GoogleContainerTools/jib) instead, which can build Docker/OCI images without a daemon. See [CAS](https://gitlab.melexis.com/gcs/cas/-/blob/master/gradle/jib.gradle) or [test-event-to-parquet](https://gitlab.melexis.com/cbs/datalake/test-event-to-parquet/-/blob/master/.gitlab-ci.yml#L189) for example projects using Jib.

It is required for team members to have a way to run linux containers on their pc for debugging purposes.

- On windows use docker desktop with WSL2 backend (**license required**), or docker in WSL2 without desktop
- On linux install either docker or podman via your package manager
- On macOS install either docker desktop (**license required**) or podman via homebrew

If a docker desktop license is required contact Piet (PDC).
