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

 
