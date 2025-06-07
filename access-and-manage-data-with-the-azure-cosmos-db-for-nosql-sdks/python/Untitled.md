# Access and manage data with the Azure Cosmos DB for NoSQL SDKs

## Implement Azure Cosmos DB for NoSQL point operations

### Introduction

The NoSQL API SDK for Azure Cosmos DB is used to perform various point operations, perform transactions, and to process bulk data. 

### Understand point operations

The **azure-cosmos** library provides robust support for modeling and interacting with items in an Azure Cosmos DB container. As a Python developer, it's important to structure your item definitions carefully to align with the requirements of the database and your application's use case.

At a minimum, each item in a container must include two attributes:

- A string attribute named **id**, which acts as the unique identifier for the item.
- A string attribute that corresponds to the **partition key** path of the container.

Here's an example of a Python class representing a minimal item structure:

```python
class Item:
    def __init__(self, id: str, partition_key: str):
        self.id = id
        self.partition_key = partition_key
```

You can enhance this class by adding more attributes of various types, including basic types like strings and numbers, or complex types like nested objects and lists.

```python
class Item:
    def __init__(
        self,
        id: str,
        partition_key: str,
        money: float,
        boolean: bool,
        tags: list[str],
        numbers: float,
        more_numbers: int,
        sophisticated: dict,
        one_to_many: list[dict]
    ):
        self.id = id
        self.partition_key = partition_key
        self.money = money
        self.boolean = boolean
        self.tags = tags
        self.numbers = numbers
        self.more_numbers = more_numbers
        self.sophisticated = sophisticated
        self.one_to_many = one_to_many
```

#### Example Scenario: Modeling a Product

Let’s use a fictional scenario for the remainder of this module. You need to create a **Product** class to represent products in your inventory. Each product includes:

- A unique **id**.
- A **name** of the product.
- A **categoryId** as the **partition key**.
- A **price** of the product.
- A list of **tags** for categorization.

Here’s how the class might look in Python:

```python
class Product:
    def __init__(self, id: str, name: str, category_id: str, price: float, tags: list[str]):
        self.id = id
        self.name = name
        self.category_id = category_id  # Partition key
        self.price = price
        self.tags = tags
```

This implementation is flexible and easy to use for various operations in your application. If, for any reason, you need to adjust the names of the attributes to match your business requirements, you can use serialization techniques to map attribute names in Python to property names in JSON.

For example, suppose you want to use the name **internal_id** in your Python code while still using **id** as the JSON property name:

```python
class Product:
    def __init__(self, internal_id: str, name: str, category_id: str, price: float, tags: list[str]):
        self.internal_id = internal_id
        self.name = name
        self.category_id = category_id  # Partition key
        self.price = price
        self.tags = tags

    def to_dict(self):
        return {
            "id": self.internal_id,  # Map internal_id to id
            "name": self.name,
            "categoryId": self.category_id,
            "price": self.price,
            "tags": self.tags,
        }
```

>**Note**: If you are working with existing Python classes that cannot be modified, consider using helper methods like `to_dict()` to convert objects into the format expected by Azure Cosmos DB for NoSQL. This allows you to reuse existing types while avoiding technical debt.

With these foundational modeling techniques, you can ensure that your Python application effectively interacts with Azure Cosmos DB for NoSQL.
### Create documents

To create a new item in an Azure Cosmos DB container, you first need to create an instance of your item model. In Python, this process can be achieved by defining an instance of the **Product** class.

```python
saddle = Product(
        internal_id="2a7816bf-9a3c-4f33-b7d7-84efb3923538",
        name="Road Warrior Saddle",
        category_id="26C74104-40BC-4541-8EF5-9892F7F03D72",
        price=65.15,
        tags=["black", "cushioned", "leather"]
    ).to_dict()
```

Recall that the **Product** class has a `to_dict` method that converts the instance to a dictionary. This class also maps the `internal_id` property to the `id` property, which Azure Cosmos DB requires.

Let’s assume there’s already a variable of type `azure.cosmos.Container` named **container**.

You can use the **create_item** method to create a new item in the container.

```python
container.create_item(body=saddle)
```

This invocation of the method creates the new item, but it doesn't provide metadata about the result of the operation. Alternatively, you can capture the response of the operation in a variable. The `get_response_headers()` method retrieves metadata about the operation, such as the request charge and the ETag value of the created item, which allows for optimistic concurrency scenarios.

```python
response = container.create_item(body=saddle)
# Get response headers
headers = response.get_response_headers()

# Retrieve the created item
item = response

# Extract metadata from headers
request_charge = headers.get('x-ms-request-charge')
etag = headers.get('etag')

# Output the metadata
print(f"Request Charge: {request_charge}")
print(f"etag: {etag}")

print(f"Item created: {item}")
```

- `x-ms-request-charge`: The request units (RUs) consumed by the operation.
- `etag`: A unique value that represents the version of the item.

#### Handling Exceptions

The Python SDK raises exceptions for various scenarios. You can handle these exceptions using a `try-except` block. The **CosmosHttpResponseError** exception includes useful information such as the HTTP status code. Some common HTTP status codes you might encounter include:

|Code|Title|Reason|
|---|---|---|
|400|Bad Request|Something was wrong with the item in the body of the request|
|403|Forbidden|Container was likely full|
|409|Conflict|Item in container likely already had a matching id|
|413|RequestEntityTooLarge|Item exceeds max entity size|
|429|TooManyRequests|Current request exceeds the maximum RU/s provisioned for the container|

Here’s an example of handling these exceptions:

```python
from azure.cosmos.exceptions import CosmosHttpResponseError

try:
    container.create_item(body=saddle)
except CosmosHttpResponseError as ex:
    if ex.status_code == 409:  # Conflict
        print("Conflict: Item with the same id already exists.")
    elif ex.status_code == 429:  # Too Many Requests
        print("Too many requests: Reduce request rate or increase RU/s provisioned.")
    elif ex.status_code == 400:  # Bad Request
        print("Bad request: Check the structure of the item being sent.")
    else:
        print(f"HTTP error occurred: Status code {ex.status_code}, message: {ex.message}")
except Exception as ex:
    print(f"An unexpected error occurred: {ex}")
```

This implementation ensures that you can create items, retrieve metadata for diagnostics, and handle errors gracefully.
### Read a document

