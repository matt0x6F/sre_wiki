Performance Testing Microservices
======

Performance testing microservices is not all that different from a monolith.

* Come up with a metric for establishing throughput
 * **Example:** For web servers or websites I use *requests per second* (rps)
* Scale your microservice down to it's lowest increment
* Mock all the endpoints and datasources your microservice talks to
 * The goal is to isolate just the microservice code
* Load test your microservice with something like [Locust](https://locust.io/) that can simulate multiple users in a realistic fashion
* Your load testing metric should be universal
 * You should use it in your SLI's as well
 * This ensures a common language to discuss performance in
