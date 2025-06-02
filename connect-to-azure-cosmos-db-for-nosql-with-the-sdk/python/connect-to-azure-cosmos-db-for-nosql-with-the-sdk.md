# Connect to Azure Cosmos DB for NoSQL with the SDK
## Use the Azure Cosmos DB for NoSQL SDK
### Introduction

There are various SDKs (Software Development Kits) available to connect to the Azure Cosmos DB for NoSQL from many popular programming languages including, but not limited to:

- .NET (C#)
- Java
- Python
- JavaScript (Node.js)
- GO
### Understand the SDK

Microsoft offers SDKs (Software Development Kits) for Azure Cosmos DB for NoSQL in various programming languages. In this module, we focus on the .NET SDK (C#), Python SDK, and JavaScript SDK (Node.js) for Azure Cosmos DB for NoSQL. Select the tab that corresponds to the SDK you want to explore.

The **azure-cosmos** library is the latest version of the Python SDK for Azure Cosmos DB for NoSQL.

The library is open-source and hosted online on GitHub at **Azure/azure-sdk-for-python**. The open-source project conforms to the Microsoft Open Source Code of Conduct and accepts contributions and suggestions from the community.

The Azure-cosmos library includes a namespace of the same name with common classes that you explore later in this module including, but not limited to:

|**Class**|**Description**|
|---|---|
|azure.cosmos.**CosmosClient**|Client-side logical representation of an Azure Cosmos DB account and the primary class used for the SDK|
|azure.cosmos.**DatabaseProxy**|Interface that logically represents a database client-side and includes common operations for database management|
|azure.cosmos.**ContainerProxy**|Interface that logically represents a container client-side and includes common operations for container management|

>**Note**: You shouldn't directly instantiate the **DatabaseProxy** and **ContainerProxy** classes. Instead, you should use the **CosmosClient** class to interact with databases and containers.

### Import from package manager


The Azure Cosmos DB SDKs (Software Development Kits) are hosted on **NuGet**, **PyPI**, and **npm** for .NET, Python, and JavaScript respectively. To import the SDK into your project, you must use the package manager for the respective programming language. Select the tab that corresponds to the SDK you want to explore.

The **azure-cosmos** library, including all of its previous versions, are hosted on **PyPI** to make it easier to import the library into a Python application.

#### Installing the Python package

To install the `azure-cosmos` package, you can use `pip`, the standard package installer for Python. This installation can be done in one of two ways:

##### Install the latest version of the package

Invoke the `pip install` command with only the name of the package. This command installs the latest stable version of the **azure-cosmos** library.

```bash
pip install azure-cosmos
```

>**Tip**: This command only imports stable versions of the package. If a newer preview of the package is available, it imports the older stable version. If no stable version is available, it doesn't import the package at all.

##### Install a specific version of the package

Invoke the pip install command with the name of the package and specify the version using `==`. For example, this command installs version **4.8.0** of the **azure-cosmos** library.

```bash
pip install azure-cosmos==4.8.0
```

>**Tip**: Specifying the version is necessary if you need to install a preview version or a specific version that matches your project's requirements.

##### Python project dependencies file

When using a virtual environment or managing dependencies, it's common to list packages in a **requirements.txt** file. This file should include the name of the package and the version, if necessary. Here's an example with the **4.8.0** version of the **azure-cosmos** library.

```bash
azure-cosmos==4.8.0
```

>**Note**: If you run `pip freeze` after installing packages, it will create a **requirements.txt** file with the currently installed versions, which you can use to replicate the environment later.

### Connect to an online account

Once the client library is imported, you can begin using the namespaces and classes within your project.

#### Import the library

Before using the library, you need to import the **CosmosClient** and other necessary classes from the **azure.cosmos** module. Importing these classes allows you to access types without needing to fully qualify each type.

```python
from azure.cosmos import CosmosClient, PartitionKey, ThroughputProperties, ContainerProxy, exceptions
```

#### Use the CosmosClient class

The three most common ways to create an instance for the **CosmosClient** class is to instantiate it with one of the following three constructors:

- A constructor that takes a single string value representing the connection string for the account.
- A constructor that takes two string values representing the endpoint and a key for the account.
- A constructor that takes a string value representing the endpoint and a token credential that enables Microsoft Entra ID authentication.

>**Note**: You can always retrieve the connection string, endpoint, or any of the keys from the Azure portal. For the examples in this section, we use a fictional endpoint of **https­://dp420.documents.azure.com:443/** and a sample key of **fDR2ci9QgkdkvERTQ==**.

>**Tip**: Using the CosmosClient class with the Microsoft Identity Platform directly for Microsoft Entra ID authentication is considered a best practice. Read the [security guidance for Azure Cosmos DB for NoSQL](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/security/) for more information.

##### Use with a connection string

The **CosmosClient** class can be instantiated by passing the connection string. This example uses a connection string in the format `AccountEndpoint=<account-endpoint>;AccountKey=<account-key>`.

```python
connection_string = "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ=="

client = CosmosClient.from_connection_string(connection_string)
```

##### Use with an endpoint and key

Alternatively, you can create an instance of the **CosmosClient** class by providing the endpoint and the key as two separate parameters.

```python
endpoint = "https://dp420.documents.azure.com:443/"
key = "fDR2ci9QgkdkvERTQ=="

client = CosmosClient(endpoint, key)
```

##### Use with an endpoint and token credential

You can also use a constructor of the **CosmosClient** class that takes in an **endpoint** and a **token credential**. This constructor is used when you want to authenticate using Microsoft Entra ID. This example uses the fictional endpoint and a token credential.

```python
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
endpoint = "https://dp420.documents.azure.com:443/"

client = CosmosClient(url=endpoint, credential=credential)
```

#### Read properties of the account

>**Tip**: At this point, the client instance is just a logical representation of the Azure Cosmos DB account. No connection is made until you perform an operation.

Once the client instance is created, you can use it to perform various operations. For example, you can use the **client.get_database_account()** method to access account properties.

```python
account_info = client.get_database_account()
```

The `account_info` object represents an instance of the `DatabaseAccount`, which includes useful properties, such as:

|**Property**|**Description**|
|---|---|
|**EnableMultipleWritableLocations**|Flag on the Azure Cosmos account that indicates if writes can take place in multiple locations|
|**ReadableLocations**|A list of readable locations for the account|
|**WritableLocations**|A list of writable locations for the account|
|**ConsistencyPolicy**|The default consistency level for the account|

#### Interact with a database

Once you have a client instance, you can retrieve or create a database using one of three methods:

- Retrieve an existing database using its name.
- Create a new database by passing a unique database name.
- Check for the existence of the database and either create or retrieve it automatically.

Any of these methods return a **DatabaseProxy** instance that you can use to interact with the database.

##### Retrieve an existing database

```python
database = client.get_database_client("cosmicworks")
```

##### Create a new database

```python
database = client.create_database("cosmicworks")
```

##### Create a database if it doesn't already exist

```python
database = client.create_database_if_not_exists("cosmicworks")
```

#### Interact with a container

With a database instance, you can retrieve or create a container using one of the following methods:

- Retrieve an existing container by name.
- Create a new container by specifying a unique container name, partition key path, and manually provisioned throughput.
- Check for the existence of the container and either create or retrieve it automatically.

Each method returns a **ContainerProxy** instance that you can use to interact with the container.

##### Retrieve an existing container

```python
container = database.get_container_client("products")
```

##### Create a new container

```python
container = database.create_container(
    id="products",
    partition_key=PartitionKey(path="/categoryId"),
    throughput=ThroughputProperties(auto_scale_max_throughput=1000)
)
```

##### Create a container if it doesn't already exist

```python
container = database.create_container_if_not_exists(
    id="products",
    partition_key=PartitionKey(path="/categoryId"),
    offer_throughput=ThroughputProperties(auto_scale_max_throughput=1000)
)
```
### Implement client singleton

The `CosmosClient` in the Python SDK is lightweight, and the Azure Cosmos DB SDK for Python automatically handles connection management efficiently. However, for performance and resource efficiency, the recommendation is to reuse the same `CosmosClient` instance throughout your application.

- The Python SDK is thread-safe, so a single instance can be shared across multiple threads.
- Creating and disposing of multiple instances unnecessarily increases resource consumption.

#### Best Practices

- Create a single `CosmosClient` instance and reuse it throughout the lifetime of your application.
- In long-running applications, store the `CosmosClient` instance in a globally accessible location, such as a module-level variable or a dependency injection container.

>**Tip**: If you're using a framework like FastAPI or Flask, consider creating the `CosmosClient` once during app startup and reusing it across requests.

### Configure connectivity mode

The **CosmosClient** class in the Python SDK provides several options that can be configured to customize the behavior of the client when connecting to an account. These options include, but aren't limited to:

- Custom consistency level used specifically for the client instance
- The preferred account region

#### Overriding default client options

When a client connects to an Azure Cosmos DB account using the **CosmosClient** class, the following default assumptions are made:

- The client connects to the first writable (primary) region of your account.
- The client uses the default consistency level for read requests.
- The client uses the **Gateway** connection mode (Direct mode isn't available in the Python SDK).

>**Note**: Other assumptions exist and can be configured as needed. This section covers the most common configurations.

To customize these options, pass a configuration dictionary or keyword arguments to the **CosmosClient** constructor. The following examples show how to configure the client with different options.

##### Changing the current consistency level

Every Azure Cosmos DB for NoSQL account has a default consistency level configured. The Python SDK allows you to configure a different consistency level for all read requests made with the client. This example configures a client to use **eventual** consistency.

```python
from azure.cosmos import CosmosClient, ConsistencyLevel

client = CosmosClient(
    url="https://dp420.documents.azure.com:443/",
    credential="fDR2ci9QgkdkvERTQ==",
    consistency_level=ConsistencyLevel.Eventual
)
```

The `ConsistencyLevel` class includes several options, such as:

- Bounded Staleness
- Consistent Prefix
- Eventual
- Session
- Strong

>**Tip**: The **consistency_level** setting can only be used to _weaken_ the consistency level for reads. It can't be strengthened or applied to writes.

##### Setting the preferred application region

By default, the client connects to the first writable region for operations. You can specify a preferred region by using the **preferred_locations** parameter. This example sets the preferred region to **West US**.

```python
client = CosmosClient(
    url="https://dp420.documents.azure.com:443/",
    credential="fDR2ci9QgkdkvERTQ==",
    preferred_locations=["West US"]
)
```

>**Tip**: If your account isn't configured for multi-region writes, the client always uses the single writable region for write operations, and this setting only impacts read operations.

You can also set multiple preferred regions for a custom failover/priority list. This example configures the client to try **West US** first and then **East US**.

```python
client = CosmosClient(
    url="https://dp420.documents.azure.com:443/",
    credential="fDR2ci9QgkdkvERTQ==",
    preferred_locations=["West US", "East US"]
)
```

#### Other configuration options

While the Python SDK doesn't support setting the connection mode to **Direct**, it includes other configurable options for **CosmosClient**:

- **Retry policies**: Configure retry policies to handle transient failures.
- **Timeouts**: Set custom time-outs for client operations.
- **User agent**: Customize the user agent string for telemetry and tracking.

Example showing custom time out configuration:

```python
client = CosmosClient(
    url="https://dp420.documents.azure.com:443/",
    credential="fDR2ci9QgkdkvERTQ==",
    connection_timeout=10  # Sets a custom connection timeout of 10 seconds
)
```
### Connect to Azure Cosmos DB for NoSQL with the SDK

The Azure SDK for Python is a suite of client libraries that provides a consistent developer interface to interact with many Azure services. Client libraries are packages that you would use to consume these resources and interact with them.

#### Prepare your development environment

If you have not already cloned the lab code repository for **Build copilots with Azure Cosmos DB** and set up your local environment, view the [Setup local lab environment](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/00-setup-lab-environment.html) instructions to do so.

#### Create an Azure Cosmos DB for NoSQL account

If you already created an Azure Cosmos DB for NoSQL account for the **Build copilots with Azure Cosmos DB** labs on this site, you can use it for this lab and skip ahead to the [next section](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/01-sdk-connect.html#install-the-azure-cosmos-library). Otherwise, view the [Setup Azure Cosmos DB](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/common/instructions/00-setup-cosmos-db.html) instructions to create an Azure Cosmos DB for NoSQL account that you will use throughout the lab modules and grant your user identity access to manage data in the account by assigning it to the **Cosmos DB Built-in Data Contributor** role.

#### Install the azure-cosmos library

The **azure-cosmos** library is available on **PyPI** for easy installation into your Python projects.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/01-sdk-connect** folder.
    
2. Open the context menu for the **python/01-sdk-connect** folder and then select **Open in Integrated Terminal** to open a new terminal instance.
    
    > 📝 This command will open the terminal with the starting directory already set to the **python/01-sdk-connect** folder.
    
3. Create and activate a virtual environment to manage dependencies:
    
    ```bash
    python -m venv venv
    source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
    ```
    
4. Install the [azure-cosmos](https://pypi.org/project/azure-cosmos) package using the following command:
    
    ```bash
    pip install azure-cosmos
    ```
    
5. Install the [azure-identity](https://pypi.org/project/azure-identity) library, which allows us to use Azure authentication to connect to the Azure Cosmos DB workspace, using the following command:
    
    
    ```bash
    pip install azure-identity
    ```
    
6. Close the integrated terminal.
    
#### Use the azure-cosmos library

Once the Azure Cosmos DB library from the Azure SDK for Python has been imported, you can immediately use its classes to connect to an Azure Cosmos DB for NoSQL account. The **CosmosClient** class is the core class used to make the initial connection to an Azure Cosmos DB for NoSQL account.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/01-sdk-connect** folder.
    
2. Open the blank Python file named **script.py**.
    
3. Add the following `import` statement to import the **CosmosClient** and the **DefaultAzureCredential** classes:
    
    
    ```python
    from azure.cosmos import CosmosClient
    from azure.identity import DefaultAzureCredential
    ```
    
4. Add variables named **endpoint** and **credential** and set the **endpoint** value to the **endpoint** of the Azure Cosmos DB account you created earlier. The **credential** variable should be set to a new instance of the **DefaultAzureCredential** class:
    
    
    ```python
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    ```
    
    > 📝 For example, if your endpoint is: **https://dp420.documents.azure.com:443/**, the statement would be: **endpoint = “https://dp420.documents.azure.com:443/”**.
    
5. Add a new variable named **client** and initialize it as a new instance of the **CosmosClient** class using the **endpoint** and **credential** variables:
    
    
    ```python
    client = CosmosClient(endpoint, credential=credential)
    ```
    
6. Add a function named **main** to read and print account properties:
    
    
    ```python
    def main():
        account_info = client.get_database_account()
        print(f"Consistency Policy:	{account_info.ConsistencyPolicy}")
        print(f"Primary Region: {account_info.WritableLocations[0]['name']}")
    
    if __name__ == "__main__":
        main()
    ```
    
7. Your **script.py** file should now look like this:
    
    
    ```python
    from azure.cosmos import CosmosClient
    from azure.identity import DefaultAzureCredential
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    client = CosmosClient(endpoint, credential=credential)
    
    def main():
        account_info = client.get_database_account()
        print(f"Consistency Policy:	{account_info.ConsistencyPolicy}")
        print(f"Primary Region: {account_info.WritableLocations[0]['name']}")
    
    if __name__ == "__main__":
        main()
    ```
    
8. **Save** the **script.py** file.
    

#### Test the script

Now that the Python code to connect to the Azure Cosmos DB for NoSQL account is complete, you can test the script. This script will print the default consistency level and the name of the first writable region. When you created the account, you specified a location, and you should expect to see that same location value printed as the result of this script.

1. In **Visual Studio Code**, open the context menu for the **python/01-sdk-connect** folder and then select **Open in Integrated Terminal** to open a new terminal instance.
    
2. Before running the script, you must log into Azure using the `az login` command. At the terminal window, run:

    
    ```bash
    az login
    ```
    
3. Run the script using the `python` command:
    
    ```bash
    python script.py
    ```
    
4. The script will now output the default consistency level and the first writable region. For example, if the default consistency level for the account is **Session**, and the first writable region was **East US**, the script would output:

    
    ```text
    Consistency Policy:   {'defaultConsistencyLevel': 'Session'}
    Primary Region: East US
    ```
    
5. Close the integrated terminal.
    
6. Close **Visual Studio Code**.

## Configure the Azure Cosmos DB for NoSQL SDK
### Introduction

Often, you want to configure the Azure Cosmos DB for NoSQL SDK to enable common scenarios, troubleshoot problems, improve performance, or gather deeper insight. The SDK includes a rich set of options to configure your applications to perform and be managed in a way that is useful to your team.
### Enable offline development

As you begin to use Azure Cosmos DB across multiple projects, eventually there might be a need to use and test Azure Cosmos DB in a local environment. With the option to test locally, you can validate new code quickly without creating a new instance in the cloud. The Azure Cosmos DB emulator is a great tool for common Dev+Test workflows that developers may need to implement on their local machine.

#### Azure Cosmos DB emulator

The Azure Cosmos DB emulator is a local environment that is useful to develop and test applications locally without incurring the costs or complexity of an Azure subscription.

The emulator is available to run in **Windows**, **Linux**, or as a **Docker** container image.

The emulator is available as a download from the [**Microsoft Learn** website](https://learn.microsoft.com/en-us/azure/cosmos-db/local-emulator) and supports various APIs depending on the platform. The NoSQL API is universally supported across all platforms.

>**Tip**: You may optionally install the [new Linux-based Azure Cosmos DB Emulator (in preview)](https://learn.microsoft.com/en-us/azure/cosmos-db/emulator-linux), which is available as a Docker container. It supports running on a wide variety of processors and operating systems.

The Docker container image for the emulator is published to the Microsoft Container Registry and is syndicated across various container registries such as **Docker Hub**. To obtain the Docker container image from Docker Hub, use the Docker CLI to **pull** the image from `mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator`.

```bash
docker pull mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator
```

#### Configuring the SDK to connect to the emulator

The Azure Cosmos DB emulator uses the same APIs as the cloud service, so connecting to the emulator isn't different from connecting to the cloud service. The emulator uses a single fixed account with a static authentication key that is the same across all instances

First, the emulator's endpoint is `https://localhost:<port>/` using SSL with the default port set to 8081. In Python code, you can configure this endpoint as a string variable using this example line of code.

```python
endpoint = "https://localhost:8081/"
```

The emulator's key is a static well-known authentication key. The default value for this key is `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`. In Python code, you can save this key as a variable using this example line of code.

```python
key = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="
```

>**Tip**: You can start the emulator using the **/Key** option to generate a new key instead of using the default key.

Once those variables are set, create the **CosmosClient** like you typically do for a cloud-based account.

```python
client = CosmosClient(endpoint, key)
```
### Handle connection errors

While most of your requests are fine, there are some scenarios where a request can fail for a temporary reason. In these scenarios, it's both normal and expected for you to retry your request after a reasonable amount of time.

#### Built-in retry

A transient error is an error that has an underlying cause that soon resolves itself. Applications that connect to your database should be built to expect these transient errors. The Azure Cosmos DB for NoSQL SDKs (Software Development Kits) for .NET, Python, and JavaScript have built-in logic to handle common transient failures for read and query requests. The SDK does NOT automatically retry write requests as they aren't idempotent.

>**Tip**: Try to always use the latest version of the SDK. The retry logic that is built in is constantly being improved in newer releases.

If you're writing an application that experiences a write failure, it's up to your application code to implement retry logic. This logic is considered a best practice.

As an application developer, it's important to understand the HTTP status codes where retrying your request makes sense. These codes include, but aren't limited to:

- **429**: Too many requests
- **449**: Concurrency error
- **500**: Unexpected service error
- **503**: Service unavailable

>**Tip**: If you experience 50x errors indicating issues with service availability, you can file an Azure support issue to receive technical support or report an issue.

There are HTTP error codes, such as **400 (bad request)**, **401 (not authorized)**, **403 (forbidden)**, and **404 (not found)** that indicate a failure client-side that should be fixed in application code and not retried.

### Implement threading and parallelism

There are several best practices and options you can implement in your Python application when using the Azure Cosmos DB Python SDK to ensure optimal performance for your workloads.

#### Avoid resource-related time-outs

Many request time-outs occur due to high CPU or resource utilization on client machines, rather than issues happening on the Azure Cosmos DB service side. Monitor resource usage on client machines, and scale out your application appropriately to avoid SDK errors or retries due to local resource exhaustion.

#### Use asynchronous queries

The Azure Cosmos DB Python SDK supports asynchronous operations through Python's `asyncio` framework and provides classes for asynchronous programming under the [`azure.cosmos.aio`](https://learn.microsoft.com/en-us/python/api/azure-cosmos/azure.cosmos.aio) namespace. For example, you can use the `query_items` method asynchronously:

```python
from azure.cosmos.aio import CosmosClient
import asyncio

endpoint = "<cosmos-endpoint>"
key = "<cosmos-key>"

client = CosmosClient(endpoint, key)

async def query_items_async(client):
    # Get database and container clients
    database = client.get_database_client("cosmicworks")
    container = database.get_container_client("products")

    # Define the query and parameters
    query = "SELECT * FROM c WHERE c.categoryId = @categoryId"
    parameters = [{"name": "@categoryId", "value": "bikes"}]

    # Perform the query asynchronously
    async for item in container.query_items(
        query=query,
        parameters=parameters
    ):
        print(item)

    # Close the client
    await client.close()

asyncio.run(query_items_async(client))
```

Before you use the `azure.cosmos.aio` classes for asynchronous operations, ensure that you installed the `aiohttp` library (`pip install aiohttp`).

Using `asyncio` ensures that multiple operations can be handled concurrently without blocking the execution of other code.

Avoid blocking asynchronous execution by improperly mixing synchronous and asynchronous methods, such as:

```python
iterator = client.query_items(query=query, parameters=parameters).result()  # Incorrect
```

Instead, always use `await` for asynchronous calls.

#### Use iterators for queries

The Python SDK includes built-in iterators to retrieve query results efficiently without blocking other operations. Avoid collecting all query results eagerly, as it can result in large memory usage and block other operations.

##### Inefficient example:

```python
results = list(container.query_items(
    query="SELECT * FROM c WHERE c.categoryId = 'bikes'"
))
```

##### Efficient example using an iterator:

```python
iterator = container.query_items(
    query="SELECT * FROM c WHERE c.categoryId = 'bikes'"
)

async for item in iterator:
    print(item)
```

Using iterators allows you to handle data in a memory-efficient manner and process results as they arrive.

#### Configure query options for performance

When you're issuing a query, the Python SDK allows you to configure several options via the `QueryIterable` or `query_items` methods to improve performance.

##### Max item count

All query results in Azure Cosmos DB for NoSQL are returned in "pages" of results. The `max_item_count` parameter specifies the number of items to return in each page. The service default is 100 items per page.

Example with a `max_item_count` of 500:

```python
iterator = container.query_items(
    query="SELECT * FROM c",
    max_item_count=500
)
```

>**Tip**: If you use a `max_item_count` of `-1`, ensure the total response size doesn't exceed the service limit of 4 MB.

##### Partition key usage

Whenever possible, include the partition key in your query filter. This partition key reduces the query scope to a single partition, improving performance significantly.

Example:

```python
iterator = container.query_items(
    query="SELECT * FROM c WHERE c.categoryId = @categoryId",
    parameters=[{"name": "@categoryId", "value": "bikes"}],
    partition_key="bikes"
)
```

#### Perform batch operations

The Python SDK allows batch operations on items within the same partition key. Use the `TransactionalBatch` class to perform multiple operations atomically.

Example:

```python
# Define the partition key and batch operations
partition_key = "socks"
batch = [
    ("create", ({"id": "sock7", "categoryId": partition_key, "name": "Red Racing Socks"},)),
    ("create", ({"id": "sock8", "categoryId": partition_key, "name": "White Racing Socks"},))
]

# Execute the batch
batch_response = container.execute_item_batch(batch, partition_key=partition_key)

# Print the resource body results to see the created items
for result in batch_response:
    print(result.get("resourceBody"))
```

>**Note**: Batch operations must target a single partition key and can include up to 100 operations or 4 MB in size.

>**Note**: These settings are explored in more detail in other Azure Cosmos DB for NoSQL modules on issuing queries using the SDK.

### Configure logging

Proper logging is essential for monitoring and debugging your Azure Cosmos DB application. The Python SDK provides multiple mechanisms for enabling logging, capturing diagnostics, and fine-tuning the amount of information logged.

#### Overview of Logging Features

The Azure Cosmos DB Python SDK integrates with Python's standard `logging` module. By default:

- Basic HTTP session information (for example, URLs and headers) is logged at the `INFO` level.
- Detailed request and response logging, including bodies and unredacted headers, is available at the `DEBUG` level.

##### Key Capabilities

1. **Global Logging**: Enable logging at the client level to capture diagnostics for all operations.
2. **Per-Operation Logging**: Enable detailed logging for individual operations.
3. **Enhanced Diagnostics**: Use `CosmosHttpLoggingPolicy` to capture more debugging information specific to Cosmos DB.

#### Enabling Basic Logging

The Python SDK uses the `logging` module for diagnostics. To start, you can configure basic logging to capture HTTP session details.

##### Example: Basic Logging Configuration

```python
import sys
import logging
from azure.cosmos import CosmosClient

# Create a logger for the Azure SDK
logger = logging.getLogger("azure")
logger.setLevel(logging.DEBUG)  # Set log level to DEBUG for detailed output

# Configure console output
handler = logging.StreamHandler(stream=sys.stdout)
logger.addHandler(handler)

# Initialize the CosmosClient with global logging enabled
client = CosmosClient("<cosmos-endpoint>", "<cosmos-key>", logging_enable=True)

# Perform an operation to observe logs
database = client.create_database_if_not_exists("cosmicworks")
print(f"Database created or retrieved: {database.id}")
```

`logging_enable=True` enables detailed logging at the client level, capturing all HTTP requests and responses. This setup is ideal for debugging during development.

#### Enabling Per-Operation Logging

If you need detailed logging for a specific operation but not globally for the client, you can enable it per operation.

##### Example: Operation-Level Logging

```python
# Perform an operation with logging enabled specifically for this request
database = client.create_database("cosmicworks", logging_enable=True)
print(f"Database created: {database.id}")
```

This approach limits logging to individual operations, reducing log noise in production environments.

#### Enhanced Diagnostics with `CosmosHttpLoggingPolicy`

The SDK provides an extended logging policy, `CosmosHttpLoggingPolicy`, which builds on Azure's `HttpLoggingPolicy`. This policy captures more diagnostic information specific to Cosmos DB, such as elapsed request times and error messages.

##### Example: Using `CosmosHttpLoggingPolicy`

```python
import logging
from azure.cosmos import CosmosClient

# Create a logger for the Azure SDK
logger = logging.getLogger("azure")
logger.setLevel(logging.DEBUG)

# Configure file output for logs
handler = logging.FileHandler(filename="cosmos_logs.txt")
logger.addHandler(handler)

# Initialize the CosmosClient with enhanced diagnostics logging
client = CosmosClient(
    "<cosmos-endpoint>", 
    "<cosmos-key>", 
    logger=logger, 
    enable_diagnostics_logging=True
)

# Perform an operation to observe enhanced diagnostics
database = client.create_database_if_not_exists("cosmicworks")
print(f"Database created or retrieved: {database.id}")
```

Passing `enable_diagnostics_logging=True` to the client enables `CosmosHttpLoggingPolicy`. Logs include more details relevant to Cosmos DB, such as response timings and diagnostic headers.

#### Combining Global and Operation-Level Logging

You can mix global logging and per-operation logging to gain granular control over what is logged.

##### Example: Combining Logging Levels

```python
# Initialize the CosmosClient with enhanced diagnostics
client = CosmosClient(
    "<cosmos-endpoint>", 
    "<cosmos-key>", 
    enable_diagnostics_logging=True
)

# Perform an operation with a custom logger
logger = logging.getLogger("azure.operation")
logger.setLevel(logging.DEBUG)
handler = logging.StreamHandler(stream=sys.stdout)
logger.addHandler(handler)

# Use the custom logger for a specific operation
database = client.create_database("cosmicworks", logger=logger)
print(f"Database created: {database.id}")
```

This approach allows enhanced diagnostics at the client level while fine-tuning logging for specific operations.
### Configure the Azure Cosmos DB Python SDK for Offline Development

The Azure Cosmos DB Emulator is a local tool that emulates the Azure Cosmos DB service for development and testing. The emulator supports the NoSQL API and can be used in place of the cloud service when developing code using the Azure SDK for Python.

#### Prepare your development environment

If you have not already cloned the lab code repository for **Build copilots with Azure Cosmos DB** and set up your local environment, view the [Setup local lab environment](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/00-setup-lab-environment.html) instructions to do so.

#### Start the Azure Cosmos DB Emulator

If you are using a hosted lab environment, it should already have the emulator installed. If not, refer to the [installation instructions](https://docs.microsoft.com/azure/cosmos-db/local-emulator) to install the Azure Cosmos DB Emulator. Once the emulator has started, you can retrieve the connection string and use it to connect to the emulator using the Azure SDK for Python.

> 💡 You may optionally install the [new Linux-based Azure Cosmos DB Emulator (in preview)](https://learn.microsoft.com/azure/cosmos-db/emulator-linux), which is available as a Docker container. It supports running on a wide variety of processors and operating systems.

1. Start the **Azure Cosmos DB Emulator**.
    
    > 💡 If you are using Windows, the Azure Cosmos DB Emulator is pinned to both the Windows taskbar and Start Menu. If it does not start from the pinned icons, try opening it by double-clicking on the **C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe** file.
    
2. Wait for the emulator to open your default browser and navigate to the **https://localhost:8081/_explorer/index.html** landing page.
    
3. In the **Quickstart** pane, note the **Primary Connection String**. You will use this connection string later.
    

> 📝 Sometimes the landing page does not successfully load, even though the emulator is running. If this happens, you can use the well-known connection string to connect to the emulator. The well-known connection string is: `AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

#### Install the azure-cosmos library

The **azure-cosmos** library is available on **PyPI** for easy installation into your Python projects.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/02-sdk-offline** folder.
    
2. Open the context menu for the **python/02-sdk-offline** folder and then select **Open in Integrated Terminal** to open a new terminal instance.
    
    > 📝 This command will open the terminal with the starting directory already set to the **python/02-sdk-offline** folder.
    
3. Create and activate a virtual environment to manage dependencies:
    
    ```bash
    python -m venv venv
    source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
    ```
    
4. Install the [azure-cosmos](https://pypi.org/project/azure-cosmos) package using the following command:
    
    ```bash
    pip install azure-cosmos
    ```
    

#### Connect to the Emulator from the Python SDK

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/02-sdk-offline** folder.
    
2. Open the blank Python file named **script.py**.
    
3. Add the following code to connect to the emulator, create a database, and print its ID:
    
    ```python
    from azure.cosmos import CosmosClient, PartitionKey
       
    # Connection string for the Azure Cosmos DB Emulator
    connection_string = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="
        
    # Initialize the Cosmos client
    client = CosmosClient.from_connection_string(connection_string)
        
    # Create a database
    database_name = "cosmicworks"
    database = client.create_database_if_not_exists(id=database_name)
        
    # Print the database ID
    print(f"New Database: Id: {database.id}")
    ```
    
4. **Save** the **script.py** file.
    

#### Run the script

1. Use the same terminal window in **Visual Studio Code** that you used to set up the Python environment for this lab. If you close it, open the context menu for the **python/02-sdk-offline** folder and then select **Open in Integrated Terminal** to open a new terminal instance.
    
2. Run the script using the `python` command:
    
    ```bash
    python script.py
    ```
    
3. The script creates a database named `cosmicworks` in the emulator. You should see output similar to the following:
    
    ```text
    New Database: Id: cosmicworks
    ```
    

#### Create and View a New Container

You can extend the script to create a container within the database.

##### Updated Code

1. Modify the `script.py` file to add the following code at the bottom of the file for creating a container:
    
    ```python
    # Create a container
    container_name = "products"
    partition_key_path = "/categoryId"
    throughput = 400
        
    container = database.create_container_if_not_exists(
        id=container_name,
        partition_key=PartitionKey(path=partition_key_path),
        offer_throughput=throughput
    )
        
    # Print the container ID
    print(f"New Container: Id: {container.id}")
    ```
    

##### Run the Updated Script

1. Run the updated script using the following command:
    
    
    ```bash
    python script.py
    ```
    
2. The script creates a container named `products` in the emulator. You should see output similar to the following:
    
    
    ```text
    New Container: Id: products
    ```
    

##### Verify the Results

1. Switch to the browser where the emulator’s Data Explorer is open.
    
2. Refresh the **NoSQL API** to observe the new **cosmicworks** database and **products** container.
    

#### Stop the Azure Cosmos DB Emulator

It is important to stop the emulator when you are done using it to free up system resources. Follow the steps below based on your operating system:

##### On macOS or Linux:

If you started the emulator in a terminal window, follow these steps:

1. Locate the terminal window where the emulator is running.
    
2. Press `Ctrl + C` to terminate the emulator process.
    

Alternatively, if you need to stop the emulator process manually:

1. Open a new terminal window.
    
2. Use the following command to find the emulator process:
    
    ```bash
    ps aux | grep CosmosDB.Emulator
    ```
    

Identify the **PID** (Process ID) of the emulator process in the output. Use the kill command to terminate the emulator process:

```bash
kill <PID>
```

##### On Windows:

1. Locate the Azure Cosmos DB Emulator icon in the Windows System Tray (near the clock on the taskbar).
    
2. Right-click on the emulator icon to open the context menu.
    
3. Select **Exit** to shut down the emulator.
    

> 💡 It may take a minute for all instances of the emulator to exit.
