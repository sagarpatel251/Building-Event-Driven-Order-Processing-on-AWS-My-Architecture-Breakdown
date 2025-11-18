# Building-Event-Driven-Order-Processing-on-AWS-My-Architecture-Breakdown
A fully serverless, event-driven reference architecture built using Amazon Cognito, API Gateway, AWS Lambda, DynamoDB, EventBridge, CloudWatch, and SES. This repository demonstrates how to orchestrate an end-to-end workflow for processing orders using decoupled services and asynchronous patterns.

![EDA_Sagar_Patel_Updated](https://github.com/user-attachments/assets/176100e1-5245-4164-8c68-5c050c9f59dd)


📌 Architecture Overview

The system processes an order from the moment it is placed until the final confirmation is delivered.
Each service publishes events, and downstream consumers process them independently.

Main workflow:

1) Order Placed (Frontend → API Gateway)
2) Order stored in DynamoDB
3) Order event sent to EventBridge
4) Payment microservice triggered
5) Payment result published
6) Notification & Fulfillment triggered asynchronously
7) Final confirmation sent to the customer

🧩 Lambda Functions Explained 

1️) OrderServiceLambda

Purpose: Handles creation of new orders via the API.

Flow:

a) Receives HTTP request from API Gateway

b) Validates order data

c) Writes order record to DynamoDB (Orders Table)

d) Publishes OrderPlacedEvent to EventBridge

Key Responsibilities:

✔ Data validation
✔ Save to DynamoDB
✔ EventBridge integration
✔ Error handling & retries

2️) OrderStatusUpdaterLambda

Purpose: Updates order status after receiving lifecycle events.

Triggered By:

a) PaymentCompletedEvent

b) PaymentFailedEvent

c) FulfillmentCompletedEvent

Key Responsibilities:

✔ Update order record in DynamoDB
✔ Log audit trail
✔ Maintain final order status consistency

3️) PaymentProcessingLambda

Purpose: Simulates payment authorization and produces a result event.

Triggered By: OrderPlacedEvent from EventBridge.

Flow:

a) Reads order ID & amount

b) Simulates payment logic (success/failure)

Emits: PaymentCompletedEvent OR PaymentFailedEvent

Key Responsibilities:

✔ Payment workflow logic
✔ Publishing payment results
✔ Idempotency for repeated events

4️) NotificationLambda

Purpose: Sends notifications based on completed events.

Triggered By: SNS topic: OrderNotificationTopic

Flow:

a) Formats communication (email/SMS)

b) Integrates with Amazon SNS or external API

c) Logs successful notification

Key Responsibilities:

✔ Customer notifications
✔ Message formatting
✔ Error handling via DLQ (optional)

5️) FulfillmentLambda

Purpose: Simulates order packing/fulfillment pipeline.

Triggered By: PaymentCompletedEvent

Flow:

a) Marks order as ready for fulfillment

b) Generates a mock tracking ID

c) Publishes FulfillmentCompletedEvent to EventBridge

Key Responsibilities:

✔ Fulfillment simulation
✔ Tracking ID creation
✔ Event publishing

6️) AuditLoggerLambda

Purpose: Writes an audit trail of every system event.

Triggered By: EventBridge rule that listens to all events.

Flow:

a) Writes event metadata to DynamoDB or S3 or cloud watch

b) Maintains chronological event history

c) Useful for debugging and compliance

Key Responsibilities:

✔ Centralized auditing
✔ Event lifecycle visibility
✔ Long-term storage

🗄 DynamoDB Tables

| Table        | Purpose                       | Partition Key | Sort Key  |
| ------------ | ----------------------------- | ------------- | --------- |
| **Orders**   | Order state, status, metadata | orderId       | —         |
| **AuditLog** | System events and tracking    | eventId       | timestamp |

📨 EventBridge Events Used

| Event Type                  | Triggered When     | Consumed By                         |
| --------------------------- | ------------------ | ----------------------------------- |
| `OrderPlacedEvent`          | Order created      | Payment Lambda                      |
| `PaymentCompletedEvent`     | Payment authorized | Order Status updater, Fulfillment   |
| `PaymentFailedEvent`        | Payment failed     | Order Status updater, Notifications |
| `FulfillmentCompletedEvent` | Package ready      | Status updater, Notification        |


📬 Message Flow Summary

Customer → API Gateway → OrderServiceLambda → DynamoDB
    → EventBridge (OrderPlacedEvent)
        → PaymentLambda → EventBridge (PaymentCompletedEvent)
            → FulfillmentLambda → EventBridge (FulfillmentCompletedEvent)
                → NotificationLambda / StatusUpdaterLambda

🧪 Local Testing

Recommended tools:

a) Postman (API calls)

b) WS SAM / Serverless Framework (deployment)

c) DynamoDB Local (offline testing)

d) CloudWatch Logs (tracing)


📘 Lessons Learned

a) Decoupling removes inter-service dependencies and makes the system more resilient

b) EventBridge simplifies routing vs building point-to-point Lambda triggers

c) DynamoDB + event logs provide strong observability

d) Idempotency is essential when events retry

🎯 Takeaways

a) EDA is clean, scalable, and modern

b) Easier debugging via audit logs

c) Fault-tolerant by design

d) Perfect architecture pattern for microservices & modern cloud systems

