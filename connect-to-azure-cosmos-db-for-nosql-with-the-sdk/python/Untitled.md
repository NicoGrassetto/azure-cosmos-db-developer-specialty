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