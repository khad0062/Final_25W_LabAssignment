
# Final_25W_LabAssignment


 - **Table of Microservice Repositories**:  
     - A table listing each microservice repository and its GitHub link.  
     - Example:

| Service | Github Repo |
| --- |--- |
| `store-front` | [store-front-L8](https://github.com/khad0062/store-front-L8)|
| `store-admin` | [store-admin-L8](https://github.com/khad0062/store-admin-L8) |
| `order-service`| [order-service-L8](https://github.com/khad0062/order-service-L8) |
| `product-service`| [product-service-L8](https://github.com/khad0062/product-service-L8) |
| `makeline-service` | This service handles processing orders from the queue and completing them (Golang) | [makeline-service-L8](https://github.com/ramymohamed10/makeline-service-L8) |
| `ai-service` | Optional service for adding generative text and graphics creation (Python) | [ai-service-L8](https://github.com/ramymohamed10/ai-service-L8) |
| `rabbitmq` | RabbitMQ for an order queue | [rabbitmq](https://github.com/docker-library/rabbitmq) |
| `mongodb` | MongoDB instance for persisted data | [mongodb](https://github.com/docker-library/mongo) |
| `virtual-customer` | Simulates order creation on a scheduled basis (Rust) | [virtual-customer-L8](https://github.com/ramymohamed10/virtual-customer-L8) |
| `virtual-worker` | Simulates order completion on a scheduled basis (Rust) | [virtual-worker-L8](https://github.com/ramymohamed10/virtual-worker-L8) |



   - **Table of Microservice Repositories**:  
     - A table listing each microservice repository and its GitHub link.  
     - Example:
       | **Service**         | **Repository Link**                       |
       |---------------------|-------------------------------------------|
       | Store-Front         | `<https://hub.docker.com/r/khad0062/store-front>`                           |
       | Order-Service       | [Order-Service](https://hub.docker.com/r/khad0062/order-service)
       | p                       |
       | Makeline-service           | [makeline-service](https://hub.docker.com/r/khad0062/makeline-service)|

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

