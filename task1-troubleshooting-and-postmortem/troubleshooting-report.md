## Process used
First, I checked if the infrastructure and application were healthy.

I verified:
- Docker containers
- `/actuator/health` endpoint
- dependent services like PostgreSQL, Kafka, Prometheus and Grafana

After confirming the environment was healthy, I executed the requests from `api-requests.http` one by one and checked:
- HTTP status code
- response time
- response body
- application logs

---

## Issue 1 - Price updates are not applied

The price update endpoint returned `202 Accepted`, which suggested that the update was processed asynchronously through Kafka.

However, after checking the prices, the new values were not saved.

From the logs I observed:
- the message reached Kafka
- the consumer tried to process it
- but the application failed while processing the `timestamp` field

The issue was related to `LocalDateTime` serialization/deserialization.

Impact:
The user believes the update was successful, but the price is not actually updated.

---

## Issue 2 - Very slow endpoint

The endpoint:

`GET /quote?quantity=430&channel=WEB`

responded very slowly.

The request took around 46 seconds and the application reported a very large number of database reads.

Impact:
The application may become very slow for large requests and users may think the service is unavailable.

---

## Issue 3 - Endpoint returns HTTP 500

The endpoint:

`GET /quote/segment?segment=flash`

returned `500 Internal Server Error`.

From the logs I observed that the application tried to use a `null` value without validation.

The `vip` segment worked correctly, so the problem was specific to the `flash` segment.

Impact:
The functionality appears broken for specific business cases.

---

## Observability

The application already had:
- health endpoint
- Prometheus metrics
- logs

But it was missing:
- alerting for Kafka errors
- monitoring for slow requests
- tracing
- metrics for consumer failures

Because of this, some problems could remain hidden until users reported them.

Overall, the application already has a good observability foundation, but alerting and visibility for asynchronous processes could be improved.