There are two ways to read an item from a container in Azure Cosmos DB: a point read and a query read. A point read is the most efficient way to read an item because it uses the unique combination of the **id** and **partition key** of the item to retrieve it. A query read is more flexible and can retrieve multiple items based on a query. This unit focuses on point reads.

For example, if you have a 1-KB document in the container, the RU charge to perform a point read is 1 RU. The RU charge for a query read is based on the number of items returned and the complexity of the query, but the minimum RU charge is typically at least 2.3 RUs (Request Units). If you just need to read a single item, a point read is the most efficient way to do so because it can read the data directly and doesn't require the query engine to process the request.

To do a point read of an existing item from the container, we need two things.

First, we need the unique **id** of the item. Here, we store that id in a variable.

```python
item_id = "027D0B9A-F9D9-4C96-8213-C8546C4AAE71"
```

Next, define the value of the partition key for the item. We use the combination of the item id and the partition key value to perform the point read.

```python
partition_key_value = "26C74104-40BC-4541-8EF5-9892F7F03D72"
```

Once you have both the `id` and partition key, you can use the **read_item** method of the container object to perform the point read.

```python
saddle = container.read_item(item=item_id, partition_key=partition_key_value)
```

At this point, we can access properties of the **saddle** variable and print them to the console much like any local variable.

```python
formatted_name = f"New Product [{saddle['name']}]"
print(formatted_name)
```
### Update documents

We can also modify the properties of the **saddle** variable.

In this example, we change the price of the variable from the original **$27.12** to **$35.00**.

```python
saddle["price"] = 35.00
```

To persist this change, use the **replace_item** method, passing in the updated item.

```python
response = container.replace_item(item=saddle, body=saddle)
```

We can continue making updates to the item without needing to reread it. For instance, we can replace the tags array with a new array containing more accurate descriptions of the product.

```python
saddle["tags"] = ["brown", "new", "crisp"]
```

Even though we updated the document already, we don't have to read a new item before updating the item again.

```python
response = container.replace_item(item=saddle, body=saddle)
```
> **Note**: Cosmos DB supports **optimistic concurrency** to check if an item updated since the last read and return a conflict error if it has. Details on how to use this concurrency level are in the SDK documentation.
### Configure time-to-live (TTL) value for a specific document

The Azure Cosmos DB SDK allows you to configure Time-to-Live (TTL) on individual items in a container. TTL ensures that items are automatically deleted after a specified duration. To implement time-to-live (TTL) on an individual item, you can use the same strategy as you use to update an item.

The Python SDK allows you to dynamically set a new property value on an item. To set the **TimeToLive** value on an item, assign an integer to indicate how long, in seconds, you want the item to last before it automatically is purged beyond its last modified time. You do this change by updating the item object with a new **ttl** property value.

```python
saddle["ttl"] = 1000
```

Update the item using the **replace_item** method.

```python
container.replace_item(item=saddle, body=saddle)
```

However, as you recall, we defined a **Product** class to represent our items. We can define a new **TimeToLive** property that only sets the ttl property on the JSON if it’s not null. This technique is accomplished by configuring the member as a nullable int.

```python
class Product:
    def __init__(
            self,
            internal_id: str,
            name: str,
            category_id: str,
            price: float,
            tags: list[str],
            ttl: int = None):
        self.internal_id = internal_id
        self.name = name
        self.category_id = category_id  # Partition key
        self.price = price
        self.tags = tags
        self.ttl = ttl # Time to live

    def to_dict(self):
        product_dict = {
            "id": self.internal_id,  # Map internal_id to id
            "name": self.name,
            "categoryId": self.category_id,
            "price": self.price,
            "tags": self.tags
        }
        if self.ttl is not None:
            product_dict["ttl"] = self.ttl
        return product_dict
```

To work with this class after adding the **ttl** property, we can instantiate a new **Product** object and set the property value.

```python
# Perform the point read
saddle = container.read_item(item=item_id, partition_key=partition_key_value)

# Convert the response to a Product object
product = Product(
    internal_id=saddle["id"],
    name=saddle["name"],
    category_id=saddle["categoryId"],
    price=saddle["price"],
    tags=saddle["tags"],
    # Add the TTL value here if it exists
    ttl=saddle.get("ttl")
)

# Set the TTL property (in seconds)
product.ttl = 1000

# Update the item in Cosmos DB
container.replace_item(item=item_id, body=product.to_dict())
```

We use the `to_dict()` method to convert the **Product** object back to a dictionary and update the item. As you may recall, we use this method instead of the `__dict__` attribute to ensure that the **internal_id** property is properly mapped to the **id** property on the JSON object.

>**Note**: The `get("ttl")` method is used to retrieve the value of the **ttl** property from the **saddle** object. If the property doesn't exist, the method returns `None`. This value is then passed to the **Product** object when instantiated.

>**Note**: Remember, this process doesn't work if the **DefaultTimeToLive** property isn't configured at the container level. Also, a `ttl` value of `-1` disables TTL for the item.

### Delete documents

Deleting a document is similar, in process, to reading an item. You need the id and the value of the partition key path.

```python
item_id = "027D0B9A-F9D9-4C96-8213-C8546C4AAE71"
partition_key_value = "26C74104-40BC-4541-8EF5-9892F7F03D72"
```

Once you have these values, you invoke the **delete_item** method in a manner similar to the **read_item** method.

```python
container.delete_item(item=item_id, partition_key=partition_key_value)
```

Azure Cosmos DB also supports deleting all items contained within a single value for a partition key.

Store the partition key value in a variable:

```python
partition_key_value = "26C74104-40BC-4541-8EF5-9892F7F03D72"
```

With the partition key value, you invoke the **delete_all_items_by_partition_key** method.

```python
container.delete_all_items_by_partition_key(partition_key=partition_key_value)
```

The delete by partition key feature is an asynchronous, background operation that allows you to delete all documents with the same logical partition key value, using the Cosmos SDK. The delete by partition key operation is constrained to consume at most 10% of the total available RU/s on the container each second. This helps in limiting the resources used by this background task.

>**Note**: The delete all items by partition key operation is disabled by default and requires special activation by Azure Support.

### Create and update documents with the Azure Cosmos DB for NoSQL SDK

The `azure-cosmos` library includes methods to create, retrieve, update, and delete (CRUD) items within an Azure Cosmos DB for NoSQL container. Together, these methods perform some of the most common “CRUD” operations across various items within NoSQL API containers.

