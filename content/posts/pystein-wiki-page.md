---
title: "Pystein Wiki Page"
date: 2022-03-10T22:58:54-08:00
draft: false
---

(Preview) Python Programming Model
Our Goals
The current Python programming model in Azure Functions has limitations that sometimes prevents a customer from having a smooth onboarding experience. This includes the facts that there are too many files present, that the Function App structure can be confusing, and that file configuration follows Azure specific concepts rather than what Python frameworks.

To overcome these challenges, the Azure Functions Python team ideated a new programming model which eases the learning experience for new and existing customers. Specifically, the new programming model involves a single .py file (funcion_app.py) and will no longer require the 'function.json' file. Furthermore, the triggers and bindings usage will be decorators, simulating an experience similar to Flask.

Getting Started
Currently, the Python programming model is in the alpha release.

To try out the new programming model, download PyStein Custom Core Tools . Note that downloading the file will not overwrite the existing core tools in your device.

Installation & Setup
Download PyStein Custom Core Tools .
Unzip the folder to extract the files.
Option 1: Referencing the func in this folder when running "func host start".
C:\Users\test_user\functionscli\CoreTools-PyStein host start
Option 2: Alias the func to this folder. Note that this will impact your existing core tools if you don't reset it when you are done testing.

Set-Alias -Name func -Value C:\Users\test_user\functionscli\CoreTools-PyStein
Notes & Limitations
At this time, when using the attached core tools, only the new programming model will be supported
HTTP annotation is taken as an argument
By default, the authentication level is set to Functions.
The name of the function script file must be 'function_app.py'. In the case that this file is not found, the worker will fall back to the legacy indexing.
Mix and match of the legacy and new programming model is not supported
If the function app is configured with Flask framework, the HTTP bindings will not work as expected.
At this time, all testing will be local as the new programming model is not available in production.
View examples for the new programming model.

Triggers & Bindings
At this time, the new Programming model supports HTTP, Timer, Event Hub, Queue, Service Bus, and Cosmos DB. The following are examples of the implementation with the new programming model.

Event Hub

@app.function_name(name="EventHubFunc")
@app.on_event_hub_message(arg_name="myhub", event_hub_name="testhub", connection="EHConnectionString")
@app.write_event_hub_message(arg_name="outputhub", event_hub_name="testhub", connection="EHConnectionString")
def eventhub_trigger(myhub: func.EventHubEvent, outputhub: func.Out[str]):
    outputhub.set("hello")
Queue

@app.function_name(name="QueueFunc")
@app.on_queue_change(arg_name="msg", queue_name="js-queue-items", connection="storageAccountConnectionString")
@app.write_queue(arg_name="outputQueueItem", queue_name="outqueue", connection="storageAccountConnectionString")
def test_function(msg: func.QueueMessage, outputQueueItem: func.Out[str]) -> None:
    logging.info('Python queue trigger function processed a queue item: %s',
                 msg.get_body().decode('utf-8'))
    outputQueueItem.set('hello')
Service Bus

@app.function_name(name="ServiceBusFunc")
@app.on_service_bus_topic_change(arg_name="serbustopictrigger", topic_name="testtopic", connection="topicconnection", subscription_name="testsub")
@app.write_service_bus_topic(arg_name="serbustopicbinding", connection="topicconnection",  topic_name="testtopic", subscription_name="testsub")
def main(serbustopictrigger: func.ServiceBusMessage, serbustopicbinding: func.Out[str]) -> None:
    logging.info('Python ServiceBus queue trigger processed message.')

    result = json.dumps({
        'message_id': serbustopictrigger.message_id,
        'body': serbustopictrigger.get_body().decode('utf-8'),
        'content_type': serbustopictrigger.content_type,
        'expiration_time': serbustopictrigger.expiration_time,
        'label': serbustopictrigger.label,
        'partition_key': serbustopictrigger.partition_key,
        'reply_to': serbustopictrigger.reply_to,
        'reply_to_session_id': serbustopictrigger.reply_to_session_id,
        'scheduled_enqueue_time': serbustopictrigger.scheduled_enqueue_time,
        'session_id': serbustopictrigger.session_id,
        'time_to_live': serbustopictrigger.time_to_live
    }, default=str)

    logging.info(result)
    serbustopicbinding.set("topic works!!")

```python
@app.function_name(name="ServiceBusFunc")
@app.on_service_bus_queue_change(arg_name="serbustopictrigger", queue_name="inputqueue", connection="sbconnection")
@app.write_service_bus_queue(arg_name="serbustopicbinding", connection="sbconnection",  queue_name="outputqueue")
def main(serbustopictrigger: func.ServiceBusMessage, serbustopicbinding: func.Out[str]) -> None:
    logging.info('Python ServiceBus queue trigger processed message.')

    result = json.dumps({
        'message_id': serbustopictrigger.message_id,
        'body': serbustopictrigger.get_body().decode('utf-8'),
        'content_type': serbustopictrigger.content_type,
        'expiration_time': serbustopictrigger.expiration_time,
        'label': serbustopictrigger.label,
        'partition_key': serbustopictrigger.partition_key,
        'reply_to': serbustopictrigger.reply_to,
        'reply_to_session_id': serbustopictrigger.reply_to_session_id,
        'scheduled_enqueue_time': serbustopictrigger.scheduled_enqueue_time,
        'session_id': serbustopictrigger.session_id,
        'time_to_live': serbustopictrigger.time_to_live
    }, default=str)

    logging.info(result)
    serbustopicbinding.set("queue works!!")
```

Cosmos DB

@app.function_name(name="Cosmos1")
@app.on_cosmos_db_update(arg_name="triggerDocs", database_name="billdb", collection_name="billcollection", connection_string_setting="CosmosDBConnectionString", lease_collection_name="leasesstuff", create_lease_collection_if_not_exists="true")
@app.write_cosmos_db_documents(arg_name="outDoc", database_name="billdb", collection_name="outColl", connection_string_setting="CosmosDBConnectionString")
@app.read_cosmos_db_documents(arg_name="inDocs", database_name="billdb", collection_name="incoll", connection_string_setting="CosmosDBConnectionString")
def main(triggerDocs: func.DocumentList, inDocs: func.DocumentList, outDoc: func.Out[func.Document]) -> str:
    if triggerDocs:
        triggerDoc = triggerDocs[0]
        logging.info(inDocs[0]['text'])
        triggerDoc['ssss'] = 'Hello updated2!'
        outDoc.set(triggerDoc)
What's Next
View examples for the new programming model.
Let us know your feedback in the GitHub discussion .

Upcoming Features

Additional trigger and binding support for Durable functions, Event Grid, Kafka, and SQL.
Visual Studio Code support
