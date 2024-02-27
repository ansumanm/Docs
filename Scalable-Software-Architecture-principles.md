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

# Popular application design patterns
- ## Creational Patterns
  - ### Singleton pattern
    - Ensures a class has only one instance and provides a global point of access to it.
    - Useful for managing shared resources or services.
  - ### Factory Method pattern
    - Defines an interface for creating an object, but lets the subclasses alter the type of objects that will be created. It is used when a class cannot anticipate the class of object that it has to create.
  - ### Abstract Factory pattern
  - Provides an interface for creating families or related or dependent objects without specifying their concrete classes.
  - Used when the system needs to be independent of how its products are created, composed or represented.

- ## Behavioral Patterns
  - ### Observer pattern:
    - Defines a dependency between objects so that when one object changess state, all its dependents are notfied and updated automatically. This is widely used in implementing distributed event handling systems.

- ### Model-View-Controller(MVC) pattern
  Separate the application into three interconnected components:
  - #### Model
    Represents the data and teh business logic of the application.
  - #### View
    Represents the user interface
  - #### Controller
    Acts as an intermediary between model and view.

- ### Factory pattern
  Provides a way to encapsulate the creation of objects and to control their lifecycle.
  Factory could be used to create and manage database connections, ensuring that only authorized users can access the database.

- ### Observer pattern
  Allows objects to communicate with each other without being tightly coupled. An observer could monitor login attempts and notify the controller when a suspicious login attempt occurs.
    