#### Prepare your development environment

If you have not already cloned the lab code repository for **Build copilots with Azure Cosmos DB** and set up your local environment, view the [Setup local lab environment](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/00-setup-lab-environment.html) instructions to do so.

#### Create an Azure Cosmos DB for NoSQL account

If you already created an Azure Cosmos DB for NoSQL account for the **Build copilots with Azure Cosmos DB** labs on this site, you can use it for this lab and skip ahead to the [next section](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/03-sdk-crud.html#install-the-azure-cosmos-library). Otherwise, view the [Setup Azure Cosmos DB](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/common/instructions/00-setup-cosmos-db.html) instructions to create an Azure Cosmos DB for NoSQL account that you will use throughout the lab modules and grant your user identity access to manage data in the account by assigning it to the **Cosmos DB Built-in Data Contributor** role.

#### Install the azure-cosmos library

The **azure-cosmos** library is available on **PyPI** for easy installation into your Python projects.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/03-sdk-crud** folder.
    
2. Open the context menu for the **python/03-sdk-crud** folder and then select **Open in Integrated Terminal** to open a new terminal instance.
    
    > 📝 This command will open the terminal with the starting directory already set to the **python/03-sdk-crud** folder.
    
3. Create and activate a virtual environment to manage dependencies:
    
    
    ```bash
    python -m venv venv
    source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
    ```
    
4. Install the [azure-cosmos](https://pypi.org/project/azure-cosmos) package using the following command:
    
    
    ```bash
    pip install azure-cosmos
    ```
    
5. Since we are using the asynchronous version of the SDK, we need to install the `asyncio` library as well:
    
    
    ```bash
    pip install asyncio
    ```
    
6. The asynchronous version of the SDK also requires the `aiohttp` library. Install it using the following command:
    
    
    ```bash
    pip install aiohttp
    ```
    
7. Install the [azure-identity](https://pypi.org/project/azure-identity) library, which allows us to use Azure authentication to connect to the Azure Cosmos DB workspace, using the following command:
    
    
    ```bash
    pip install azure-identity
    ```
    

#### Use the azure-cosmos library

Using the credentials from the newly created account, you will connect with the SDK classes and create a new database and container instance. Then, you will use the Data Explorer to validate that the instances exist in the Azure portal.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/03-sdk-crud** folder.
    
2. Open the blank Python file named **script.py**.
    
3. Add the following `import` statement to import the **PartitionKey** class:
    
    
    ```python
    from azure.cosmos import PartitionKey
    ```
    
4. Add the following `import` statements to import the asynchronous **CosmosClient** class, **DefaultAzureCredential** class, and the **asyncio** library:
    
    
    ```python
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    ```
    
5. Add variables named **endpoint** and **credential** and set the **endpoint** value to the **endpoint** of the Azure Cosmos DB account you created earlier. The **credential** variable should be set to a new instance of the **DefaultAzureCredential** class:
    
    
    ```python
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    ```
    
    > 📝 For example, if your endpoint is: **https://dp420.documents.azure.com:443/**, the statement would be: **endpoint = “https://dp420.documents.azure.com:443/”**.
    
6. All interaction with Cosmos DB starts with an instance of the `CosmosClient`. In order to use the asynchronous client, we need to use async/await keywords, which can only be used within async methods. Create a new async method named **main** and add the following code to create a new instance of the asynchronous **CosmosClient** class using the **endpoint** and **credential** variables:
    
    
    ```python
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
    ```
    
    > 💡 Since we’re using the asynchronous **CosmosClient** client, in order to properly use it you also have to warm it up and close it down. We recommend using the `async with` keywords as demonstrated in the code above to start your clients - these keywords create a context manager that automatically warms up, initializes, and cleans up the client, so you don’t have to.
    
7. Add the following code to create a database and container if they do not already exist:
    
    
    ```python
    # Create database
    database = await client.create_database_if_not_exists(id="cosmicworks")
        
    # Create container
    container = await database.create_container_if_not_exists(
        id="products",
        partition_key=PartitionKey(path="/categoryId"),
        offer_throughput=400
    )
    ```
    
8. Underneath the `main` method, add the following code to run the `main` method using the `asyncio` library:
    
    
    ```python
    if __name__ == "__main__":
        asyncio.run(query_items_async())
    ```
    
9. Your **script.py** file should now look like this:
    
    
    ```python
    from azure.cosmos import PartitionKey
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
            # Create database
            database = await client.create_database_if_not_exists(id="cosmicworks")
        
            # Create container
            container = await database.create_container_if_not_exists(
                id="products",
                partition_key=PartitionKey(path="/categoryId")
            )
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
10. **Save** the **script.py** file.
    
11. Before running the script, you must log into Azure using the `az login` command. At the terminal window, run:
    
    
    ```bash
    az login
    ```
    
12. Run the script to create the database and container:
    
    
    ```bash
    python script.py
    ```
    
13. Switch to your web browser window.
    
14. Within the **Azure Cosmos DB** account resource, navigate to the **Data Explorer** pane.
    
15. In the **Data Explorer**, expand the **cosmicworks** database node, then observe the new **products** container node within the **NOSQL API** navigation tree.
    

#### Perform create and read point operations on items with the SDK

You will now use the set of methods in the **ContainerProxy** class to perform common operations on items within a NoSQL API container.

1. Return to **Visual Studio Code**. If it is not still open, open the **script.py** code file within the **python/03-sdk-crud** folder.
    
2. Create a new product item and assign it to a variable named **saddle** with the following properties:
    
    |Property|Value|
    |---|---|
    |**id**|_706cd7c6-db8b-41f9-aea2-0e0c7e8eb009_|
    |**categoryId**|_9603ca6c-9e28-4a02-9194-51cdb7fea816_|
    |**name**|_Road Saddle_|
    |**price**|_45.99d_|
    |**tags**|_{ tan, new, crisp }_|
    
    
    ```python
    saddle = {
        "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        "name": "Road Saddle",
        "price": 45.99,
        "tags": ["tan", "new", "crisp"]
    }
    ```
    
3. Invoke the [`create_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-create-item) method of the **container** variable passing in the **saddle** variable as the method parameter:
    
    ```python
    await container.create_item(body=saddle)
    ```
    
4. Once you are done, your code file should now include:
    
    
    ```python
    from azure.cosmos import PartitionKey
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
            # Create database
            database = await client.create_database_if_not_exists(id="cosmicworks")
        
            # Create container
            container = await database.create_container_if_not_exists(
                id="products",
                partition_key=PartitionKey(path="/categoryId")
            )
            
            saddle = {
                "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
                "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
                "name": "Road Saddle",
                "price": 45.99,
                "tags": ["tan", "new", "crisp"]
            }
                
            await container.create_item(body=saddle)
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
5. **Save** and run the script again:
    
    
    ```bash
    python script.py
    ```
    
6. Observe the new item in the **Data Explorer**.
    
7. Return to **Visual Studio Code**.
    
8. Return to the editor tab for the **script.py** code file.
    
9. Delete the following lines of code:
    
    
    ```python
    saddle = {
        "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        "name": "Road Saddle",
        "price": 45.99,
        "tags": ["tan", "new", "crisp"]
    }
        
    await container.create_item(body=saddle)
    ```
    
10. Create a string variable named **item_id** with a value of **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:
    
    
    ```python
    item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
    ```
    
11. Create a string variable named **partition_key** with a value of **9603ca6c-9e28-4a02-9194-51cdb7fea816**:
    
    ```python
    partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    ```
    
12. Invoke the [`read_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-read-item) method of the **container** variable passing in the **item_id** and **partition_key** variables as the method parameters:
    
    
    ```python
    # Read item    
    saddle = await container.read_item(item=item_id, partition_key=partition_key)
    ```
    
    > 💡 The `read_item` method allows you to perform a point read operation on an item in the container. The method requires the `item_id` and `partition_key` parameters to identify the item to read. As opposed to executing a query using Cosmos DB’s SQL query language to find the single item, the `read_item` method is more efficient and cost-effective way to retrieve a single item. Point reads can read the data directly and don’t require the query engine to process the request.
    
13. Print the saddle object using a formatted output string:
    
    
    ```python
    print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
    ```
    
14. Once you are done, your code file should now include:
    
    
    ```python
    from azure.cosmos import PartitionKey
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
            # Create database
            database = await client.create_database_if_not_exists(id="cosmicworks")
        
            # Create container
            container = await database.create_container_if_not_exists(
                id="products",
                partition_key=PartitionKey(path="/categoryId")
            )
           
            item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
            partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
       
            # Read item
            saddle = await container.read_item(item=item_id, partition_key=partition_key)
                
            print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
15. **Save** and run the script again:
    
    
    ```bash
    python script.py
    ```
    
16. Observe the output from the terminal. Specifically, observe the formatted output text with the id, name, and price from the item.
    

#### Perform update and delete point operations with the SDK

While learning the SDK, it’s not uncommon to use an online Azure Cosmos DB account or the emulator to update an item and oscillate back-and-forth between the Data Explorer and your IDE of choice as you perform an operation and check to see if your change has been applied. Here, you will do just that as you update and delete an item using the SDK.

1. Return to your web browser window or tab.
    
2. Within the **Azure Cosmos DB** account resource, navigate to the **Data Explorer** pane.
    
3. In the **Data Explorer**, expand the **cosmicworks** database node, then expand the new **products** container node within the **NOSQL API** navigation tree.
    
4. Select the **Items** node. Select the only item within the container and then observe the values of the **name** and **price** properties of the item.
    
    |**Property**|**Value**|
    |---|---|
    |**Name**|_Road Saddle_|
    |**Price**|_$45.99_|
    
    > 📝 At this point in time, these values should not have been changed since you have created the item. You will change these values in this exercise.
    
5. Return to **Visual Studio Code**. Return to the editor tab for the **script.py** code file.
    
6. Delete the following line of code:
    
    
    ```python
    print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
    ```
    
7. Change the **saddle** variable by setting the value of the price property to **32.55**:
    
    
    ```python
    saddle["price"] = 32.55
    ```
    
8. Modify the **saddle** variable again by setting the value of the **name** property to **Road LL Saddle**:
    
    
    ```python
    saddle["name"] = "Road LL Saddle"
    ```
    
9. Invoke the [`replace_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-replace-item) method of the **container** variable passing in the **item_id** and **saddle** variables as method parameters:
    
    
    ```python
    await container.replace_item(item=item_id, body=saddle)
    ```
    
