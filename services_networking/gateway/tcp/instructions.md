# TCP routing

## We want to route TCP traffic for two internal applications: a messaging service and a database. The rules are:

- We need a gateway named infra-tcp-gateway configured with listeners that use the TCP protocol.

- All TCP traffic arriving at port 7000 on the Gateway is forwarded to the messaging-svc service on port 5672 (RabbitMQ).

- All TCP traffic arriving at port 7001 on the Gateway is forwarded to the db-svc service on port 5432 (PostgreSQL).

The documentation:
https://gateway-api.sigs.k8s.io/guides/tcp/
