# Get started with Azure Cosmos DB for NoSQL
## Introduction to Azure Cosmos DB for NoSQL
### Introduction


Today's apps deliver innovation in all facets of life. For a business to remain competitive, companies must build apps and products that work with real-time data, are resilient, flexible and can support the next generation of AI capabilities.

Modern apps thrive on real-time data from different sources, shaped in different forms. An apps' usefulness is often in its ability to move and use data.

Developers require flexibility in their platforms so they can be responsive to business changes. Developers also require their entire application ecosystem to flexibly handle changes in the velocity, volume, or shape of their data. This flexibility enables developers to develop new features more rapidly than they ever have before.

#### Scenario

Suppose you work as the lead developer at a retail company. Your team is building your online storefront with AI Agents to assist customers in all aspects of their experience. You're designing the new storefront to be accessible across various devices including mobile. The team expects a spike in demand when the storefront is published and various "grand opening" sales begin.

As the lead developer, you have been tasked with identifying a database platform. The database platforms you consider should be able to service the data your team will generate and collect over time. The selected database should also be able to handle a large variety of data, at high volumes and velocity. Your database solution needs to scale quickly and with little friction in order to handle this demand that is both growing and variable. Your database should be able to support the vectorized data for search using AI Agents that handle customer requests.

#### Azure Cosmos DB

Azure Cosmos DB is a fast NoSQL database service for modern and AI app development at any scale.

Here, we look at how Azure Cosmos DB and its NoSQL API can be used for this type of business problem. We also learn a bit about how the database works. At the end, this module helps you decide if Azure Cosmos DB for NoSQL is a good choice for your solutions.

### What is Azure Cosmos DB for NoSQL


Let's start with a few definitions and a quick tour through Azure Cosmos DB for NoSQL. This overview should help you see whether Azure Cosmos DB might be a good fit for your work.

#### What is a NoSQL database?

Developers require new kinds of databases that can address the unique challenges of modern apps. NoSQL databases were designed to address needs such as:

- High volumes of data.
- Data with many different sources and forms.
- Dynamic data schemas that store different types of data.
- Using high-velocity and/or real-time data.

You define NoSQL databases by the common characteristics they share rather than by a specific formal definition. These characteristics include:

- A nonrelational data store.
- Being designed to scale out.
- Not enforcing a specific schema.

Generally, NoSQL databases don't enforce relational constraints or put locks on data, making writes fast. Also, they're often designed to horizontally scale via sharding or partitioning, which allows them to maintain high-performance regardless of size.

While there are many NoSQL data models, four broad data model families are commonly used when modeling data in a NoSQL database:

