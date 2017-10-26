# VeriCite LMS API documentation

This document will walk through the Sakai LMS integration with VeriCite to explain the steps required to integrate an LMS with VeriCite internally. You should follow along with the project: [Sakai LMS Integration](https://github.com/vericite/contentreview-impl-vericite) If you are not comfortable with Java, there is a Moodle LMS integration written in php that follows the same guidelines: [Moodle LMS Integration](https://github.com/vericite/moodle-plagiarism_vericite). We have also provided a [Postman](https://www.getpostman.com/apps) export for all available endpoints in our API: [VeriCite-LMS_API.postman_collection.json](https://github.com/vericite/LMS-API/blob/master/VeriCite-LMS_API.postman_collection.json)

## Best practices:
  * If the user doesn’t need to wait for a confirmation, do the work in a non-user thread so that they are not blocked (e.g. loading) until the VeriCite call is complete.
  * Look up scores in bulk as much as possible. For example, an instructor will have access to every submission in a site, so you might as well look up all scores at once and either cache them or store them in the database.
  * Scores that are cached or stored in a database should expire in ~20 minutes. VeriCite scores are dynamic and can change, so the LMS should never hold on to the scores for long periods of time.
  * Tokens should be used when any user facing link is displayed. A user **SHOULD NEVER** be able to see the consumer password in any HTML.
  * Token creation should be granted at the highest level accessible for that user. The more details you pass in with the creation of a token, the more restrictive it will be. For example, when creating a token for a Learner, make sure it includes the user, externalId, course, and role. When creating a token for an instructor, you can make it less restricted by sending in only the user, course and role. This way, you can cache this token and reuse it for any paper submitted to that course.
  * VeriCite will expire tokens after 30 minutes. Make tokens expire after ~20 minutes in the LMS so that the user doesn’t get presented with a URL that has an expired token.
  * Send as much data as you can in most requests. For example, if you are uploading a document for a user, you can send the user’s id, first name, last name, email, ect. and it will be updated/stored. This is good for the case of new users or new assignments that VeriCite hasn't seen. VeriCite will automatically create a record for the new user or assignment and set the data fields provided.
  * VeriCite is resilient. If a report doesn’t return a score, try re-submitting the report. There are no APIs for creating users, sites, assignemnts, context title, ect, VeriCite will create them if they don’t exist when you submit reports or update any details (e.g. assignment).
  * URL encode every parameter. If you use the Swagger export, this should be handled for you automatically.

## VeriCite LMS API Libraries

  * **Ruby**  
[VeriCite LM Ruby API](https://rubygems.org/gems/vericite_api)
An example of this code in use can be found in our Canvas integration: [vericite.rb](https://github.com/instructure/canvas-lms/blob/master/lib/vericite.rb)

  * **NodeJS/Javascript**  
We recommend using [Swagger JS library](https://github.com/swagger-api/swagger-js). This is a dynamic library which takes the [VeriCiteLmsApiV1.json](https://github.com/vericite/LMS-API/blob/master/VeriCiteLmsApiV1.json) swagger json definition that we provide.  
Ex.:
```javascript
import Swagger from 'swagger-client';
this.client = new Swagger({
  url: 'https://github.com/vericite/LMS-API/blob/master/VeriCiteLmsApiV1.json',
  success: function() {
    console.log("Client created");
    console.log(this.client.help());
    this.client.setBasePath("/lms/v1");
  }.bind(this),
  fail: function() {
    console.error("Client failed");
  }
});
```

  * **Java**  
We will be adding our API to a maven repository. An example of this code in use can be found in our Sakai integration: [Sakai Content Review](https://github.com/vericite/contentreview-impl-vericite)

  * **Php**  
We will be adding our API to a php repository. An example of this code in use can be found in our Moodle integration: [Moodle VeriCite Plugin](https://github.com/vericite/moodle-plagiarism_vericite)

  * **Python**  
[VeriCite LM Python API](https://github.com/vericite/vericite_api_python)
An example of this code in use can be found in the README file in the project


For all other languages, please put in a request and we will respond. We have also provided a [Swagger](http://swagger.io/) definition for our LMS API: [VeriCiteLmsApiV1.json](https://github.com/vericite/LMS-API/blob/master/VeriCiteLmsApiV1.json).

## Sample Integration Walkthrough

You should follow along with our Java example project: [Sakai LMS Integration](https://github.com/vericite/contentreview-impl-vericite) or our PHP example project: [Moodle LMS Integration](https://github.com/vericite/moodle-plagiarism_vericite). We also suggest plugging in our [VeriCite LMS Swagger Definition](https://github.com/vericite/LMS-API/blob/master/VeriCiteLmsApiV1.json) into the [Swagger Editor](http://editor.swagger.io/#/) to see an interactive display of our API. We have also provided a [Postman](https://www.getpostman.com/apps) export for all available endpoints in our API: [VeriCite-LMS_API.postman_collection.json](https://github.com/vericite/LMS-API/blob/master/VeriCite-LMS_API.postman_collection.json)

### Step 1: Assignment is Created or Updated:

Whenever an assignment is created or updated, pass this information over to VeriCite. This isn’t required, but just makes the integration nicer for the user. There is no harm to calling this function everytime a user makes a modification to the assignment. Remember to do this work in a non-user thread as to not block the user and to pass all parameters in encoded.

#### assignments.createUpdateAssignment

POST /assignments/{contextID}/{assignmentID}  
Creates or updates an assignment

*Authorization Headers:*
  * consumer
  * consumerSecret

*Parameters:*
  * contextID (from URL; this is the site id)
  * assignmentID (from URL; this is the assignment id)
  * encrypted (always set to true, this was used as a migration flag and will go away)
  * assignmentData (body json parameter)
  * assignmentData.assignmentTitle (if not set, assignmentId will be used)
  * assignmentData.assignmentInstructions
  * assignmentData.assignmentExcludeQuotes
  * assignmentData.assignmentExcludeSelfPlag
  * assignmentData.assignmentStoreInIndex
  * assignmentData.assignmentDueDate
  * assignmentData.assignmentGrade
  * assignmentData.assignmentAttachmentExternalContent (a list of attachments for this assignment. Not required. For every attachment provided, VeriCite will return an upload URL to store the attachment. If you want to delete an attachment, include this parameter without the attachment externalContentID)
  * assignmentData.assignmentAttachmentExternalContent.fileName (file name without extension)
  * assignmentData.assignmentAttachmentExternalContent.uploadContentType (file extension, i.e. pdf, txt, rtf, etc.)
  * assignmentData.assignmentAttachmentExternalContent.uploadContentLength (size of the file in bytes)
  * assignmentData.assignmentAttachmentExternalContent.externalContentID (this is your UID for the attachment)

  *Response:*
  ```javascript
  [
      {
          "urlPost": "https://someurl/147903070164393.reportgen?AWSAccessKeyId=AKIAJBIOQ&Expires=1507669870&Signature=4mH%2BRxLR%2B%2F2tvpqlq4jhoo%3D",
          "externalContentId": "abc12345",
          "filePath": "key/2017/10/10/147903070164393.reportgen",
          "contentLength": 4977,
          "contentType": ".rtf",
          "paperId": 147903070173110,
          "metaDataId": 147903070164393,
          "headers": {
              "x-amz-server-side-encryption": "AES256"
          }
      }
  ]
  ```
  Response is a list of URLs to send a "PUT" request with the raw file data (or empty if you did not include attachments). Make sure you dynamically set the PUT request headers based on the response headers field and do not add any extra headers (this is for the signature).

*Java example:*  
[ContentReviewServiceImpl.createAssignment](https://github.com/vericite/contentreview-impl-vericite/blob/88f084abda1a2f0aa573868c8f98b5924f235888/impl/src/java/org/sakaiproject/contentreview/impl/ContentReviewServiceImpl.java#L105)  
*Php example:*  
[lib.php save_form_elements()](https://github.com/vericite/moodle-plagiarism_vericite/blob/db75c8505ea4cc6b6db4e55ca4cc4b86271666d2/lib.php#L410)  

### Step 2: Submit a paper

When a user submits a paper, VeriCite will create or update any site, assignment or user connected to the paper. There is no need to create these objects in a different call. Your architecture should make this call in a queueing process. For example, when a user submits a paper, the paper should be put up on a queue. The queue reader would request the paper submission upload URL. The VeriCite response will include an upload URL for you to then upload the file submission to. Pass in as much data as possible for the user, assignment and context.

### reports.submitRequest

POST /reports/submit/request/{contextID}/{assignmentID}/{userID}  
Request a file submission upload URL

*Authorization Headers:*
  * consumer
  * consumerSecret

*Parameters:*
  * contextID (from URL; this is the site id)
  * assignmentID (from URL; this is the assignment id)
  * userID (from URL; this is the user id)
  * encrypted (always set to true, this was used as a migration flag and will go away)
  * reportMetaData (from body as json. Only required fields in this object externalContentData: A minimum of one request with externalContentID, fileName and uploadContentLength set. Please provide all parameters if possible)
  * reportMetaData.userFirstName
  * reportMetaData.userLastName
  * reportMetaData.userEmail
  * reportMetaData.userRole (must be “Instructor” or “Learner”; default to Learner)
  * reportMetaData.contextTitle
  * reportMetaData.assignmentTitle
  * reportMetaData.externalContentData.externalContentId (this is your UID for the paper)
  * reportMetaData.externalContentData.fileName (file name without extension)
  * reportMetaData.externalContentData.uploadContentType (file extension, i.e. pdf, txt, rtf, etc.)
  * reportMetaData.externalContentData.uploadContentLength (size of the file in bytes)

*Response:*
```javascript
[
    {
        "urlPost": "https://someurl/147903070164393.reportgen?AWSAccessKeyId=AKIAJBIOQ&Expires=1507669870&Signature=4mH%2BRxLR%2B%2F2tvpqlq4jhoo%3D",
        "externalContentId": "abc12345",
        "filePath": "key/2017/10/10/147903070164393.reportgen",
        "contentLength": 4977,
        "contentType": ".rtf",
        "paperId": 147903070173110,
        "metaDataId": 147903070164393,
        "headers": {
            "x-amz-server-side-encryption": "AES256"
        }
    }
]
```
Response is a list of URLs to send a "PUT" request with the raw file data. Make sure you dynamically set the PUT request headers based on the response headers field and do not add any extra headers (this is for the signature).



*Java example:*  
[ContentReviewServiceImpl.queue](https://github.com/vericite/contentreview-impl-vericite/blob/88f084abda1a2f0aa573868c8f98b5924f235888/impl/src/java/org/sakaiproject/contentreview/impl/ContentReviewServiceImpl.java#L578)  
*Php example:*  
[send_files.php plagiarism_vericite_send_files()](https://github.com/vericite/moodle-plagiarism_vericite/blob/master/classes/task/send_files.php#L20)  


### Step 3: Get Report Scores

Do this in bulk when possible. For Learners, you’ll probably request scores for one assignment and one user (e.g. the Learner) at a time and this is ok. However, for instructors, you’ll want to request scores for all users in an assignment. You can then cache/store these scores for 30 minutes. Do not store the scores for any longer of a period of time since VeriCite scores are dynamic and can change. *IMPORTANT*: If a score isn’t returned for a submission, then this means that VeriCite can’t find the submission. Simply re-queue the report for the missing attachment (described in Step 2). Remember to pass all parameters in encoded.

Alternatively, you can register a `report_score_changed` event webhook to be notified of any score changes to a report. Read more about our webhooks below.

#### reports.getScores

GET /reports/scores/{contextID}  
Retrieves scores for the reports  

*Authorization Headers:*
  * consumer
  * consumerSecret

*Parameters:*
  * contextID (from URL; this is the site id)
  * assignmentID (query; filter by assignment id)
  * userID (query; filter by user id)
  * externalContentID (query; find exact report by externalContentID)

*Java example:*  
[ContentReviewServiceImpl.getReviewScore](https://github.com/vericite/contentreview-impl-vericite/blob/88f084abda1a2f0aa573868c8f98b5924f235888/impl/src/java/org/sakaiproject/contentreview/impl/ContentReviewServiceImpl.java#L406)  
*Php example:*  
[lib.php plagiarism_vericite_get_scores()](https://github.com/vericite/moodle-plagiarism_vericite/blob/master/lib.php#L577)  


### Step 4: Get access urls

Access URLs are used to allow users to see a report. These are temporary URLs that will expire in 30 minutes. You can re-use them, so it is ok to cache them for 20 minutes. For Learners, you’ll probably request URLs for one assignment and one user (e.g. the Learner) at a time and this is ok. However, for instructors, you’ll more likely want to request URLs for all reports in an assignment in bulk and cache the results.

#### reports.getReportUrls

GET /reports/urls/{contextID}  
Retrieves URLS for the reports

*Authorization Headers:*
  * consumer
  * consumerSecret

*Parameters:*
  * contextID (from URL; this is the site id)
  * assignmentIDFilter (query; filter by assignment id)
  * userIDFilter (query; filter by user id)
  * externalContentIDFilter (query; find exact report by externalContentID)
  * tokenUser (ID of user who will view the report)
  * tokenUserRole (role of user who will view the report)

*Java example:*  
[ContentReviewServiceImpl.getAccessUrl](https://github.com/vericite/contentreview-impl-vericite/blob/88f084abda1a2f0aa573868c8f98b5924f235888/impl/src/java/org/sakaiproject/contentreview/impl/ContentReviewServiceImpl.java#L303)  
*Php example:*  
[lib.php PLAGIARISM_VERICITE_ACTION_REPORTS_URLS](https://github.com/vericite/moodle-plagiarism_vericite/blob/master/lib.php)  

## Webhooks

VeriCite webhooks allows you to register your own API endpoints to be called by VeriCite on specific events. For example, instead of requesting the scores from VeriCite, you can register for the `report_score_changed` event. This will notify your API with any changes to any report for each registered consumer.

The first step for using webhooks is to register your API endpoint to a specific event. This should be done at the same time a consumer's key and secret are configured. For example, each consumer should have a unique key and secret associated to them in your implementation. When you save this data, you should:
  * Get a list of all webhooks associated with this consumer.
  * Go through each webhook and decide whether to keep or delete it.
  * For any missing webhooks, make a `webhooks.createWebhook` call to register it for that consumer.

### Register a Webhook

#### webhooks.createWebhook

POST /webhooks
Registers a webhook for a specific client, event and URL.

*Authorization Headers:*
  * consumer
  * consumerSecret

*Parameters:*
  * webhook (body json parameter)
  example:
  ```javascript
  {
    "event": "report_score_changed",
    "url": "https://mydomain/webhooks/event"
  }
  ```
*Response:*
If successful, the response will be the newly registered webhook.
  ```javascript
  {
    "event": "report_score_changed",
    "url": "https://mydomain/webhooks/event"
  }
  ```


### List Webhooks

#### webhooks.getWebhooks

GET /webhooks
Retrieves webhooks for the consumer.

*Authorization Headers:*
  * consumer
  * consumerSecret

*Parameters:*
  * event (optional query parameter to filter results by event)

*Response:*
If successful, the response will be a list of all registered webhooks:
  ```javascript
  [
    {
      "event": "report_score_changed",
      "url": "https://mydomain/webhooks/event"
    }
  ]
  ```


### Delete Webhook

#### webhooks.deleteWebhook

DELETE /webhooks
Delete webhook for event and URL.

*Authorization Headers:*
  * consumer
  * consumerSecret

*Parameters:*
  * webhook (body json parameter)
  example:
  ```javascript
  {
    "event": "report_score_changed",
    "url": "https://mydomain/webhooks/event"
  }
  ```
*Response:*
If successful, the response will be the deleted webhook.
  ```javascript
  {
    "event": "report_score_changed",
    "url": "https://mydomain/webhooks/event"
  }
  ```

### Webhook Events

All webhooks events will follow the same pattern:
  * OAuth1 POST event message will be signed with the consumer's key and secret
  * `event` is the registered event type
  * `consumer` it the registered consumer for this event.
  * `payload` consist of a list of results. This list will be unique to each event.


#### report_score_changed

This event will be triggered when a report score has been updated. This includes the report score, draft status and preliminary status. It will send an OAuth1 POST message signed with the consumer's key and secret. The message is a list of report score data similar to `reports.getScores`.

*Payload:*
```javascript
  {
    "payload": [
        {
            "user": "23asdfsadf",
            "assignment": "06561f2e-8ff1-4aa0-a501-58e892ae5ec4",
            "externalContentId": "149108935274859",
            "score": 0,
            "preliminary": true,
            "draft": false
        }
    ],
    "event": "report_score_changed",
    "consumer": "example"
  }
```



## Summary overview:

You'll want to do all of the server to server communication in server side code since you should never expose the client's key and secret. You will also need to keep track of each client's key and secret for VeriCite and not just use a single one for everyone. You should have the ability for clients to set their default settings that would apply to the new assignments/activity. For example, the VeriCite API has the following advanced options: "Exclude Quotes", "Exclude Self Plagiarism" and "Store in Institutional Index". Other LMS also include settings such as "Allow students to view score" (some have the release based on times like immediately, after due date, after graded, never) or "Allow students to view reports". If you have any importing/exporting of assignments/activities, you'll want to make sure these settings persist. When it comes to our API, you should do bulk calls as much as possible and have restrictions in there that make sure you do not call our API's excessively (i.e. check for the same report's score every second until it is ready). This is handled in the other LMS's by assigning a "last retrieved" time stamp to the objects so that you only request that object's scores/urls every 5 minutes at most. VeriCite has two types of scores: preliminary and final as noted by the "preliminary" flag. Preliminary scores should be "cached" for 5 minutes before asking for the score again. "Final" scores should be cached 30+ minutes before asking for the scores again. VeriCite scores are dynamic, so you should not display a stored score that is more than 24hrs old (you can store it, just check the "time retrieved" value and ask our servers for an updated score).

URL requests have two parts to them: 1) what URLs do you want access to 2) who do you want to grant access to. For example, as instructor (role=Instructor, user=1) wants to view all student's reports for Assignment A. The "token user and role" in this case is user=1 and role=Instructor and the "filter" params would be the highest (i.e. bulk) level of filtering, so context=X and assignment=A. This will retrieve URLs for all reports in that assignment with permissions granted at the Instructor level. Alternatively, students would only be allowed to see their own URLs. You would set token user to be user/role=Student and the filter to be their exact report externalId=reportID. This will return only the one report with permissions for a Student. You should cache URLs, however URLs expire after about 60 minutes and we recommend only caching the URLs for 30 minutes. This means if the user reloads the page, you should not call our API and just use the same URL unless that URL is older than 30 minutes.

For the queue service to submit the papers to VeriCite, you will want to make sure it is done as a background job. Once you submit a paper, the preliminary score should be ready fairly quick (5-10mins). The final score should be ready within 1 hr (but allow up to 24 hrs in case there are issues with our service for any reason). When user request scores/urls, VeriCite will return either a score >= 0, score = -1 or empty result. An empty result is an indicator that we have no record of the report at all. If scores return "-1", this is an indicator that the preliminary score isn't complete (if its been past an hour, this report needs requeued). In both cases, if enough time has passed, these reports will need to be resubmitted. There are three approaches to determining whether the reports were successfully submitted/generated and requeuing if not. 1) Automatically check for these cases when a user loads the page to get the scores. If report needs requeued, automatically re-add the report to the queue service. 2) After a paper is submitted, run a background job that checks for the scores of the reports and whether they need to be requeued. With this approach, you should use a logarithmic time check with a ceiling (e.g. 2mins, 4 mins, 8 mins, 16 mins, 32 mins, 64 mins, 64 mins, 64 mins, etc for up to 24hrs). If it fails, then requeue. 3) Give the user the ability to re-submit the report manually. In all cases, the VeriCite API is smart enough to know if you just recently re-submitted the report or not and given the timing, it will or will not return a URL for you to submit to. You should have a limit to the number of requeues before considering the report as failed.

We will expect to see all of these best practices. It is best to print out the calls/parameters/responses to our APIs to review and make sure they are correctly implemented. Also make sure to apply courtesy measures to our API's, such as bulk calls whenever possible and no more than 1 call per minute for the same request.
