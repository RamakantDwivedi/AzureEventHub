# Azure Event Hub with Auzre Funtions - Resiliency Best Practices
In order to make SummaryGenerator Resilient, We need to implement following:
1. Error Handling
2. Retry Mechanism
3. At-least once delivery for the event (Idempotency)
4. Strategies for Event corrupt data

# Error handling:
  * Use Application Insights : Enable and use Application Insights to log errors and monitor the health of azure function.
  * Add try/catch block in Azure Funtion



