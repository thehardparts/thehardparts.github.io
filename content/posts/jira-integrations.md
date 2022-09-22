---
title: "Jira integrations"
date: 2022-09-22T14:25:15+02:00
draft: true
---

# Why

In Jira there are a lot of functionalities coming out of the box. We have validators, conditions and post functions to do more complex stuff.  
In some cases we even use scripted fields which get populated based on a script running in the background.

With the coming of Jira Cloud; even more possibilities were added under the form of automations. These are basically extended post functions and can do way more complex things. Maybe even way too complex...

However in some cases these possibilities are not enough:

- The desired functionality is not available
- Maintaining scripts is painful without version control systems (git)
- When business logic is involved, we rather keep isolated from Jira itself
- ...

In these cases we tend to develop a sidecar service; een service running seperatly. This can be a cloud function, kubernetes pod, app engine instance...  
Typically these sidecars talk to Jira through the REST-api. Several libraries exist to simplify this process.

# How

We communicating with Jira, we typically see these use cases:

## Simple scripts

Some people tend to create scripts which interact with the API to simplify their day to day work. For example some people have scripts running to log time through the rest API instead of opening up the UI. Others have scripts to determine the time logging of certain tickets and to fetch management metrics from tickets. Others have scripts which move tickets to a certain state if a merge is done in git. The possibilities are endless.

## Jira as datasource

Some services use Jira as a datasource to fetch data and transform the data is something else. For example qualitysidecar is fetching CRC-tickets in order to create a slide deck based on the information inside a ticket. In CSR we also extract the data from jira into another database so other users can consult certain data in a view which is ready to be consumed. 

## Jira as a target

In some cases we want to trigger some actions in Jira. For example if action A happens; we want to put a comment on ticket B and move it to the next stage. Action A can be everything; a click on a website button, a cron job triggering every few seconds etc...

## Jira as trigger

Last but not least we can use 'events' in jira to trigger an external action, these are called webhooks. In jira you can define which webhooks need to be present and when you want to send out an event. For example on ticket creation, ticket updating, putting a comment ticket...  
These actions can be used as an action to trigger a 'jira as a target'-application, or can trigger other things in the background. In case of CSR we have webhooks configured which create a spreadsheet based on jira input when a ticket moves to a certain state.

### All sidecars

Most of the sidecars within Melexis you can find here: https://gitlab.melexis.com/cbs/jira-sidecars. 

# What

## Credentials