10. Once you are done, your code file should now include:
    
    
    ```python
    from azure.cosmos import PartitionKey
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
            # Create database
            database = await client.create_database_if_not_exists(id="cosmicworks")
        
            # Create container
            container = await database.create_container_if_not_exists(
                id="products",
                partition_key=PartitionKey(path="/categoryId")
            )
            
            item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
            partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
            # Read item
            saddle = await container.read_item(item=item_id, partition_key=partition_key)
                
            saddle["price"] = 32.55
            saddle["name"] = "Road LL Saddle"
        
            await container.replace_item(item=item_id, body=saddle)
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
11. **Save** and run the script again:
    
    
    ```bash
    python script.py
    ```
    
12. Return to your web browser window or tab.
    
13. Within the **Azure Cosmos DB** account resource, navigate to the **Data Explorer** pane.
    
14. In the **Data Explorer**, expand the **cosmicworks** database node, then expand the new **products** container node within the **NOSQL API** navigation tree.
    
15. Select the **Items** node. Select the only item within the container and then observe the values of the **name** and **price** properties of the item.
    
    |**Property**|**Value**|
    |---|---|
    |**Name**|_Road LL Saddle_|
    |**Price**|_$32.55_|
    
    > 📝 At this point in time, these values should have been changed since you have observed the item.
    
16. Return to **Visual Studio Code**. Return to the editor tab for the **script.py** code file.
    
17. Delete the following lines of code:
    
    
    ```python
    # Read item
    saddle = await container.read_item(item=item_id, partition_key=partition_key)
        
    saddle["price"] = 32.55
    saddle["name"] = "Road LL Saddle"
        
    await container.replace_item(item=item_id, body=saddle)
    ```
    
18. Invoke the [`delete_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-delete-item) method of the **container** variable passing in the **item_id** and **partition_key** variables as method parameters:
    
    
    ```python
    # Delete the item
    await container.delete_item(item=item_id, partition_key=partition_key)
    ```
    
19. Save and run the script again:
    
    
    ```bash
    python script.py
    ```
    
20. Close the integrated terminal.
    
21. Return to your web browser window or tab.
    
22. Within the **Azure Cosmos DB** account resource, navigate to the **Data Explorer** pane.
    