![Diagram showing various NoSQL models including; a key-value, document, graph, and column-family store.](https://learn.microsoft.com/en-us/training/wwl-data-ai/introduction-to-azure-cosmos-db-sql-api/media/2-nosql-db.png)

Moving forward, we focus on the data model supported by Azure Cosmos DB for NoSQL: The _document_ data model.

#### Why use a NoSQL database with the document data model?

The document data model breaks data down into individual **document** entities. A document can be any structured data type, but JSON is commonly used as the data format. The Azure Cosmos DB for NoSQL supports JSON natively.

![Illustration of a hierarchical document data model that includes parent entities, child entities, and lines connecting them.](https://learn.microsoft.com/en-us/training/wwl-data-ai/introduction-to-azure-cosmos-db-sql-api/media/2-document-db.png)

A document is an atomic entity and can have its own data form, regardless of what is stored in other documents in the same database. Because of this flexibility, there's no need for a predefined schema making it easier to build new applications rapidly. Additionally, this flexibility enables scenarios where different types of data can be stored together and where models can evolve over the lifetime of an application.

#### What is a JSON document?

JavaScript Object Notation, or [JSON](https://www.json.org/), is a lightweight data format. JSON was built to be highly compatible with the literal notation of an object in the JavaScript language. Many frameworks, browsers, and even databases support JavaScript natively making JSON a popular format for transmitting and storing data.

Here's an example of a JSON document:

```json
{
  "device": {
    "type": "mobile"
  },
  "sentTime": "2019-11-12T13:08:42",
  "spoolRefs": [
    "6a86682c-be5a-4a4a-bacd-96c4d1c7ece6",
    "79e78fe2-93aa-4688-89db-a7278b034aa6"
  ]
}
```

As you can see, JSON is a relatively readable data format that clearly exposes its content. JSON is also relatively easy to parse and use in JavaScript applications.

#### What is Azure Cosmos DB for NoSQL?

Azure Cosmos DB for NoSQL is a fast NoSQL and vector database service that offers rich querying over diverse data and supports a new generation of Generative AI applications. It helps deliver configurable and reliable performance, is globally distributed, and enables rapid development.

![Illustration of a world map with four globally distributed nodes that are connected via lines.](https://learn.microsoft.com/en-us/training/wwl-data-ai/introduction-to-azure-cosmos-db-sql-api/media/2-azure-cosmos-db.png)

The NoSQL API is the core or native API for working with documents. The NoSQL API supports fast, flexible development utilizing JSON documents, a query language with a familiar syntax, and client libraries for popular programming languages. Azure Cosmos DB also provides unique capabilities such as vector indexing and search, allowing users to create a new breed of Generative AI applications over users' data that can rapidly scale efficiently.

Azure Cosmos DB for NoSQL has a few advantages such as:

- **Industry Leading Vector Database** with vector indexing and search designed to handle high-dimensional vectors, enabling efficient and accurate vector search at any scale.
- **Guaranteed speed at any scale** even through bursts—with instant, limitless elasticity, fast reads, and multi-master writes, anywhere in the world.
- **Fast, flexible app development** with SDKs for popular languages and frameworks such as .NET, Java, Python, JavaScript and GO, as well as no-ETL (extract, transform, load) analytics.
- **Ready for mission-critical applications** with guaranteed business continuity, 99.999-percent availability, and enterprise-grade security.
- **Fully managed and cost-effective** with a fully featured serverless offering as well as instant, automatic and dynamic scaling that responds to application needs.

These capabilities make Azure Cosmos DB ideally suited for modern application development. Azure Cosmos DB for NoSQL is especially suited for applications that:

- Experience unpredictable spikes and dips in traffic
- Generate lots of data
- Need to deliver real-time user experiences
- Are depended upon for business continuity

The Azure Cosmos DB for NoSQL can store native JSON documents with flexible schema. Data is indexed automatically and is available for query using a flavor of the SQL query language designed for JSON data. The NoSQL API can be accessed using SDKs for popular frameworks such as [.NET](https://learn.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-sdk-dotnet-standard), [Python](https://learn.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-sdk-python), [Java](https://learn.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-sdk-java-v4), [Node.js](https://learn.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-sdk-node) and [GO](https://learn.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-sdk-go)

### How does Azure Cosmos DB for NoSQL work

Now that we know the basics of Azure Cosmos DB, let's see what resources and information are required to start working with an account. This information should help you decide whether Azure Cosmos DB for NoSQL works for your data set. Also, it should help you decide how much, if any, extra configuration is necessary.

#### What are the components of Azure Cosmos DB for NoSQL?

To begin using Azure Cosmos DB, you first create various resources in Azure such as accounts, databases, containers, and items.

![Diagram showing how an Azure Cosmos DB for NoSQL account is the parent resource to a database, which is itself a parent resource to a container.](https://learn.microsoft.com/en-us/training/wwl-data-ai/introduction-to-azure-cosmos-db-sql-api/media/3-resource-hierarchy.png)

##### Accounts

**Accounts** are the fundamental units of high availability and tenant isolation for SaaS applications. At the account level, you can configure the region[s] for your data in Azure Cosmos DB for NoSQL. Accounts also contain the globally unique DNS name used for API requests. You can also set the default consistency level for requests at the account level. You can manage or create accounts using the Azure portal, Azure Resource Manager templates, the Azure CLI, or Azure PowerShell.

##### Databases

Each account can contain one or more **Databases**. A database is a logical unit of management for containers in Azure Cosmos DB for NoSQL.

##### Containers

**Containers** are the fundamental unit of scalability in Azure Cosmos DB for NoSQL. With Azure Cosmos DB, you provision throughput at the container level. You can also optionally configure an indexing policy or a default time-to-live value at the container level. Azure Cosmos DB for NoSQL will automatically and transparently partition the data in a container.

##### Items

The NoSQL API for Azure Cosmos DB stores individual documents in JSON format as _items_ within the container. Azure Cosmos DB for NoSQL natively supports JSON files and can provide fast and predictable performance because write operations on JSON documents are atomic.

![Diagram showing various items stored in a container.](https://learn.microsoft.com/en-us/training/wwl-data-ai/introduction-to-azure-cosmos-db-sql-api/media/3-item-hierarchy.png)

#### Partitioning & Partition Keys

Every Azure Cosmos DB for NoSQL container is required to specify a **partition key path** that is used to distribute data for scale out. Behind the scenes, Azure Cosmos DB for NoSQL uses this path to logically partition data using **partition key values**. For example, consider the following JSON document:

```json
{
  "id": "35b5bf7d-5f0e-4209-b7cb-8c5c70c3bb59",
  "deviceDisplayName": "shared-printer",
  "acquiredYear": 2019,
  "department": {
    "name": "information-technology",
    "metadata": {
      "location": "floor-5-unit-27"
    }
  },
  "queuedDocuments": [
    {
      "sender": "user-293749329",
      "sentTime": "2019-07-26T05:12:37",
      "pages": 5,
      "spoolRef": "3f4b759c-3230-4269-a88e-de7620ad91c0"
    },
    {
      "device": {
        "type": "mobile"
      },
      "sentTime": "2019-11-12T13:08:42",
      "spoolRefs": [
        "6a86682c-be5a-4a4a-bacd-96c4d1c7ece6",
        "79e78fe2-93aa-4688-89db-a7278b034aa6"
      ]
    }
  ]
}
```

If your container specifies a partition key **path** of `/department/name`, then the partition key **value** of this document would be `information-technology`. Behind the scenes, Azure Cosmos DB for NoSQL automatically manages the physical resources necessary to support your data workload.

Selecting a partition key path for a container is critical to allow applications to scale and is one of the most important design decisions for a new workload. Review the [choosing a partition key](https://learn.microsoft.com/en-us/azure/cosmos-db/partitioning-overview#choose-partitionkey) documentation for a deeper technical explanation and best practices.

### When should you use Azure Cosmos DB for NoSQL

Azure Cosmos DB for NoSQL is a fully managed NoSQL database service for modern and AI app development. It provides guaranteed single-digit millisecond response times, 99.999-percent availability and [vector database capabilities](https://learn.microsoft.com/en-us/azure/cosmos-db/vector-database), backed by SLAs with automatic and instant scalability.

For enterprise scenarios, Azure Cosmos DB for NoSQL has a comprehensive suite of financially backed [service level agreements (SLAs)](https://azure.microsoft.com/support/legal/sla/cosmos-db/) that cover throughput, consistency, availability, and latency.

#### Common use cases for the Azure Cosmos DB for NoSQL

As a fast NoSQL database with a flexible API and vector indexing and search capabilities, Azure Cosmos DB for NoSQL is well suited for many types and sizes of applications. From the very small scale, to high-performance applications with global ambition. Speed and flexibility make Azure Cosmos DB for NoSQL great for Generative AI, web, retail, IoT, gaming, and mobile applications. Azure Cosmos DB for NoSQL is a good fit for applications that require flexibility, low cost, fast response times, and the ability to scale to massive volume or velocity.

##### Generative AI

Generative AI applications can be diverse and unpredictable. These workloads require a database platform that is cost-efficient, responsive and scalable. Users can store vectors directly in their documents with traditional schema-free data and high-dimensional vectors as other properties. This colocation of data and vectors allows for efficient indexing and searching, as the vectors are stored in the same logical unit as the data they represent. Keeping vectors and data together simplifies data management, AI application architectures, and the efficiency of vector-based operations.

![Architectural diagram for a Generative AI workload showing a multi-tenant AI Agent application for end users. New or updated data is ingested with Change Feed, vectorized and stored. Users interact through the AI Agents which perform vector search and generate responses with the chat history stored in Azure Cosmos DB.](https://learn.microsoft.com/en-us/training/wwl-data-ai/introduction-to-azure-cosmos-db-sql-api/media/4-ai-case.png)

In this example, customers are taking transactional and operational data and vectorizing it to be used for vector search by multiple AI Agents serving customers. Azure Cosmos DB's Change Feed is used to handle ingestion and vectorization of new or updated data, making it available in near real-time for users. Customers interacting with these agents generate prompts and completions which are also stored as their chat history in Azure Comsos DB and used to provide a semantic cache for improved cost and performance.

##### Retail/marketing

Azure Cosmos DB for NoSQL is a great fit for retail and marketing workloads that can experience dramatic and unexpected swings in usage at any point throughout the year. The elastic scale of Azure Cosmos DB for NoSQL ensures that the database platform can handle requests during peak usage, and save money during nonpeak times.

![Architectural diagram for a retail workload showing a user browser connecting to the website on Azure App Service supported by an Azure Blob Storage account containing static site data. Behind the scenes, an Azure Cosmos DB for NoSQL account with a container for inventory data and a container for shopping cart data is used by the App Service Web App and an Azure Search instance that builds a searchable catalog by indexing the Azure Cosmos DB for NoSQL account with inventory data.](https://learn.microsoft.com/en-us/training/wwl-data-ai/introduction-to-azure-cosmos-db-sql-api/media/4-retail-case.png)

In this example, a JavaScript web application, built on content stored in Azure Blob Storage, uses Azure Cosmos DB for NoSQL as it's backing database. Multiple accounts are used to manage different facets of the solution such as the shopping cart, inventory, or catalog. The solution then uses Azure Search to index the Azure Cosmos DB for NoSQL data, providing a rich search experience to end users.

##### Web/mobile

Many modern social applications generate a plethora of user-generated content that is diverse in quantity, shape, and volume. Azure Cosmos DB for NoSQL is a great candidate for this workload as this API can store data of varying schemas. Consider the NoSQL API for data that may have schemas that change or evolve over time as the company's initiatives expand into new areas.

![Architectural diagram for a web workload showing a user browser connecting to a URL that is connected to  Azure Traffic Manager to determine the correct redirect destination. Then three Azure App Service instances in three Azure regions (North Europe, West US, East US) are connected to a globally distributed Azure Cosmos DB for NoSQL account.](https://learn.microsoft.com/en-us/training/wwl-data-ai/introduction-to-azure-cosmos-db-sql-api/media/4-web-case.png)

In this example, a user is using a URL to access a web site in their browser. The URL points to Azure Traffic Manager, which then uses a built-in algorithm to determine which Azure App Service endpoint to redirect the user to. Since Azure Cosmos DB for NoSQL is capable of global distribution, you only need one account that is replicated across multiple regions.

#### Module Scenario

Consider the scenario from the beginning of this module:

> Suppose you work as the lead developer at a retail company. Your team is building your online storefront with support for AI Agents to provide a rich experience for users. You're designing the new storefront to be accessible across various devices including mobile. The team expects a spike in demand when the storefront is published and various "grand opening" sales begin.

One key part of your store's success is the ability for the company to notify users of shipping updates regardless of what device they place the order on or are currently using. Your team has worked hard on a sophisticated system to manage detailed order status tracking. The tight integration of Azure Cosmos DB with other Azure services, let's you consider building solutions that use order data in Azure Cosmos DB for NoSQL to send notification to your user's mobile devices. The notifications alert them when their package ships, or is out for delivery.

![Architectural diagram for a retail workload showing a growing number of users ordering products and a collection of compute resources handling requests from the storefront instances. Behind the compute resources, Azure Cosmos DB stores purchase data. Then, Azure Synapse Link connects Azure Cosmos DB to Azure Synapse Analytics for deeper analytics. Finally, Azure Functions, triggered off of change feed, processing data events that then trigger an Azure Logic Apps workflow to perform business operations such as notifying the user on their mobile device of new events.](https://learn.microsoft.com/en-us/training/wwl-data-ai/introduction-to-azure-cosmos-db-sql-api/media/4-retail-scenario.png)

This example is similar to the example from the introduction of this module. To build on the first example, your team has decided to introduce Azure Cosmos DB for NoSQL as the database of choice. Now, your team can use Azure Synapse Link to prepare and aggregate data for deeper analysis using Azure Synapse Analytics. Your team can also use services such as Azure Functions to react to data events with Azure Cosmos DB, and then trigger an Azure Logic Apps workflow that sends notifications to mobile devices.

## Try Azure Cosmos DB for NoSQL
### Introduction

The first step to getting started with Azure Cosmos DB is to create a new account. You will learn, here, the basic hierarchy of resources in an Azure Cosmos DB for NoSQL account and how to create an account along with those resources.

### Explore resources

An Azure Cosmos DB for NoSQL account is composed of a basic hierarchy of resources that include:

- An account
- One or more databases
- One or more containers
- Many items

![Diagram explaining the hierarchy of Azure Cosmos DB resources including an account, then a child set of databases, child set of containers, and then finally a child set of items.](https://learn.microsoft.com/en-us/training/wwl-data-ai/try-azure-cosmos-db-sql-api/media/2-hiearchy.png)

Let's explore each item in this hierarchy.

#### Account

Each tenant of the Azure Cosmos DB service is created by provisioning a database account. Accounts are the fundamental units of data distribution, high availability and security. At the account level, you can configure the region[s] for your data in Azure Cosmos DB for NoSQL. Accounts also contain the globally unique DNS name used for API requests

![Diagram explaining the resource hierarchy with account highlighted and associated with a DNS name and key.](https://learn.microsoft.com/en-us/training/wwl-data-ai/try-azure-cosmos-db-sql-api/media/2-account.png)

#### Database

A database is a logical unit of management for containers in Azure Cosmos DB for NoSQL. Within the database, you can find one or more containers.

![Diagram explaining the resource hierarchy with database highlighted and multiple example child containers.](https://learn.microsoft.com/en-us/training/wwl-data-ai/try-azure-cosmos-db-sql-api/media/2-database-diag.png)

#### Container

Containers are the fundamental unit of scalability in Azure Cosmos DB for NoSQL. Typically, you provision throughput at the container level but can use Serverless as well. Azure Cosmos DB for NoSQL will automatically and transparently partition the data in a container using the document property you select as a partition key for the container. You can also optionally configure indexing policies or a default time-to-live value at the container level.

![Diagram explaining the resource hierarchy with a set of containers highlighted.](https://learn.microsoft.com/en-us/training/wwl-data-ai/try-azure-cosmos-db-sql-api/media/2-container-diag.png)

#### Item[s]

An Azure Cosmos DB for NoSQL resource container is a schema-agnostic container of arbitrary user-generated JSON items. The NoSQL API for Azure Cosmos DB stores individual documents in JSON format as items within the container. Azure Cosmos DB for NoSQL natively supports JSON files and can provide fast and predictable performance because write operations on JSON documents are atomic.

>**Tip**: Containers can also store JavaScript based stored procedures, triggers and user-defined-functions (UDFs)

![Diagram explaining the resource hierarchy with items highlighted and other example children resources of containers.](https://learn.microsoft.com/en-us/training/wwl-data-ai/try-azure-cosmos-db-sql-api/media/2-item.png)
### Review basic operations

There are a few basic operations that you will need to perform anytime you create any Azure Cosmos DB for NoSQL account resource in Azure.

#### Creating a new account

The first step to getting started with Azure Cosmos DB is to create a new account.

When creating a new account in the portal, you must first select an API for your workload. The API selection cannot be changed after the account is created. For the remainder of this section, we will assume that the NoSQL API has been selected.

[![Screenshot showing the select API option in the portal with a list of all current APIs including SQL, MongoDB, Graph, Table, and Cassandra.](https://learn.microsoft.com/en-us/training/wwl-data-ai/try-azure-cosmos-db-sql-api/media/3-select-api.png)](https://learn.microsoft.com/en-us/training/wwl-data-ai/try-azure-cosmos-db-sql-api/media/3-select-api.png#lightbox)

Next, the Azure portal will use a step-by-step wizard with tabs for various configuration options. Here you can configure options such as:

- The globally unique name of your account
- The location (Azure region) for the account
- Capacity mode (provisioned throughput or serverless)

[![Screenshot showing the wizard with various tabs and options for creating a new Azure Cosmos DB for NoSQL account.](https://learn.microsoft.com/en-us/training/wwl-data-ai/try-azure-cosmos-db-sql-api/media/3-account-wizard.png)](https://learn.microsoft.com/en-us/training/wwl-data-ai/try-azure-cosmos-db-sql-api/media/3-account-wizard.png#lightbox)

>**Note**: Only the options in the **Basics** tab are required to create an Azure Cosmos DB account.

#### Creating a new database

Databases are logical units of management in Azure Cosmos DB for NoSQL, and don't require much to create. You only need a unique database name within the account to create a new database.

>**Note**: However, if you choose to provision throughput at the database level, configuring the database may require additional steps. This is explored deeper in other Azure Cosmos DB for NoSQL topics.

#### Creating a new container

Containers are the primary unit of scalability in Azure Cosmos DB for NoSQL. When creating a container, you should specify:

- The parent database
- A unique name for the container with the database
- The path for the partition key value
- _Optional_: provisioned throughput if not using a Serverless account.

The Azure Cosmos DB service will automatically and transparently partition your data based on the value of the partition key for each individual item.

#### Creating simple items

Once the database and container resources exist, you are ready to create your first item. In Azure Cosmos DB for NoSQL, an item is a JSON document.

>**Note**: JavaScript Object Notation (JSON) is an open standard file format, and data interchange format, that uses human-readable text to store and transmit data objects consisting of attribute–value pairs and array data types (or any other serializable value)

JSON is a language-independent data format with well-defined data types and near universal support across a diverse range of services and programing languages. Here is an example of a JSON document that could be an item in an Azure Cosmos DB account:

```json
{
  "id": "0012D555-C7DE",
  "type": "customer",
  "fullName": "Franklin Ye",
  "title": null,
  "emailAddress": "fye@cosmic.works",
  "creationDate": "2014-02-05",
  "addresses": [
    {
      "addressLine": "1796 Westbury Drive",
      "cityStateZip": "Melton, VIC 3337 AU"
    },
    {
      "addressLine": "9505 Hargate Court",
      "cityStateZip": "Bellflower, CA 90706 US"
    }
  ],
  "password": {
    "hash": "GQF7qjEgMk=",
    "salt": "12C0F5A5"
  },
  "salesOrderCount": 2
}
```

### Create an Azure Cosmos DB for NoSQL account

Before diving too deeply into Azure Cosmos DB, it’s important to get a handle on the basics of creating the resources you will use the most. In most scenarios, you will need to be comfortable creating accounts, databases, containers, and items. In a real-world scenario, you should also have a few basic queries “on hand” to test that you created all of your resources correctly.

In this lab, you’ll create a new Azure Cosmos DB account using the for NoSQL. You will then use the Data Explorer to create a database, a container, and two items. Finally, you will query the database for the items you created.

#### Create a new Azure Cosmos DB account

Azure Cosmos DB is a cloud-based NoSQL database service that supports multiple APIs. When provisioning an Azure Cosmos DB account for the first time, you will select which of the APIs you want the account to support (for example, **Mongo API** or **NoSQL API**).

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
    |**Capacity mode**|**Provisioned throughput**|
    |**Apply Free Tier Discount**|**Do Not Apply**|
    
7. Select **Review + Create** to navigate to the **Review + Create** tab, and then select **Create**.
    
    > 📝 It can take 10-15 minutes for the Azure Cosmos DB for NoSQL account to be ready for use.
    
8. Observe the **Deployment** pane. When the deployment is complete, the pane will update with a **Deployment successful** message.
    
9. Still within the **Deployment** pane, select **Go to resource**.
    

#### Use the Data Explorer to create a new database and container

The Data Explorer will be your primary tool to manage the Azure Cosmos DB for NoSQL database and containers in the Azure portal. You will create a basic database and container to use in this lab.

1. From within the **Azure Cosmos DB account** pane, select **Data Explorer** from the resource menu.
    
2. In the **Data Explorer** pane, select **New Container**.
    
3. In the **New Container** popup, enter the following values for each setting, and then select **OK**:
    
    |**Setting**|**Value**|
    |---|---|
    |**Database id**|_cosmicworks_|
    |**Share throughput across containers**|_Unckecked_|
    |**Container id**|_products_|
    |**Partition key**|_/categoryId_|
    |**Container throughput (autoscale)**|_Manual_|
    |**RU/s**|_400_|
    
4. Back in the **Data Explorer** pane, expand the **cosmicworks** database node and then observe the **products** container node within the hierarchy.
    

#### Use the Data Explorer to create new items

The Data Explorer also includes a suite of features to query, create, and manage items in an Azure Cosmos DB for NoSQL container. You will create two basic items using raw JSON in the Data Explorer.

1. In the **Data Explorer** pane, expand the **cosmicworks** database node, expand the **products** container node, and then select **Items**.
    
2. Select **New Item** from the command bar, and in the editor, replace the placeholder JSON item with the following content:
    
    
    ```json
     {
       "categoryId": "4F34E180-384D-42FC-AC10-FEC30227577F",
       "categoryName": "Components, Pedals",
       "sku": "PD-R563",
       "name": "ML Road Pedal",
       "price": 62.09
     }
    ```
    
3. Select **Save** from the command bar to add the first JSON item.
    
4. Back in the **Items** tab, select **New Item** from the command bar. In the editor, replace the placeholder JSON item with the following content:
    
    
    ```json
     {
       "categoryId": "75BF1ACB-168D-469C-9AA3-1FD26BB4EA4C",
       "categoryName": "Bikes, Touring Bikes",
       "sku": "BK-T18Y-44",
       "name": "Touring-3000 Yellow, 44",
       "price": 742.35
     }
    ```
    
5. Select **Save** from the command bar to add the second JSON item.
    
6. In the **Items** tab, observe the two new items in the **Items** pane.
    

## Use the Data Explorer to issue a basic query

Finally, the Data Explorer has a built-in query editor that is used to issue queries, observe the results, and measure impact in terms of request units per second (RU/s).

1. In the **Data Explorer** pane, select **New SQL Query**.
    
2. In the query tab, select **Execute Query** to view a standard query that selects all items without any filters.
    
3. Delete the contents of the editor area.
    
4. Replace the placeholder query with the following content:
    
    
    ```sql
     SELECT * FROM products p WHERE p.price > 500
    ```
    
    > 📝 This query will select all items where the **price** is greater than $500.
    
5. Select **Execute Query**.
    
6. Observe the results of the query, which should include a single JSON item and all of its properties.
    
7. In the **Query** tab, select **Query Stats**.
    
8. Still in the **Query** tab, observe the value of the **Request Charge** field within the **Query Statistics** section.
    
    > 📝 Typically, the request charge for this simple query is between 2 and 3 RU/s when the container size is small.
    
9. Close your web browser window or tab.