---
title: "Electronic Ewafermapping"
date: 2022-09-09T14:25:15+02:00
draft: true
---

# Why

In Melexis we test wafers which consist out of dies. These dies are in later processes turned into the chips we all know and test here in final test.  
The process of testing these wafers is called 'probing'; we basically check which dies are good and which are not conform our standards.

In a not so far past, we used to physically mark those bad dies with small ink dots. 
Marking these dies makes sure that these dies are not tested later on (with other testconditions) and subcontractors do not use them while dicing the wafer.
However this was a very costly operation and also resulted in a lot of loss due to ink spilling etc.

Therefore the inkless- and electronic wafermapping-applications were developed. By creating a visual representation of wafer, which we call a wafermap, we can deliver our subcontractors and later processes a clean inkless wafer, accompanied with a electronic wafermap. 

Inkless together with electronic wafermapping (EWAF) is the system responsible for importing such wafermaps (when we receive wafers from subcontractor or if subcons did some extra tests on these wafers) and exporting these wafermaps to subcons. The latter is very important; if our subcontractors/customers decide to start testing a lot of wafers at their end, we need to make sure our wafermaps are also available at their end. 

# How

## Importing wafermaps

When a subcontractor does some additional testing, they provide us wafermaps in an agreed upon format. This mostly happens by putting the maps on a (s)ftp-server. 
Next up we have services polling these (s)ftp's periodically and converting the wafermaps on the (s)ftp to our THxx format. 

Next up the wafermaps are put in our wafermapdatabase. The wafermaps can be used within Melexis from that point on; for example when stacking later on in postprocessing pure.

## Exporting wafermaps

When we test our wafers on several conditions, we tend to stack all relevant wafermaps (raw conditions, PAT, RIP...) into one postprocess-wafermap. This wafermap is a single wafermap where all results of the tested wafermaps for that lot and wafer are reflected in. When our planners decide to ship a lot to a customer or subcontractor, we typically also want to provide these customers/subcontractors our wafermaps. 

By doing this at this very moment in time, we make sure the wafermaps are present at the customer end before the wafers physically arrive. Again we put wafermap on an (s)ftp of the customer/subcon in an agreed upon format. This also means that at this time we convert the wafermap from THxx to an agreed upon format. For example SINF, CARSEM...

# What

There are several microservices here in play interacting with eachother. Also the flow is different for exporting and importing wafermaps.

## Importing wafermaps

We have several applications running which poll (s)ftp servers from partners. These applications are divide per format; for example all customers providing us maps in carsem format are bundled in the carsem application. Most of these applications are small services deployed on kubernetes, though we have some leftover services still running in the big EWAF application which is a docker image running on a docker host in ewaf.colo.elex.be. 

These applications are java/kotlin spring applications mainly built with [Apache Camel](https://camel.apache.org/)-components. We basically set up a route in which we poll the sftp, convert the wafermap from format X to THxx and put the map in our wafermap database with an HTTP post request.

We used to keep track of which wafermaps we imported in the past in a idempotent database. This prevent importing the same wafermaps over and over again on each poll. However polling huge folders seemed to be a problem. Therefore we now tend to move the processed wafermaps on the ftp in a _processed or _backup folder, depending on the customer. You can find this back in the camel-endpoints of the application.

## Exporting wafermaps

There are basically three triggers to send wafermaps to a customer/partner:
- An inventory move done in oracle to OSP IN = the lot went to status 'OUTSIDE PROCESSING' at a partner
- A red exception has been thrown when trying to ship a lot = the lot is going to get physically sold/sent to a customer
- A manual trigger is given in the website ewafermap.colo.elex.be

In all three cases, the big concept is the same.

### 1. viiperevents

viiperevents is the application which queries the oracle database for new inventory moves and 'red exceptions'. When such a row is queried, the row is converted into an event and put on the broker. That message in turn gets consumed by ewaf. 

### 2. inkless

When the message ends up in ewaf, ewaf sends a message to inkless in order to download the map and do some validations on the selected map. Note that inkless is downloading the postprocessing (stacked) wafermap from the cloud wafermap database. The validations are checking that the map has the correct shape (row/col count) and contains the right qty of dies inside the wafermap versus what is being sent by oracle. Next that message is sent back to ewaf.

### 3. ewaf and sidecars

The message from ewaf (containing a reference to all selected wafermaps which are temporarily stored in the wafermapdatastore) is ending up in ewaf. Note that next to ewaf, there are also more recent modules consuming these messages. This is a side effect of slowly decommissioning ewaf and in the meantime extract part by part of the application and deploy them as seperate services. 

Just like importing, these applications are grouped by format. For example we used to sent wafermaps to carsem in ewaf, but now the carsem module is a seperate service for which we can configure all customers who desire carsem maps.

Both EWAF and the sidecars (carsem, sinf, xfab) first check that if the event from inkless (containing the partner/customer name) are relevant for them. If so, the event gets consumed. Next up the wafermaps are downloaded and extra configuration is fetched from the ewaf config section. This config gets configured on ewafermap.colo.elex.be. Here the filename template can get set, the refdies can be configured etc... Depending on the service and format, different config might be needed or even obliged. 

When all config is fetched, the map gets converted to the desired format and are put on the (s)ftp of the customer/partner.

Note that this config module is also something which is running inside EWAF.

### 4. confirmations

For customers which get triggered with a red exception, there is a special rule. These lots can't physically get shipped in oracle as long as the red exception is present. The wafermaps first need to get confirmed at the customer end. This can be done by polling the same ftp to see if the maps we but there are actually present or in special cases by receiving an automated e-mail (conti). 

These confirmation messages (from ewaf, conti-confirmation or even seperate services themselves) are then again consumed by ewaf, whereafter ewaf is removing these red exceptions automatically. We do this to prevent sending wafers to customers (not partners for which the sla's are less strict) without the wafermaps being present on their end. If this would occur, we get big and costly claims from companies like conti because they have to stop production in order to ask the wafermaps to us.