23. In the **Data Explorer**, expand the **cosmicworks** database node, then expand the new **products** container node within the **NOSQL API** navigation tree.
    
24. Select the **Items** node. Observe that the items list is now empty.
    
25. Close your web browser window or tab.
    
26. Close **Visual Studio Code**.

## Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
### Introduction

When you write multi-document transactions in the programming language of your choice, it gives you power and flexibility. You can manage, version, and optimize your code using your team’s practices in the way you want without having to adapt to a language you may or may not want to use.
### Create a transactional batch with the SDK

Transactional batch requests enable you to perform multiple operations simultaneously within the same partition key. All operations are executed in the specified order, and the transaction is committed only if every operation succeeds. If any operation fails, the entire transaction is reverted. Consider a fictional scenario where two used bicycle accessories are created in a container, a **worn saddle,** and a **rusty handlebar**. For simplicity's sake, they have short unique identifiers and category identifiers.

```python
saddle = ("create", (
    {"id": "0120", "name": "Worn Saddle", "categoryId": "accessories-used"},
))
handlebar = ("create", (
    {"id": "012A", "name": "Rusty Handlebar", "categoryId": "accessories-used"},
))
```

Transactional Batch operations look similar to the singular operations APIs, and are tuples containing (`operation_type_string`, `args_tuple`, `batch_operation_kwargs_dictionary`), with the kwargs dictionary being optional.

The `operation_type_string` is a string that represents the operation type, such as "create", "upsert", "replace", "read", "patch", or "delete".

To execute the batch, use the `execute_item_batch` method of the container object. The first argument is a list of the operations to be executed in the batch. In our example, we have two operations to create items within the same partition key value.

```python
# Partition key
partition_key = "accessories-used"

batch = [
    saddle,
    handlebar
]

try:
    # Execute the batch
    batch_response = container.execute_item_batch(batch, partition_key=partition_key)

    # Check the response and print results
    for result in batch_response:
        print(result.get("resourceBody"))
except exceptions.CosmosHttpResponseError as e:
    print(f"Failed to execute batch: {e.message}")
```

The transactional batch supports operations with the **same logical partition key**. Operations with different logical partition keys fail. In the following example, the transactional batch fails with a bad request due to having a different logical partition key.

```python
partition_key = "accessories-used"

batch = [
    ("create", ({"id": "0120", "name": "Worn Saddle", "categoryId": "accessories-used"},)),
    ("create", ({"id": "012C", "name": "Pristine Handlebar", "categoryId": "accessories-new"},))
]
batch_response = container.execute_item_batch(batch, partition_key=partition_key)
```
### Review batch operation results with the SDK

The **CosmosList** object returned by the `execute_item_batch` method provides the means to examine the results of each operation in a batch request. Each operation's result can be accessed using zero-based indexing and provides details such as the HTTP status code and the operation's response body.

You can iterate through all operations in the batch to access the results:

```python
for result in batch_response:
    resource_body = result.get("resourceBody")
    print(f"Operation result: {resource_body}")
```

You can examine the status code, response body, and request charge of individual operations using the `get` method of the batch response objects.

```python
first_item_result = batch_response[0]
first_item_status_code = first_item_result.get("statusCode")
first_item_request_charge = first_item_result.get("requestCharge")
first_item_result = first_item_result.get("resourceBody")
print(f"First item status: {first_item_status_code}; request charge: {first_item_request_charge}; result: {first_item_result}")

second_item_result = batch_response[1]
second_item_status_code = second_item_result.get("statusCode")
second_item_request_charge = second_item_result.get("requestCharge")
second_item_result = second_item_result.get("resourceBody")
print(f"Second item status: {second_item_status_code}; request charge: {second_item_request_charge}; result: {second_item_result}")
```

When handling errors in batch operations, use the `CosmosBatchOperationError` exception to identify the failed operation and its index:

```python
try:
    batch_response = container.execute_item_batch(batch, partition_key=partition_key)
except exceptions.CosmosBatchOperationError as e:
    error_operation_index = e.error_index
    error_operation_response = e.operation_responses[error_operation_index]
    error_operation = batch[error_operation_index]
    print("\nError operation: {}, error operation response: {}\n".format(error_operation, error_operation_response))
```
### Batch multiple point operations together with the Azure Cosmos DB for NoSQL SDK

The `azure-cosmos` Python SDK provides the `execute_item_batch` method for executing multiple point operations in a single logical step. This allows developers to efficiently bundle multiple operations together and determine if they completed successfully server-side.

#### Prepare your development environment

If you have not already cloned the lab code repository for **Build copilots with Azure Cosmos DB** and set up your local environment, view the [Setup local lab environment](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/00-setup-lab-environment.html) instructions to do so.

#### Create an Azure Cosmos DB for NoSQL account

