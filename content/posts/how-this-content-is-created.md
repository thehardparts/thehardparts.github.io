---
title: "How This Content Is Created"
date: 2022-09-09T13:31:15+02:00
draft: true
author: Brecht Hoflack
---

# Why
The idea of this blog is to share the knowledge between the team(s).
This should aide onboarding,  and also help as a reference.

# How
This is a living document where changes can be added by the whole team.

# What

## Contribute your content:

- clone this repository:

```git clone git@gitlab.melexis.com:thehardparts/thehardparts.git```
- checkout your own branch:

```git checkout -b <<new topic>>```
- create a new page:

```hugo new posts/<<my-new-page>>.md```
- write your content in markdown
- commit and push the changes

```git commit posts/<<my-new-page>>.md```
- push the changes

```git push origin <<new topic>>```
- create a merge request
- ask a colleague to merge it

## How to raise issues:
- create an issue in the gitlab project:

[gitlab issues](https://gitlab.melexis.com/thehardparts/thehardparts/-/issues)

## Which structure should i use
- try to keep a why / how / what structure
  - why: the broad context of the topic
  - how: how things have been implemented
  - what: which actions will be taken

- evaluate technology using steps suggested in "beyond the goal"
  - which rules currently apply before applying the change
  - which rules will be obsoleted after applying the change
  - which new rules will be required
