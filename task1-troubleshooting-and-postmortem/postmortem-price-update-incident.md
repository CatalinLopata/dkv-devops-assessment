## Incident Summary

The `POST /price-update` endpoint returned `202 Accepted`, but the updated prices were not applied successfully.

The request appeared successful from the client perspective, but the backend processing failed during Kafka message consumption.

---

# Impact

- users believed the price update was successful
- pricing data remained outdated
- no visible error was returned to the client
- issue could remain hidden without log investigation

---

# Root Cause

The Kafka consumer failed while processing the `timestamp` field because of a `LocalDateTime` serialization/deserialization issue.

As a result:
- the message reached Kafka successfully
- but processing failed before the database update was completed

---

# Detection

The issue was identified during manual API testing.

Investigation steps:
- checked application logs
- tested endpoints from `api-requests.http`
- verified Kafka-related logs
- validated database values after processing

The logs showed deserialization errors during Kafka message processing.

---

# Resolution

Recommended fix:
- configure proper `LocalDateTime` serialization/deserialization
- improve validation and exception handling during Kafka processing
- return clearer processing status to the client

---

# Preventive Actions

To reduce similar incidents in the future, I would recommend:
- alerting for Kafka consumer failures
- dead letter queue (DLQ) for failed messages
- metrics for consumer processing errors
- integration tests for Kafka workflows
- tracing for asynchronous operations