---
title: "Reverse API Proxy"
seoTitle: "Reverse API Proxy"
seoDescription: "Create a Reverse API Proxy"
datePublished: Tue Dec 12 2023 06:48:21 GMT+0000 (Coordinated Universal Time)
cuid: clq1ze74s000j08lk6yhifuw3
slug: reverse-api-proxy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1702363294476/11bfd103-5e50-4a0a-808a-b75b32f59108.png
tags: apis, apigee

---

### What is a Reverse proxy?

* It receives the request from the client, processes it, and calls the backend service to get the data that the client requests.
    
* Send the response back to the client.
    

**Scenario**: The client sends a request message to the backend service to get the employee information according to its ID.

**Apigee Implementation**: Create a reverse API Proxy by using the backend service URL(target endpoint)  
target endpoint: [https://jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users)  
Run the above endpoint in Postman by adding `id:1` in the parameter below and see the output.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701874090192/378ea6b7-a253-4b00-99fc-e0cb3000238b.png align="center")

* Click on **API proxies**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701873839277/d4a530ad-3ede-4475-be29-220af7f8af1f.png align="center")
    
* Click on **CREATE NEW.**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701873919230/6ea044ee-58fc-4a32-b88d-28716fc609b8.png align="center")
    
* Click on **Reverse proxy.**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701873958357/7cd89f80-6d0d-4b56-a04d-e3687d2027f4.png align="center")
    
* Add the details as shown below. And click on **Next**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701874322682/4628d4ce-6bf6-41f4-81af-95c83ba00573.png align="center")
    
* Click on **Next** by selecting the **Security** option as Pass-through as shown in the image.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701874460640/125b431f-8e50-4274-ac3c-24051356b239.png align="center")
    
* select the environment (**eval**) where we are deploying this Proxy. As shown in the image. Click on **Create and deploy**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701874591279/af5eb72e-0822-4018-b9fd-7624e7ba6d98.png align="center")
    
* Click on **Edit proxy**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701874649576/5978b02d-352b-409c-8537-80db85269885.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701874719349/c7e72396-f10a-4eb4-94c5-d3f21083943a.png align="center")
    
* We successfully created the Reverse proxy with the target endpoint ([https://jsonplaceholder.typicode.com/users?id=1](https://jsonplaceholder.typicode.com/users?id=1)).
    
* The client sent a request message including id: 1 in the query parameter to the API proxy. Then proxy hits the target(Backend service) and gets the data as requested by the client. And respond to the client with a response message.  
    request message: [https://34.36.30.222.nip.io/reverseproxy?id=1](https://34.36.30.222.nip.io/reverseproxy?id=1)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702362548764/4d6537d4-19d4-4201-8f19-9226b4765e09.png align="center")