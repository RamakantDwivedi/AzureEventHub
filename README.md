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

# Retries:
  * Set Max Retry Count:
  * Retry strategy : Exponential back off strategy(Preferred) vs. Fixed delay offers more flexibility for retry delay intervals and is commonly used when integrating with REST endpoints, and other Azure services. [Code Snippet](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-hubs?tabs=in-process%2Cextensionv5&pivots=programming-language-csharp#host-json)
     "clientRetryOptions":{
                "mode" : "exponential",
                "tryTimeout" : "00:01:00",
                "delay" : "00:00:00.80",
                "maximumDelay" : "00:01:00",
                "maximumRetries" : 3
            }
   
  * Implement Logging 

