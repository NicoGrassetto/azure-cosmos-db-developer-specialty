# Plan and implement Azure Cosmos DB for NoSQL
## Plan Resource Requirements
### Introduction

Creating a new Azure Cosmos DB account often requires making many configuration choices that can, at first, be daunting. While the defaults fit numerous scenarios, it makes the most sense to familiarize yourself with the configuration options to ensure that your account and resources are optimally configured for your solution. 
### Understand throughput

In our basic hierarchy of resources, an Azure Cosmos DB for NoSQL database is a unit of management for a set of schema-agnostic containers. Each container is a unit of scalability for both **throughput** and **storage**.

Containers are partitioned horizontally across compute within an Azure region. Container are distributed across all partitions within a container and all Azure Regions you configure in your Azure Cosmos DB for NoSQL account.

Even with the throughput distributed across the partitions for a container and regions in an account, with Dynamic Autoscale, only those partitions and regions in which requests are occurring will be scaled up. Dynamic Autoscale ensures that costs reflect only the throughput consumed by an application.

#### Container-level throughput provisioning

![Throughput provisioned at container level](https://learn.microsoft.com/en-gb/training/wwl-data-ai/plan-resource-requirements/media/2-container.png)

Any throughput provisioned exclusively at the container level is reserved only for this container. This throughput is available only for this container all the time. This throughput is also financially backed by SLAs (Service Level Agreements).

Users creating new accounts should look to start with Serverless as it only bills when requests are made. Serverless makes it cost efficient when developing new applications. It also is suitable for production-grade applications with infrequent requests or otherwise low usage requirements.

>**Note**: Users can also provision throughput for a database that is shared across all containers. However, this should only be used when all of the containers have roughly the same volume of requests and data as it is not possible to guarantee performance when these vary widely across containers.

### Evaluate throughput requirements

Request units are a rate-based currency. They are used to make it simple to talk about physical resources like memory, CPU, and IO when performing requests in Azure Cosmos DB. For example, it’s easier to think of 10 request units as roughly twice as much as five request units in a relative sense without worrying about the physical resources that are abstracted away. Request units are used to measure both foreground and background activities.

Every request consumes a fixed number of request units, including but not limited to:

- Reads
- Writes (including indexing operations)
- Queries

#### Configuring throughput

When you create a database or container in Azure Cosmos DB, you can provision request units in an increment of request units per second (or RU/s for short). For _standard provisioned throughput_, the minimum amount you can provision is 400 RU/s. For _autoscale provisioned throughput_, the minimum is 1000 RU/s that will scale down to 100 RU/s.

#### Estimating ad-hoc RU/s consumption

Some RU/s are normalized across various access methods, making many common operations predictable. Using this knowledge, you can perform some basic estimations for simple workloads. For example, you can estimate the RU/s required for common database operations such as one RU for a read operation for a 1-KB document and six RU for a write operation of a 1-KB document with indexing turned off.

![Request units diagram with estimates](https://learn.microsoft.com/en-gb/training/wwl-data-ai/plan-resource-requirements/media/3-request-units.png)

Using this strategy, you should identify your solution's query and access patterns to make an educated guess as to how many request units will be needed in Azure Cosmos DB. To accomplish this, you will want information such as:

- Top five queries
- Number of read operations per second
- Number of write operations per second

>**Tip**: Measuring RU/s for queries should be done at scale. Measuring queries running on a single physical partition will not yield significant data on the actual throughput used in your real world scenario once it is deployed and scaled out.

You can use a spreadsheet application to build a quick table to figure out a rough estimate of your needed request unit capacity. Here's a quick example:

|**Operation type**|**Number of requests per second**|**Number of RU per request**|**RU/s needed**|
|---|---|---|---|
|**Write Single Document**|10,000|10|100,000|
|**Top Query #1**|700|100|70,000|
|**Top Query #2**|200|100|20,000|
|**Top Query #3**|100|100|10,000|
|**Total RU/s**|||200,000 RU/s|

>**Tip**: You can also run a proof of concept application, and use the **request charge** property of the SDK to measure the real-world RU charge of running the operations that you intend to make against Azure Cosmos DB.

### Evaluate data storage requirements

Planning for a new Azure Cosmos DB account is composed of two components; throughput and storage. While we have already discussed throughput, data in Azure Cosmos DB will also consume SSD storage billed per GB per month.

#### Migrating existing transactional workloads

The Azure Cosmos DB Capacity Calculator is a calculator surfaced as an online form to plug in details about your existing data workload to help estimate your application's storage and throughput requirements and translate it to a cost estimate in terms of Azure Cosmos DB.

[![Screenshot of the Azure Cosmos DB Capacity Calculator](https://learn.microsoft.com/en-gb/training/wwl-data-ai/plan-resource-requirements/media/4-calculator.png)](https://learn.microsoft.com/en-gb/training/wwl-data-ai/plan-resource-requirements/media/4-calculator.png#lightbox)

>**Note**: The costs detailed in this example may not accurately reflect current storage costs or your current region.

The calculator will inquire about details such as:

- Total data stored
- Whether you expect to perform near real-time analytics
- The anticipated size of documents
- Point reads per second
- Queries per second

After your rough estimate and proof of concept, use the capacity calculator to refine your estimate further to a much more accurate cost to run your solution in Azure Cosmos DB.
### Time-to-live (TTL)

Azure Cosmos DB allows you to set the length of time documents live in the database before being automatically purged. A document's "time-to-live" (TTL) is measured in seconds from the last modification and can be set at the container level with the ability to override on a per-item basis.

Once set at the container level, Azure Cosmos DB will automatically purge documents at the specified time since they were last modified. The TTL value is defined as an integer in seconds.

>**Tip**: The maximum TTL value is 2147483647.

TTL expiration is a background task performed in the background using request units and is scheduled when quiescent.

#### Configuring TTL on a container

The TTL value for a container is configured using the `DefaultTimeToLive` property of the container's JSON object.

|**DefaultTimeToLive**|**Expiration**|
|---|---|
|_Does not exist_|Items are not automatically expired|
|`-1`|Items will not expire by default|
|_n_|_n_ seconds after last modified time|

The TTL value for an item is configured by setting the `ttl` path of the item. The TTL value for an item will only work if the `DefaultTimeToLive` property is configured for the parent container. If the `ttl` path is configured for the item, it will override the `DefaultTimeToLive` property of the parent container.

#### Examples

|**Container.DefaultTimeToLive**|**Item.ttl**|**Expiration in seconds**|
|---|---|---|
|`1000`|_null_|`1000`|
|`1000`|`-1`|_This item will never expire_|
|`1000`|`2000`|`2000`|

|**Container.DefaultTimeToLive**|**Item.ttl**|**Expiration in seconds**|
|---|---|---|
|_null_|_null_|_This item will never expire_|
|_null_|`-1`|_This item will never expire_|
|_null_|`2000`|_TTL is disabled at the container level. This item will never expire._|
### Plan for data retention with time-to-live (TTL)

Azure Cosmos DB only charges for storage you directly consume in real time, and you don't have to prereserve storage in advance. In high-write scenarios, TTL (Time to Live) values can be used to save on data storage costs in Azure Cosmos DB. Data that already shipped out to data warehouses or aggregated and stored in other forms or elsewhere can be immediately purged. This purge ensures that you only keep fresh and relevant data in local SSD storage.

Consider solutions such to aggregate and migrate data such as:

- Change feed
- Azure Fabric
- Azure Blob Storage

When you are designing your solution, to minimize storage costs, game plan how long your data will need to be retained in Azure Cosmos DB before the data is migrated across your entire Azure solution space.
## Configure Azure Cosmos DB for NoSQL
### Introduction

Applications come in many different patterns, they may have predictable usage, or they could be spiked with sudden traffic surges. Azure Cosmos DB has multiple ways to host workloads that map directly to how applications run in the real world, whether the applications are predictable or entirely growth-based.
### Serverless

Azure Cosmos DB serverless is a consumption-based model where each request consumes request units. The consumption model eliminates the need to pre-provision throughput request units ahead of time.

Remember, when using Azure Cosmos DB, you typically express database options as a cost described in Request Units per second.

#### What are use cases for serverless?

![Diagram of individual requests consuming RU/s](https://learn.microsoft.com/en-gb/training/wwl-data-ai/configure-azure-cosmos-db-sql-api/media/2-serverless.png)

Serverless is great for development and testing as well as applications with unpredictable or bursty traffic. You can use serverless with an application such as:

- A new application with hard to forecast users loads
- A new prototype application within your organization
- Serverless compute integration with a service like Azure Functions
- Just getting started with Azure Cosmos DB as a new developer
- Low traffic application that doesn’t send or receive numerous data
### Compare serverless vs. provisioned throughput

How do you choose between serverless and provisioned throughput?

#### Compare workloads

Provisioned throughput is ideal for workloads with predictable traffic patterns that require sustained and predictable performance with minimal variance.

On the other hand, serverless can handle workloads that have wildly varying traffic and low average-to-peak traffic ratios.

#### Compare request units

Provisioned throughput makes some number of request units available each second to each container. The number of request units can be updated either manually or via autoscale.

Serverless doesn’t require any planning or automatic provisioning and can deliver throughput up to a documented service limit.

#### Compare global distribution

Provisioned throughput supports distributing your data to an unlimited number of Azure regions.

Serverless accounts can only run in a single Azure region but can be used with Availability Zones in a single region.

#### Compare storage limits

Provisioned throughput allows you to store unlimited data in a container.

Serverless allows up to 1 TB of data in a container.

#### Migrating between serverless and provisioned throughput

Serverless accounts are capable of being migrated to autoscale throughput if the container needs more resources than serverless can provide. Accounts can also be migrated from autoscale to serverless if needed.
### Autoscale throughput

We can often make an educated guess about where our workload is in terms of throughput, but we don’t know exactly where it lands until it’s in production. We also may know our operational tolerances. We know:

- The maximum amount of money we're willing to spend
- The minimum amount of performance we're ready to tolerate.

With all of this information in mind, we could define a range. This range would represent our application running at a comfortable performance level without overspending without our knowledge.

![Application with usage oscillating between maximum potential spend and minimum performance](https://learn.microsoft.com/en-gb/training/wwl-data-ai/configure-azure-cosmos-db-sql-api/media/4-autoscale-1.png)

With Azure Cosmos DB autoscale, we can define a range of request units per second (RU/s) to scale our container automatically and instantly. The throughput RU/s is scaled based on real-time usage instantly.

![Autoscale between max and min RU/s](https://learn.microsoft.com/en-gb/training/wwl-data-ai/configure-azure-cosmos-db-sql-api/media/4-autoscale-2.png)

Autoscale is great for workloads with variable or unpredictable traffic patterns and can minimize unused capacity that would typically be pre-provisioned.
### Compare autoscale vs. standard (manual) throughput

How do you choose between autoscale and standard throughput?

#### Compare workloads

Standard throughput is again suited for workloads with steady traffic.

Autoscale throughput is better suited for unpredictable traffic. Autoscale can ensure that your actual Azure Cosmos DB provisioned throughput oscillates between your minimal acceptable performance and maximum allowed spend.

#### Compare request units

Standard throughput requires a static number of request units to be assigned ahead of time.

With autoscale, you only set the maximum, and the minimum billed will be 10% of the maximum when there are zero requests.

#### Compare scenarios

You want to use standard throughput provisioning in scenarios when your team can accurately predict the amount of throughput your application needs, and your team suspects these needs will not change over time. Also throughput provisioning is ideal for scenarios where the full RU/s provisioned is consumed for > 66% of hours per month.

Autoscale throughput is helpful if your team cannot predict your throughput needs accurately or otherwise use the max throughput amount for < 66% of hours per month.

#### Compare rate-limiting

The standard throughput will always remain static at the set RU/s that is provisioned. Requests beyond this will be rate-limited, with a response indicating that a wait should be attempted before retrying.

Autoscale will scale up to the max RU/s before similarly rate-limiting responses.
### Migrate between standard (manual) and autoscale throughput

Existing containers can be migrated to and from autoscale using the Azure portal, Azure CLI, or Azure PowerShell. During the migration process, the system will automatically apply a request unit per second (RU/s) value to the container.

>**Note**: You can always change this RU/s value after the migration has occurred.

### Configure throughput for Azure Cosmos DB for NoSQL with the Azure portal

One of the most important things to wrap your head around is configuring throughput in Azure Cosmos DB for NoSQL. To create an Azure Cosmos DB for NoSQL container, you must first create an account and then a database; in that order.

In this lab, you will provision throughput using various methods in the Data Explorer. You will provision throughput either manually or using autoscale, at the database and the container level.

#### Create a serverless account

Let’s start simple by creating a serverless account. There’s not much to configure here since everything is serverless. When we create our database and container, we don’t have to provision throughput at all. You will see all of that as we step into creating this account.

1. In a new web browser window or tab, navigate to the Azure portal (`portal.azure.com`).
    
2. Sign into the portal using the Microsoft credentials associated with your subscription.
    
3. Within the **Azure services** category, select **Create a resource**, and then select **Azure Cosmos DB**.
    
    > 💡 Alternatively; expand the **≡** menu, select **All Services**, in the **Databases** category, select **Azure Cosmos DB**, and then select **Create**.
    
4. In the **Select API option** pane, select the **Create** option within the **Azure Cosmos DB for NoSQL** section.
    
5. Within the **Create Azure Cosmos DB Account** pane, observe the **Basics** tab.
    
6. On the **Basics** tab, enter the following values for each setting:
    
    |**Setting**|**Value**|
    |---|---|
    |**Workload Type**|**Learning**|
    |**Subscription**|**Use your existing Azure subscription.** _All resources must belong to a resource group. Every resource group must belong to a subscription._|
    |**Resource Group**|**Use existing or create a new resource group.** _All resources must belong to a resource group._|
    |**Account Name**|**Enter any globally unique name.** _The globally unique account name. This name will be used as part of the DNS address for requests. The portal will check the name in real time._|
    |**Location**|**Choose any available region.** _Select the geographical region from which your database will initially be hosted._|
    |**Capacity mode**|**Select Serverless**|
    
7. Select **Review + Create** to navigate to the **Review + Create** tab, and then select **Create**.
    
    > 📝 It can take 10-15 minutes for the Azure Cosmos DB for NoSQL account to be ready for use.
    
8. Observe the **Deployment** pane. When the deployment is complete, the pane will update with a **Deployment successful** message.
    
9. Still within the **Deployment** pane, select **Go to resource**.
    
10. From within the **Azure Cosmos DB account** pane, select **Data Explorer** from the resource menu.
    
11. In the **Data Explorer** pane, expand **New Container** and then select **New Database**.
    
12. In the **New Database** popup, enter the following values for each setting, and then select **OK**:
    
    |**Setting**|**Value**|
    |---|---|
    |**Database id**|_`cosmicworks`_|
    
13. Back in the **Data Explorer** pane, observe the **cosmicworks** database node within the hierarchy.
    
14. In the **Data Explorer** pane, select **New Container**.
    
15. In the **New Container** popup, enter the following values for each setting, and then select **OK**:
    
    |**Setting**|**Value**|
    |---|---|
    |**Database id**|_Use existing_ &vert; _cosmicworks_|
    |**Container id**|_`products`_|
    |**Partition key**|_`/category/name`_|
    
16. Back in the **Data Explorer** pane, expand the **cosmicworks** database node and then observe the **products** container node within the hierarchy.
    
17. Return to the **Home** of the Azure portal.
    

#### Create a provisioned account

Now, we are going to create a provisioned throughput account with more traditional configuration options. This type of account will open up a world of configuration options for us which can be a bit overwhelming. We are going to walk through a few examples of database and container pairings that are possible here.

1. Within the **Azure services** category, select **Create a resource**, and then select **Azure Cosmos DB**.
    
    > 💡 Alternatively; expand the **≡** menu, select **All Services**, in the **Databases** category, select **Azure Cosmos DB**, and then select **Create**.
    
2. In the **Select API option** pane, select the **Create** option within the **Azure Cosmos DB for NoSQL** section.
    
3. Within the **Create Azure Cosmos DB Account** pane, observe the **Basics** tab.
    
4. On the **Basics** tab, enter the following values for each setting:
    
    |**Setting**|**Value**|
    |---|---|
    |**Workload Type**|**Learning**|
    |**Subscription**|**Use your existing Azure subscription.** _All resources must belong to a resource group. Every resource group must belong to a subscription._|
    |**Resource Group**|**Use existing or create a new resource group.** _All resources must belong to a resource group._|
    |**Account Name**|**Enter any globally unique name.** _The globally unique account name. This name will be used as part of the DNS address for requests. The portal will check the name in real time._|
    |**Location**|**Choose any available region.** _Select the geographical region from which your database will initially be hosted._|
    |**Capacity mode**|**Provisioned throughput**|
    |**Apply Free Tier Discount**|**Do Not Apply**|
    |**Limit the total amount of throughput that can be provisioned on this account**|**Unchecked**|
    
5. Select **Review + Create** to navigate to the **Review + Create** tab, and then select **Create**.
    
    > 📝 It can take 10-15 minutes for the Azure Cosmos DB for NoSQL account to be ready for use.
    
6. Observe the **Deployment** pane. When the deployment is complete, the pane will update with a **Deployment successful** message.
    
7. Still within the **Deployment** pane, select **Go to resource**.
    
8. From within the **Azure Cosmos DB account** pane, select **Data Explorer** from the resource menu.
    
9. In the **Data Explorer** pane, expand **New Container** and then select **New Database**.
    
10. In the **New Database** popup, enter the following values for each setting, and then select **OK**:
    
    |**Setting**|**Value**|
    |---|---|
    |**Database id**|_`nothroughputdb`_|
    |**Provision throughput**|_Unchecked_|
    
11. Back in the **Data Explorer** pane, observe the **nothroughputdb** database node within the hierarchy.
    
12. In the **Data Explorer** pane, select **New Container**.
    
13. In the **New Container** popup, enter the following values for each setting, and then select **OK**:
    
    |**Setting**|**Value**|
    |---|---|
    |**Database id**|_Use existing_ &vert; _nothroughputdb_|
    |**Container id**|_`requiredthroughputcontainer`_|
    |**Partition key**|_`/primarykey`_|
    |**Container throughput**|_Manual_|
    |**RU/s**|_`400`_|
    
14. Back in the **Data Explorer** pane, expand the **nothroughputdb** database node and then observe the **requiredthroughputcontainer** container node within the hierarchy.
    
15. In the **Data Explorer** pane, expand **New Container** and then select **New Database**.
    
16. In the **New Database** popup, enter the following values for each setting, and then select **OK**:
    
    |**Setting**|**Value**|
    |---|---|
    |**Database id**|_`manualthroughputdb`_|
    |**Provision throughput**|_Checked_|
    |**Database throughput**|_Manual_|
    |**RU/s**|_`400`_|
    
17. Back in the **Data Explorer** pane, observe the **manualthroughputdb** database node within the hierarchy.
    
18. In the **Data Explorer** pane, select **New Container**.
    
19. In the **New Container** popup, enter the following values for each setting, and then select **OK**:
    
    |**Setting**|**Value**|
    |---|---|
    |**Database id**|_Use existing_ &vert; _manualthroughputdb_|
    |**Container id**|_`childcontainer`_|
    |**Partition key**|_`/primarykey`_|
    |**Provision dedicated throughput for this container**|_Checked_|
    |**Container throughput**|_Manual_|
    |**RU/s**|_`1000`_|
    
20. Back in the **Data Explorer** pane, expand the **manualthroughputdb** database node and then observe the **childcontainer** container node within the hierarchy.

## Move data into and out of Azure Cosmos DB for NoSQL
### Introduction

Shortly after creating a new Azure Cosmos DB account, it’s not uncommon to migrate new data into the account. There are services both in and out of Azure that you may want to consider for this task.
### Move data by using Azure Data Factory

**Azure Data Factory** is a native service to extract data, transform it, and load it across sinks and stores in an entirely serverless fashion. From a data integration perspective, this means you can marshal data from one datastore to another, regardless of the nuances of each, as long as you can reasonably transform the data between each data paradigm.

#### Setup

Azure Cosmos DB for NoSQL is available as a **linked service** within Azure Data Factory. This type of linked service is supported both as a **source** of data ingest and as a target (**sink**) of data output. For both, the configuration is identical. Make sure you enabled the managed identity and role-based access control (RBAC) permissions on your Cosmos DB account. You can configure the service using the Azure portal, or alternatively using a JSON object.

```json
{
    "name": "<example-name-of-linked-service>",
    "properties": {
        "type": "CosmosDb",
        "typeProperties": {
            "accountEndpoint": "https://<cosmos-account-name>.documents.azure.com:443/",
            "authenticationType": "ManagedIdentity"
        }
    }
}
```

#### Read from Azure Cosmos DB

In Azure Data Factory, when reading data from Azure Cosmos DB for NoSQL, we must configure our linked service as a **source**. This setting reads data in. To configure this option, we must create a SQL query of the data we want to read in. For example, we write a query such as `SELECT id, categoryId, price, quantity, name FROM products WHERE price > 500` to filter the items from the container that we want to read in from Azure Cosmos DB for NoSQL to Azure Data Factory to be transformed and then eventually loaded into our destination data store.

In Azure Data Factory, our **source** activity has a configuration JSON object that we can use to set properties such as the query:

```json
{
    "source": {
        "type": "CosmosDbSqlApiSource",
        "query": "SELECT id, categoryId, price, quantity, name FROM products WHERE price > 500",
        "preferredRegions": [
            "East US",
            "West US"
        ]        
    }
}
```

#### Write to Azure Cosmos DB

In Azure Data Factory, when storing data to Azure Cosmos DB for NoSQL, we must configure our linked service as a **sink**. This setting loads our data. To configure this option, we must set our write behavior. For example, we might want to insert our data, or we might want to upsert our data and overwrite any items that might have a matching unique identifier (**id** field).

Our **sink** activity also had a configuration JSON object:

```json
"sink": {
    "type": "CosmosDbSqlApiSink",
    "writeBehavior": "upsert"
}
```
### Move data by using a Kafka connector

**Apache Kafka** is an open-source platform used to stream events in a distributed manner. Many companies use Kafka for large-scale high-performance data integration scenarios. **Kafka Connect** is a tool within their suite to stream data between Kafka and other data systems. Understandably, this can include Azure Cosmos DB as a source of data or a target (sink) of data.

#### Setup

The Kafka Connect connectors for Azure Cosmos DB is available as an open-source project on GitHub at [microsoft/kafka-connect-cosmosdb](https://github.com/microsoft/kafka-connect-cosmosdb). Instructions for downloading and installing the JAR file manually are available at the repository.

##### Configuration

Four configuration properties should be set to properly configure connectivity to an Azure Cosmos DB for NoSQL account.

|**Property**|**Value**|
|---|---|
|**connect.cosmos.connection.endpoint**|Account endpoint URI|
|**connect.cosmos.master.key**|Account key|
|**connect.cosmos.databasename**|Name of the database resource|
|**connect.cosmos.containers.topicmap**|Using CSV format, a mapping of the Kafka topics to containers|

##### Topics to containers map

Each container should be mapped to a topic. For example, suppose you would like the **products** container to be mapped to the **prodlistener** topic and the **customers** container to the **custlistener** topic. In that case, you should use the following CSV mapping string: `prodlistener#products,custlistener#customers`.

#### Write to Azure Cosmos DB

Let’s write data to Azure Cosmos DB by creating a topic. In Apache Kafka, all messages are sent via topics.

You can create a new topic using the **kafka-topics** command. This example will make a new topic named **prodlistener**.

```bash
kafka-topics --create \
    --zookeeper localhost:2181 \
    --topic prodlistener \
    --replication-factor 1 \
    --partitions 1
```

The following command will start a producer so you can write three records to the prodlistener topic.

```bash
kafka-console-producer \
    --broker-list localhost:9092 \
    --topic prodlistener
```

And in the console, you can then enter these three records to the topic. Once this is done, these records will be committed to the Azure Cosmos DB for NoSQL container mapped to the topic (**products**).

```json
{"id": "0ac8b014-c3f4-4db0-8a1f-434bab460938", "name": "handlebar", "categoryId": "78148556-4e84-44be-abae-9755dde9c9e3"}
{"id": "54ba00da-50cf-44d8-b122-1d18bd1db400", "name": "handlebar", "categoryId": "eb642a5e-0c6f-4c83-b96b-bb2903b85e59"}
{"id": "381dde84-e6c2-4583-b66c-e4a4116f7d6e", "name": "handlebar", "categoryId": "cf8ae707-6d74-4563-831a-06e15a70a0dc"}
```

#### Read from Azure Cosmos DB

You can create a source connector in Kafka Connect using a JSON configuration object. In this sample configuration below, most of the properties should be left unchanged, but be sure to change the following values:

|**Property**|**Description**|
|---|---|
|**connect.cosmos.connection.endpoint**|Your actual account endpoint URI|
|**connect.cosmos.master.key**|Your actual account key|
|**connect.cosmos.databasename**|The name of your actual account database resource|
|**connect.cosmos.containers.topicmap**|Using CSV format, a mapping of your actual Kafka topics to containers|


```json
{
  "name": "cosmosdb-source-connector",
  "config": {
    "connector.class": "com.azure.cosmos.kafka.connect.source.CosmosDBSourceConnector",
    "tasks.max": "1",
    "key.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "connect.cosmos.task.poll.interval": "100",
    "connect.cosmos.connection.endpoint": "<cosmos-endpoint>",
    "connect.cosmos.master.key": "<cosmos-key>",
    "connect.cosmos.databasename": "<cosmos-database>",
    "connect.cosmos.containers.topicmap": "<kafka-topic>#<cosmos-container>",
    "connect.cosmos.offset.useLatest": false,
    "value.converter.schemas.enable": "false",
    "key.converter.schemas.enable": "false"
  }
}
```

As an illustrative example, using this example configuration table:

|**Property**|**Description**|
|---|---|
|**connect.cosmos.connection.endpoint**|`https://dp420.documents.azure.com:443/`|
|**connect.cosmos.master.key**|`C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`|
|**connect.cosmos.databasename**|`cosmicworks`|
|**connect.cosmos.containers.topicmap**|`prodlistener#products`|

Here is an example configuration file:

```json
{
  "name": "cosmosdb-source-connector",
  "config": {
    "connector.class": "com.azure.cosmos.kafka.connect.source.CosmosDBSourceConnector",
    "tasks.max": "1",
    "key.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "connect.cosmos.task.poll.interval": "100",
    "connect.cosmos.connection.endpoint": "https://dp420.documents.azure.com:443/",
    "connect.cosmos.master.key": "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==",
    "connect.cosmos.databasename": "cosmicworks",
    "connect.cosmos.containers.topicmap": "prodlistener#products",
    "connect.cosmos.offset.useLatest": false,
    "value.converter.schemas.enable": "false",
    "key.converter.schemas.enable": "false"
  }
}
```

Once configured, data from the Azure Cosmos DB change feed will be published to a Kafka topic.
### Move data by using Stream Analytics

**Azure Stream Analytics** is a real-time event-processing engine designed to process fast streaming data from multiple sources simultaneously. It can aggregate, analyze, transform, and even move data around to other data stores for more profound and further analysis.

#### Setup

Azure Stream Analytics supports multiple output sinks including Azure Cosmos DB for NoSQL.

>**Note**: As of this time, only the NoSQL API is supported.

##### Configuration

Configuring the Azure Cosmos DB for NoSQL output consists of either selecting the account within your subscription or providing your credentials, which commonly include:

|**Property**|**Description**|
|---|---|
|`Output alias`|An alias to refer to this output in the query|
|`Account ID`|Account endpoint URI|
|`Account Key`|Account key|
|`Database`|Name of the database resource|
|`Container name`|Name of the container|

The database and container must already exist in the Azure Cosmos DB for NoSQL account before using the output sink.

#### Write to Azure Cosmos DB

Query results from Azure Stream Analytics will be processed as JSON output when written to Azure Cosmos DB for NoSQL.

Additionally, items are **upserted** to Azure Cosmos DB for NoSQL based on the value of the **id** field. Items are typically inserted into Azure Cosmos DB for NoSQL. If an item already exists with the same unique id, then the operation is assumed to be an **update** operation instead of an **insert** operation.
### Move data by using the Azure Cosmos DB Spark connector

With **Azure Synapse Analytics** and **Azure Synapse Link** for **Azure Cosmos DB**, you can create a cloud-native hybrid transactional and analytical processing (HTAP) to run analytics over your data in Azure Cosmos DB for NoSQL. This connection enables integration over your data pipeline on both ends of your data world, Azure Cosmos DB and Azure Synapse Analytics.

#### Setup

First, you should make sure **Synapse Link** is enabled at the account level. This can be accomplished using the Azure portal or by using the Azure CLI:

```bash
az cosmosdb create --name <name> --resource-group <resource-group> --enable-analytical-storage true
```

You can also use Azure PowerShell:

```powershell
New-AzCosmosDBAccount -ResourceGroupName <resource-group> -Name <name>  -Location <location> -EnableAnalyticalStorage true
```

When creating a container, you should enable analytical storage at the container level on a per container basis. Again this can be accomplished with the portal.

This can also be accomplished with the CLI:

```bash
az cosmosdb sql container create --resource-group <resource-group> --account <account> --database <database> --name <name> --partition-key-path <partition-key-path> --throughput <throughput> --analytical-storage-ttl -1
```

Or with Azure PowerShell:

```powershell
New-AzCosmosDBSqlContainer -ResourceGroupName <resource-group> -AccountName <account> -DatabaseName <database> -Name <name> -PartitionKeyPath <partition-key-path> -Throughput <throughput> -AnalyticalStorageTtl -1
```

>**Tip**: You can also use the various developers SDKs to enable or disable either analytical storage on a per-container level or Synapse Link at the account level.

#### Read from Azure Cosmos DB

>**Note**: The next couple of Python examples should be performed within your Azure Synapse Analytics workspace.

There are two options to query data from Azure Cosmos DB for NoSQL. First, you can choose to load to a Spark DataFrame where the metadata is cached. This example uses Python to load a Spark DataFrame that points to an Azure Cosmos DB for NoSQL account.

```python
productsDataFrame = spark.read.format("cosmos.olap")\
    .option("spark.synapse.linkedService", "cosmicworks_serv")\
    .option("spark.cosmos.container", "products")\
    .load()
```

Alternatively, you can create a Spark table that points to the Azure Cosmos DB for NoSQL directly. You can then run SparkSQL queries against the Spark table without impacting the underlying store. This example uses Python to create a Spark table.

```python
create table products_qry using cosmos.olap options (
    spark.synapse.linkedService 'cosmicworks_serv',
    spark.cosmos.container 'products'
)
```

## Write to Azure Cosmos DB

>**Note**: The next couple of Python examples should be performed within your Azure Synapse Analytics workspace.

If we want to write new data to Azure Cosmos DB from our Spark DataFrame, we can use the following Python script to append the data in a DataFrame to an existing container.

```python
productsDataFrame.write.format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "cosmicworks_serv")\
    .option("spark.cosmos.container", "products")\
    .mode('append')\
    .save()
```

>**Note**: This operation will impact our existing transaction workloads and will consume request units on the Azure Cosmos DB for NoSQL container[s].

We can even take it further and stream data from a DataFrame, starting from a checkpoint. We can also append this streaming data to an existing container using this example Python script.

```python
query = productsDataFrame\
    .writeStream\
    .format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "cosmicworks_serv")\
    .option("spark.cosmos.container", "products")\
    .option("checkpointLocation", "/tmp/runIdentifier/")\
    .outputMode("append")\
    .start()

query.awaitTermination()
```
### Migrate existing data using Azure Data Factory

In Azure Data Factory, Azure Cosmos DB is supported as a source of data ingest and as a target (sink) of data output.

In this lab, we will populate Azure Cosmos DB using a helpful command-line utility and then use Azure Data Factory to move a subset of data from one container to another.

#### Create and seed your Azure Cosmos DB for NoSQL account

You will use a command-line utility that creates a **cosmicworks** database and a **products** container at **4,000** request units per second (RU/s). Once created, you will adjust the throughput down to 400 RU/s.

To accompany the products container, you will create a **flatproducts** container manually that will be the target of the ETL transformation and load operation at the end of this lab.

1. In a new web browser window or tab, navigate to the Azure portal (`portal.azure.com`).
    
2. Sign into the portal using the Microsoft credentials associated with your subscription.
    
3. Select **+ Create a resource**, search for _Cosmos DB_, and then create a new **Azure Cosmos DB for NoSQL** account resource with the following settings, leaving all remaining settings to their default values:
    
    |**Setting**|**Value**|
    |---|---|
    |**Workload Type**|**Learning**|
    |**Subscription**|_Your existing Azure subscription_|
    |**Resource group**|_Select an existing or create a new resource group_|
    |**Account Name**|_Enter a globally unique name_|
    |**Location**|_Choose any available region_|
    |**Capacity mode**|_Provisioned throughput_|
    |**Apply Free Tier Discount**|_Do Not Apply_|
    |**Limit the total amount of throughput that can be provisioned on this account**|_Unchecked_|
    
    > 📝 Your lab environments may have restrictions preventing you from creating a new resource group. If that is the case, use the existing pre-created resource group.
    
4. Wait for the deployment task to complete before continuing with this task.
    
5. Go to the newly created **Azure Cosmos DB** account resource and navigate to the **Keys** pane.
    
6. This pane contains the connection details and credentials necessary to connect to the account from the SDK. Specifically:
    
    1. Notice the **PRIMARY CONNECTION STRING** field. You will use this **connection string** value later in this exercise.
7. Keep the browser tab open, as we will return to it later.
    
8. Start **Visual Studio Code**.
    
    > 📝 If you are not already familiar with the Visual Studio Code interface, review the [Get Started guide for Visual Studio Code](https://code.visualstudio.com/docs/getstarted/tips-and-tricks)
    
9. In **Visual Studio Code**, open the **Terminal** menu and then select **New Terminal** to open a new terminal instance.
    
10. Install the [cosmicworks](https://www.nuget.org/packages/cosmicworks) command-line tool for global use on your machine.
    
    ```powershell
     dotnet tool install --global CosmicWorks --version 2.3.1
    ```
    
    > 💡 This command may take a couple of minutes to complete. This command will output the warning message (*Tool ‘cosmicworks’ is already installed’) if you have already installed the latest version of this tool in the past.
    
11. Run cosmicworks to seed your Azure Cosmos DB account with the following command-line options:
    
    |**Option**|**Value**|
    |---|---|
    |**-c**|_The connection string value you checked earlier in this lab_|
    |**–number-of-employees**|_The cosmicworks command populates your database with both employees and products containers with 1000 and 200 items respectively, unless specified otherwise_|
    
    ```powershell
     cosmicworks -c "connection-string" --number-of-employees 0 --disable-hierarchical-partition-keys
    ```
    
    > 📝 For example, if your endpoint is: **https­://dp420.documents.azure.com:443/** and your key is: **fDR2ci9QgkdkvERTQ==**, then the command would be: `cosmicworks -c "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ==" --number-of-employees 0 --disable-hierarchical-partition-keys`
    
12. Wait for the **cosmicworks** command to finish populating the account with a database, container, and items.
    
13. Close the integrated terminal.
    
14. Switch back to the web browser, open a new tab and navigate to the Azure portal (`portal.azure.com`).
    
15. Select **Resource groups**, then select the resource group you created or viewed earlier in this lab, and then select the **Azure Cosmos DB account** resource you created in this lab.
    
16. Within the **Azure Cosmos DB** account resource, navigate to the **Data Explorer** pane.
    
17. In the **Data Explorer**, expand the **cosmicworks** database node, expand the **products** container node, and then select **Items**.
    
18. Observe and select the various JSON items in the **products** container. These are the items created by the command-line tool used in previous steps.
    
19. Select the **Scale** node. In the **Scale** tab, select **Manual**, update the **required throughput** setting from **4000 RU/s** to **400 RU/s** and then **Save** your changes**.
    
20. In the **Data Explorer** pane, select **New Container**.
    
21. In the **New Container** popup, enter the following values for each setting, and then select **OK**:
    
    |**Setting**|**Value**|
    |---|---|
    |**Database id**|_Use existing_ &vert; _cosmicworks_|
    |**Container id**|_`flatproducts`_|
    |**Partition key**|_`/category`_|
    
22. Back in the **Data Explorer** pane, expand the **cosmicworks** database node and then observe the **flatproducts** container node within the hierarchy.
    
23. Return to the **Home** of the Azure portal.
    

#### Create Azure Data Factory resource

Now that the Azure Cosmos DB for NoSQL resources are in place, you will create an Azure Data Factory resource and configure all of the necessary components and connections to perform a one-time data movement from one NoSQL API container to another to extract data, transform it, and load it to another NoSQL API container.

1. Select **+ Create a resource**, search for _Data Factory_, and then create a new **Data Factory** resource with the following settings, leaving all remaining settings to their default values:
    
    |**Setting**|**Value**|
    |---|---|
    |**Subscription**|_Your existing Azure subscription_|
    |**Resource group**|_Select an existing or create a new resource group_|
    |**Name**|_Enter a globally unique name_|
    |**Region**|_Choose any available region_|
    |**Version**|_V2_|
    
    > 📝 Your lab environments may have restrictions preventing you from creating a new resource group. If that is the case, use the existing pre-created resource group.
    
2. Wait for the deployment task to complete before continuing with this task.
    
3. Go to the newly created **Data Factory** resource and select **Launch studio**.
    
    > 💡 Alternatively, you can navigate to (`adf.azure.com/home`), select your newly created Data Factory resource, and then select the home icon.
    
4. From the home screen. Select the **Ingest** option to begin the quick wizard to perform a one-time copy data at scale operation and move to the **Properties** step of the wizard.
    
5. Starting with the **Properties** step of the wizard, in the **Task type** section, select **Built-in copy task**.
    
6. In the **Task cadence or task schedule** section, select **Run once now** and then select **Next** to move to the **Source** step of the wizard.
    
7. In the **Source** step of the wizard, in the **Source type** list, select **Azure Cosmos DB for NoSQL**.
    
8. In the **Connection** section, select **+ New connection**.
    
9. In the **New connection (Azure Cosmos DB for NoSQL)** popup, configure the new connection with the following values, and then select **Create**:
    
    |**Setting**|**Value**|
    |---|---|
    |**Name**|_`CosmosSqlConn`_|
    |**Connect via integration runtime**|_AutoResolveIntegrationRuntime_|
    |**Authentication method**|_Account key_ &vert; _Connection string_|
    |**Account selection method**|_From Azure subscription_|
    |**Azure subscription**|_Your existing Azure subscription_|
    |**Azure Cosmos DB account name**|_Your existing Azure Cosmos DB account name you chose earlier in this lab_|
    |**Database name**|_cosmicworks_|
    
10. Back in the **Source data store** section, within the **Source tables** section, select **Use query**.
    
11. In the **Table name** list, select **products**.
    
12. In the **Query** editor, delete the existing content and enter the following query:
    
    ```sql
     SELECT 
         p.name, 
         p.categoryName as category, 
         p.price 
     FROM 
         products p
    ```
    
13. Select **Preview data** to test the query’s validity. Select **Next** to move to the **Destination** step of the wizard.
    
14. In the **Destination** step of the wizard, in the **Destination type** list, select **Azure Cosmos DB for NoSQL**.
    
15. In the **Connection** list, select **CosmosSqlConn**.
    
16. In the **Target** list, select **flatproducts** and then select **Next** to move to the **Settings** step of the wizard.
    
17. In the **Settings** step of the wizard, in the **Task name** field, enter **`FlattenAndMoveData`**.
    
18. Leave all remaining fields to their default blank values and then select **Next** to move to the final step of the wizard.
    
19. Review the **Summary** of the steps you have selected in the wizard and then select **Next**.
    
20. Observe the various steps in the deployment. When the deployment has finished, select **Finish**.
    
21. Return to the browser tab that has your **Azure Cosmos DB account** and navigate to the **Data Explorer** pane.
    
22. In the **Data Explorer**, expand the **cosmicworks** database node, select the **flatproducts** container node, and then select **New SQL Query**.
    
23. Delete the contents of the editor area.
    
24. Create a new SQL query that will return all documents where the **name** is equivalent to **HL Headset**:
    
    ```sql
     SELECT 
         p.name, 
         p.category, 
         p.price 
     FROM
         flatproducts p
     WHERE
         p.name = 'HL Headset'
    ```
    
25. Select **Execute Query**.
    
26. Observe the results of the query.
    
27. Close your web browser window or tab.
