# Azure Event Hub with Azure Funtions - Resiliency Best Practices


## [Resiliency checklist for Event Hub](https://learn.microsoft.com/en-us/azure/architecture/checklist/resiliency-per-service)

1. 

In order to make SummaryGenerator Resilient, We need to implement following :
1. Error Handling in Azure Function
2. Retry Mechanism in Azure Function
3. At-least once delivery for the event (Idempotency)
4. Strategies for Event corrupt data

## Error handling:
  * Use Application Insights : Enable and use Application Insights to log errors and monitor the health of azure function.
  * Implement Logging
  * Add [try/catch](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reliable-event-processing) block in Azure Function, After processing custom retry logic in case of failure [please refer Retries Mechanism section] under the catch block log the exception details and move to next event for processing.

## Retries Mechanism:

  * Set Max Retry Count: 2 (Configurable)
  * Set Delay: 2 Sec (Configurable)
  * Azure Function will mainly perform below five tasks:
    1. Triggerd Azure Function will have event data as batch of event
    2. Check duplicacy for the event based on Call Id.
    3. Schema Validation for the event data.
    4. Generate Summary [LLM API Call]
    5. Generate the event and insert in another event hub
    Instead of Azure function retry policy, We will implement custom / configured retry logic for last couple of steps i.e. Generate Summary / Generate Event.
## Idempotency (Handling duplicate events):
To implement Idempotency we can follow below steps:
  * Azure Function to Read event from event hub
  * Azure Function will extract unique Call Id from event
  * Azure Function will Check if unique Call Id is already present in Azure Redis cache.
  * If Call ID is not present in Azure Redis cache Azure Function will process event and save unique Call Id with event process status (processing completed) in to Azure Redis cache
  * If Call ID is present, Azure Function will not process the event and move to process next event.
![EventHub](../llm/img/EventHub.png)

## Strategies for corrupt data

1. We can create and implement custom solution inside azure function.
