# CST8915 Final Lab Project - Cloud-Native App for Best Buy

## Application Overview
The **Best Buy App** includes the following components:

| Service              | Purpose                                                             |
|:--------------------|:--------------------------------------------------------------------|
| **Store-Front**      | A customer-facing application where users can browse products and place orders. |
| **Store-Admin**      | An internal application for employees to manage product listings and monitor incoming orders. |
| **Order-Service**    | Manages the creation of orders and dispatches order data to a managed queue for processing. |
| **Product-Service**  | Provides CRUD functionality for managing product information. |
| **Makeline-Service** | Consumes order data from the queue and handles order preparation and completion. |
| **AI-Service**       | Utilizes GPT-4 and DALL·E 3 to automatically generate product descriptions and images. |
| **Database**         | A MongoDB instance used for storing persistent product and order data. |

## Architecture Diagram
![image](https://github.com/user-attachments/assets/f2558167-bdbe-4491-b828-2f4b17d43401)    

Customers use the store-front to examine merchandise while placing their orders but the store-front retrieves information from product-service then delivers it to customers. The order information passes from store-front to order-service which performs encryption before Azure Service Bus receives it. Makeline-service monitors messages in Azure Service Bus until it obtains the most recent entry which it saves into the mongodb database. Store-admin service retrieves order information through makeline-service from mongodb yet also makes use of ai-service for new product description generation (gpt-4) and image generation (dall-e-3).  

## Deployment Instructions

## Step 1: Clone BestBuy Demo Repository
aps-all-in-one.yaml: Defines the application deployment configuration, including integration with the service bus.  
secrets.yaml: Stores sensitive data, specifically the OpenAI service key.  
config-maps.yaml: Contains application configuration values and environment settings.  

## Step 2: Azure AKS Cluster Setup Guide

### 1️ Log in to Azure Portal
- Visit [https://portal.azure.com](https://portal.azure.com/) and sign in with your Azure account.

---

### 2️ Create a Resource Group
- In the Azure Portal search bar, type **Resource Groups** and select it.  
- Click **Create** and follow the prompts to set up a new resource group for this deployment.

---

### 3️ Create an AKS Cluster
- In the Azure Portal search bar, type **Kubernetes services** and select it.  
- Click **Create** and choose **Kubernetes cluster**.  
- Under the `Basics` tab, fill in the following details:

| Field                         | Value                              |
|:------------------------------|:------------------------------------|
| **Subscription**              | Select your active subscription.    |
| **Resource group**            | Choose the one you just created.    |
| **Cluster preset configuration** | `Dev/Test`                          |
| **Region**                    | Same as your resource group (e.g. `Canada`). |
| **Availability zones**        | `None`                               |
| **AKS pricing tier**          | `Free`                               |
| **Kubernetes version**        | `Default`                            |
| **Automatic upgrade**         | `Disabled`                           |
| **Automatic upgrade scheduler** | `No schedule`                      |
| **Node security channel type**| `None`                               |
| **Security channel scheduler**| `No schedule`                        |
| **Authentication and Authorization** | `Local accounts with Kubernetes RBAC` |

- Under **Node pools**:
  - Create a master pool and any additional worker pools as needed.
  - Set the **node size** to `D2as_v4`.

---

### 4️ Connect to the AKS Cluster

- After the cluster is deployed, go to your cluster’s overview page in the Azure Portal.  
- Click **Connect**.  
- In the **Azure CLI** tab, you’ll find connection commands.  

> If you don’t have the Azure CLI installed, follow the instructions here: [Install Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

**Steps to connect:**

1. Log in to your Azure account:

   ```bash
   az login
   ```

2. Set your subscription (replace `subscription-id` with the actual ID):

   ```bash
   az account set --subscription 'subscription-id'
   ```

3. Configure `kubectl` access to your AKS cluster (replace placeholders):

   ```bash
   az aks get-credentials --resource-group <your resource group> --name <your cluster name>
   ```

4. Test your connection by listing the cluster nodes:

   ```bash
   kubectl get nodes
   ```
## AI Service and Application Deployment Setup Guide

---

## Step 3: Set Up AI Backing Services

To enable AI-generated product descriptions and images in the Bestbuy Demo app, you’ll need to deploy **Azure OpenAI Services** for both GPT-4 (text) and DALL-E 3 (images).

### 3.1 Create an Azure OpenAI Service Instance

1. **Log in to Azure Portal**: [https://portal.azure.com/](https://portal.azure.com/)
2. **Create a New Resource**:
   - Click **Create a Resource**.
   - Search for **Azure OpenAI** in the Marketplace.
3. **Configure the Resource**:
   - Region: `East US` (ensures GPT-4 and DALL-E 3 availability)
   - Resource Group: Create or select an existing one.
   - Pricing Tier: `Standard`
4. **Deploy the Resource**: Click **Review + Create**, then **Create**.

### 3.2 Deploy GPT-4 and DALL-E 3 Models

1. Go to your newly created **Azure OpenAI Resource**.
2. In **Model Deployments**, click **Add Deployment**.
3. Deploy **GPT-4**:
   - Select **GPT-4** model.
   - Name it (e.g., `gpt-4-deployment`).
4. Deploy **DALL-E 3** similarly:
   - Choose **DALL-E 3**.
   - Name it (e.g., `dalle-3-deployment`).
5. **Save Configuration Details** for both:
   - Deployment Name
   - Endpoint URL

### 3.3 Retrieve and Encode API Keys

1. In your **Azure OpenAI Resource**, go to **Keys and Endpoints**.
2. Copy **API Key 1** and **Endpoint URL**.
3. Encode your API key in Base64:

   ```bash
   echo -n "<your-api-key>" | base64
   ```

---

## Step 4: Update AI Service Deployment Configuration

### 4.1 Update Secrets YAML

- Edit `secrets-openai.yaml`.
- Replace `OPENAI_API_KEY` with your Base64-encoded API key.

### 4.2 Update Deployment YAML

- Edit `bb-all-in-one-servicebus.yaml`.
- Update these placeholders:

```yaml
- name: AZURE_OPENAI_API_VERSION
  value: "2024-07-01-preview"
- name: AZURE_OPENAI_DEPLOYMENT_NAME
  value: "gpt-4-deployment"
- name: AZURE_OPENAI_ENDPOINT
  value: "https://<your-openai-resource-name>.openai.azure.com/"
- name: AZURE_OPENAI_DALLE_ENDPOINT
  value: "https://<your-openai-resource-name>.openai.azure.com/"
- name: AZURE_OPENAI_DALLE_DEPLOYMENT_NAME
  value: "dalle-3-deployment"
```

---

## Step 5: Deploy Secrets and the Application

### 5.1 Deploy Secrets

Apply your updated secret file:

```bash
kubectl apply -f secrets-openai.yaml
```

Verify deployment:

```bash
kubectl get configmaps
kubectl get secrets
```

### 5.2 Deploy Bestbuy Demo Application

Apply the deployment YAML to your AKS cluster:

```bash
kubectl apply -f bb-all-in-one-servicebus.yaml
```

Check the deployment:

```bash
kubectl get pods
kubectl get services
```

Access **store-front** and **store-admin** services via `EXTERNAL-IP:80`.

---

## Step 6: Implement CI/CD

A `ci-cd.yaml` is added to all Git repos (excluding `virtual-customer` and `virtual-worker`).

### 6.1 Set Up GitHub Secrets

For each forked repo:
- Go to **Settings > Secrets and variables > Actions**.
- Add these secrets:
  - `DOCKER_USERNAME`: Your Docker Hub username.
  - `DOCKER_PASSWORD`: Your Docker Hub password.
  - `KUBE_CONFIG_DATA`: Base64 content of your Kubernetes config file.

Generate `KUBE_CONFIG_DATA` with:

```bash
cat ~/.kube/config | base64 -w 0 > kube_config_base64.txt
```

Use the file content for `KUBE_CONFIG_DATA` in GitHub.

### 6.2 Set Up Repository Variables

In **Settings > Secrets and variables > Actions**, add:
- `DOCKER_IMAGE_NAME`: Example `store-front-bb-v1`
- `DEPLOYMENT_NAME`: Example `store-front`
- `CONTAINER_NAME`: Example `store-front`

---

✅ **All done — AI services and Bestbuy Demo are configured and ready to deploy!**

## Microservice Repositories

| Service         | Repository Link                                                                 |
|:----------------|:-------------------------------------------------------------------------------|
| Store-Front     | [Store-Front Repo](https://github.com/AbdulRafay2325/store-front-Bestbuy)               |
| Store-Admin     | [Store-Admin Repo](https://github.com/AbdulRafay2325/store-admin-Bestbuy)               |
| Order-Service   | [Order-Service Repo](https://github.com/AbdulRafay2325/order-service-Bestbuy)           |
| Product-Service | [Product-Service Repo](https://github.com/AbdulRafay2325/product-service-Bestbuy)       |
| Makeline-Service| [Makeline-Service Repo](https://github.com/AbdulRafay2325/makeline-service-Bestbuy)     |
| Ai-Service      | [Ai-Service Repo](https://github.com/AbdulRafay2325/Microservice-repos/blob/main/ai-service-Bestbuy)                 |
| Virtual-Worker  | [Virtual-Worker Repo](https://github.com/AbdulRafay2325/virtual-worker-Bestbuy)         |
| Virtual-Customer| [Virtual-Customer Repo](https://github.com/AbdulRafay2325/virtual-customer-Bestbuy)     |

## Docker Images

| Service         | Docker Image Link                                                              |
|:----------------|:----------------------------------------------------------------------------|
| Store-Front     | [Docker Hub](https://hub.docker.com/r/abdulrafay2325/store-front-bestbuy)               |
| Store-Admin     | [Docker Hub](https://hub.docker.com/r/abdulrafay2325/store-admin-bestbuy)               |
| Order-Service   | [Docker Hub](https://hub.docker.com/r/abdulrafay2325/order-service-bestbuyf)             |
| Product-Service | [Docker Hub](https://hub.docker.com/r/abdulrafay2325/product-service-bestbuyf)           |
| Makeline-Service| [Docker Hub](https://hub.docker.com/r/abdulrafay2325/makeline-service-bestbuy)          |
| Ai-Service      | [Docker Hub](https://hub.docker.com/r/abdulrafay2325/ai-service-bestbuy)                |



## Conclusion: Challenges Faced During Deployment

Throughout this deployment process, a few challenges surfaced:

- **Azure OpenAI Service Regional Availability**: Faced initial availability issues for GPT-4 and DALL-E 3 models in the preferred regions, which required adjustments to deploy resources specifically in the `East US` region to proceed.

- **YAML Configuration Precision**: Encountered minor syntax issues in the YAML configuration files, particularly with indentation and environment variable placeholders, leading to deployment errors that required careful review and debugging.
