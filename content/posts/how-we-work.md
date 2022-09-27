---
title: "How We Work"
date: 2022-09-26T16:02:44+02:00
draft: false
---

In DPA we try to follow a lean and continuous improvement process.
The target of the process is to share our knowledge,  and to remove all of the cruft from the process.

While developing,  we heavily rely on the following tools:

- git for version control
- gitlab for sharing our code,  and for gitlab ci
- slack for team communication

# The process
To enable us to support this we use the following process ( we constantly iterate between these steps ):

## Event storming
To gather context of the business,  and the domains we are working in we make use of eventstorming.
Event storming is a chaotic way of gathering events by inviting a lot of stakeholders,  and by asking them to create post-its for each item in their process (= event).
From these events a timeline can be created,  which can then be used to discover the business domains and current issues.

This is a great place to discover the domains,  the language used in the domains,  and to make connections between developers and business persons.

## Design discussions
Normally we have a business analyst or architect doing the design of the system in more detail.
Once we have the design in sufficient detail,  the system is discussed in team again.

We use estimations as a base for the discussion.
To do this,  we create a breakdown of each part of the system,  and discuss it until there is concensus in the team that we have enough information for estimating it.

From this epics are created,  and they are used for planning.

## Planning
We use kanban to process our items.
Kanban allows us to create an overview over different projects,  and should enable us to share work between projects.

We have different columns ( = stages ) on the board for the stages in our process.
Each of these columns has a limit of the number of items that can be handled in parallel there.
Once the limits are reached,  the team should work on removing the items before adding new items.

We also limit the amound of items in the backlog.
When a team member is ready to process a new item,  he/she can take the top item from the todo list.

## Developing
We use TDD to develop our software.
This means that we add a test for each use case,  and then develop the software till the test is ok ( green ).
Once the test is green,  the software can be refactored with confidence.

While developing we use the following ground rules:
- the code should be correct
- it should be concise and descriptive

Only optimise for speed when the code is actually a bottleneck.

Create a commit in git with the jira code for your improvement / new feature / bug ticket ( we have a git hook to do this [automatically](https://gitlab.melexis.com/cbs/gitlab-webhook-integrations/link_to_jira) ).

## Code review
When an item is ready,  we assign a colleague to do code review.
This enables sharing of knowledge,  and should bring everybody on the same level of code quality.
You can assign a code review by setting a reviewer in the merge request of gitlab.

Once a reviewer has been set,  our slackbot ( @jean-ralphio [source](https://gitlab.melexis.com/cbs/gitlab-webhook-integrations/jean-ralphio-slackbot) ) should automatically alert the assigned reviewer,  and set the jira ticket in the correct state.

If the reviewer has feedback,  the code should be updated,  and reviewed again.

## Performing Code review
You are alerted on slack when a merge request is assigned to you.

To perform code review,  check if the code has been properly tested,  and if the coding style is up to standards.
We are looking for descriptive,  self explaining code that is preferably functional.
If you have remarks,  you can add these to the line in gitlab.

When you are finished,  you can add a DONE comment to the merge request.
This will trigger our slackbot to notify the developer to update the code.

If the code is ok,  you can merge it into the master.
This will deploy the code to uat.

## Deploying
We rely on gitlab ci pipelines to automatically build,  test,  deploy and release.

We typically have 3 deployment targets:
- test for every push to gitlab
- uat for integration testing,  when a branch is merged to the master branch
- prod,  when a tag is pushed

## Releasing
Once code is approved on the test and uat stage,  it can be deployed on production.
To do this,  create a tag with the new [semantic version](https://semver.org),  and push it to gitlab.

This will automatically trigger a pipeline to deploy our software to production.

## Retrospective

### Dapt status
Every 2 weeks we have a team retrospective.
There we all come up with things that were bad,  good,  and ideas for improvements.

### Status meeting Dapt
Every 2 weeks we report on the status of the things we did in the past week.

# What is expected from you

- Contact the rest of the team when you're stuck or when something is unclear
- Critical thinking
- Push for high quality code
- Have fun
