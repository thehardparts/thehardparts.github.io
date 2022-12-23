---
title: "Pure"
date: 2022-12-22T16:36:00+01:00
draft: true
---

# Postprocessing pure

## Why

In Melexis we design chips,  and ensure that the chips we design are delivered up to a specification.
To do this,  we have a production environment where we test our chips for different conditions.
This testing is done when the chips are still on a wafer ( probing ),  and after the chips have been packaged ( final test ).

## How

We keep track of all results during the testing of our chips.

To predict which chips might fail in the future ( and as required by our customers ),  we started doing postprocessing on our data.
This started with rip,  where we do scratch detection on the wafermaps to predict which chip might turn bad.

Later we started doing outlier detection on our analog data,  which we call pat.

Over the years we built more and more modules for postprocessing.
We now run ai models trained with customer returns,  do preprocessing on our analog data,  integrate partner data, ...

## What

Postprocessing pure is designed as a list of independent components that are glued together by the business logic.
The business can define recipes per item / device or lotname,  which declare the steps that should be executed for a job.

A job is triggered when a job in Oracle is moved to postprocessing in Oracle.
This is intercepted by trigger_pure,  which will build the correct recipe for this job.

Once the recipe has been created,  it's triggered to the coordinator.
The coordinator is responsible for executing jobs,  and making them fault tolerant.
It will call every service in the order defined in the recipe,  and will retry if any system error would occur.
If a call was succcessfull,  it will take the results and pass it to the next command which has a dependency to the result.

If a job was successfull,  it's moved to the succeeded folder in cloud storage.
If it failed ( after several retries ) because of system errors,  it's moved to the system error folder.
While the job is moved to the user error folder when the recipe contained configuration errors.

