---
title: "Monolithic vs Microservices architecture based on APIs"
seoTitle: "Monolithic vs Microservices architecture based on APIs"
seoDescription: "What is monolithic and microservice architecture. Key challenges faced in monolithic and how we overcome those challenges using microservices with example."
datePublished: Thu Oct 12 2023 14:31:34 GMT+0000 (Coordinated Universal Time)
cuid: clnna2xgw000308mk97modzdm
slug: monolithic-vs-microservices-architecture-based-on-apis
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1695834028246/2b9d86f1-7083-4dc1-a26d-e238ce780ab1.png
tags: microservices, apis, monolithic-architecture

---

## What is a Microservice?

* Microservice is a small, loosely coupled distributed service.
    
* Microservice is an API.
    
* It can be either SOAP or REST API. The API must have less number of operations with respect to a single use case or module (Ex: In an e-commerce website there would be so many modules such as searching products, add to cart, payment, checkouts, and so on) each module has N number of operations.
    
* It should be small enough to be called a microservice.
    

## What is a Monolithic service?

Monolithic architecture is a software design pattern in which a software application is written as a single coherent piece of code.  
As a result, changes to one part of the code base will affect the rest of the application as well.  
These applications are designed to handle multiple related tasks.  
They’re typically complex applications that encompass several tightly coupled functions.

### Let's understand how microservice and monolithic service differ by using one project requirement.

**Project requirement:** Create a web app/API that would be used by customers to view and manage hotel bookings and their customer profiles through the web browser.

**To fulfill the above requirement we need access to data, those are:**  
Customer data: Database and Salesforce  
Hotel data: SAP  
Hotel Booking data: Salesforce

### Monolithic architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697021553490/0fe8dbec-537d-4020-af82-b5375667e32a.jpeg align="center")

The functionality of monolithic applications is implemented into various modules like Customer and Hotel Booking Data, Customer Data, and Hotel Data. These modules connect with each other and each module connects to their respective databases to enable the required functionality.

**Challenges facing by this architecture**

* One module interacts with another module at a code level, whenever any code modification is done in one module, might affect the other module
    
* if one module is broken down because of some modification, it will cause another module to be broken down.
    
* If we do small modifications in one module, We have to redeploy the entire application.
    
* We cannot precisely control the amount of RAM and processing power given to individual modules.
    
* It is not possible to handle more traffic in one particular module, because we can't increase the RAM and processing power assigned to that particular module. We need to do this at the application level.  
    That means we can neither scale up nor scale down individual modules in the monolithic application. we need to scale up or scale down the entire application.
    
* Governing individual modules with different security schemes is practically difficult.
    
* Currently, this application is designed to be consumed from a Web browser. But we can't able to consume these applications on mobile devices or some other client platform.
    
    Because consuming this application on a mobile device requires an entirely different API, This new API should be created entirely from scratch.
    

All these challenges can be overcome by using a microservices architecture.

> Now let's see how to design these microservices for this project requirement.

### Microservice architecture

Before creating microservices for project requirements. First, we need to understand what are the various resources and the corresponding data sources required to acquire the above project requirement. Then we need to create a pair that includes one resource and its data source.

* **Resources**  
    \- Customers data  
    \- Hotels data  
    \- Hotel bookings data
    
* **Data Sources**  
    \- Database  
    \- SAP  
    \- Salesforce
    
* **Resource and Data source pairs:** For each, we need to create a microservice(API), each API should connect with the corresponding data source. Each API should expose CRUD(Create, Read, Update, Delete) operations from their corresponding data source.  
    \- Customers data (in) Database and (in) Salesforce  
    \- Hotels data (in) SAP  
    \- Hotel Bookings data (in) Salesforce
    

**1) System APIs** are APIs that connect to data systems and perform data operations.

> Note: Understand clearly, here API is created to perform the CRUD operations on the Database.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697035177564/03ca1a79-0d97-4eb6-9b1f-1bba26d99018.png align="center")

Using system APIs we expose CRUD operations corresponding to the data. And we can perform additional function functionality such as data transformation and data validation.

> example: Assume that customer API is connected to a database and fetching the data using JDBC, then the fetched data is in Java object format. And if we don't want to expose this Java object as it is. If we want to expose this data in JSON format, then we have to transform this data into JSON format, and we can perform data validation in system APIs.  
> We can't perform any business functionality and Data processing in System APIs.

**2) Process APIs** are the APIs that will connect to one or more system APIs, consume data from system APIs, and then perform logical functionality, data processing, execute business validations, and so on.  
Process APIs don't need to connect with all system APIs and we can have one or more process APIs based on the requirements.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697106772038/7f4edf77-e3c7-471f-a14e-959ffad0c305.png align="center")

**3) Experience APIs** are APIs that will be on top of the process APIs to provide the additional requirements to the APIs, that are specific to different client devices.

These APIs can also interact with System APIs if required. And do all final manipulation before sending it back to the original client application.

These APIs are optional, if we are supporting a single client device or if multiple client devices are expecting the same data structure, same data, and same number of operations. No need to create Experience APIs.

> example: if process API exposes 50 parameters inside the data and those 50 parameters can be viewed completely in case of web browser, but can't view all 50 parameters in mobile and tablet. Because there may be difficulties, their screen may be small, they may not have a bigger bandwidth to receive all the data in a faster manner, and users can experience latency to receive the data. We can also provide some additional functionality like GPS to mobile devices that may not needed in web browsers.  
> Then it is clear that every client device may have different requirements on the data and also may expect different numbers of operations based on the abilities of the client devices.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697109094175/bdc929c9-16c7-4a0f-81eb-7a7bbc34e1b8.png align="center")

### How we eliminated the challenges of monolithic approach over Microservices architecture:

* In this approach, we have N number of microservices(APIs) for individual modules and functionality. Here APIs are not communicated at the code level.
    
    It will interact at the API level. So if there is any breakdown in one API will not affect the other API
    
* Then we can deploy a separate microservice application to a different server.
    
* If something changes in one API will not affect the other API.
    
* If we do any modifications in one API, no need to redeploy the entire application. It is enough to redeploy the particular API.
    
* We can scale up or scale down the RAM and processing power at individual APIs.
    
* In the future if we want to support one or more types of client devices. We create experience API based on the requirements of that client by accessing data from the system and process APIs.