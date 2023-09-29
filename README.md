# Azure Event Hub with Azure Funtions - Resiliency Best Practices

 

In order to make SummaryGenerator Resilient, We need to implement following :

 

## [Resiliency checklist for Event Hub](https://learn.microsoft.com/en-us/azure/architecture/checklist/resiliency-per-service#event-hubs)

 

1. [Use checkpoints](https://www.oag.com/en/knowledge/checkpointing-with-azure-event-hubs): An event consumer should write its current position to persistent storage at some predefined interval. If the consumer experiences a fault (for example, the consumer crashes, or the host fails), then a new instance can resume reading the stream from the last recorded position.

In our case - we can store combination of Partition Id and Id in Redis cache after a configurable time interval and in case of process failure, when consumer will restart it will fetch Partion id and id from redis cache and start processing from there.

 

2. Handle duplicate messages(Idempotency): If an event consumer fails, message processing is resumed from the last recorded checkpoint. Any messages that were already processed after the last checkpoint will be processed again. Therefore, message processing logic must be idempotent. In our scenario we are going to implement it by following way:

    * Azure Function to Read event from event hub

    * Azure Function will extract unique Call Id from event

    * Azure Function will Check if unique Call Id is already present in Azure Redis cache.

    * If Call ID is not present in Azure Redis cache Azure Function will process event and save unique Call Id with event process status (processing completed) in to Azure Redis cache

    * If Call ID is present, Azure Function will not process the event and move to process next event.

    ![EventHub](../llm/img/EventHub.png)

 

3. Handle exceptions: An event consumer typically processes a batch of events in a loop. If a single message contains an exception, it should not block entire batch, hence exception handling is required. In our scenario we can implement in following way:

   

    * Use Application Insights : Enable and use Application Insights to log errors and monitor the health of azure function.

    * Implement Logging

    * Add [try/catch](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reliable-event-processing) block in Azure Function, After processing custom retry logic in case of failure [please refer Retries Mechanism section] under the catch block log the exception details and move to next event for processing.

 

4. Retries Mechanism:

    * Set Max Retry Count: 2 (Configurable)

    * Set Delay: 2 Sec (Configurable)

    * Azure Function will mainly perform below five tasks:

        1. Triggerd Azure Function will have event data as batch of event

        2. Check duplicacy for the event based on Call Id.

        3. Schema Validation for the event data.

        4. Generate Summary [LLM API Call]

        5. Generate the event and insert in another event hub

 

    Instead of Azure function retry policy, We will implement custom / configured retry logic for last couple of steps i.e. Generate Summary / Generate Event.

 

5. Strategies for corrupt data: In our scenario, We can use defined schema and can validate event data against defined schema. If validation fails we can log error and move to further processing.

 

6. Configure zone redundancy: When zone redundancy is enabled on event hub namespace, Event Hubs automatically replicates changes between multiple availability zones.

 

7. [Circuit Breaker Pattern for Azure Function](https://binduc.medium.com/circuit-breaker-pattern-for-azure-function-678ca0f7c3b5): It will be useful in the scenario like if downstream service is not available or Upstream event producer changed the message schema etc,in such scenarios, there is no point in running the Function indefinitely, we should break the circuit and send a notification to the support or admin team about the possible problem in the event processing pipeline, we can break the circuit with the help of App Insight, Monitor and Azure automation.

![circuit-breaker-pattern](../llm/img/circuit-breaker-pattern.png)
