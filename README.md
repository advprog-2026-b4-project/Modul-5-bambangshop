# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber
is defined as an interface. Explain based on your understanding of Observer design patterns,
do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model
struct is enough?

the classic observer pattern from the head first book requires an interfacce so a publisher can
polymorphicvally call an update() method on various in-memory subscriber obejcts. In bambangshop, 
the publisher and subscriber are completely seperate applications, and the publisher notifies every 
subscriber using the exact same mechanism in which sending an HTTP POST request to their specific URL.
because there are no different in app notification behaviours to abstract away, a single subscirber model 
struct that acts purely as a data ocntainer for the subscirber utl and deails is perfectly sufficient


2. id in Program and url in Subscriber is intended to be unique. Explain based on your
understanding, is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently
use is necessary for this case?

using a vec is not sufficient for this case. First, checking for unique id or url in a vec is slow because 
the program has to scan the entire list one by one. second, because a web server handles many request at the exact same time, 
a vec would force those request to wait in line to edit the data, causing lag. a dashmap is necessary because it allows 
instant lookups to prevent duplicates, and it is specially designed to let multiple requests safely update the data at the same time without crashing

3. When programming using Rust, we are enforced by rigorous compiler constraints to make a
thread-safe program. In the case of the List of Subscribers (SUBSCRIBERS) static variable, we
used the DashMap external library for thread safe HashMap. Explain based on your
understanding of design patterns, do we still need DashMap or we can implement Singleton
pattern instead?

we still need dashmap because the singleton pattern and dashmap solve two different problems. 
the singleton pattern only ensures that there is a single, global list of subbscribers in the application. 
However, because a web application is multi-threaded, multiple request might try to modify that single list 
at the exact same time. the singleton pattern does not prevent these threads from crashing into each other. 
therefore, we sill need dashmap to enforce thread safety and allow concurrent read/write operations 
on that single global instance, satisfying rust strict compiler constraints

#### Reflection Publisher-2

1. In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. Model in MVC covers both data storage and business logic. Explain based on your understanding of design principles, why we need to separate “Service” and “Repository” from
a Model?

Seperating the service and repository from themodel is necessary to follow the single responsibility principle and prevent the model from becoming bloated and difficult to to maintain, by splitting them up, the model acts as a simple data container, the repository is strictly responsible for interating with the database, and the service focuses entirely on executing the busines slogic. this seperation of concerns makes the codebase much cleaners, easire to test, and more flexible, because changing how data is stored in the repository will not break the business rules in the service

2. What happens if we only use the Model? Explain your imagination on how the interactions
between each model (Program, Subscriber, Notification) affect the code complexity for
each model?

if we only use the model, we create a model that are fat where the data structure, database operations, and business logic are all tangled together. the complexity of each model skyrockets because tgey become highly dependent on each other. for example, the program model would not only store product data, but it woould also have to fetch subscriber data, format notification messages, and send http requests. this makes the code incredibly difficult to read, hard to maintain, and nearly impossible to test, because we cannot trigger one action without accidentally triggering the entire notification process

3. Have you explored more about Postman? Tell us how this tool helps you to test your current work. You might want to also list which features in Postman you are interested in or feel like it is helpful to help your Group Project or any of your future software engineering projects.

in this bambangshop tutorial, postman has been essential for testing the REST API independently, without needing to build a frontend interface. it allowed me to easily act as a client by sending POST request to the publisher to create new products. more importantly, it helped me verify that the observer pattern was functioning correctly across different instances, i could trigger a product creation on port 8000 and confirm that the notification payload was successfully routed to the receiver on port 8001. being able to directly inspect the JSON body, headers, and HTTP status codes made debugging the notification routing mechanism much more straightforward

for my future, this is incredibly helpful for parallel development. if the backend team is still building out complex logic or database schemas, we can use postman to spin up a mock server that returns predetermined JSON responses. this ensures frontend development is never blocked waiting for the real API to be finished

#### Reflection Publisher-3
1. Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and Pull model (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?

In this tutorial, we are utilizing the Push model variation of the Observer Pattern. This is evident in how the Publisher handles data distribution: when a new product is created, the Publisher constructs a complete Notification payload (containing the product title, type, and status) and actively "pushes" it to the Subscribers via an HTTP POST request. The Subscribers simply receive and process this data. If this were a Pull model, the Publisher would only send a minimal alert that a change occurred, and the Subscriber would then be forced to send a subsequent request back to the Publisher to "pull" the actual product details. The Push model is highly efficient here because it eliminates the need for that extra round-trip network request.

2. What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull)

If we were to use the Pull model variation of the Observer Pattern in this tutorial, the Publisher would only send a lightweight alert (a "ping") to the Subscribers, and the Subscribers would then have to make a separate API request back to the Publisher to fetch the full product details. An advantage of this approach is data control, the Subscriber can decide whether or not to fetch heavy payloads, saving bandwidth if the data isn't immediately needed. However, the disadvantages heavily outweigh the pros for this specific application. The Pull model introduces significant network latency by requiring two HTTP requests for every single event. Furthermore, it risks overwhelming the Publisher, if a ping is broadcasted to many subscribers, they will all simultaneously request the data, potentially causing a server bottleneck. Because our payload consists of simple, lightweight strings, the Push model is far more efficient and less complex.

3. Explain what will happen to the program if we decide to not use multi-threading in the notification process.

If we do not use multi-threading in the notification process, the application will suffer from severe performance bottlenecks due to synchronous, blocking operations. In a single-threaded approach, the Publisher must send HTTP requests to each subscriber one by one. If we have hundreds of subscribers, or if even a single subscriber's server is slow or unresponsive, the entire loop is delayed. This forces the original user who created the product to experience massive loading times, waiting for all notifications to finish processing before receiving their 201 Created response. By using multi-threading or asynchronous tasks, we can fire off all notification requests concurrently in the background, ensuring the web server remains fast, responsive, and resilient to individual subscriber failures