If you already created an Azure Cosmos DB for NoSQL account for the **Build copilots with Azure Cosmos DB** labs on this site, you can use it for this lab and skip ahead to the [next section](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/04-sdk-batch.html#install-the-azure-cosmos-library). Otherwise, view the [Setup Azure Cosmos DB](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/common/instructions/00-setup-cosmos-db.html) instructions to create an Azure Cosmos DB for NoSQL account that you will use throughout the lab modules and grant your user identity access to manage data in the account by assigning it to the **Cosmos DB Built-in Data Contributor** role.

#### Install the azure-cosmos library

The **azure-cosmos** library is available on **PyPI** for easy installation into your Python projects.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/04-sdk-batch** folder.
    
2. Open the context menu for the **python/04-sdk-batch** folder and then select **Open in Integrated Terminal** to open a new terminal instance.
    
    > 📝 This command will open the terminal with the starting directory already set to the **python/04-sdk-batch** folder.
    
3. Create and activate a virtual environment to manage dependencies:
    
    ```bash
    python -m venv venv
    source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
    ```
    
4. Install the [azure-cosmos](https://pypi.org/project/azure-cosmos) package using the following command:
    
    ```bash
    pip install azure-cosmos
    ```
    
5. Since we are using the asynchronous version of the SDK, we need to install the `asyncio` library as well:
    
    ```bash
    pip install asyncio
    ```
    
6. The asynchronous version of the SDK also requires the `aiohttp` library. Install it using the following command:
    
    ```bash
    pip install aiohttp
    ```
    
7. Install the [azure-identity](https://pypi.org/project/azure-identity) library, which allows us to use Azure authentication to connect to the Azure Cosmos DB workspace, using the following command:
    
    ```bash
    pip install azure-identity
    ```
    

#### Use the azure-cosmos library

Using the credentials from the newly created account, you will connect with the SDK classes and create a new database and container instance. Then, you will use the Data Explorer to validate that the instances exist in the Azure portal.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/03-sdk-crud** folder.
    
2. Open the blank Python file named **script.py**.
    
3. Add the following `import` statement to import the **PartitionKey** class:
    
    
    ```python
    from azure.cosmos import PartitionKey
    ```
    
4. Add the following `import` statements to import the asynchronous **CosmosClient** class, **DefaultAzureCredential** class, and the **asyncio** library:
    
    
    ```python
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    ```
    
5. Add variables named **endpoint** and **credential** and set the **endpoint** value to the **endpoint** of the Azure Cosmos DB account you created earlier. The **credential** variable should be set to a new instance of the **DefaultAzureCredential** class:
    
    
    ```python
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    ```
    
    > 📝 For example, if your endpoint is: **https://dp420.documents.azure.com:443/**, the statement would be: **endpoint = “https://dp420.documents.azure.com:443/”**.
    
6. All interaction with Cosmos DB starts with an instance of the `CosmosClient`. In order to use the asynchronous client, we need to use async/await keywords, which can only be used within async methods. Create a new async method named **main** and add the following code to create a new instance of the asynchronous **CosmosClient** class using the **endpoint** and **credential** variables:
    
    
    ```python
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
    ```
    
    > 💡 Since we’re using the asynchronous **CosmosClient** client, in order to properly use it you also have to warm it up and close it down. We recommend using the `async with` keywords as demonstrated in the code above to start your clients - these keywords create a context manager that automatically warms up, initializes, and cleans up the client, so you don’t have to.
    
7. Add the following code to create a database and container if they do not already exist:
    
    
    ```python
    # Create database
    database = await client.create_database_if_not_exists(id="cosmicworks")
        
    # Create container
    container = await database.create_container_if_not_exists(
        id="products",
        partition_key=PartitionKey(path="/categoryId"),
        offer_throughput=400
    )
    ```
    
8. Underneath the `main` method, add the following code to run the `main` method using the `asyncio` library:
    
    
    ```python
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
9. Your **script.py** file should now look like this:
    
    ```python
    from azure.cosmos import exceptions, PartitionKey
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
            # Create database
            database = await client.create_database_if_not_exists(id="cosmicworks")
        
            # Create container
            container = await database.create_container_if_not_exists(
                id="products",
                partition_key=PartitionKey(path="/categoryId")
            )
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
10. **Save** the **script.py** file.
    
11. Before running the script, you must log into Azure using the `az login` command. At the terminal window, run:
    
    
    ```bash
    az login
    ```
    
12. Run the script to create the database and container:
    
    
    ```bash
    python script.py
    ```
    
13. Switch to your web browser window.
    
14. Within the **Azure Cosmos DB** account resource, navigate to the **Data Explorer** pane.
    
15. In the **Data Explorer**, expand the **cosmicworks** database node, then observe the new **products** container node within the **NOSQL API** navigation tree.
    

#### Creating a transactional batch

First, let’s create a simple transactional batch that makes two fictional products. This batch will insert a worn saddle and a rusty handlebar into the container with the same “used accessories” category identifier. Both items have the same logical partition key, ensuring that we will have a successful batch operation.

1. Return to **Visual Studio Code**. If it is not still open, open the **script.py** code file within the **python/04-sdk-batch** folder.
    
2. Create two dictionaries representing products: a **worn saddle** and a **rusty handlebar**. Both items share the same partition key value of **“9603ca6c-9e28-4a02-9194-51cdb7fea816”**.
    
    
    ```python
    saddle = ("create", (
        {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
    ))
    
    handlebar = ("create", (
        {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
    ))
    ```
    
3. Define the partition key value.
    
    ```python
    partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    ```
    
4. Create a batch containing the two items.
    
    ```python
    batch = [saddle, handlebar]
    ```
    
5. Execute the batch using the `execute_item_batch` method of the `container` object and print the response for each item in the batch.

```python
try:
        # Execute the batch
        batch_response = await container.execute_item_batch(batch, partition_key=partition_key)

        # Print results for each operation in the batch
        for idx, result in enumerate(batch_response):
            status_code = result.get("statusCode")
            resource = result.get("resourceBody")
            print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
    except exceptions.CosmosBatchOperationError as e:
        error_operation_index = e.error_index
        error_operation_response = e.operation_responses[error_operation_index]
        error_operation = batch[error_operation_index]
        print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
    except Exception as ex:
        print(f"An error occurred: {ex}")
```

1. Once you are done, your code file should now include:
    
    ```python
    from azure.cosmos import exceptions, PartitionKey
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
            # Create database
            database = await client.create_database_if_not_exists(id="cosmicworks")
        
            # Create container
            container = await database.create_container_if_not_exists(
                id="products",
                partition_key=PartitionKey(path="/categoryId")
            )
    
            saddle = ("create", (
                {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
            ))
            handlebar = ("create", (
                {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
            ))
            
            partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
            
            batch = [saddle, handlebar]
                
            try:
                # Execute the batch
                batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
            
                # Print results for each operation in the batch
                for idx, result in enumerate(batch_response):
                    status_code = result.get("statusCode")
                    resource = result.get("resourceBody")
                    print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
            except exceptions.CosmosBatchOperationError as e:
                error_operation_index = e.error_index
                error_operation_response = e.operation_responses[error_operation_index]
                error_operation = batch[error_operation_index]
                print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
            except Exception as ex:
                print(f"An error occurred: {ex}")
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
2. **Save** and run the script again:
    
    ```bash
    python script.py
    ```
    
3. The output should indicate a successful status code for each operation.
    

#### Creating an errant transactional batch

Now, let’s create a transactional batch that will error purposefully. This batch will attempt to insert two items that have different logical partition keys. We will create a flickering strobe light in the “used accessories” category and a new helmet in the “pristine accessories” category. By definition, this should be a bad request and return an error when performing this transaction.

1. Return to the editor tab for the **script.py** code file.
    
2. Delete the following lines of code:
    
    ```python
    saddle = ("create", (
        {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
    ))
    handlebar = ("create", (
        {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
    ))
    
    partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    
    batch = [saddle, handlebar]
    ```
    
3. Modify the script to create a new **flickering strobe light** and a **new helmet** with different partition key values.
    
    ```python
    light = ("create", (
        {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
    ))
    helmet = ("create", (
        {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
    ))
    ```
    
4. Define the partition key value for the batch.
    
    ```python
    partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    ```
    
5. Create a new batch containing the two items.
    
    
    ```python
    batch = [light, helmet]
    ```
    
6. Once you are done, your code file should now include:
    
    ```python
    from azure.cosmos import exceptions, PartitionKey
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
            # Create database
            database = await client.create_database_if_not_exists(id="cosmicworks")
        
            # Create container
            container = await database.create_container_if_not_exists(
                id="products",
                partition_key=PartitionKey(path="/categoryId")
            )
    
            light = ("create", (
                {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
            ))
            helmet = ("create", (
                {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
            ))
            
            partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
            
            batch = [light, helmet]
                
            try:
                # Execute the batch
                batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
            
                # Print results for each operation in the batch
                for idx, result in enumerate(batch_response):
                    status_code = result.get("statusCode")
                    resource = result.get("resourceBody")
                    print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
            except exceptions.CosmosBatchOperationError as e:
                error_operation_index = e.error_index
                error_operation_response = e.operation_responses[error_operation_index]
                error_operation = batch[error_operation_index]
                print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
            except Exception as ex:
                print(f"An error occurred: {ex}")
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
7. **Save** and run the script again:
    
    ```bash
    python script.py
    ```
    
8. Observe the output from the terminal. The status code on the second item (the “New Helmet”) should be **400** for **Bad Request**. This occurred because all items within the transaction did not share the same partition key value as the transactional batch.
    
9. Close the integrated terminal.
    
10. Close **Visual Studio Code**.

### Implement optimistic concurrency control

Using the SDK to read an item and then update the same item in a subsequent operation carries some inherent risk. Another operation could potentially come in from a separate client and change the underlying document before the first client’s update operation is finalized. This conflict could create a "lost update" situation. Let’s illustrate this conflict with an example.

Here's a typical Python code example with a separate read and update operation.

```python
item_id = "01AC0"
partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"

# Read the item
item_response = container.read_item(item=item_id, partition_key=partition_key)
product = item_response

# Update the product price
product["price"] = 50.0

# Upsert the item back to the container
container.upsert_item(body=product)
```

Since read and write in this example are distinct operations, there's a latency between these operations. This latency is represented in this diagram as _n_.

![Diagram of the N latency between read and update.](https://learn.microsoft.com/en-us/training/wwl-data-ai/perform-cross-document-transactional-operations-azure-cosmos-db-sql-api/media/5-latency.png)

This latency can be as short as milliseconds or seconds in computer code but could still be catastrophic enough to lose potential updates. Some user-facing applications, where user input causes a longer latency between a read and update operation, can cause a longer _n_ value and a higher potential for lost updates. This issue can be resolved by implementing **optimistic concurrency control**.


```python
item_response = container.read_item(item=item_id, partition_key=partition_key)
product = item_response
```

Each item has an **ETag** value. This value is updated when the item is updated. You can retrieve the ETag value of the item by observing the headers from the _read_ operation.

There are two ways you can access the ETag value of the item. One way is to use the **get_response_headers** method. With this method, the ETag value is named **etag**.

```python
headers = item_response.get_response_headers()
etag = headers.get("etag")
```

Another way is to use the **get** method on the item response. With this method, the ETag value is named **_etag**.

```python
etag = item_response.get("_etag")
```

To prevent lost updates, you can use the **if-match** condition to see if the **ETag** still matches the current ETag header of the item server-side as part of your update request.

```python
headers = {"If-Match": etag}
container.upsert_item(body=product, headers=headers)
```

If the **ETag** value doesn't match the current ETag header of the item server-side, the operation fails, and you need to re-read the item to get the latest version of the item before trying to update it again. The failure code is **412 Precondition Failed**.

The updates to the Python code only required minor changes to implement optimistic concurrency control to ensure that your update operations didn't lose changes previously saved server-side by competing clients.

```python
item_id = "01AC0"
partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"

# Read the item
item_response = container.read_item(item=item_id, partition_key=partition_key)
product = item_response
etag = item_response.get("_etag")

# Update the product price
product["price"] = 50.0

# Use optimistic concurrency control
headers = {"If-Match": etag}
container.upsert_item(body=product, headers=headers)
```
## Process bulk data in Azure Cosmos DB for NoSQL
### Introduction

If you’re using a high throughput database, it’s not uncommon for you to want to send a high volume of data to the database with as much throughput as possible. What if you wanted to send millions of overnight logs to your Azure Cosmos DB for NoSQL account?
### Create bulk operations with the SDK (Python)

Bulk execution must be enabled by creating a new instance of the **CosmosClient** with the `enable_bulk=True` parameter.

```python
from azure.cosmos import CosmosClient

# Create the Cosmos client with bulk execution enabled
client = CosmosClient(
    url="https://<cosmos-account-name>.documents.azure.com:443/",
    credential="<your-key-or-credential>",
    enable_bulk=True
)
```

You can also use **DefaultAzureCredential** from the `azure.identity` package to authenticate with a managed identity.

```python
from azure.identity import DefaultAzureCredential
from azure.cosmos import CosmosClient

# Configure the account endpoint
account_endpoint = "https://<cosmos-account-name>.documents.azure.com:443/"

# Use DefaultAzureCredential to authenticate with Managed Identity
credential = DefaultAzureCredential()

# Create CosmosClient with the account endpoint and Managed Identity
client = CosmosClient(account_endpoint, credential, enable_bulk=True)
```

For context, how do we usually perform a single _Create Item_ operation? Here, we invoke the **create_item** method, which is an asynchronous coroutine in Python.

```python
await container.create_item(body=product, partition_key=product["partitionKeyValue"])
```

Under the hood, we could handle the coroutine objects and even add them to lists. Here's an example where we create two tasks for two _Create Item_ operations that would create two products. We don’t start the tasks yet; we add them to a list to start them later.

```python
import asyncio

concurrent_tasks = []

first_task = container.create_item(body=first_product, partition_key=first_product["partitionKeyValue"])
concurrent_tasks.append(first_task)

second_task = container.create_item(body=second_product, partition_key=second_product["partitionKeyValue"])
concurrent_tasks.append(second_task)
```

This code is a bit verbose; we could make it cleaner. Here, we have a method called **get_our_products_from_somewhere** that generates **250,000** products. We then create a list of tasks and use a clean Python `for` loop.

```python
products_to_insert = get_our_products_from_somewhere()

concurrent_tasks = []

for product in products_to_insert:
    task = container.create_item(body=product, partition_key=product["partitionKeyValue"])
    concurrent_tasks.append(task)
```

For each product in the products list, we add a task to create an item in our Azure Cosmos DB for NoSQL container. No action is yet taken. Even better, no other Cosmos DB-specific code is written other than the **container.create_item** part.

When we invoke **asyncio.gather**, the SDK kicks in to create batches to group our operations by physical partition, then distribute the requests to run concurrently. Grouping operations greatly improves efficiency by reducing the number of back-end requests and allowing batches to be dispatched to different physical partitions in parallel.

```python
await asyncio.gather(*concurrent_tasks)
```

Once each batch is done, the SDK translates the batches back to the results for the client-side application. This action is seamless and transparent to the developer.
### Review bulk operation caveats

There are some caveats to consider when developing for bulk operations that are different than designing for typical Azure Cosmos DB for NoSQL applications.

#### Throughput consumption

The provisioned throughput in request units per second (RU/s) is higher than if the operations were executed individually. This increase should be considered as you evaluate total throughput requirements when measured against other operations that will happen concurrently.

#### Latency impact

When the SDK is attempting to fill a batch and doesn’t quite have enough items, it will wait 100 milliseconds for more items. This wait can affect overall latency.

#### Document size

The SDK automatically creates batches for optimization with a maximum of 2 Mb (or 100 operations). Smaller items can take advantage of this optimization, with oversized items having an inverse effect.

### Implement bulk best practices

While it is straightforward to add bulk support to your applications, there are a couple of best practices.

#### Configure the partition key

You are not required to provide the partition key for many of the operations on the Container class; the SDK will determine it automatically from your class. However, this will add to your overhead in a bulk scenario and could create needless complexity. It’s a good practice to provide the partition key to the operation if you already have it.

#### Use stream API in serialize-deserialize scenarios

If you're building an API, avoid unnecessary serialization and deserialization. For example, you are sometimes forced to deserialize and serialize going to and from some database platforms. With Azure Cosmos DB for NoSQL, you can use the Stream variants of common item operations to avoid unnecessary performance overhead. This is especially true when using the bulk features of the SDK.

#### Configure worker task per partition key

If your items are already separated into logical partition keys, you can create a list of worker tasks per partition key. Each worker task in that list can then spawn child tasks for each operation within that logical partition key. This setup would de facto create a hierarchy of tasks which coordination of per item operations.

### Move multiple documents in bulk with the Azure Cosmos DB for NoSQL SDK (Python Version)

The easiest way to learn how to perform a bulk operation is to attempt to push many documents to an Azure Cosmos DB for NoSQL account in the cloud. Using the bulk features of the SDK, this can be done with some minor help from the `asyncio` and `aiohttp` libraries in Python.

#### Prepare your development environment

If you have not already cloned the lab code repository for DP-420 to the environment where you’re working on this lab, follow these steps to do so. Otherwise, open the previously cloned folder in Visual Studio Code.

1. Start Visual Studio Code.
2. Open the command palette and run `Git: Clone` to clone the https://github.com/microsoftlearning/dp-420-cosmos-db-dev GitHub repository in a local folder of your choice.
3. Once the repository has been cloned, open the local folder you selected in Visual Studio Code.
#### Create an Azure Cosmos DB for NoSQL account and configure the SDK project

1. In a new browser tab, go to https://portal.azure.com.
2. Sign in and create a new Azure Cosmos DB for NoSQL account with the following settings:

|Setting|Value|
|---|---|
|Workload Type|Learning|
|Subscription|Your Azure subscription|
|Resource group|Existing or new|
|Account Name|Globally unique name|
|Location|Any available region|
|Capacity mode|Provisioned throughput|
|Apply Free Tier Discount|Do Not Apply|

3. After deployment, go to the **Keys** pane and note the **URI** and **PRIMARY KEY**.
4. In **Data Explorer**, create a new container:

|Setting|Value|
|---|---|
|Database id|cosmicworks|
|Share throughput across containers|Do not select|
|Container id|products|
|Partition key|/categoryId|
|Container throughput|Autoscale 4000|

#### Bulk inserting 25,000 documents

Install the required packages:

```bash
pip install azure-cosmos faker aiohttp
```

Create a new Python script named `bulk_insert.py` and add the following code:

```python
import asyncio
import uuid
from faker import Faker
from azure.cosmos.aio import CosmosClient
from azure.cosmos import PartitionKey, exceptions

# Replace with your actual endpoint and key
endpoint = "<cosmos-endpoint>"
key = "<cosmos-key>"

# Initialize Faker
fake = Faker()

# Define the product model
def generate_product():
    return {
        "id": str(uuid.uuid4()),
        "name": fake.unique.word(),
        "price": round(fake.pyfloat(left_digits=3, right_digits=2, positive=True, min_value=10, max_value=1000), 2),
        "categoryId": fake.word()
    }

async def main():
    # Create Cosmos client with bulk execution enabled
    async with CosmosClient(endpoint, key, consistency_level="Session") as client:
        database = await client.create_database_if_not_exists("cosmicworks")
        container = await database.create_container_if_not_exists(
            id="products",
            partition_key=PartitionKey(path="/categoryId"),
            offer_throughput=4000
        )

        # Generate 25,000 fake products
        products_to_insert = [generate_product() for _ in range(25000)]

        # Create tasks for bulk insert
        tasks = [
            container.create_item(body=product, partition_key=product["categoryId"])
            for product in products_to_insert
        ]

        # Execute all tasks concurrently
        await asyncio.gather(*tasks)

        print("Bulk tasks complete")

if __name__ == "__main__":
    asyncio.run(main())
```
#### Run the script

In the terminal:

```bash
python bulk_insert.py
```

The application should run silently and complete in about 1–2 minutes.
#### Observe the results

1. Go to the **Data Explorer** in the Azure portal.
2. Expand the `cosmicworks` database and the `products` container.
3. Select **Items** to view the inserted documents.
4. Run the following SQL query to count the documents:

```sql
SELECT COUNT(1) FROM c
```

Execute the query and observe the count.