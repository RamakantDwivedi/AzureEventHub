# Azure Event Hub with Auzre Funtions - Resiliency Best Practices
In order to make SummaryGenerator Resilient, We need to implement following:
1. Error Handling
2. Retry Mechanism
3. At-least once delivery for the event (Idempotency)
4. Strategies for Event corrupt data

# Error handling:
  * Use Application Insights : Enable and use Application Insights to log errors and monitor the health of azure function.
  * Add try/catch block in Azure Funtion
  * Implement Logging 

# Retries Mechanism:
  * Set Max Retry Count:
  * Retry strategy : Exponential back off strategy(Preferred) vs. Fixed delay offers more flexibility for retry delay intervals and is commonly used when integrating with REST endpoints, and other Azure services. [Code Snippet](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-hubs?tabs=in-process%2Cextensionv5&pivots=programming-language-csharp#host-json)
    
     "clientRetryOptions":{
                "mode" : "exponential",
                "tryTimeout" : "00:01:00",
                "delay" : "00:00:00.80",
                "maximumDelay" : "00:01:00",
                "maximumRetries" : 3
            }
   
  # Idempotency(Handling duplicate events):
  To implement this we can follow below steps:
  * Azure Function to Read event from event hub
  * Azure Function will extract unique Call Id from event
  * Azure Function will Check if unique Call Id is already present in Azure table storage table
  * If Call ID is not present in Azure table storage table Azure Function will process event and save unique Call Id with event process status (processing completed) in to Azure table storage
  * If Call ID is present, Azure Function will not process the event and move to process next event.

# Strategies for corrupt data
In case if event contains corrupt data, we can avoid it if producer/consumer use same schema.
![image](https://github.com/RamakantDwivedi/AzureEventHub/assets/68191772/81b97c11-3d1a-4cf6-95f8-c44017f60057)

    