In order to be able to talk with the API, we need a token as jira server does not support username/password authenticating anymore. We need to create this token [here](https://id.atlassian.com/manage-profile/security/api-tokens). Keep good track of this token as you can reuse it a lot. Note that you can create a token for every use case you have in order to distinguish later on what token did which call.

## Simple CURL examples

When you created a token, you can simply test it out by doing an http request:

    curl -u "ars@melexis.com:<TOKEN>" https://mlxtest.atlassian.net/rest/api/2/issue/SCPM-831 

When all is working fine, you'll get a request like this: 

    {"expand":"renderedFields,names,schema,operations,editmeta,changelog,versionedRepresentations,customfield_17052.requestTypePractice,customfield_17662.cmdb.label,customfield_17662.cmdb.objectKey,customfield_17662.cmdb.attributes,customfield_17531.properties,customfield_17538.cmdb.label,customfield_17538.cmdb.objectKey,customfield_17538.cmdb.attributes,customfield_17520.properties,customfield_17519.properties","id":"4989805","self":"https://mlxtest.atlassian.net/rest/api/2/issue/4989805","key":"SCPM-831","fields":{"customfield_17263":null,"customfield_17262":null,"parent":{"id":"4989792","key":"SCPM-818","self":"https://mlxtest.atlassian.net/rest/api/2/issue/4989792","fields":{"summary":"Functional test L567 2022-09-21 09:46:53.867445","status":{"self":"https://mlxtest.atlassian.net/rest/api/2/status/6","description":"The issue is considered finished, the resolution is correct. Issues which are closed can be reopened.","iconUrl":"https://mlxtest.atlassian.net/images/icons/statuses/closed.png","name":"Closed","id":"6","statusCategory":{"self":"https://mlxtest.atlassian.net/rest/api/2/statuscategory/3","id":3,"key":"done","colorName":"green","name":"Done"}},"priority":{"self":"https://mlxtest.atlassian.net/rest/api/2/priority/5","iconUrl":"https://mlxtest.atlassian.net/images/icons/priorities/trivial.svg","name":"Trivial","id":"5"},"issuetype":{"self":"https://mlxtest.atlassian.net/rest/api/2/issuetype/11502","id":"11502","description":"","iconUrl":"https://mlxtest.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/19880?size=medium","name":"New Process","subtask":false,"avatarId":19880,"hierarchyLevel":0}}},"customfield_17261":null,"customfield_17267":null,"customfield_17266":null,"customfield_17264":null,"customfield_17269":null,"customfield_17268":null,"customfield_10750":null,"resolution":null,"customfield_10751":null,"customfield_10752":null,"customfield_10984":null,"customfield_10985":null,"customfield_10986":null,"customfield_10987":null,"customfield_10988":null,"customfield_10989":null,"customfield_17252":null,"lastViewed":"2022-09-21T12:01:59.181+0200","customfield_17251":null,"customfield_17256":null,"customfield_17255":null,"customfield_17259":null,"customfield_17258":null,"customfield_17257":null,"customfield_10980":null,"customfield_10981":null,"customfield_10982":null,"customfield_11951":null,"customfield_11950":null,"customfield_10983":null,"customfield_10973":null,"customfield_10974":null,"customfield_10975":null,"customfield_10976":null,"customfield_10977":null,"aggregatetimeoriginalestimate":null,"customfield_10978":null,"customfield_10979":null,"issuelinks":[],"assignee":{"self":"https://mlxtest.atlassian.net/rest/api/2/user?accountId=6130ce07f0bf520069579257","accountId":"6130ce07f0bf520069579257","emailAddress":"pab@melexis.com","avatarUrls":{"48x48":"https://secure.gravatar.com/avatar/b4c9a5d5adfeac7fbdaa7bf89443fef0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FPB-3.png","24x24":"https://secure.gravatar.com/avatar/b4c9a5d5adfeac7fbdaa7bf89443fef0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FPB-3.png","16x16":"https://secure.gravatar.com/avatar/b4c9a5d5adfeac7fbdaa7bf89443fef0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FPB-3.png","32x32":"https://secure.gravatar.com/avatar/b4c9a5d5adfeac7fbdaa7bf89443fef0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FPB-3.png"},"displayName":"Paschasia Bisschop","active":true,"timeZone":"Europe/Paris","accountType":"atlassian"},"customfield_10970":null,"customfield_10971":null,"customfield_10972":null,"customfield_10851":null,"customfield_10962":null,"customfield_10964":null,"customfield_10966":null,"customfield_10967":null,"customfield_10968":null,"customfield_10969":null,"customfield_17351":null,
    ...

Note that this giant thing is a JSON representation of the ticket SCPM-831. You'll find back a lot of information if you search for certain aspects in it. For example: 

    "assignee":{"self":"https://mlxtest.atlassian.net/rest/api/2/user?accountId=6130ce07f0bf520069579257","accountId":"6130ce07f0bf520069579257","emailAddress":"pab@melexis.com","avatarUrls":{"48x48":"https://secure.gravatar.com/avatar/b4c9a5d5adfeac7fbdaa7bf89443fef0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FPB-3.png","24x24":"https://secure.gravatar.com/avatar/b4c9a5d5adfeac7fbdaa7bf89443fef0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FPB-3.png","16x16":"https://secure.gravatar.com/avatar/b4c9a5d5adfeac7fbdaa7bf89443fef0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FPB-3.png","32x32":"https://secure.gravatar.com/avatar/b4c9a5d5adfeac7fbdaa7bf89443fef0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FPB-3.png"},"displayName":"Paschasia Bisschop","active":true,"timeZone":"Europe/Paris","accountType":"atlassian"}

    "summary":"Functional test L567 2022-09-21 09:46:53.867445"

Note that you'll see a lot of other fields as well which don't sound that familiar:

    "customfield_17268":null

This is a customfield defined in Jira; but you only see the internal ID. In order to know which field it represents you need to get the fields mapping: 

    curl -u "ars@melexis.com:<TOKEN>" https://mlxtest.atlassian.net/rest/api/3/field

Again you get a giant response where you can lookup your field:

    {"id":"customfield_17268","key":"customfield_17268","name":"Obsoleted on docserver","untranslatedName":"Obsoleted on docserver","custom":true,"orderable":true,"navigable":true,"searchable":true,"clauseNames":["cf[17268]","Obsoleted on docserver","Obsoleted on docserver[Checkboxes]"],"schema":{"type":"array","items":"option","custom":"com.atlassian.jira.plugin.system.customfieldtypes:multicheckboxes","customId":17268}}

Here you see the weird field is the custom fields [Obsoleted on docserver](https://mlxtest.atlassian.net/secure/admin/ConfigureCustomField!default.jspa?customFieldId=17268) and it's a dropdown field with multiple options. 

## Postman

Some people prefer a graphical interface to do these commands rather than a command line tool. Postman is here a good solution.  
In this tool you can do the same as described above; but in a graphical way. Also code snippets can get generated if you want to.

1. Let's create a ticket! First consult the api documentation:

    https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/#api-rest-api-3-issue-post

2. Next set up the authentication:

    ![postman1](/thehardparts/postman1.png)

    Note in the headers section you'll find back your information encoded in authorization section of your request.

3. Let's post our body:

    ![postman2](/thehardparts/postman2.png)

    For more info about RESTful API's consult this [page](https://en.wikipedia.org/wiki/Representational_state_transfer).

4. Observe the response:

    ![postman3](/thehardparts/postman3.png)

    Note that your response ends up in the 2xx response range; which means you got a succes response. Other response codes can indicatie other issues.  
    For example if you search for an issue which does not exist you'll get a 404 response. Not sure what the response code means? Look it up at https://http.cat/. 

## Client libraries

While doing these http requests yourselves is perfectly possible for all kind of programming languages, a lot of programming languages offer an abstraction layer on top. We'll briefly discuss some examples.

### Python

A nice library to interact with Python is this [one](https://jira.readthedocs.io/examples.html#quickstart)

Instead of having to specify the full HTTP request; this snippet does the trick:

    from jira import JIRA

    jira = JIRA(<URL>, basic_auth=(<email>, <token>))
    issue = jira.issue("JRA-1330")

    # Comment an issue
    jira.add_comment(issue, "Comment text")

    # Change the issue's summary and description.
    issue.update(
        summary="I'm different!", description="Changed the summary to be different."
    )

    # Change the issue without sending updates
    issue.update(notify=False, description="Quiet summary update.")

    # You can update the entire labels field like this
    issue.update(fields={"labels": ["AAA", "BBB"]})

    # Or modify the List of existing labels. The new label is unicode with no
    # spaces
    issue.fields.labels.append("new_text")
    issue.update(fields={"labels": issue.fields.labels})

    # Send the issue away for good.
    issue.delete()

### Java/kotlin

Unfortunately atlassian is no longer offering a maintained java library. Therefore we created our own: https://gitlab.melexis.com/cbs/jira-sidecars/jira-cloud-lib

Again; a lot of the http requests are abstracted away in simple method calls:

    	val jiraUrl: String = System.getenv("BASE_URL")
		val jiraUser: String = System.getenv("SCAR_API_USERNAME")
		val jiraPassword: String = System.getenv("SCAR_API_PASSWORD")
		jiraService = Config.createJiraService(jiraUrl, jiraUser, jiraPassword)

        fun createJiraService(url: String, user: String, password: String): JiraService {
            val apiClient = ApiClient(RestTemplate())
            apiClient.apply {
                setUsername(user)
                setPassword(password)
                basePath = url
            }
            return JiraService(apiClient)
        }

        private fun JiraService.createIssue(project: String, values: Map<String, Any>): String {
            val map : MutableMap<String, Any> = mutableMapOf(
                "project" to mapOf("key" to project),
                "issuetype" to mapOf("id" to "27".toLong()),
                "summary" to "testje",
                "description" to JiraService.CommentBodyBuilder("Test").build(),
                "assignee" to mapOf("accountId" to "61e8022893885f00696a5e59")
		)

		val fieldMapping = IssueFieldsApi(apiClient).fields.associate { it.name to it.id }

		values.forEach{ (key, value) -> map[fieldMapping[key] ?: throw Error("no key found for $key")] = value }
		val issuesApi = IssuesApi(apiClient)
		return issuesApi.createIssue(mapOf("fields" to map), false).key
	}

