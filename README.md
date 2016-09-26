# VeriCite LMS API documentation

This document will walk through the Sakai LMS integration with VeriCite to explain the steps required to integrate an LMS with VeriCite internally. You should follow along with the project: [Sakai LMS Integration](https://github.com/vericite/contentreview-impl-vericite) If you are not comfortable with Java, there is a Moodle LMS integration written in php that follows the same guidelines: [Moodle LMS Integration](https://github.com/vericite/moodle-plagiarism_vericite).

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
An example of this code in use can be found in our Canvas integration: [vericite.rb](https://github.com/baholladay/canvas-lms/blob/vericite-master/lib/vericite.rb)

  * **NodeJS/Javascript**  
We recommend using [Swagger JS library](https://github.com/swagger-api/swagger-js). This is a dynamic library which takes the [VeriCiteLmsApiV1.json](https://github.com/vericite/LMS-API/blob/master/VeriCiteLmsApiV1.json) swagger json definition that we provide.  
Ex.:
`
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

`

  * **Java**  
We will be adding our API to a maven repository. An example of this code in use can be found in our Sakai integration: [Sakai Content Review](https://github.com/vericite/contentreview-impl-vericite)

  * **Php**  
We will be adding our API to a php repository. An example of this code in use can be found in our Moodle integration: [Moodle VeriCite Plugin](https://github.com/vericite/moodle-plagiarism_vericite)

For all other languages, please put in a request and we will respond. We have also provided a [Swagger](http://swagger.io/) definition for our LMS API: [VeriCiteLmsApiV1.json](https://github.com/vericite/LMS-API/blob/master/VeriCiteLmsApiV1.json).
