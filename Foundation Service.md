# Foundation 
## Architecture Overview

The Foundation architecture proposes a microservice oriented architecture implementation with 
multiple autonomous microservices (each one owning its own data/db) and implementing different 
approaches within each microservice (simple CRUD vs. DDD/CQRS patterns) using Http as the 
communication protocol between the client apps and the microservices and supports asynchronous 
communication for data updates propagation across multiple services based on Integration Events and 
an Event Bus (a light message broker, to choose between RabbitMQ or Azure Service Bus, underneath) 
plus other features

# Downloads 
The latest stable release of the Foundation Framework is available on **[NuGet](http://10.10.1.21:4100/)**. 
_Pre-release builds are available._


# Framework Overview
## Backend Architecure
![Foudation Backend](https://demographicstorage.blob.core.windows.net/public/Foundation_Backend.png)
