# CST8915FinalLabProject Cloud-Native App for Best Buy

## Application Overview
The **Best Buy App** includes the following components:

| Service              | Description                                                  |                                     
| -------------------- | ------------------------------------------------------------ |  
| **Store-Front**      | Customer-facing app for browsing and placing orders.         |                                         
| **Store-Admin**      | Employee-facing app for managing products and viewing orders . |                                          
| **Order-Service**    | Handles order creation and sends data to the managed order queue. | 
| **Product-Service**  | Handles CRUD operations for product data.                    |                                          
| **Makeline-Service** | Processes and completes orders by reading from the order queue. |                                          
| **AI-Service**       | Generates product descriptions and images. We use gpt-4 and dall-e-3 services.                  | 
| **Database**         | MongoDB for persisting order and product data.               |                                          

## Architecture Diagram
![image](https://github.com/user-attachments/assets/f2558167-bdbe-4491-b828-2f4b17d43401)    

Customers use the store-front to examine merchandise while placing their orders but the store-front retrieves information from product-service then delivers it to customers. The order information passes from store-front to order-service which performs encryption before Azure Service Bus receives it. Makeline-service monitors messages in Azure Service Bus until it obtains the most recent entry which it saves into the mongodb database. Store-admin service retrieves order information through makeline-service from mongodb yet also makes use of ai-service for new product description generation (gpt-4) and image generation (dall-e-3).  

