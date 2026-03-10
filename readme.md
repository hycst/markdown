## CST8915 – Full-stack Cloud-native Development

# Midterm project: 
Extending the Algonquin Pet Store with an Order Analytics Microservice

## Professor:   Ramy Mohamed
## Student:  Hesheng Yang, Youssuf Hichri , Usama Iqbal

### Youtube Video Link
-  Link to Youtube Part1, Demo for application running locally : https://youtu.be/kJ3Ht9iGcm8
-  Link to Youtube Part2, Demo for application running on Azure : https://youtu.be/F0iaijkEP2Y
-  ink to Application running on Azure second account: https://youtu.be/tdb4v1IvAvU
-  Link to Analytics service running as web app communicating with RabbitMQ as embedded server from second account (final part): https://youtu.be/ia0mgLSnxbw

### Source code
-  https://github.com/hycst/8915mid-order-service
-  https://github.com/hycst/8915-store-front
-  https://github.com/hycst/8915-order-analytics-service
-  https://github.com/hycst/8915-store-front

## Brief overview of our order-analytics-service (language choice + architecture decisions)
###  What it is:
In the mid term project, the order-analytics-service is a new microservice, that consumes order messages from RabbitMQ queue order_queue , and exposes REST analytics endpoints so we can see the details orders, total profit, and top products.
### Development Language choice:
We used node.js for one but because we had multiple members wanting to see how it would perform with a different stack we also had another member develop the service in python, both implementations complied with microservices well, and it has strong RabbitMQ + REST libraries, furthermore, it can deploy easily to Azure App Service. Because two full fledged solutions were developed by the group, that is the reason why there are multiple videos as well as the submission, the videos talk about both solutions developed namely the python application as well as the node.js application for order analytics. 

Futhermore, we team member implemented the service in Python as second option, since Python has mature messaging libraries and is commonly used for data processing and analytics workloads.

### 1) Architecture decisions (key points):
-	Event-driven design: order-service publishes events → analytics service consumes events.
- Separation of concerns: order-service focuses on receiving orders and publishing; analytics service focuses on processing + analytics.
- In-memory storage (for the assignment): consumed orders are stored in a local array / map in the service (simple, fast, easy to demo).
-	12-Factor compliance (first 4):
    -  Codebase: we separate GitHub repo for analytics service.
    -  Dependencies: we declared them in `package.json` / `requirements.txt`.
    -  Config: RabbitMQ parameters, such as host, user, password, port, and service port, come from environment variables.
    -  Backing services: RabbitMQ is treated as an attached service via a connection string.

### 2) The RabbitMQ consumer pattern we chose and why
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

### 3) Live demo running locally (place orders → show analytics endpoints)
What we will show locally:
1.	Start RabbitMQ (Docker or local)
2.	Start all services:
    -	product-service (3030)
    -	order-service (3000)
    -	store-front (8080 dev)
    -	order-analytics-service (4000)
3.	Place 5 or more orders using the store-front UI (different products + quantities)
4.	Verify analytics service is consuming messages (console logs show “received order…”)

Endpoints to show:
-  GET /health → returns status like { "status": "ok" }
-  GET /orders → shows the list of consumed orders (proof it consumed messages)
-  GET /analytics/summary → shows totals:
    - 	totalOrders
    - 	totalRevenue
    - 	averageOrderValue
-  GET /analytics/products → grouped by product name:
    - 	orderCount
    - 	totalRevenue
-  GET /analytics/top-products → top 3 by total quantity


### 4) Live demo running on Azure (show deployed services → place orders → verify analytics)
What we will show in Azure Portal:
•	Azure VM running RabbitMQ (ports 5672 and 15672)
•	Azure App Service for:
    - 	order-service (Node runtime)
    - 	product-service (Python runtime)
    - 	order-analytics-service (our runtime)
•	Azure Static Web Apps for store-front
Steps in the Azure demo:
1.	Open the deployed store-front URL (Static Web Apps)
2.	Place a few orders
3.	Show RabbitMQ Management UI on the VM:
o	order_queue message activity (messages arrive and then disappear after being consumed)
4.	Open analytics service endpoints on Azure:
    - 	/health
    - 	/analytics/summary
    - 	/analytics/top-products
5.	Confirm results changed after placing orders (numbers increase)
Important configuration callout (12-Factor):
•	In App Service → Configuration, we set env vars like:
    - 	RABBITMQ_HOST (VM public IP or DNS)
    - 	RABBITMQ_PORT=5672
    - 	RABBITMQ_USER
    - 	RABBITMQ_PASS
    - 	PORT=4000 (or whatever Azure expects)
•	“No secrets are hardcoded; everything is config-based.”
________________________________________
### 5) Walkthrough of the architecture diagram (what to explain)
Why this design is cloud-native:
-	 Each component is independently deployable and scalable.
-  RabbitMQ decouples order ingestion from analytics.
-  Config via environment variables supports different environments (local vs Azure).
