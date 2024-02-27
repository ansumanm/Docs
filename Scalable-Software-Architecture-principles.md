# Principles of scalable software architeture
- ### Loose Coupling
  - Reduce dependencies between different components of a system.
  - Changes in one component have minimal impact on others.
  - This flexibility allows for independent development, deployment and scaling making the system more adaptable and resilient.
- ### Modularity
  - Break down a complex system into smaller, self-contained modules or components.
  - Each module has well-defined responsibilities and interfaces, allowing for independent development and deployment.
  - Modularity enables horizontal scaling by replicating modules as needed, ensuring system can handle increased loads effectively.
- ### Stateless Architecture
  - Each request from a client contains all the information necessary to process it, eliminating the need for servers to maintain session states.
- ### Caching
  - Involves storing frequently acccessed data or computed results in a cache, closer to the application layer.
  - Reduces load on backend systems and improves response times.

# Common patterns for Scalable Software Architecture
- ### Microservices
  - It involves breaking down a monolithic application into smaller, loosly coupled services, each responsible for a specific business capability.
  - These services can be developed, deployed and scaled independently, allowing for greater agility and scalability.
  - Microservices enable fault isolation. Issues in one service do not affect the entire system.
- ### Load Balancing
  - Distribute incoming network traffic across multiple servers or computing resources.
  - With load balancing, you can scale horizontally, by adding more servers or instances as the traffic increases, ensuring high availability and performance.
- ### Vertical Scaling:
  - Involves adding more resources, such as CPU, memory or storage to a single server or instance.
 
# Best practices for scalable software architecture
- ### Use Cloud Computing
  - IaaS - Infrastructure as a service
  - PaaS - Platform as a service
- ### Monitor System Performance
- ### Carry Out Automated Testing
