CST8915 – Full-stack Cloud-native Development

Midterm project: 
Extending the Algonquin Pet Store with an Order Analytics Microservice

Professor:   Ramy Mohamed
Student:  Hesheng Yang, Youssuf Hichri , Usama Iqbal

Link to Youtube Part1, Demo for application running locally : https://youtu.be/kJ3Ht9iGcm8
Link to Youtube Part2, Demo for application running on Azure : https://youtu.be/F0iaijkEP2Y
Link to Application running on Azure second account: https://youtu.be/tdb4v1IvAvU
Link to Analytics service running as web app communicating with RabbitMQ as embedded server from second account (final part): https://youtu.be/ia0mgLSnxbw

Brief overview of our order-analytics-service (language choice + architecture decisions)
What it is:
In the mid term project, the order-analytics-service is a new microservice, that consumes order messages from RabbitMQ queue order_queue , and exposes REST analytics endpoints so we can see the details orders, total profit, and top products.
Development Language choice:
We used node.js, the reason is, it fits microservices well, and it has strong RabbitMQ + REST libraries, furthermore, it can deploy easily to Azure App Service.

Architecture decisions (key points):
•	Event-driven design: order-service publishes events → analytics service consumes events.
•	Separation of concerns: order-service focuses on receiving orders and publishing; analytics service focuses on processing + analytics.
•	In-memory storage (for the assignment): consumed orders are stored in a local array / map in the service (simple, fast, easy to demo).
•	12-Factor compliance (first 4):
o	Codebase: we separate GitHub repo for analytics service
o	Dependencies:  we declared in package.json / requirements.txt
o	Config: for RabbitMQ parameters, for example, host/user/pass/port and service port come from environment variables
o	Backing service: RabbitMQ treated as an attached service via connection string

2) The RabbitMQ consumer pattern we chose and why
Pattern chosen: Competing Consumers (Worker Consumer) + manual ACK.
How it works:
•	In the project, our service opens a persistent connection to RabbitMQ and listens to order_queue.
•	When a message arrives, it will:
1.	parse JSON
2.	update analytics data (store order, update totals)
3.	send an ACK so RabbitMQ removes the message
The reason select the pattern::
•	Reliable processing: with manual ACK, we only acknowledge after successful processing. If the service crashes before ACK, the message can be re-delivered.
•	Scales easily: if we run multiple instances of order-analytics-service (locally or on Azure), they can share the same queue and split messages automatically.
•	Best fit for this queue setup: order-service sends directly to order_queue using the default exchange, so a single queue + consumer group is the simplest and clearest architecture for analytics.

3) Live demo running locally (place orders → show analytics endpoints)
What we will show locally:
1.	Start RabbitMQ (Docker or local)
2.	Start all services:
o	product-service (3030)
o	order-service (3000)
o	store-front (8080 dev)
o	order-analytics-service (4000)
3.	Place 5 or more orders using the store-front UI (different products + quantities)
4.	Verify analytics service is consuming messages (console logs show “received order…”)

Endpoints to show:
•	GET /health → returns status like { "status": "ok" }
•	GET /orders → shows the list of consumed orders (proof it consumed messages)
•	GET /analytics/summary → shows totals:
o	totalOrders
o	totalRevenue
o	averageOrderValue
•	GET /analytics/products → grouped by product name:
o	orderCount
o	totalRevenue
•	GET /analytics/top-products → top 3 by total quantity


4) Live demo running on Azure (show deployed services → place orders → verify analytics)
What we will show in Azure Portal:
•	Azure VM running RabbitMQ (ports 5672 and 15672)
•	Azure App Service for:
o	order-service (Node runtime)
o	product-service (Python runtime)
o	order-analytics-service (our runtime)
•	Azure Static Web Apps for store-front
Steps in the Azure demo:
1.	Open the deployed store-front URL (Static Web Apps)
2.	Place a few orders
3.	Show RabbitMQ Management UI on the VM:
o	order_queue message activity (messages arrive and then disappear after being consumed)
4.	Open analytics service endpoints on Azure:
o	/health
o	/analytics/summary
o	/analytics/top-products
5.	Confirm results changed after placing orders (numbers increase)
Important configuration callout (12-Factor):
•	In App Service → Configuration, we set env vars like:
o	RABBITMQ_HOST (VM public IP or DNS)
o	RABBITMQ_PORT=5672
o	RABBITMQ_USER
o	RABBITMQ_PASS
o	PORT=4000 (or whatever Azure expects)
•	“No secrets are hardcoded; everything is config-based.”
________________________________________
5) Walkthrough of the architecture diagram (what to explain)
Why this design is cloud-native:
•	Each component is independently deployable and scalable.
•	RabbitMQ decouples order ingestion from analytics.
•	Config via environment variables supports different environments (local vs Azure).
