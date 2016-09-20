<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [Zendesk Tracker](Zendesk Tracker.md)

**THIS WEBHOOK HAS NOT BEEN RELEASED YET.**

This guide will explain how to configure your Zendesk account so that whenever a ticket is created, closed, or commented on, Zendesk will send the relevant information to your Snowplow collector in the form of an unstructured event.

1. [Setting up a Cloudfront collector as a Zendesk extension](#extension)
2. [Setting up a trigger to track ticket creation](#ticket-creation)
3. [Setting up a trigger to track comment creation](#comment-creation)
4. [Setting up an automation to track tickets being closed](#ticket-resolution)

<a name="extension" />
## 1. Setting up a Cloudfront collector as a Zendesk extension

You can configure Zendesk to automatically send GET requests to a collector. The first step is to set up a Zendesk "extension" pointing at the collector.

Log in to Zendesk. Click the cogwheel-shaped "Admin" icon located at the bottom-left corner of the Dashboard page to take you to the Admin page.

In the "SETTINGS" menu, click on "Extensions":

[[/setup-guide/images/webhooks/zendesk/extensions-button.png]]

Click "add target":

[[/setup-guide/images/webhooks/zendesk/add-extension.png]]

Choose "URL target" from the list of target types to add:

[[/setup-guide/images/webhooks/zendesk/extension-form.png]]

Name the new extension something like "Cloudfront-Collector".

In the URL field, enter "http://d34uzc5hjrimh8.cloudfront.net/i?e=ue", replacing "d34uzc5hjrimh8" with your Cloudfront subdomain.

You can also add "&tna={{MY ZENDESK NAMESPACE}}" to this URL, where {{MY ZENDESK NAMESPACE}} is a label for the extension. This label will be attached to all events fired by the extension, so you can later check where a given event came from.

Set the Attribute Name field to "ue_pr". Whenever a trigger uses the extension to fire an event, a "ue_pr" field will be added to the GET request querystring containing the event's properties.

Select "Create Target" from the drop-down menu and click the Submit button.

[[/setup-guide/images/webhooks/zendesk/url-target.png]]

We have set up our collector as a Zendesk extension. We can now add triggers or automations which send GET requests to the collector whenever certain events occur.

<a name="ticket-creation" />
## 2. Setting up a trigger to track ticket creation

From the Admin page, select "Triggers" from the "BUSINESS RULES" menu:

[[/setup-guide/images/webhooks/zendesk/business-rules.png]]

Click "add trigger": 

[[/setup-guide/images/webhooks/zendesk/add-trigger-button.png]]

Name the trigger something like "Ticket-Creation-Trigger".

From the drop-down menus in the "Meet all of the following conditions" section, select "Ticket: Is..." and "Created".

In the "Perform these actions" section, select "Notifications: Notify target" and "Cloudfront-Collector".

In the Message box, paste the following:

```javascript
{"data":{"data":{"via":"{{ticket.via}}","ticket_type":"{{ticket.ticket_type}}","updated_at":"{{ticket.updated_at}}","assignee":{"first_name":"{{ticket.assignee.first_name}}","last_name":"{{ticket.assignee.last_name}}","name":"{{ticket.assignee.name}}","language":"{{ticket.assignee.language}}","tags":"{{ticket.assignee.tags}}","locale":"{{ticket.assignee.locale}}","notes":"{{ticket.assignee.notes}}","time_zone":"{{ticket.assignee.time_zone}}","id":"{{ticket.assignee.id}}","phone":"{{ticket.assignee.phone}}","extended_role":"{{ticket.assignee.extended_role}}","role":"{{ticket.assignee.role}}","details":"{{ticket.assignee.details}}","signature":"{{ticket.assignee.signature}}","organization":"{{ticket.assignee.organization}}","external_id":"{{ticket.assignee.external_id}}","email":"{{ticket.assignee.email}}"},"url_with_protocol":"{{ticket.url_with_protocol}}","id":"{{ticket.id}}","title":"{{ticket.title}}","priority":"{{ticket.priority}}","score":{{ticket.score}},"updated_at_with_timestamp":"{{ticket.updated_at_with_timestamp}}","current_user":{"first_name":"{{ticket.current_user.first_name}}","last_name":"{{ticket.current_user.last_name}}","name":"{{ticket.current_user.name}}","language":"{{ticket.current_user.language}}","tags":"{{ticket.current_user.tags}}","locale":"{{ticket.current_user.locale}}","notes":"{{ticket.current_user.notes}}","time_zone":"{{ticket.current_user.time_zone}}","id":"{{ticket.current_user.id}}","phone":"{{ticket.current_user.phone}}","extended_role":"{{ticket.current_user.extended_role}}","role":"{{ticket.current_user.role}}","details":"{{ticket.current_user.details}}","signature":"{{ticket.current_user.signature}}","organization":"{{ticket.current_user.organization}}","external_id":"{{ticket.current_user.external_id}}","email":"{{ticket.current_user.email}}"},"organization.name":"{{ticket.organization.name}}","status":"{{ticket.status}}","due_date":"{{ticket.due_date}}","due_date_with_timestamp":"{{ticket.due_date_with_timestamp}}","description":"{{ticket.description}}","tags":"{{ticket.tags}}","cc_names":"{{ticket.cc_names}}","link":"{{ticket.link}}","requester":{"first_name":"{{ticket.requester.first_name}}","last_name":"{{ticket.requester.last_name}}","name":"{{ticket.requester.name}}","language":"{{ticket.requester.language}}","tags":"{{ticket.requester.tags}}","locale":"{{ticket.requester.locale}}","notes":"{{ticket.requester.notes}}","time_zone":"{{ticket.requester.time_zone}}","id":"{{ticket.requester.id}}","phone":"{{ticket.requester.phone}}","extended_role":"{{ticket.requester.extended_role}}","role":"{{ticket.requester.role}}","details":"{{ticket.requester.details}}","signature":"{{ticket.requester.signature}}","organization":"{{ticket.requester.organization}}","external_id":"{{ticket.requester.external_id}}","email":"{{ticket.requester.email}}"},"in_business_hours":{{ticket.in_business_hours}},"created_at_with_timestamp":"{{ticket.created_at_with_timestamp}}","account":"{{ticket.account}}","url":"{{ticket.url}}","created_at":"{{ticket.created_at}}","external_id":"{{ticket.external_id}}"},"schema":"iglu:com.zendesk.zendesk/ticket_opened/jsonschema/1-0-0"},"schema":"iglu:com.snowplowanalytics.snowplow/unstruct_event/jsonschema/1-0-0"}
```

Submit the new trigger. It should look something like this:

[[/setup-guide/images/webhooks/zendesk/ticket-creation-trigger.png]]

<a name="comment-creation" />
# Set up a trigger to track ticket comment creation

As before, navigate to the trigger creation page.

Name the trigger something like "Comment-Creation-Trigger".

From the drop-down menus in the "Meet all of the following conditions" section, add two conditions:
* "Ticket: Is..." and "Updated" 
* "Ticket: Comment is..." and "Present (public or private)"

In the "Perform these actions" section, select "Notifications: Notify target" and "Cloudfront-Collector".

In the Message box, paste the following:

```javascript
{"data": {"data": {"latest_comment.author.name": "{{ticket.latest_comment.author.name}}", "url": "{{ticket.url}}", "latest_comment.created_at": "{{ticket.latest_comment.created_at}}", "latest_comment.created_at_with_time": "{{ticket.latest_comment.created_at_with_time}}", "latest_comment.is_public": {{latest_comment.is_public}}, "id": "{{ticket.id}}""}, "schema": "iglu:com.zendesk.zendesk/ticket_commented/jsonschema/1-0-0"}, "schema": "iglu:com.snowplowanalytics.snowplow/unstruct_event/jsonschema/1-0-0"}
```

Submit the new trigger. It should look something like this:

[[/setup-guide/images/webhooks/zendesk/comment-creation-trigger.png]]

<a name="ticket-resolution" />
## 4. Setting up an automation to track tickets being closed

From the Admin page, select "Automations" from the "BUSINESS RULES" menu:

[[/setup-guide/images/webhooks/zendesk/business-rules.png]]

You should have an automation which closes tickets. By default it will be called something like "Close ticket 4 days after status is set to solved":

[[/setup-guide/images/webhooks/zendesk/automation.png]]

As before, in the "Perform these actions" section, choose "Notifications: Notify target" and "Cloudfront-Collector" from the drop-down menus.

Copy and paste the following into the Message box:

```javascript
{"data":{"data":{"via":"{{ticket.via}}","ticket_type":"{{ticket.ticket_type}}","updated_at":"{{ticket.updated_at}}","assignee":{"first_name":"{{ticket.assignee.first_name}}","last_name":"{{ticket.assignee.last_name}}","name":"{{ticket.assignee.name}}","language":"{{ticket.assignee.language}}","tags":"{{ticket.assignee.tags}}","locale":"{{ticket.assignee.locale}}","notes":"{{ticket.assignee.notes}}","time_zone":"{{ticket.assignee.time_zone}}","id":"{{ticket.assignee.id}}","phone":"{{ticket.assignee.phone}}","extended_role":"{{ticket.assignee.extended_role}}","role":"{{ticket.assignee.role}}","details":"{{ticket.assignee.details}}","signature":"{{ticket.assignee.signature}}","organization":"{{ticket.assignee.organization}}","external_id":"{{ticket.assignee.external_id}}","email":"{{ticket.assignee.email}}"},"url_with_protocol":"{{ticket.url_with_protocol}}","id":"{{ticket.id}}","title":"{{ticket.title}}","priority":"{{ticket.priority}}","score":{{ticket.score}},"updated_at_with_timestamp":"{{ticket.updated_at_with_timestamp}}","current_user":{"first_name":"{{ticket.current_user.first_name}}","last_name":"{{ticket.current_user.last_name}}","name":"{{ticket.current_user.name}}","language":"{{ticket.current_user.language}}","tags":"{{ticket.current_user.tags}}","locale":"{{ticket.current_user.locale}}","notes":"{{ticket.current_user.notes}}","time_zone":"{{ticket.current_user.time_zone}}","id":"{{ticket.current_user.id}}","phone":"{{ticket.current_user.phone}}","extended_role":"{{ticket.current_user.extended_role}}","role":"{{ticket.current_user.role}}","details":"{{ticket.current_user.details}}","signature":"{{ticket.current_user.signature}}","organization":"{{ticket.current_user.organization}}","external_id":"{{ticket.current_user.external_id}}","email":"{{ticket.current_user.email}}"},"organization.name":"{{ticket.organization.name}}","status":"{{ticket.status}}","due_date":"{{ticket.due_date}}","due_date_with_timestamp":"{{ticket.due_date_with_timestamp}}","description":"{{ticket.description}}","tags":"{{ticket.tags}}","cc_names":"{{ticket.cc_names}}","link":"{{ticket.link}}","requester":{"first_name":"{{ticket.requester.first_name}}","last_name":"{{ticket.requester.last_name}}","name":"{{ticket.requester.name}}","language":"{{ticket.requester.language}}","tags":"{{ticket.requester.tags}}","locale":"{{ticket.requester.locale}}","notes":"{{ticket.requester.notes}}","time_zone":"{{ticket.requester.time_zone}}","id":"{{ticket.requester.id}}","phone":"{{ticket.requester.phone}}","extended_role":"{{ticket.requester.extended_role}}","role":"{{ticket.requester.role}}","details":"{{ticket.requester.details}}","signature":"{{ticket.requester.signature}}","organization":"{{ticket.requester.organization}}","external_id":"{{ticket.requester.external_id}}","email":"{{ticket.requester.email}}"},"in_business_hours":{{ticket.in_business_hours}},"created_at_with_timestamp":"{{ticket.created_at_with_timestamp}}","account":"{{ticket.account}}","url":"{{ticket.url}}","created_at":"{{ticket.created_at}}","external_id":"{{ticket.external_id}}"},"schema":"iglu:com.zendesk.zendesk/ticket_closed/jsonschema/1-0-0"},"schema":"iglu:com.snowplowanalytics.snowplow/unstruct_event/jsonschema/1-0-0"}
```

The result should look something like this:

[[/setup-guide/images/webhooks/zendesk/ticket-closed-automation.png]]

[Back to top](#top)

[Return to setup guide](Setting-up-Snowplow)
