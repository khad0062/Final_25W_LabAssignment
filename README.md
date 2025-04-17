
# Final_25W_LabAssignment


 - **Table of Microservice Repositories**:  
     - A table listing each microservice repository and its GitHub link. 

| Service |Description|Github Repo |
| --- |--- |---|
| `store-front` |Web app for customers to place orders (Vue.js)| [store-front](https://github.com/khad0062/store-front-L8)|
| `store-admin` |Web app used by store employees to view orders in queue and manage products (Vue.js)| [store-admin](https://github.com/khad0062/store-admin-L8) |
| `order-service`|This service is used for placing orders (Javascript)	| [order-service](https://github.com/khad0062/order-service-L8) |
| `product-service`|This service is used to perform CRUD operations on products (Rust)	| [product-service](https://github.com/khad0062/product-service-L8) |
| `makeline-service` | This service handles processing orders from the queue and completing them (Golang) | [makeline-service](https://github.com/khad0062/makeline-service-L8) |
| `ai-service` | Optional service for adding generative text and graphics creation (Python) | [ai-service](https://github.com/khad0062/ai-service-L8) |
| `mongodb` | MongoDB instance for persisted data | [mongodb](https://github.com/khad0062/mongo) |
| `virtual-customer` | Simulates order creation on a scheduled basis (Rust) | [virtual-customer](https://github.com/khad0062/virtual-customer) |
| `virtual-worker` | Simulates order completion on a scheduled basis (Rust) | [virtual-worker](https://github.com/khad0062/virtual-worker) |



   - **Table of Microservice Repositories**:  
     - A table listing each microservice repository and its GitHub link.  
     - Example:
       | **Service**         | **Docker Hub Link**                       |
       |---------------------|-------------------------------------------|
       | Store-Front         | `<https://hub.docker.com/r/khad0062/store-front>`                           |
       | store-admin         | [store-admin](https://hub.docker.com/r/khad0062/store-admin)     |
       | Order-Service       | [Order-Service](https://hub.docker.com/r/khad0062/order-service)
       | product-service     |  [product-service](https://hub.docker.com/r/khad0062/product-service)                  |
       | makeline-service    | [makeline-service](https://hub.docker.com/r/khad0062/makeline-service)|
       | ai- serivce         |  [ai-service](https://hub.docker.com/r/khad0062/ai-service)  |
       | virtual-worker      |  [virtual-worker](https://hub.docker.com/r/khad0062/virtualworker)  |
       |

# Step 5: Set Up the Azure Service Bus

You'll set up a Service Bus namespace, create a queue, and configure both the Order and Makeline services to communicate securely using shared access policies.

---

## Task 1: Create the Service Bus and Queue

Use the Azure CLI to create your Service Bus namespace and queue:

```bash
az servicebus namespace create \
  --name asb-cst8915-la2 \
  --resource-group rg-cst8915-la2

az servicebus queue create \
  --name orders \
  --namespace-name asb-cst8915-la2 \
  --resource-group rg-cst8915-la2
```

---

## Task 2: Set Up Shared Access Policies

### Sender Policy (for Order Service):

```bash
az servicebus queue authorization-rule create \
  --name sender \
  --namespace-name asb-cst8915-la2 \
  --resource-group rg-cst8915-la2 \
  --queue-name orders \
  --rights Send
```

### Listener Policy (for Makeline Service):

```bash
az servicebus queue authorization-rule create \
  --name listener \
  --namespace-name asb-cst8915-la2 \
  --resource-group rg-cst8915-la2 \
  --queue-name orders \
  --rights Listen
```

---

## Task 3: Configure the Order Service

### Get the Access Key:

- Go to the **Azure portal > Service Bus resource > Shared Access Policies**
- Click on the **sender** policy and copy the **Primary Key**

### Base64 Encode the Key:

```bash
echo -n "<sender-access-key>" | base64
```
> Replace `<sender-access-key>` with the actual key you copied.

### Get the Hostname:

- Go to the **Overview** section of the Service Bus resource and copy the **Host name**

### Update Your Deployment Files:

- Edit `secrets.yaml` and replace `Base64-encoded-Service-Bus-Sender-Key` with the encoded key.
- Edit `order-service.yaml` and set `ORDER_QUEUE_HOSTNAME` to the hostname.

---

## Task 4: Configure the Makeline Service

### Get the Access Key:

- Go to **Shared Access Policies > listener** and copy the **Primary Key**

### Base64 Encode the Key:

```bash
echo -n "<listener-access-key>" | base64
```
> Replace `<listener-access-key>` with the actual key.

### Get the Hostname:

- Again, from the **Overview** section, copy the **Host name**

### Update Your Deployment Files:

- Edit `secrets.yaml` and replace `Base64-encoded-Service-Bus-Listener-Key` with the encoded value.
- Edit `makeline-service.yaml` and set `ORDER_QUEUE_URI` to:

```plaintext
amqps://<hostname>
```
> Replace `<hostname>` with the actual hostname from Azure.

