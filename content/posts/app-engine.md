---
title: "App Engine"
date: 2022-06-09T12:01:04+02:00
draft: true
---

# App engine

App engine was one of the first SAAS offerings Google created.
It's a true serverless solution,  that allows you to focus on the code and minimizes the administration required.

Currently App Engine comes in 2 flavors:
- App engine standard
- App engine flex

In this document we will focus on App Engine standard,  as App Engine flex has more or less been succeeded by Google Cloud Run.

## Why

- Appengine allows you to focus on your code,  as all of the infrastructure is managed by Google.
- Your code is scalable,  as every new request is (theoretically) spawning a new instance.


## Install the google cloud sdk

On mac:

```
brew install google-cloud-sdk
```

Now you should be able to login with gcloud:

```
gcloud auth login
```

And set the default GCP project:

```
gcloud config set project ...
```

## Java
### Deploying a new project in spring boot
#### Generate a new project
Goto https://start.spring.io and generate a new ( maven ) spring boot application.

#### Configure the pom.xml
Open the pom.xml,  and do the following steps:

- set packaging to war
- add the appengine-maven-plugin in build > plugins

```
            <plugin>
                <groupId>com.google.cloud.tools</groupId>
                <artifactId>appengine-maven-plugin</artifactId>
                <version>2.4.2</version>
                <configuration>
                </configuration>
            </plugin>
```

#### Add appengine-web.xml to src/main/webapp/WEB-INF/

Remove the web.xml file,  and create a new appengine-web.xml file. 

```
<?xml version="1.0" encoding="utf-8"?>
<!--
Copyright 2017 Google Inc.
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
    <threadsafe>true</threadsafe>
    <runtime>java8</runtime>
    <system-properties>
        <property name="java.util.logging.config.file" value="WEB-INF/classes/logging.properties"/>
    </system-properties>
    <service>**the project name**</service>
    <env-variables>
    </env-variables>
</appengine-web-app>
```

#### Deploy a test application

After doing these changes you should be able to deploy your hello world application to google cloud:

```
mvn install appengine:deploy
```