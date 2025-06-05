# Execute queries and build a Generative AI application with Azure Cosmos DB
## Query the Azure Cosmos DB for NoSQL
### Introduction

The Azure Cosmos DB for NoSQL supports Structured Query Language (SQL) as a JSON query language.
### Understand NoSQL query language

Azure Cosmos DB for NoSQL uses the already popular Structured Query Language (SQL) syntax to perform queries over semi-structured data. If you ran queries in database platforms like MySQL or SQL Server, then you may already have some of the tools necessary to write queries in Azure Cosmos DB for NoSQL.

For this module, we focus on a fictional container of **products** with the following structure:

|**Property**|**Value**|
|---|---|
|**id**|_String_ \| _unique identifier_|
|**categoryId**|_String_ \| _partition key_|
|**categoryName**|_String_|
|**sku**|_String_|
|**name**|_String_|
|**description**|_String_|
|**price**|_Number_|
|**tags**|_Array_ \| _[ String id, String name ]_|

Here's an example of a JSON object that would be in this container:

```json
{
    "id": "86FD9250-4BD5-42D2-B941-1C1865A6A65E",
    "categoryId": "F3FBB167-11D8-41E4-84B4-5AAA92B1E737",
    "categoryName": "Components, Touring Frames",
    "sku": "FR-T67U-58",
    "name": "LL Touring Frame - Blue, 58",
    "description": "The product called \"LL Touring Frame - Blue, 58\"",
    "price": 333.42,
    "tags": [
        {
            "id": "764C1CC8-2E5F-4EF5-83F6-8FF7441290B3",
            "name": "Tag-190"
        },
        {
            "id": "765EF7D7-331C-42C0-BF23-A3022A723BF7",
            "name": "Tag-191"
        }
    ]
}
```

### Create queries with NoSQL

A basic SQL query in Azure Cosmos DB for NoSQL would be similar to the same query in any other database platform; it would be composed of a few essential components:

- The `SELECT` keyword
- Either an asterisk to indicate all possible fields or an inclusive list of fields
- The `FROM` keyword followed by the data source (container)

Here's a basic query that returns all fields from a container:

```sql
SELECT * FROM products
```

Here's another query that returns only a few fields from a container:

```sql
SELECT 
    products.id, 
    products.name, 
    products.price, 
    products.categoryName 
FROM 
    products
```

One interesting caveat here's that it doesn’t matter what name is used here for the source, as this source references the source moving forward. You can think of this source as a variable. It’s not uncommon to use a single letter from the container name:

```sql
SELECT
    p.name, 
    p.price
FROM 
    p
```

You can use any word or phrase like you would in developer code:

```sql
SELECT
    supercalifragilisticexpialidocious.id,
    supercalifragilisticexpialidocious.categoryId
FROM 
    supercalifragilisticexpialidocious
```

Alternatively, you can alias the data source and use the alias if you prefer:

```sql
SELECT 
    alternativealias.id, 
    alternativealias.name 
FROM 
    reallyinterestingdatasource alternativealias
```

We can also filter our queries using the `WHERE` keyword. In this example, we filter the list of products to those products that have a price that is between 50and100:

```sql
SELECT
    p.name, 
    p.categoryName,
    p.price
FROM 
    products p
WHERE
    p.price >= 50 AND
    p.price <= 100
```
### Project query results

When you develop middle-tier and API applications, there's a tendency to build highly complex solutions to translate database results to something that the business application can understand and use. This workaround often occurs because the database platform is inflexible and must store the data in some fixed schema that can never be changed.

One of the great things about JSON is that it’s compatible with various developer platforms making it highly flexible. Azure Cosmos DB for NoSQL extends the SQL query language by adding functionality to manipulate the JSON results of your query so you can change the query result to map to the schema and shape that your developer team needs.

Let’s look at an example.

In the previous unit, you ran this query:

```sql
SELECT
    p.name, 
    p.categoryName,
    p.price
FROM 
    products p
WHERE
    p.price >= 50 AND
    p.price <= 100
```

And here's an example result:

```json
{
    "name": "LL Bottom Bracket",
    "categoryName": "Components, Bottom Brackets",
    "price": 53.99
}
```

While this result is acceptable, your Python team needs this result mapped to this Python class and doesn't want to write extra code to accomplish this task.

```python
class ProductAdvertisement:
    def __init__(self, name, category, scanner_data):
        self.name = name
        self.category = category
        self.scanner_data = scanner_data

class ScannerData:
    def __init__(self, price):
        self.price = price
```

The first change that could be made is to use a SQL alias to change the **categoryName** property to **category**. This change is accomplished by adding an `AS` keyword to the existing query:

```sql
SELECT
    p.name, 
    p.categoryName AS category,
    p.price
FROM 
    products p
WHERE
    p.price >= 50 AND
    p.price <= 100
```

This new query results in this JSON output:

```json
{
    "name": "LL Bottom Bracket",
    "category": "Components, Bottom Brackets",
    "price": 53.99
}
```

The following change requires us to think about how we want to change the structure of our JSON output. Before changing the query, we need to think about how our JSON object should change. We, essentially, need to create a child JSON object. In this example, we have a child `scannerData` object with a property for `price`:

```json
{
    "name": "LL Bottom Bracket",
    "category": "Components, Bottom Brackets",
    "scannerData": {
        "price": 53.99
    }
}
```

How does this child affect the query? We need to create a field that defines a JSON object with a single property named **price** that references the **p.price** property and an alias of **scannerData**. This expression would look like this:

```sql
{ "price": p.price } AS scannerData
```

Altogether, the entire query looks like this:

```sql
SELECT
    p.name, 
    p.categoryName AS category,
    { "price": p.price } AS scannerData
FROM 
    products p
WHERE
    p.price >= 50 AND
    p.price <= 100
```

#### Reviewing specific properties in query results

Sometimes you want to shape your query results to drill down to specific properties. Two keywords are useful in these scenarios.

First, consider a scenario where you would like to find all of the category names in your container. You could use this query to get all of the container names for every item:

```sql
SELECT
    p.categoryName
FROM
    products p
```

This query returns a JSON result set

Unfortunately, there would be repeated values within the result set:

```json
[
    {
        "categoryName": "Components, Road Frames"
    },
    {
        "categoryName": "Components, Touring Frames"
    },
    {
        "categoryName": "Bikes, Touring Bikes"
    },
    {
        "categoryName": "Clothing, Vests"
    },
    {
        "categoryName": "Accessories, Locks"
    },
    {
        "categoryName": "Components, Pedals"
    },
    {
        "categoryName": "Components, Touring Frames"
    },
...
```

Instead, you can use the `DISTINCT` keyword only to return unique values in the result set.

```sql
SELECT DISTINCT
    p.categoryName
FROM
    products p
```

Let’s consider another scenario. If your Python developers wanted to consume this list of category names, they would need to create a Python wrapper class to consume this list:

```python
class CategoryReader:
    def __init__(self, category_name):
        self.category_name = category_name

# Developers read this as List[CategoryReader]
```

This extra step is both needless and unnecessary. It can quickly become cumbersome as you need to do this multiple times for multiple types in your container[s]. But, if you have a query that returns an object with only a single property, you can use the `VALUE` keyword to flatten the result set to an array of a simple type.

```sql
SELECT DISTINCT VALUE
    p.categoryName
FROM
    products p
```

```json
[
    "Components, Road Frames",
    "Components, Touring Frames",
    "Bikes, Touring Bikes",
    "Clothing, Vests",
    "Accessories, Locks",
    "Components, Pedals",
...
```

```python
# Developers read this as List[str]
```

The `VALUE` keyword can even be used on its own without the `DISTINCT` keyword:


```sql
SELECT VALUE
    p.name
FROM
    products p
```


```json
[
    "LL Road Frame - Red, 60",
    "LL Touring Frame - Blue, 58",
    "Touring-1000 Yellow, 54",
    "Classic Vest, L",
    "Cable Lock",
    "ML Road Pedal",
    "LL Touring Frame - Yellow, 62",
...
```
### Implement type-checking in queries

One of Azure Cosmos DB for NoSQL’s advantages as a data store, is its flexibility to store data with varying structures and shapes. As the developer crafting queries for this data, the responsibility for type checking often falls on your queries. The SQL query language for the NoSQL API includes a suite of built-in functions to make it possible for you to check the types of properties or expressions on the fly when they're variable or unknown.

Up until now, the sample data structure is simple and easy to understand. But let’s consider some possible exceptions.

Each **product** item in the container has a property named **tags**. The tags property is an array of objects with **id** and **name** properties. The assumption, until now, is that the tags array always exists for every product in the container. But if we remove that baseline assumption, we could have a situation where a new product item is inserted into the container without a tag property such as this example:

```json
{
    "id": "6374995F-9A78-43CD-AE0D-5F6041078140",
    "categoryid": "3E4CEACD-D007-46EB-82D7-31F6141752B2",
    "sku": "FR-R38R-60",
    "name": "LL Road Frame - Red, 60",
    "price": 337.22
}
```

First, we can use the `IS_DEFINED` built-in function to check if the **tags** property exists at all in this item:

```sql
SELECT
    IS_DEFINED(p.tags) AS tags_exist
FROM
    products p
```


```json
[
    {
        "tags_exist": false
    }
]
```

Let’s say that the tags property does exist, but it’s not an array; it’s another type of property:

```json
{
    "id": "6374995F-9A78-43CD-AE0D-5F6041078140",
    "categoryid": "3E4CEACD-D007-46EB-82D7-31F6141752B2",
    "sku": "FR-R38R-60",
    "name": "LL Road Frame - Red, 60",
    "price": 337.22,
    "tags": "fun, sporty, rad"
}
```

We can use the `IS_ARRAY` built-in function to check if the tags property is an array:

```sql
SELECT
    IS_ARRAY(p.tags) AS tags_is_array
FROM
    products p
```

We can also check if the tags property is _null_ or not using the `IS_NULL` built-in function:

```sql
SELECT
    IS_NULL(p.tags) AS tags_is_null
FROM
    products p
```

There are even more built-in functions for different scenarios involving other data types.

For example, consider a situation where different data stores persist pricing information inconsistently. Some persist pricing information using string data, while others may store pricing information using numbers. The built-in `IS_NUMBER` function could be used in a WHERE expression of our queries:

```sql
SELECT
    p.id,
    p.price, 
    (p.price * 1.25) AS priceWithTax
FROM
    products p
WHERE
    IS_NUMBER(p.price)
```

We could also use the built-in `IS_STRING` function to see if our price is a string and not apply any formatting:

```sql
SELECT
    p.id,
    p.price
FROM
    products p
WHERE
    IS_STRING(p.price)
```

There are other built-in type checking functions including `IS_OBJECT` and `IS_BOOLEAN`.
### Use built-in functions

The SQL query language for the Azure Cosmos DB for NoSQL ships with built-in functions for common tasks in a query. In this unit, we walk through a brief set of examples of those functions.

Let’s start with an example where the name and the category are concatenated in the query result. For this example, the CONCAT built-in string function is used to concatenate these two fields together with a single vertical bar in the middle:

```sql
SELECT VALUE
    CONCAT(p.name, ' | ', p.categoryName)
FROM
    products p
```

For the next example, the query returns a flattened array with a single field, _sku_. Unfortunately, the _sku_ may, or may not, be in lowercase. To solve for this issue, the LOWER built-in function is used to manipulate the string to all lowercase characters.

```sql
SELECT VALUE 
    LOWER(p.sku) 
FROM 
    products p
```

For this last example, the query is intended to filter out products that shouldn’t be retired yet by using the GetCurrentDateTime built-in function in a WHERE expression:

```sql
SELECT 
    *
FROM
    products p
WHERE
    p.retirementDate >= GetCurrentDateTime()
```

>**Tip**: These examples aren't a comprehensive list of built-in functions for the Azure Cosmos DB for NoSQL query language.

### Execute queries in the SDK

The `azure-cosmos` Python SDK provides efficient methods for querying data using SQL-like syntax, handling pagination, and streaming results back to the client.

To start, let’s use a straightforward SQL command that returns all products:

```sql
SELECT * FROM products p
```

In Python, this query can be passed directly as a string to the `query_items` method.

Define a Python dictionary to represent the structure of your items. For example, consider a product structure like this:

```python
product = {
    "id": "string",
    "name": "string",
    "price": "string"
}
```

Next, use the **query_items** method. The method accepts the SQL query string and options for partition key and other configurations. The results are streamed efficiently using a loop. Within the loop, add your code to handle each item; in this example, each item’s **id**, **name**, and **price** is output to the console:

```python
query = "SELECT * FROM products p"
for item in container.query_items(
    query=query,
    enable_cross_partition_query=True):
    print(f"{item['id']}\t{item['name']}\t{item['price']}")
```

### Execute a query with the Azure Cosmos DB for NoSQL SDK

The latest version of the Python SDK for Azure Cosmos DB for NoSQL simplifies querying a container and iterating over result sets using Python’s modern features.

The `azure-cosmos` library has built-in functionality to make querying Azure Cosmos DB efficient and straightforward.

#### Prepare your development environment

If you have not already cloned the lab code repository for **Build copilots with Azure Cosmos DB** and set up your local environment, view the [Setup local lab environment](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/00-setup-lab-environment.html) instructions to do so.

#### Create an Azure Cosmos DB for NoSQL account

If you already created an Azure Cosmos DB for NoSQL account for the **Build copilots with Azure Cosmos DB** labs on this site, you can use it for this lab and skip ahead to the [next section](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/05-sdk-queries.html#create-azure-cosmos-db-database-and-container-with-sample-data). Otherwise, view the [Setup Azure Cosmos DB](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/common/instructions/00-setup-cosmos-db.html) instructions to create an Azure Cosmos DB for NoSQL account that you will use throughout the lab modules and grant your user identity access to manage data in the account by assigning it to the **Cosmos DB Built-in Data Contributor** role.

#### Create Azure Cosmos DB database and container with sample data

If you already created an Azure Cosmos DB database named **cosmicworks-full** and container within it named **products**, which is preloaded with sample data, you can use it for this lab and skip ahead to the [next section](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/05-sdk-queries.html#install-the-azure-cosmos-library). Otherwise, follow the steps below to create a new sample database and container.

1. Within the newly created **Azure Cosmos DB** account resource, navigate to the **Data Explorer** pane.
    
2. In the **Data Explorer**, select **Launch quick start** on the home page.
    
3. Within the **New Container** form, enter the following values:
    
    - **Database id**: `cosmicworks-full`
    - **Container id**: `products`
    - **Partition key**: `/categoryId`
    - **Analytical store**: `Off`
4. Select **OK** to create the new container. This process will take a minute or two while it creates the resources and preloads the container with sample product data.
    
5. Keep the browser tab open, as we will return to it later.
    
6. Switch back to **Visual Studio Code**.

#### Install the azure-cosmos library

The **azure-cosmos** library is available on **PyPI** for easy installation into your Python projects.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/05-sdk-queries** folder.
    
2. Open the context menu for the **python/05-sdk-queries** folder and then select **Open in Integrated Terminal** to open a new terminal instance.
    
    > 📝 This command will open the terminal with the starting directory already set to the **python/05-sdk-queries** folder.
    
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
    

#### Iterate over the results of a SQL query using the SDK

Using the credentials from the newly created account, you will connect with the SDK classes and connect to the database and container you provisioned in an earlier step, and iterate over the results of a SQL query using the SDK.

You will now use an iterator to create a simple-to-understand loop over paginated results from Azure Cosmos DB. Behind the scenes, the SDK will manage the feed iterator and ensure subsequent requests are invoked correctly.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/05-sdk-queries** folder.
    
2. Open the blank Python file named **script.py**.
    
3. Add the following `import` statements to import the asynchronous **CosmosClient** class, **DefaultAzureCredential** class, and the **asyncio** library:
    
    
    ```python
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    ```
    
4. Add variables named **endpoint** and **credential** and set the **endpoint** value to the **endpoint** of the Azure Cosmos DB account you created earlier. The **credential** variable should be set to a new instance of the **DefaultAzureCredential** class:
    
    
    ```python
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    ```
    
    > 📝 For example, if your endpoint is: **https://dp420.documents.azure.com:443/**, the statement would be: **endpoint = “https://dp420.documents.azure.com:443/”**.
    
5. All interaction with Cosmos DB starts with an instance of the `CosmosClient`. In order to use the asynchronous client, we need to use async/await keywords, which can only be used within async methods. Create a new async method named **main** and add the following code to create a new instance of the asynchronous **CosmosClient** class using the **endpoint** and **credential** variables:
    
    
    ```python
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
    ```
    
    > 💡 Since we’re using the asynchronous **CosmosClient** client, in order to properly use it you also have to warm it up and close it down. We recommend using the `async with` keywords as demonstrated in the code above to start your clients - these keywords create a context manager that automatically warms up, initializes, and cleans up the client, so you don’t have to.
    
6. Add the following code to connect to the database and container you created earlier:
    
    
    ```python
    database = client.get_database_client("cosmicworks-full")
    container = database.get_container_client("products")
    ```
    
7. Create a query string variable named `sql` with a value of `SELECT * FROM products p`.
    
    
    ```python
    sql = "SELECT * FROM products p"
    ```
    
8. Invoke the [`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items) method with the `sql` variable as a parameter to the constructor.
    
    ```python
    result_iterator = container.query_items(
        query=sql
    )
    ```
    
9. The **query_items** method returned an asynchronous iterator that we store in a variable named `result_iterator`. This means that each object from the iterator is an awaitable object and does not yet contain the query result. Add the code below to create an async **for** loop to await each query result as you iterate over the asynchronous iterator and print the `id`, `name`, and `price` of each item.
    
    
    ```python
    # Perform the query asynchronously
    async for item in result_iterator:
        print(f"[{item['id']}]	{item['name']}	${item['price']:.2f}")
    ```
    
10. Underneath the `main` method, add the following code to run the `main` method using the `asyncio` library:
    
    
    ```python
    if __name__ == "__main__":
        asyncio.run(query_items_async())
    ```
    
11. Your **script.py** file should now look like this:
    
    
    ```python
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
    
            database = client.get_database_client("cosmicworks-full")
            container = database.get_container_client("products")
        
            sql = "SELECT * FROM products p"
                
            result_iterator = container.query_items(
                query=sql
            )
                
            # Perform the query asynchronously
            async for item in result_iterator:
                print(f"[{item['id']}]	{item['name']}	${item['price']:.2f}")
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
12. **Save** the **script.py** file.
    
13. Before running the script, you must log into Azure using the `az login` command. At the terminal window, run:
    
    
    ```bash
    az login
    ```
    
14. Run the script to create the database and container:
    
    
    ```bash
    python script.py
    ```
    
15. The script will now output every product in the container.
    

#### Perform a query within a logical partition

In the previous section, you queried all items in the container. By default, the async **CosmosClient** performs cross-partition queries. Because of this, the query you executed (`"SELECT * FROM products p"`) caused the query engine to scan all partitions in the container. As a best practice, you should always query within a logical partition to avoid cross-partition queries. Doing so ultimately saves you money and improves performance.

In this section, you will perform a query within a logical partition by including the partition key in the query.

1. Return to the editor tab for the **script.py** code file.
    
2. Delete the following lines of code:
    
    
    ```python
    result_iterator = container.query_items(
        query=sql
    )
        
    # Perform the query asynchronously
    async for item in result_iterator:
        print(f"[{item['id']}]	{item['name']}	${item['price']:.2f}")
    ```
    
3. Modify the script to create a **partition_key** variable to store the Category ID value for jerseys. Add the **partition_key** as a parameter to the **query_items** method. This ensures that the query is executed within the logical partition for the jerseys category.
    
    
    ```python
    partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"
    
    result_iterator = container.query_items(
        query=sql,
        partition_key=partition_key
    )
    ```
    
4. In the previous section, you performed an async for loop directly on the asynchronous iterator (`async for item in result_iterator:`). This time, you’ll asynchronously create a complete list of the actual query results. This code performs the same action as the for-loop example you previously used. Add the following lines of code to create a list of results and print the results:
    
    
    ```python
    item_list = [item async for item in result_iterator]
    
    for item in item_list:
        print(f"[{item['id']}]	{item['name']}	${item['price']:.2f}")
    ```
    
5. Your **script.py** file should now look like this:
    
    
    ```python
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
    
            database = client.get_database_client("cosmicworks-full")
            container = database.get_container_client("products")
        
            sql = "SELECT * FROM products p"
                
            partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"
    
            result_iterator = container.query_items(
                query=sql,
                partition_key=partition_key
            )
        
            # Perform the query asynchronously
            item_list = [item async for item in result_iterator]
        
            for item in item_list:
                print(f"[{item['id']}]	{item['name']}	${item['price']:.2f}")
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
6. **Save** the **script.py** file.
    
7. Run the script to create the database and container:
    
    
    ```bash
    python script.py
    ```
    
8. The script will now output every product within the jersey category, effectively performing an in-partition query.
    
9. Close the integrated terminal.
    
10. Close **Visual Studio Code**.

### Author complex queries with the Azure Cosmos DB for NoSQL
### Introduction

It's not uncommon for JSON documents to include child documents and arrays. These more complex documents require SQL queries with more complex expressions than ones that are available in the typical SQL language used with typical relational databases. 
### Create cross-product queries

A JOIN in Azure Cosmos DB for NoSQL is different from a JOIN in a relational database as its only scope is a single item. A JOIN creates a cross-product between different sections of a single item.

Let’s take this example JSON object, which has a **name** property and an array with three objects that each have their own **group** property:

```json
{
    "id": "E08E4507-9666-411B-AAC4-519C00596B0A",
    "name": "Men's Bib-Shorts",
    "groups": [
        {
            "group": "accessories"
        },
        {
            "group": "new"
        },
        {
            "group": "sale"
        }
    ]
}

```

If you create a cross-product of the **name** and **group** properties, you create a JSON array with permutations of possible combinations of names and groups, making it easier for your applications to iterate over items in the array:

```json
[
    {
        "name": "Men's Bib-Shorts",
        "group": "accessories"
    },
    {
        "name": "Men's Bib-Shorts",
        "group": "new"
    },
    {
        "name": "Men's Bib-Shorts",
        "group": "sale"
    }
]
```

So, how can you create this type of cross-product in a SQL query? The `JOIN` keyword in Azure Cosmos DB for NoSQL returns all possible combinations of values within two sets. Let’s use a different example JSON object with a more complex group of tags:

```json
{
    "id": "80D3630F-B661-4FD6-A296-CD03BB7A4A0C",
    "categoryId": "629A8F3C-CFB0-4347-8DCC-505A4789876B",
    "categoryName": "Clothing, Vests",
    "sku": "VE-C304-L",
    "name": "Classic Vest, L",
    "description": "A worn brown classic vest that was a trade-in apparel item",
    "price": 32.4,
    "tags": [
        {
            "id": "2CE9DADE-DCAC-436C-9D69-B7C886A01B77",
            "name": "apparel",
            "class": "group"
        },
        {
            "id": "CA170AAD-A5F6-42FF-B115-146FADD87298",
            "name": "worn",
            "class": "trade-in"
        },
        {
            "id": "CA170AAD-A5F6-42FF-B115-146FADD87298",
            "name": "no-damaged",
            "class": "trade-in"
        }
    ]
}
```

The corresponding query for this example is structured like most `SELECT FROM` query, but also includes the `JOIN` keyword, which references the **tags** property and aliases it with the letter **t**. Then, we add the **t.name** to the list of projected fields in the query results:

```sql
SELECT 
    p.id,
    p.name,
    t.name AS tag
FROM 
    products p
JOIN
    t IN p.tags
```

The result of this query is a JSON array that includes three objects for the single JSON item in the container:

```json
[
    {
        "id": "80D3630F-B661-4FD6-A296-CD03BB7A4A0C",
        "name": "Classic Vest, L",
        "tag": "apparel"
    },
    {
        "id": "80D3630F-B661-4FD6-A296-CD03BB7A4A0C",
        "name": "Classic Vest, L",
        "tag": "worn"
    },
    {
        "id": "80D3630F-B661-4FD6-A296-CD03BB7A4A0C",
        "name": "Classic Vest, L",
        "tag": "no-damaged"
    }
]
```
### Implement correlated subqueries

We can optimize JOIN expressions further by writing subqueries to filter the number of array items we want to include in the cross-product set.

Let’s examine the example JSON object again:

```json
{
    "id": "80D3630F-B661-4FD6-A296-CD03BB7A4A0C",
    "categoryId": "629A8F3C-CFB0-4347-8DCC-505A4789876B",
    "categoryName": "Clothing, Vests",
    "sku": "VE-C304-L",
    "name": "Classic Vest, L",
    "description": "A worn brown classic vest that was a trade-in apparel item",
    "price": 32.4,
    "tags": [
        {
            "id": "2CE9DADE-DCAC-436C-9D69-B7C886A01B77",
            "name": "apparel",
            "class": "group"
        },
        {
            "id": "CA170AAD-A5F6-42FF-B115-146FADD87298",
            "name": "worn",
            "class": "trade-in"
        },
        {
            "id": "CA170AAD-A5F6-42FF-B115-146FADD87298",
            "name": "no-damaged",
            "class": "trade-in"
        }
    ]
}
```

In this example, we include tags that are in both classes **trade-in** and **group**. What if we want to filter out the **group** tags?

We can rewrite our JOIN expression by writing a subquery to filter out the group tags using a subquery:

```sql
SELECT VALUE t FROM t IN p.tags WHERE t.class = 'trade-in'
```

If we add this subquery to the entire all-up query, it becomes this query:

```sql
SELECT 
    p.id,
    p.name,
    t.name AS tag
FROM 
    products p
JOIN
    (SELECT VALUE t FROM t IN p.tags WHERE t.class = 'trade-in') AS t
```

Our final JSON result array would then be as follows with one less result in the set:

```json
[
    {
        "id": "80D3630F-B661-4FD6-A296-CD03BB7A4A0C",
        "name": "Classic Vest, L",
        "tag": "worn"
    },
    {
        "id": "80D3630F-B661-4FD6-A296-CD03BB7A4A0C",
        "name": "Classic Vest, L",
        "tag": "no-damaged"
    }
]
```
### Implement variables in queries

We can implement many common cross-product queries on the SDK side and may want to add filters to prevent the queries from exploding in result size and complexity. Using the available classes and methods within the SDK, we can add query parameters to quickly adjust the values in a `WHERE` filter for a SQL query.

Let’s look at an example SQL query that uses a `JOIN` and a `WHERE` filter:

```sql
SELECT 
    p.name,
    t.name AS tag
FROM 
    products p
JOIN
    t IN p.tags
WHERE
    p.price > 500
```

In Python, we would typically create a query definition using the following syntax with the value of **500** hard-coded in a string value:

```python
query_text = "SELECT p.name, t AS tag FROM products p JOIN t IN p.tags WHERE p.price > 500"
query = {"query": query_text}
```

However, using parameters to the query dynamically, we can adjust the `WHERE` filter values at runtime. Here's how to add a parameter using the Python SDK:

```python
query_text = "SELECT p.name, t AS tag FROM products p JOIN t IN p.tags WHERE p.price > @lower"

query = {
    "query": query_text,
    "parameters": [
        {"name": "@lower", "value": 500}
    ]
}
```

You can even use multiple parameters in more complex queries:

```python
query_text = (
    "SELECT p.name, t AS tag "
    "FROM products p JOIN t IN p.tags "
    "WHERE p.price >= @lower AND p.price <= @upper"
)

query = {
    "query": query_text,
    "parameters": [
        {"name": "@lower", "value": 500},
        {"name": "@upper", "value": 1000}
    ]
}
```

#### Benefits of Parameterized Queries

- **Improved Security**: Helps prevent SQL injection attacks.
- **Flexibility**: Allows you to modify query filters dynamically without changing the SQL query structure.
- **Code Reusability**: Makes it easier to reuse query definitions across multiple queries with different parameter values.

### Paginate query results

The Python SDK for Azure Cosmos DB supports asynchronous iteration for retrieving results, but it also allows you to manually paginate through result sets using a feed iterator. In scenarios where you wish to manually control pagination and the size of each page, you can configure these options and retrieve results incrementally.

First, define the SQL query string that you wish to execute.

```python
sql = "SELECT * FROM products WHERE p.price > 500"
```

The `query_items` method allows you to specify options such as `max_item_count` to limit the number of items returned per page.

```python
iterator = container.query_items(
    query=query,
    enable_cross_partition_query=True,
    max_item_count=100  # Set maximum items per page
)
```

The feed iterator contains a **by_page** method that returns an iterator of pages. Each page contains a list of items that can be iterated over. Use a for loop to iterate over each page and another for loop to iterate over each item.

```python
for page in iterator.by_page():
    for product in page:
        print(f"[{product['id']}]	{product['name']}	${product['price']:.2f}")
```
### Paginate cross-product query results with the Azure Cosmos DB for NoSQL SDK

Azure Cosmos DB queries will typically have multiple pages of results. Pagination is done automatically server-side when Azure Cosmos DB cannot return all query results in one single execution. In many applications, you will want to write code using the SDK to process your query results in batches in a performant manner.

#### Prepare your development environment

If you have not already cloned the lab code repository for **Build copilots with Azure Cosmos DB** and set up your local environment, view the [Setup local lab environment](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/00-setup-lab-environment.html) instructions to do so.

#### Create an Azure Cosmos DB for NoSQL account

If you already created an Azure Cosmos DB for NoSQL account for the **Build copilots with Azure Cosmos DB** labs on this site, you can use it for this lab and skip ahead to the [next section](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/06-sdk-pagination.html#create-azure-cosmos-db-database-and-container-with-sample-data). Otherwise, view the [Setup Azure Cosmos DB](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/common/instructions/00-setup-cosmos-db.html) instructions to create an Azure Cosmos DB for NoSQL account that you will use throughout the lab modules and grant your user identity access to manage data in the account by assigning it to the **Cosmos DB Built-in Data Contributor** role.

#### Create Azure Cosmos DB database and container with sample data

If you already created an Azure Cosmos DB database named **cosmicworks-full** and container within it named **products**, which is preloaded with sample data, you can use it for this lab and skip ahead. Otherwise, follow the steps below to create a new sample database and container.

**Click to expand/collapse steps to create database and container with sample data**

#### Install the azure-cosmos library

The **azure-cosmos** library is available on **PyPI** for easy installation into your Python projects.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/06-sdk-pagination** folder.
    
2. Open the context menu for the **python/06-sdk-pagination** folder and then select **Open in Integrated Terminal** to open a new terminal instance.
    
    > 📝 This command will open the terminal with the starting directory already set to the **python/06-sdk-pagination** folder.
    
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
    

#### Paginate through small result sets of a SQL query using the SDK

When processing query results, you must make sure your code progresses through all pages of results and checks to see if any more pages are remaining before making subsequent requests.

1. In **Visual Studio Code**, in the **Explorer** pane, browse to the **python/06-sdk-pagination** folder.
    
2. Open the blank Python file named **script.py**.
    
3. Add the following `import` statements to import the asynchronous **CosmosClient** class, **DefaultAzureCredential** class, and the **asyncio** library:
    
    
    ```python
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    ```
    
4. Add variables named **endpoint** and **credential** and set the **endpoint** value to the **endpoint** of the Azure Cosmos DB account you created earlier. The **credential** variable should be set to a new instance of the **DefaultAzureCredential** class:
    
    
    ```python
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    ```
    
    > 📝 For example, if your endpoint is: **https://dp420.documents.azure.com:443/**, the statement would be: **endpoint = “https://dp420.documents.azure.com:443/”**.
    
5. All interaction with Cosmos DB starts with an instance of the `CosmosClient`. In order to use the asynchronous client, we need to use async/await keywords, which can only be used within async methods. Create a new async method named **main** and add the following code to create a new instance of the asynchronous **CosmosClient** class using the **endpoint** and **credential** variables:
    
    
    ```python
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
    ```
    
    > 💡 Since we’re using the asynchronous **CosmosClient** client, in order to properly use it you also have to warm it up and close it down. We recommend using the `async with` keywords as demonstrated in the code above to start your clients - these keywords create a context manager that automatically warms up, initializes, and cleans up the client, so you don’t have to.
    
6. Add the following code to connect to the database and container you created earlier:
    
    
    ```python
    database = client.get_database_client("cosmicworks-full")
    container = database.get_container_client("products")
    ```
    
7. Create a new variable named **sql** of type _string_ with a value of **SELECT * FROM products WHERE products.price > 500**:
    
    
    ```python
    sql = "SELECT * FROM products WHERE products.price > 500"
    ```
    
8. Invoke the [`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items) method with the `sql` variable as a parameter to the constructor. Set the `max_item_count` to `50` to limit the number of items returned in each page.
    
    ```python
    iterator = container.query_items(
        query=sql,
        max_item_count=50  # Set maximum items per page
    )
    ```
    
9. Create an async **for** loop that asynchronously invokes the [`by_page`](https://learn.microsoft.com/python/api/azure-core/azure.core.paging.itempaged?view=azure-python#azure-core-paging-itempaged-by-page) method on the iterator object. This method returns a page of results each time it is called.
    
    
    ```python
    async for page in iterator.by_page():
    ```
    
10. Within the async **for** loop, asynchronously iterate over the paginated results and print the `id`, `name`, and `price` of each item.
    
    
    ```python
    async for product in page:
        print(f"[{product['id']}]	{product['name']}	${product['price']:.2f}")
    ```
    
11. Underneath the `main` method, add the following code to run the `main` method using the `asyncio` library:
    
    
    ```python
    if __name__ == "__main__":
        asyncio.run(query_items_async())
    ```
    
12. Your **script.py** file should now look like this:
    
    
    ```python
    from azure.cosmos.aio import CosmosClient
    from azure.identity.aio import DefaultAzureCredential
    import asyncio
    
    endpoint = "<cosmos-endpoint>"
    credential = DefaultAzureCredential()
    
    async def main():
        async with CosmosClient(endpoint, credential=credential) as client:
            # Get database and container clients
            database = client.get_database_client("cosmicworks-full")
            container = database.get_container_client("products")
        
            sql = "SELECT * FROM products WHERE products.price > 500"
            
            iterator = container.query_items(
                query=sql,
                max_item_count=50  # Set maximum items per page
            )
            
            async for page in iterator.by_page():
                async for product in page:
                    print(f"[{product['id']}]	{product['name']}	${product['price']:.2f}")
    
    if __name__ == "__main__":
        asyncio.run(main())
    ```
    
13. **Save** the **script.py** file.
    
14. Before running the script, you must log into Azure using the `az login` command. At the terminal window, run:
    
    ```bash
    az login
    ```
    
15. Run the script to create the database and container:
    
    
    ```bash
    python script.py
    ```
    
16. The script will now output pages of 50 items at a time.
    
    > 💡 The query will match hundreds of items in the products container.
    
17. Close the integrated terminal.
    
18. Close **Visual Studio Code**.

### Build Generative AI applications with Azure Cosmos DB
### Introduction

Generative AI applications are transformative tools with the potential to revolutionize productivity and decision-making. These intelligent assistants, powered by artificial intelligence and data analytics, offer contextual insights and automated support. They can streamline complex workflows and, importantly, adapt to user requirements, making them indispensable in the ever-evolving world of AI.

This learning module delves into the intricacies of building Generative AI application using Python, a versatile and widely used programming language, and Azure Cosmos DB for NoSQL, a globally distributed, multi-model database service provided by Microsoft. Integrating Python and Azure Cosmos DB for NoSQL enables developers to create scalable and adaptive Generative AI applications, offering a powerful combination of flexibility and robustness.

#### What are Generative AI applications?

Generative AI applications are advanced AI assistants designed to augment human capabilities and improve productivity by providing intelligent, context-aware support, automating repetitive tasks, and enhancing decision-making processes. For instance, an AI Generative AI application can help in code review and suggest improvements in software development. In customer service, it can handle routine queries, freeing up human agents for more complex issues. In data analysis, it can identify patterns and trends in large datasets. AI Generative AI applications can be employed in diverse fields such as these, and many more.

#### Why use Python?

Python's simplicity and readability make it a popular programming language for AI and machine learning projects. Its extensive libraries and frameworks, such as LangChain, FastAPI, and many others, provide robust tools for developing sophisticated Generative AI applications. Python's versatility allows developers to iterate and experiment quickly, making it a top choice for building AI applications.

#### Azure Cosmos DB for NoSQL

Azure Cosmos DB is a fully managed NoSQL database service that offers high availability, low latency, and seamless scalability. Its ability to handle various data models, including document, key-value, graph, and column-family, makes it a robust backend for sophisticated generative AI applications.

More importantly, when building AI Generative AI applications, Azure Cosmos DB can serve as both a data store and a vector store, a specialized database optimized for storing and retrieving vectors, which are mathematical representations of data points. This functionality seamlessly integrates vector search capabilities within a unified database system. These features make Azure Cosmos DB an excellent platform for implementing retrieval augmented generation (RAG). RAG enhances the capabilities of large language models (LLMs) like OpenAI's GPT-4 and helps solve the problem of using private corporate information with LLMs. These models are trained on vast datasets that are snapshots of public information at a specific point in time, meaning they don't include the latest public data or private corporate information. Furthermore, while LLMs possess broad general knowledge, incorporating a RAG process can help focus their responses more accurately on a specific domain, often required for AI Generative AI applications.
### Configure the Vector Search and storage feature of Azure Cosmos DB NoSQL

Azure Cosmos DB for NoSQL includes a vector search feature, which provides a robust method for storing and querying high-dimensional vectors. This capability is essential for Generative AI applications requiring an integrated vector search capability. Azure Cosmos DB for NoSQL's vector database allows embeddings to be stored, indexed, and queried alongside the original data, meaning each document in your database can contain high-dimensional vectors and traditional schema-free data. Keeping the vector embeddings and the original data they represent together facilitates better multi-modal data operations and enables greater data consistency, scale, and performance. It also simplifies data management, AI application architectures, and the efficiency of vector-based operations.

#### Enable Vector Search in Azure Cosmos DB for NoSQL

Enabling the Vector Search for NoSQL API feature in Azure Cosmos DB can be done via the [Azure portal](https://portal.azure.com/) or the Azure command-line interface (Azure CLI).

In the Azure portal, it's listed under **Features**.

![Screenshot of the Features page for the Azure Cosmos DB NoSQL database is displayed, with the Vector Search for NoSQL API feature highlighted in the features list.](https://learn.microsoft.com/en-us/training/wwl-data-ai/build-generative-ai-applications-with-azure-cosmos-db-nosql/media/2-cosmos-db-for-nosql-features.png)

After reviewing the feature description, you can enable the **Vector Search for NoSQL API** feature to make it available in your account.

![Screenshot of the Vector Search for NoSQL API enrollment dialog.](https://learn.microsoft.com/en-us/training/wwl-data-ai/build-generative-ai-applications-with-azure-cosmos-db-nosql/media/2-enable-vector-search-for-nosql-api.png)

Alternatively, you can enable Vector Search via the Azure CLI by executing the following command:

```bash
az cosmosdb update \
 --resource-group <resource-group-name> \
 --name <cosmos-db-account-name> \
 --capabilities EnableNoSQLVectorSearch
```

#### Define container vector policy

Once the Vector Search feature is enabled on your Azure Cosmos DB for NoSQL account, you must define a vector embedding policy for the containers where you want to store vectors. This policy informs the Azure Cosmos DB query engine how to handle vector properties in the `VectorDistance` system function. It also instructs the vector indexing policy of necessary details, should you specify one.

> 📝 The vector search feature is currently not supported on the existing containers, so you need to create a new container and specify the container-level vector embedding policy and the vector indexing policy at the time of container creation.

The following information is included in the container vector policy:

- `path`: The path of the property containing the vector embeddings.
- `datatype`: The type of the elements in the vector. The default is `Float32`.
- `dimensions`: This property is the number of dimensions in or length of each vector and will be driven by the model used to create embeddings.
- `distanceFunction`: The technique used to compute distance or similarity between vectors. The available options are Euclidean (default), cosine, and dot product.

In vector search, distance functions determine how similar or different vectors are within a multidimensional space. Popular methods include Euclidean distance, cosine similarity, and Manhattan distance, each offering unique ways to compare the vectors.

#### Improve vector search efficiency with vector indexing

Azure Cosmos DB for NoSQL supports the creation of vector search indexes on top of stored embeddings. A vector search index is a container of vectors in latent space that enables a semantic similarity search across all data (vectors) contained within. Vector indexes are specialized for high-dimensional vector data, increasing the efficiency of performing vector searches using the `VectorDistance` system function in Azure Cosmos DB for NoSQL. The following types of vector index policies are supported:

|Type|Description|Max dimensions|
|---|---|---|
|flat|Stores vectors on the same index as other indexed properties.|505|
|quantizedFlat|Quantizes (compresses) vectors before storing on the index. This policy can improve latency and throughput at the cost of a small amount of accuracy.|4096|
|diskANN|Creates an index based on DiskANN for fast and efficient approximate search.|4096|

Vector searches reduced latency, increase throughput, and decreased RU consumption when using a vector index. A vector index search allows for a prompt preprocessing step where information can be semantically retrieved from an index and then used to generate a factually accurate prompt for the LLM (Large Language Model) to reason over. This process provides knowledge augmentation and domain-specific focus to an LLM. The vector search index returns a list of documents whose vector field is semantically similar to the incoming message. The original text stored within the same document augments the LLM's prompt, which the LLM then uses to respond to the requestor based on the information provided.

### Enable Vector Search for Azure Cosmos DB for NoSQL

Azure Cosmos DB for NoSQL provides an efficient vector indexing and search capability designed to store and query high-dimensional vectors efficiently and accurately at any scale. To take advantage of this capability, you must enable your account to use the _Vector Search for NoSQL API_ feature.

#### Prepare your development environment

If you have not already cloned the lab code repository for **Build copilots with Azure Cosmos DB** and set up your local environment, view the [Setup local lab environment](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/00-setup-lab-environment.html) instructions to do so.

#### Create an Azure Cosmos DB for NoSQL account

If you already created an Azure Cosmos DB for NoSQL account for the **Build copilots with Azure Cosmos DB** labs on this site, you can use it for this lab and skip ahead to the [next section](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/07-01-enable-azure-cosmos-db-nosql-vector-search.html#enable-vector-search-for-nosql-api). Otherwise, view the [Setup Azure Cosmos DB](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/common/instructions/00-setup-cosmos-db.html) instructions to create an Azure Cosmos DB for NoSQL account that you will use throughout the lab modules and grant your user identity access to manage data in the account by assigning it to the **Cosmos DB Built-in Data Contributor** role.

#### Enable Vector Search for NoSQL API

In this task, you will enable the _Vector Search for NoSQL API_ feature in your Azure Cosmos DB account using the Azure CLI.

1. From the toolbar in the [Azure portal](https://portal.azure.com/), open a Cloud Shell.
    
    [![The Cloud Shell icon is highlighted on the Azure portal's toolbar.](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/media/07-azure-portal-toolbar-cloud-shell.png)](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/media/07-azure-portal-toolbar-cloud-shell.png)
    
2. At the Cloud Shell prompt, ensure your exercise subscription is used for subsequent commands by running `az account set -s <SUBSCRIPTION_ID>`, replacing the `<SUBSCRIPTION_ID>` placeholder token with the id of the subscription you are using for this exercise.
    
3. Enable the _Vector Search for NoSQL API_ feature by executing the following command from the Azure Cloud Shell, replacing the `<RESOURCE_GROUP_NAME>` and `<COSMOS_DB_ACCOUNT_NAME>` tokens with the name of your resource group and Azure Cosmos DB account name, respectively.
    
    ```bash
      az cosmosdb update \
        --resource-group <RESOURCE_GROUP_NAME> \
        --name <COSMOS_DB_ACCOUNT_NAME> \
        --capabilities EnableNoSQLVectorSearch
    ```
    
4. Wait for the command to run successfully before exiting the Cloud Shell.
    
5. Close the Cloud Shell.
    

#### Create a database and container for hosting vectors

1. Select **Data Explorer** from the left-hand menu of your Azure Cosmos DB account in the [Azure portal](https://portal.azure.com/), then select **New Container**.
    
2. In the **New Container** dialog:
    
    1. Under **Database id**, select **Create new** and enter “CosmicWorks” into the database id field.
    2. In the **Container id** box, enter the name “Products.”
    3. Assign “/category_id” as the **Partition key.**
        
        [![Screenshot of the New Container settings specified above entered into the dialog.](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/media/07-azure-cosmos-db-new-container.png)](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/media/07-azure-cosmos-db-new-container.png)
        
    4. Scroll to the bottom of the **New Container** dialog, expand **Container Vector Policy**, and select **Add vector embedding**.
        
    5. In the **Container Vector Policy** settings section, set the following:
        
        |Setting|Value|
        |---|---|
        |**Path**|Enter _/embedding_.|
        |**Data type**|Select _float32_.|
        |**Distance function**|Select _cosine_.|
        |**Dimensions**|Enter _1536_ to match the number of dimensions produced by OpenAI’s `text-embedding-3-small` model.|
        |**Index type**|Select _diskANN_.|
        |**Quantization byte size**|Leave this blank.|
        |**Indexing search list size**|Accept the default value of _100_.|
        
        [![Screenshot of the Container Vector Policy specified above entered into the New Container dialog.](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/media/07-azure-cosmos-db-container-vector-policy.png)](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/media/07-azure-cosmos-db-container-vector-policy.png)
        
    6. Select **OK** to create the database and container.
        
    7. Wait for the container to be created before proceeding. It may take several minutes for the container to be ready.
### Generate embeddings using Azure OpenAI Service

Vectors, also known as embeddings or vector embeddings, are mathematical representations of data in a high-dimensional space. Each dimension corresponds to a feature of the data in this space, and tens of thousands of dimensions might be used to represent sophisticated data. A vector's position in this space represents its characteristics. Words, phrases, or entire documents, and images, audio, and other types of data, can all be vectorized. Because data is represented as a vector, vector search can identify matching data across different data types.

An embedding is a data representation format that machine learning models and algorithms can efficiently utilize. The embedding is an information-dense representation of the semantic meaning of a piece of text. Each embedding is a vector of floating point numbers. Hence, the distance between two embeddings in the vector space correlates with the semantic similarity between two inputs in the original format.

#### Generate embeddings with Azure OpenAI

Azure OpenAI is a cutting-edge service that integrates OpenAI's advanced language models with Microsoft's Azure platform, providing a secure and scalable environment for developers. This powerful combination enables the creation of intelligent applications that can understand and generate human-like text, essential for tasks like natural language processing, text summarization, and sentiment analysis. A fundamental application of Azure OpenAI is building sophisticated virtual assistants, or Generative AI applications, capable of handling complex queries, offering personalized responses, and seamlessly integrating with other services. By using an embedding model, developers can generate vector representations of textual data and store them in a vector store like Azure Cosmos DB for NoSQL. This approach facilitates efficient and accurate similarity searches, significantly enhancing the Generative AI application's ability to retrieve relevant information and provide contextually rich interactions.

![Diagram of using an embedding model to generate vectors and store them in Azure Cosmos DB for NoSQL.](https://learn.microsoft.com/en-us/training/wwl-data-ai/build-generative-ai-applications-with-azure-cosmos-db-nosql/media/4-vector-embedding-storage-with-azure-cosmos-db-nosql.png)

Embeddings are generated by sending data to an embedding model, where it transforms into a vector. Azure OpenAI provides several models for creating embeddings, including the `text-embedding-ada-002`, `text-embedding-3-small`, and `text-embedding-3-large` models. The Azure OpenAI SDK for Python can be used to create an Azure OpenAI client, which provides a method for producing embeddings.

```python
from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from openai import AzureOpenAI

# Enable Microsoft Entra ID RBAC authentication
credential = DefaultAzureCredential()
token_provider = get_bearer_token_provider(
 credential,
    "https://cognitiveservices.azure.com/.default"
)

# Instantiate an Azure OpenAI client
client = AzureOpenAI(
    api_version = AZURE_OPENAI_API_VERSION,
    azure_endpoint = AZURE_OPENAI_ENDPOINT,
    azure_ad_token_provider = token_provider
)

# Generate embeddings for input text
response = client.embeddings.create(
    input = "Build Generative AI applications with Python and Azure Cosmos DB for NoSQL",
    model = "text-embedding-3-large"
)

# Retrieve the generated embedding
embedding = response.data[0].embedding
```

Knowing the number of dimensions the model produces is crucial when defining a container vector policy in Azure Cosmos DB for NoSQL. It also plays a role in selecting an appropriate vector indexing policy. The dimensionality of the vectors created is dictated by the model used to generate them. In the previous example, the `text-embedding-3-large` model was used, which, by default, created vectors containing 3,072 dimensions. When defining a container vector policy, you must specify that number in the **Dimensions** property. Likewise, you need to choose an index type that supports the number of dimensions used.

Once created, the embeddings can be stored in a vector database, such as Azure Cosmos DB for NoSQL.

#### Common development patterns for generating and storing vectors

Integrating Azure OpenAI and Azure Cosmos DB for NoSQL to generate and store vector embeddings typically requires an automated worker process that ensures efficiency, real-time processing, and seamless integration. Some common patterns that are used to accomplish this integration include:

1. **Azure Function with Cosmos DB Change Feed Trigger**
    
    An effective pattern for generating embeddings as data is inserted into Cosmos DB involves using an Azure Function in combination with the Cosmos DB change feed. The function's Cosmos DB trigger is invoked when documents are inserted or updated in the container. The function calls an Azure OpenAI embedding model to create vectors, updates the documents with these embeddings, and writes them back into Cosmos DB. This approach ensures real-time processing and immediate availability of vector embeddings for subsequent searches.
    
2. **Batch Processing with Azure Data Factory**
    
    Batch processing can be an efficient approach for bulk data operations on documents, whether they already exist in a Cosmos DB container or reside in another data store. You can use Azure Data Factory to orchestrate the process of generating vector embeddings. Data Factory can extract data from its source, send it to Azure OpenAI for embedding generation, and then write the enriched data into Cosmos DB. This method is beneficial for initial data loading or periodic updates where real-time processing isn't critical.
    
3. **Microservices Architecture**
    
    A microservices architecture can provide a solution that offers more granular control of the processes involved. Embedding generation can be encapsulated within a dedicated microservice. This service interacts with Azure OpenAI and Cosmos DB, generating vector embeddings as needed. Other services can call this microservice whenever they need to update or retrieve embeddings, ensuring a modular and maintainable system design.
    

By adopting these patterns, your applications can efficiently integrate vector embeddings, using the capabilities of Azure OpenAI and Cosmos DB for NoSQL to enhance search and data analysis.

### Generate vector embeddings with Azure OpenAI and store them in Azure Cosmos DB for NoSQL

Azure OpenAI provides access to OpenAI’s advanced language models, including the `text-embedding-ada-002`, `text-embedding-3-small`, and `text-embedding-3-large` models. By leveraging one of these models, you can generate vector representations of textual data, which can be stored in a vector store like Azure Cosmos DB for NoSQL. This facilitates efficient and accurate similarity searches, significantly enhancing a copilot’s ability to retrieve relevant information and provide contextually rich interactions.

In this lab, you will create an Azure OpenAI service and deploy an embedding model. You will then use Python code to create Azure OpenAI and Cosmos DB clients using their respective Python SDKs to generate vector representations of product descriptions and write them into your database.

> 🛑 The previous exercise in this module is a prerequisite for this lab. If you still need to complete that exercise, please finish it before continuing, as it provides the infrastructure required for this lab.

#### Create an Azure OpenAI service

Azure OpenAI provides REST API access to OpenAI’s powerful language models. These models can be easily adapted to your specific task including but not limited to content generation, summarization, image understanding, semantic search, and natural language to code translation.

1. In a new web browser window or tab, navigate to the Azure portal (`portal.azure.com`).
    
2. Sign into the portal using the Microsoft credentials associated with your subscription.
    
3. Select **Create a resource**, search for _Azure OpenAI_, and then create a new **Azure OpenAI** resource with the following settings, leaving all remaining settings to their default values:
    
    |Setting|Value|
    |---|---|
    |**Subscription**|_Your existing Azure subscription_|
    |**Resource group**|_Select an existing or create a new resource group_|
    |**Region**|_Choose an available region that supports the `text-embedding-3-small` model_ from the [list of supporting regions](https://learn.microsoft.com/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-embeddings#tabpanel_3_standard-embeddings).|
    |**Name**|_Enter a globally unique name_|
    |**Pricing Tier**|_Choose Standard 0_|
    
    > 📝 Your lab environments may have restrictions preventing you from creating a new resource group. If that is the case, use the existing pre-created resource group.
    
4. Wait for the deployment task to complete before continuing with the next task.
    

#### Deploy an embedding model

To use Azure OpenAI to generate embeddings, you must first deploy an instance of the desired embedding model within your service.

1. Navigate to your newly created Azure OpenAI service in the Azure portal (`portal.azure.com`).
    
2. On the **Overview** page of the Azure OpenAI service, launch **Azure AI Foundry** by selecting the **Go to Azure AI Foundry portal** link on the toolbar.
    
3. In Azure AI Foundry, select **Deployments** from the left-hand menu.
    
4. On the **Model deployments** page, select **Deploy model** and select **Deploy base model** from the dropdown.
    
5. From the list of models, select `text-embedding-3-small`.
    
    > 💡 You can filter the list to display only _Embeddings_ models using the inference tasks filter.
    
    > 📝 If you do not see the `text-embedding-3-small` model, you may have selected an Azure region that does not currently support that model. In this case, you can use the `text-embedding-ada-002` model for this lab. Both models generate vectors with 1536 dimensions, so no changes are required to the container vector policy you defined on the `Products` container in Azure Cosmos DB.
    
6. Select **Confirm** to deploy the model.
    
7. On the **Model deployments** page in Azure AI Foundry, note the **Name** of the `text-embedding-3-small` model deployment, as you will need this later in this exercise.
    

#### Deploy a chat completion model

In addition to the embedding model, you will need a chat completion model for your copilot. You will use OpenAI’s `gpt-4o` large language model to generate responses from your copilot.

1. While still on the **Model deployments** page in Azure AI Foundry, select the **Deploy model** button again and choose **Deploy base model** from the dropdown.
    
2. Select the **gpt-4o** chat completion model from the list.
    
3. Select **Confirm** to deploy the model.
    
4. On the **Model deployments** page in Azure AI Foundry, note the **Name** of the `gpt-4o` model deployment, as you will need this later in this exercise.
    

#### Assign the Cognitive Services OpenAI User RBAC role

To allow your user identity to interact with the Azure OpenAI service, you can assign your account the **Cognitive Services OpenAI User** role. Azure OpenAI Service supports Azure role-based access control (Azure RBAC), an authorization system for managing individual access to Azure resources. Using Azure RBAC, you assign different team members different levels of permissions based on their needs for a given project.

> 📝 Microsoft Entra ID’s Role-Based Access Control (RBAC) for authenticating against Azure services like Azure OpenAI enhances security through precise access controls tailored to user roles, effectively reducing unauthorized access risks. Streamlining secure access management using Entra ID RBAC makes a more efficient and scalable solution for leveraging Azure services.

1. In the Azure portal (`portal.azure.com`), navigate to your Azure OpenAI resource.
    
2. Select **Access Control (IAM)** on the left navigation pane.
    
3. Select **Add**, then select **Add role assignment**.
    
4. On the **Role** tab, select the **Cognitive Services OpenAI User** role, then select **Next**.
    
5. On the **Members** tab, select assign access to a user, group, or service principal, and select **Select members**.
    
6. In the **Select members** dialog, search for your name or email address and select your account.
    
7. On the **Review + assign** tab, select **Review + assign** to assign the role.
    

#### Create a Python virtual environment

Virtual environments in Python are essential for maintaining a clean and organized development space, allowing individual projects to have their own set of dependencies, isolated from others. This prevents conflicts between different projects and ensures consistency in your development workflow. By using virtual environments, you can manage package versions easily, avoid dependency clashes, and keep your projects running smoothly. It’s a best practice that keeps your coding environment stable and dependable, making your development process more efficient and less prone to issues.

1. Using Visual Studio Code, open the folder into which you cloned the lab code repository for **Build copilots with Azure Cosmos DB** learning module.
    
2. In Visual Studio Code, open a new terminal window and change directories to the `python/07-build-copilot` folder.
    
3. Create a virtual environment named `.venv` by running the following command at the terminal prompt:
    
    
    ```bash
     python -m venv .venv 
    ```
    
    The above command will create a `.venv` folder under the `07-build-copilot` folder, which will provide a dedicated Python environment for the exercises in this lab.
    
4. Activate the virtual environment by selecting the appropriate command for your OS and shell from the table below and executing it at the terminal prompt.
    
    |Platform|Shell|Command to activate virtual environment|
    |---|---|---|
    |POSIX|bash/zsh|`source .venv/bin/activate`|
    ||fish|`source .venv/bin/activate.fish`|
    ||csh/tcsh|`source .venv/bin/activate.csh`|
    ||pwsh|`.venv/bin/Activate.ps1`|
    |Windows|cmd.exe|`.venv\Scripts\activate.bat`|
    ||PowerShell|`.venv\Scripts\Activate.ps1`|
    
5. Install the libraries defined in `requirements.txt`:
    
    
    ```bash
     pip install -r requirements.txt
    ```
    
    The `requirements.txt` file contains a set of Python libraries you will use throughout this lab.
    
    |Library|Version|Description|
    |---|---|---|
    |`azure-cosmos`|4.9.0|Azure Cosmos DB SDK for Python - Client library|
    |`azure-identity`|1.19.0|Azure Identity SDK for Python|
    |`fastapi`|0.115.5|Web framework for building APIs with Python|
    |`openai`|1.55.2|Provides access to the Azure OpenAI REST API from Python apps.|
    |`pydantic`|2.10.2|Data validation using Python type hints.|
    |`requests`|2.32.3|Send HTTP requests.|
    |`streamlit`|1.40.2|Transforms Python scripts into interactive web apps.|
    |`uvicorn`|0.32.1|An ASGI web server implementation for Python.|
    |`httpx`|0.27.2|A next-generation HTTP client for Python.|
    

#### Add a Python function to vectorize text

The Python SDK for Azure OpenAI provides access to both synchronous and asynchronous classes that can be used to create embeddings for textual data. This functionality can be encapsulated in a function in your Python code.

1. In the **Explorer** pane within Visual Studio Code, navigate to the `python/07-build-copilot/api/app` folder and open the `main.py` file located within it.
    
    > 📝 This file will serve as the entry point to a backend Python API you will build in the next exercise. In this exercise, you will provide a handful of async functions that can be used to import data with embeddings into Azure Cosmos DB that will be leveraged by the API.
    
2. To use the asychronous Azure OpenAI SDK for Python, import the library by adding the following code to the top of the `main.py` file:
    
    ```python
    from openai import AsyncAzureOpenAI
    ```
    
3. You will be accessing Azure OpenAI and Cosmos DB asynchronously using Azure authentication and the Entra ID RBAC roles you previously assigned to your user identity. Add the following line below the `openai` import statement at the top of the file to import the required classes from the `azure-identity` library:
    
    ```python
    from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    ```
    
    > 📝 To ensure you can securely interact with Azure services from your API, you will use the Azure Identity SDK for Python. This approach allows you to avoid having to store or interact with keys from code, instead leveraging the RBAC roles you assigned to your account for access to Azure Cosmos DB and Azure OpenAI in the previous exercises.
    
4. Create variables to store the Azure OpenAI API version and endpoint, replacing the `<AZURE_OPENAI_ENDPOINT>` token with the endpoint value for your Azure OpenAI service. Also, create a variable for the name of your embedding model deployment. Insert the following code below the `import` statements in the file:
    
    ```python
    # Azure OpenAI configuration
    AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
    AZURE_OPENAI_API_VERSION = "2024-10-21"
    EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    ```
    
    If your embedding deployment name differs, update the value assigned to the variable accordingly.
    
    > 💡 The API version of `2024-10-21` was the latest GA release version as of the time of this writing. You can use that or a new version, if one is available. The API specs documentation contains a [table with the latest API versions](https://learn.microsoft.com/azure/ai-services/openai/reference#api-specs).
    
    > 📝 The `EMBEDDING_DEPLOYMENT_NAME` is the **Name** value you noted after deploying the `text-embedding-3-small` model in Azure AI Foundry. If you need to refer back to it, launch Azure AI Foundry, navigate to the **Deployments** page and locate the deployment whose **Model name** is `text-embedding-3-small`. Then, copy the **Name** field value of that item. If you deployed the `text-embedding-ada-002` model, use the name for that deployment.
    
5. Use the Azure Identity SDK for Python’s `DefaultAzureCredential` class to create an asynchronous credential for accessing Azure OpenAI and Azure Cosmos DB using Microsoft Entra ID RBAC authentication by inserting the following code below the variable declarations:
    
    ```python
    # Enable Microsoft Entra ID RBAC authentication
    credential = DefaultAzureCredential()
    ```
    
6. To handle the creation of embeddings, insert the following, which adds a function to generate embeddings using an Azure OpenAI client:
    
    
    ```python
    async def generate_embeddings(text: str):
        """Generates embeddings for the provided text."""
        # Create an async Azure OpenAI client
        async with AsyncAzureOpenAI(
            api_version = AZURE_OPENAI_API_VERSION,
            azure_endpoint = AZURE_OPENAI_ENDPOINT,
            azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
        ) as client:
            response = await client.embeddings.create(
                input = text,
                model = EMBEDDING_DEPLOYMENT_NAME
            )
            return response.data[0].embedding
    ```
    
    Creation of the Azure OpenAI client does not require the `api_key` value because it is retrieving a bearer token using the Azure Identity SDK’s `get_bearer_token_provider` class.
    
7. The `main.py` file should now look similar to the following:
    
    ```python
    from openai import AsyncAzureOpenAI
    from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
        
    # Azure OpenAI configuration
    AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
    AZURE_OPENAI_API_VERSION = "2024-10-21"
    EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
        
    # Enable Microsoft Entra ID RBAC authentication
    credential = DefaultAzureCredential()
        
    async def generate_embeddings(text: str):
        """Generates embeddings for the provided text."""
        # Create an async Azure OpenAI client
        async with AsyncAzureOpenAI(
            api_version = AZURE_OPENAI_API_VERSION,
            azure_endpoint = AZURE_OPENAI_ENDPOINT,
            azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
        ) as client:
            response = await client.embeddings.create(
                input = text,
                model = EMBEDDING_DEPLOYMENT_NAME
            )
            return response.data[0].embedding
    ```
    
8. Save the `main.py` file.
    

#### Test the embedding function

To ensure the `generate_embeddings` function in the `main.py` file is working correctly, you will add a few lines of code at the bottom of the file to allow it to be run directly. These lines allow you to execute the `generate_embeddings` function from the command line, passing in the text to embed.

1. Add a **main guard** block containing a call to `generate_embeddings` at the bottom of the `main.py` file:
    
    ```python
    if __name__ == "__main__":
        import asyncio
        import sys
        
        async def main():
            print(await generate_embeddings(sys.argv[1]))
            # Close open async credential sessions
            await credential.close()
            
        asyncio.run(main())
    ```
    
    > 📝 The `if __name__ == "__main__":` block is commonly referred to as the **main guard** or **entry point** in Python. It ensures that certain code is only executed when the script is run directly, and not when it is imported as a module in another script. This practice helps in organizing code and makes it more reusable and modular.
    
2. Save the `main.py` file, which should now look like:
    
    ```python
    from openai import AsyncAzureOpenAI
    from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
        
    # Azure OpenAI configuration
    AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
    AZURE_OPENAI_API_VERSION = "2024-10-21"
    EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
        
    # Enable Microsoft Entra ID RBAC authentication
    credential = DefaultAzureCredential()
        
    async def generate_embeddings(text: str):
        """Generates embeddings for the provided text."""
        # Create an async Azure OpenAI client
        async with AsyncAzureOpenAI(
            api_version = AZURE_OPENAI_API_VERSION,
            azure_endpoint = AZURE_OPENAI_ENDPOINT,
            azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
        ) as client:
            response = await client.embeddings.create(
                input = text,
                model = EMBEDDING_DEPLOYMENT_NAME
            )
            return response.data[0].embedding
    
    if __name__ == "__main__":
        import asyncio
        import sys
        
        async def main():
            print(await generate_embeddings(sys.argv[1]))
            # Close open async credential sessions
            await credential.close()
            
        asyncio.run(main())
    ```
    
3. In Visual Studio Code, open a new integrated terminal window.
    
4. Before running the API, which will send requests to Azure OpenAI, you must log into Azure using the `az login` command. At the terminal window, run:
    
    
    ```bash
    az login
    ```
    
5. Complete the login process in your browser.
    
6. At the terminal prompt, change directories to `python/07-build-copilot`.
    
7. Ensure the intgrated terminal window is running within your Python virutal environment by activating your virtual environment using a command from the table below, selecting the appropriate command for your OS and shell.
    
    |Platform|Shell|Command to activate virtual environment|
    |---|---|---|
    |POSIX|bash/zsh|`source .venv/bin/activate`|
    ||fish|`source .venv/bin/activate.fish`|
    ||csh/tcsh|`source .venv/bin/activate.csh`|
    ||pwsh|`.venv/bin/Activate.ps1`|
    |Windows|cmd.exe|`.venv\Scripts\activate.bat`|
    ||PowerShell|`.venv\Scripts\Activate.ps1`|
    
8. At the terminal prompt, change directories to `api/app`, then execute the following command:
    
    ```python
    python main.py "Hello, world!"
    ```
    
9. Observe the output in the terminal window. You should see an array of floating point number, which is the vector representation of the “Hello, world!” string. It should look similiar to the following abbreviated output:
    
    ```bash
    [-0.019184619188308716, -0.025279032066464424, -0.0017195191467180848, 0.01884828321635723...]
    ```
    

#### Build a function for writing data to Azure Cosmos DB

Using the Azure Cosmos DB SDK for Python, you can create a function that allows upserting documents into your database. An upsert operation will update a record if a match is found and insert a new record if one is not.

1. Return to the open `main.py` file in Visual Studio Code and import the async `CosmosClient` class from the Azure Cosmos DB SDK for Python by inserting the following line just below the `import` statements already in the file:
    
    
    ```python
    from azure.cosmos.aio import CosmosClient
    ```
    
2. Add another import statement to reference the `Product` class from the _models_ module in the `api/app` folder. The `Product` class defines the shape of products in the Cosmic Works dataset.
    
    
    ```python
    from models import Product
    ```
    
3. Create a new group of variables containing configuration values associated with Azure Cosmos DB and add them to the `main.py` file below the Azure OpenAI variables you inserted previously. Ensure you replace the `<AZURE_COSMOSDB_ENDPOINT>` token with the endpoint for your Azure Cosmos DB account.
    
    ```python
    # Azure Cosmos DB configuration
    AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
    DATABASE_NAME = "CosmicWorks"
    CONTAINER_NAME = "Products"
    ```
    
4. Add a function named `upsert_product` for upserting (update or insert) documents into Cosmos DB, inserting the following code below the `generate_embeddings` function in the `main.py` file:
    
    ```python
    async def upsert_product(product: Product):
        """Upserts the provided product to the Cosmos DB container."""
        # Create an async Cosmos DB client
        async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
            # Load the CosmicWorks database
            database = client.get_database_client(DATABASE_NAME)
            # Retrieve the product container
            container = database.get_container_client(CONTAINER_NAME)
            # Upsert the product
            await container.upsert_item(product)
    ```
    
5. Save the `main.py` file, which should now look like:
    
    ```python
    from openai import AsyncAzureOpenAI
    from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    from azure.cosmos.aio import CosmosClient
    from models import Product
        
    # Azure OpenAI configuration
    AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
    AZURE_OPENAI_API_VERSION = "2024-10-21"
    EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
    # Azure Cosmos DB configuration
    AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
    DATABASE_NAME = "CosmicWorks"
    CONTAINER_NAME = "Products"
        
    # Enable Microsoft Entra ID RBAC authentication
    credential = DefaultAzureCredential()
        
    async def generate_embeddings(text: str):
        """Generates embeddings for the provided text."""
        # Create an async Azure OpenAI client
        async with AsyncAzureOpenAI(
            api_version = AZURE_OPENAI_API_VERSION,
            azure_endpoint = AZURE_OPENAI_ENDPOINT,
            azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
        ) as client:
            response = await client.embeddings.create(
                input = text,
                model = EMBEDDING_DEPLOYMENT_NAME
            )
            return response.data[0].embedding
    
    async def upsert_product(product: Product):
        """Upserts the provided product to the Cosmos DB container."""
        # Create an async Cosmos DB client
        async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
            # Load the CosmicWorks database
            database = client.get_database_client(DATABASE_NAME)
            # Retrieve the product container
            container = database.get_container_client(CONTAINER_NAME)
            # Upsert the product
            await container.upsert_item(product)
        
    if __name__ == "__main__":
        import sys
        print(generate_embeddings(sys.argv[1]))
    ```
    

#### Vectorize sample data

You are now ready to test both the `generate_embeddings` and `upsert_document` functions together. To do this, you will overwrite the `if __name__ == "__main__"` main guard block with code that downloads a sample data file containing Cosmic Works product information from GitHub and then vectorizes the `description` field of each product, and upserts the documents into the `Products` container in your Azure Cosmos DB database.

> 📝 This approach is being used to demonstrate the techniques for generating with Azure OpenAI and storing embeddings in Azure Cosmos DB. In a real-world scenario, however, a more robust approach, such as using an Azure Function triggered by the Azure Cosmos DB change feed would be more appropiate for handling adding embeddings to existing and new documents.

1. In the `main.py` file, overwrite the `if __name__ == "__main__":` code block with the following:
    
    ```python
    if __name__ == "__main__":
        import asyncio
        from models import Product
        import requests
        
        async def main():
            product_raw_data = "https://raw.githubusercontent.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs/refs/heads/main/data/07/products.json?v=1"
            products = [Product(**data) for data in requests.get(product_raw_data).json()]
        
            # Call the generate_embeddings function, passing in an argument from the command line.    
            for product in products:
                print(f"Generating embeddings for product: {product.name}", end="\r")
                product.embedding = await generate_embeddings(product.description)
                await upsert_product(product.model_dump())
        
            print("All products with vectorized descriptions have been upserted to the Cosmos DB container.")
            # Close open credential sessions
            await credential.close()
        
        asyncio.run(main())
    ```
    
2. At the open integrated terminal prompt in Visual Studio Code, run the `main.py` file again using the command:
    
    
    ```python
    python main.py
    ```
    
3. Wait for the code execution to complete, indicated by a message indicating all products with vectorized descriptions have been upserted to the Cosmos DB container. It may take up to 10 - 15 minutes for the vectorization and data upsert process to complete for the 295 records in the products dataset. If not all products are inserted, you can rerun `main.py` using the command above to add the remaining products.
    

#### Review upserted sample data in Cosmos DB

1. Return to the Azure portal (`portal.azure.com`) and navigate to your Azure Cosmos DB account.
    
2. Select the **Data Explorer** for the left navigation menu.
    
3. Expand the **CosmicWorks** database and **Products** container, then select **Items** under the container.
    
4. Select several random documents within the container and ensure the `embedding` field is populated with a vector array.

### Build Generative AI applications with Azure Cosmos DB NoSQL and Python

Generative AI applications, the next generation of intelligent assistants, are redefining productivity by providing context-aware support, enhancing decision-making processes, and automating complex workflows. With Azure Cosmos DB for NoSQL Python vector search, developers can build advanced Generative AI applications that deliver precise and efficient solutions by integrating the power of Python and its vast assortment of libraries and tools. Python's versatility in data manipulation integrated with the robust vector search capabilities of Azure Cosmos DB allows Generative AI applications to handle complex data queries and provide real-time insights efficiently. Additionally, vector search plays a crucial role when implementing a Retrieval-Augmented Generation (RAG) pattern. It enables the AI to retrieve the most relevant documents from a vast corpus based on similarity to the input query, thus enhancing the generation of accurate and contextually relevant responses. This synergy allows users to focus on strategic tasks while the AI Generative AI applications manage the heavy lifting of data processing and analysis.

#### Configure a Python virtual environment

Virtual environments in Python are critical for maintaining a clean and organized development environment, offering numerous benefits that can significantly enhance your coding experience. They allow each project to have its own set of dependencies, isolated from others, which prevents conflicts and ensures a consistent development workflow. This isolation is beneficial when deploying projects to production, as it ensures that the exact versions of dependencies used during development are maintained, reducing the risk of unexpected bugs and incompatibilities. Furthermore, virtual environments make it easier to collaborate with other developers by providing a consistent setup across different machines and development stages. By using virtual environments, you can easily manage package versions, avoid dependency clashes, and ensure your projects run smoothly. This best practice is essential for a stable and dependable coding environment, making your development process more efficient and less prone to issues.

Creating a Python virtual environment can be easily accomplished using a command similar to the following command, which creates a virtual environment named `.venv` in the directory in which the command is run:

```bash
python -m venv .venv
```

Once created, you can activate the virtual environment by selecting the appropriate command for your OS and shell from the following table.

|Platform|Shell|Command to activate virtual environment|
|---|---|---|
|POSIX|bash/zsh|`source .venv/bin/activate`|
||fish|`source .venv/bin/activate.fish`|
||csh/tcsh|`source .venv/bin/activate.csh`|
||pwsh|`.venv/bin/Activate.ps1`|
|Windows|cmd.exe|`.venv\Scripts\activate.bat`|
||PowerShell|`.venv\Scripts\Activate.ps1`|

After activating the virtual environment, any required Python libraries can be installed using the `pip install` command. Typically, required libraries and their versions are maintained in a `requirements.txt` file. This file allows versions to be specified and retained within the project, ensuring all libraries and dependencies are maintained between environments. These files store dependencies in the following format:

```
azure-cosmos==4.9.0
azure-identity==1.19.0
fastapi==0.115.5
openai==1.55.2
pydantic==2.10.2
requests==2.32.3
streamlit==1.40.2
uvicorn==0.32.1
```

Adding all libraries listed in this file can also be accomplished using the `pip install` command by running:

```bash
pip install -r requirements.txt
```

#### Securely access Azure resources using Entra ID RBAC

Utilizing Microsoft Entra ID's Role-Based Access Control (RBAC) for authenticating against Azure services like Azure OpenAI and Azure Cosmos DB presents several key benefits over key-based methods. Entra ID RBAC enhances security through precise access controls tailored to user roles, effectively reducing unauthorized access risks. It also streamlines user management, enabling administrators to dynamically assign and modify permissions without the hassle of distributing and maintaining cryptographic keys. Furthermore, this approach enhances compliance and auditability by aligning with organizational policies and facilitating comprehensive access monitoring and review. Entra ID RBAC makes a more efficient and scalable solution for using Azure services by streamlining secure access management.

The following code snippet demonstrates authenticating and configuring a client for interacting with Azure OpenAI services using Microsoft Entra ID RBAC (role-based access control) authentication.

```python
from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from openai import AzureOpenAI

# Enable Microsoft Entra ID RBAC authentication
token_provider = get_bearer_token_provider(
    DefaultAzureCredential(),
    "https://cognitiveservices.azure.com/.default"
)

client = AzureOpenAI(
    api_version = AZURE_OPENAI_API_VERSION,
    azure_endpoint = AZURE_OPENAI_ENDPOINT,
    azure_ad_token_provider = token_provider
)
```

The `token_provider = get_bearer_token_provider(DefaultAzureCredential(), "https://cognitiveservices.azure.com/.default")` previous line creates a token provider using the `DefaultAzureCredential` and the Azure Cognitive Services scope URL. `DefaultAzureCredential` handles the authentication process, and the `get_bearer_token_provider` utility function returns a token provider that obtains access tokens for Azure services. The `AzureOpenAI` client then uses the created `token_provider` to authenticate requests to the Azure OpenAI service using Microsoft Entra ID RBAC.

#### Generative AI application architecture

The architecture of using separate frontend and backend layers provides significant extensibility, allowing for the seamless integration of extra functionalities over time. Once the initial Generative AI application is developed, incorporating new features, such as LangChain orchestration, into the APIs becomes straightforward.

![A diagram showing a high-level Generative AI application architecture diagram, showing a UI developed in Python using Streamlit, a backend API written in Python, and interactions with Azure Cosmos DB and Azure OpenAI.](https://learn.microsoft.com/en-us/training/wwl-data-ai/build-generative-ai-applications-with-azure-cosmos-db-nosql/media/6-copilot-high-level-architecture-diagram.png)

This modular design enables developers to enhance the backend with advanced capabilities without disrupting the existing front end. For instance, LangChain can be added to handle complex workflows and chain multiple tasks, boosting the Generative AI application's functionality. This flexibility ensures the system remains scalable and adaptable, ready to incorporate future advancements, and efficiently meet evolving user needs.

#### Create a UI with Streamlit

Streamlit is a powerful open-source Python library that enables rapid development of interactive web applications, making it an ideal choice for building a Generative AI application's user interface. With its intuitive API, developers can quickly create dynamic, responsive chat interfaces that facilitate real-time interactions. Streamlit's built-in support for various widgets and its seamless integration with popular data visualization tools allow for the easy incorporation of chat functionality, enabling users to communicate with the Generative AI application efficiently. By using Streamlit, you can streamline the process of constructing a user-friendly and engaging interface, ensuring a smooth and productive experience for users interacting with your Generative AI application.

```python
# Create a Generative AI application UI with Streamlit
import streamlit as st
import requests

st.set_page_config(page_title="Cosmic Works Generative AI application", layout="wide")

def send_message_to_copilot(message: str, chat_history: list = []) -> str:
    """Send a message to the Generative AI application chat endpoint."""
    try:
        api_endpoint = "http://localhost:8000"
        request = {"message": message, "chat_history": chat_history}
        response = requests.post(f"{api_endpoint}/chat", json=request, timeout=60)
        print('response:', response.content)
        return response.json()
    except Exception as e:
        st.error(f"An error occurred: {e}")
        return ""

def main():
    """Main function for the Cosmic Works Product Management Generative AI application UI."""

    st.write(
        """
        # Cosmic Works Product Management Generative AI application
    
        Welcome to Cosmic Works Product Management Generative AI application, a tool for managing and finding bicycle-related products in the Cosmic Works system.
    
        **Ask the Generative AI application to apply or remove a discount on a category of products or to find products.**
        """
    )

    if "messages" not in st.session_state:
        st.session_state.messages = []

    # Display message from the history on app rerun.
    for message in st.session_state.messages:
        with st.chat_message(message["role"]):
            st.markdown(message["content"])

    # React to user input
    if prompt := st.chat_input("What can I help you with today?"):
        with st.spinner("Awaiting the Generative AI application's response to your question..."):
            # Display user message in chat message container
            with st.chat_message("user"):
                st.markdown(prompt)
            
            # Send user message to Generative AI application and get response
            response = send_message_to_copilot(prompt, st.session_state.messages)            

            # Display assistant response in chat message container
            with st.chat_message("assistant"):
                st.markdown(response)
            
            # Add the current user message and assistant response messages to chat history
            st.session_state.messages.append({"role": "user", "content": prompt})
            st.session_state.messages.append({"role": "assistant", "content": response})
```

#### Build a backend API with Python and FastAPI

FastAPI is a modern framework for developing APIs with Python, which is well-suited for creating robust backend APIs. When designing a Generative AI application UI with Streamlit, FastAPI can serve as the powerful backend engine for handling interactions with Azure services like Azure OpenAI and Cosmos DB. Using FastAPI's efficient request handling, you can quickly build endpoints that enable communication between the Streamlit front end and Azure's services. This setup ensures that user queries to the Generative AI application are processed smoothly, allowing real-time responses and efficient data management. FastAPI's simplicity and high performance make it an excellent choice for building the backend infrastructure needed to support advanced Generative AI applications.

```python
from fastapi import FastAPI
from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from openai import AzureOpenAI

app = FastAPI()

# Enable Microsoft Entra ID RBAC authentication
token_provider = get_bearer_token_provider(
    DefaultAzureCredential(),
    "https://cognitiveservices.azure.com/.default"
)

client = AzureOpenAI(
    api_version = AZURE_OPENAI_API_VERSION,
    azure_endpoint = AZURE_OPENAI_ENDPOINT,
    azure_ad_token_provider = token_provider
)

@app.get("/chat")
async def root(message: str, deployment_name: str = "gpt-4o"):
    messages = [{"role": "user", "content": message}]
    completion = client.chat.completions.create(
        model=deployment_name,
        messages=messages,
    )
    return completion
```

#### Leverage private data through function calling

Function calling in Azure OpenAI allows the seamless integration of external APIs or tools directly into your model’s output. When the model detects a relevant request, it constructs a JSON object with the necessary parameters, which you then execute. The result is returned to the model, enabling it to deliver a comprehensive final response enriched with external data.

When using function calling, there are several steps you need to perform in code. First, you must create a function that is called to perform an action. This method is a regular Python function. For example, the following function, `apply_discount`, accepts a discount amount, such as 0.1 for 10%, and a product category and applies that discount amount to every product matching that category:

```python
async def apply_discount(discount: float, product_category: str) -> str:
    """Apply a discount to products in the specified category."""
    # Load the database
    database = cosmos_client.get_database_client(DATABASE_NAME)
    # Retrieve the container
    container = database.get_container_client(CONTAINER_NAME)

    query_results = container.query_items(
        query = """
        SELECT * FROM Products p WHERE LOWER(p.category_name) = LOWER(@product_category)
        """,
        parameters = [
            {"name": "@product_category", "value": product_category}
        ]
    )

    # Apply the discount to the products
    async for item in query_results:
        item['discount'] = discount
        item['sale_price'] = item['price'] * (1 - discount) if discount > 0 else item['price']
        await container.upsert_item(item)

    return f"A {discount}% discount was successfully applied to {product_category}." if discount > 0 else f"Discounts on {product_category} removed successfully."
```

Next, you must provide a JSON-formatted function definition that the LLM (Larger Language Model) uses to understand how to interact with the function.

```json
{
   "type": "function",
   "function": {
         "name": "apply_discount",
         "description": "Apply a discount to products in the specified category",
         "parameters": {
            "type": "object",
            "properties": {
               "discount": {"type": "number", "description": "The percent discount to apply."},
               "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
            },
            "required": ["discount", "product_category"]
         }
   }
}
```

The function definition must be added to an array of `tools` that can be passed into the LLM. It contains definitions for all functions you want to make available to your models. For example, the following `tools` array contains definitions for two functions, `apply_discount` and `get_category_names`.

```python
# Define function calling tools
tools = [
   {
      "type": "function",
      "function": {
            "name": "apply_discount",
            "description": "Apply a discount to products in the specified category",
            "parameters": {
               "type": "object",
               "properties": {
                  "discount": {"type": "number", "description": "The percent discount to apply."},
                  "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
               },
               "required": ["discount", "product_category"]
            }
      }
   },
   {
      "type": "function",
      "function": {
            "name": "get_category_names",
            "description": "Retrieves the names of all product categories"
      }
   }
]
```

When using function calling with Azure OpenAI, you must make two calls to the LLM. The first allows the LLM to determine what "tools" to use, and the second generates a completion using a prompt enriched with output from the function calls.

The first call requires you to provide the Azure OpenAI client with the tools array and then capture the tool outputs in the response from that call:

```python
# First API call, providing the model to the defined functions
response = aoai_client.chat.completions.create(
   model = COMPLETION_DEPLOYMENT_NAME,
   messages = messages,
   tools = tools,
   tool_choice = "auto"
)

# Process the model's response
response_message = response.choices[0].message
messages.append(response_message)
```

You must then use the requested function calls to "handle" the response, where you make the calls to the functions requested by the LLM. This handler lets you call the functions and capture their output so it can be written into the conversation and used by the LLM on the next API call.

```python
# Handle function call outputs
if response_message.tool_calls:
   for call in response_message.tool_calls:
      if call.function.name == "apply_discount":
            func_response = apply_discount(**json.loads(call.function.arguments))
            messages.append(
               {
                  "role": "tool",
                  "tool_call_id": call.id,
                  "name": call.function.name,
                  "content": func_response
               }
            )
      elif call.function.name == "get_category_names":
            func_response = get_category_names()
            messages.append(
               {
                  "role": "tool",
                  "tool_call_id": call.id,
                  "name": call.function.name,
                  "content": json.dumps(func_response)
               }
            )
else:
   print("No function calls were made by the model.")
```

Finally, you make the second call to the LLM, passing in a `messages` collection enriched with outputs from your function calls.

```python
# Second API call, asking the model to generate a response
final_response = aoai_client.chat.completions.create(
   model = COMPLETION_DEPLOYMENT_NAME,
   messages = messages
)

# Output the completion response
print(final_response.choices[0].message.content)
```

#### Use prompt engineering to provide a persona to your Generative AI application

Prompt engineering is a crucial technique in artificial intelligence and natural language processing. It guides AI models in generating desired outputs by crafting specific, well-structured inputs (prompts) that steer the AI's responses toward achieving accurate and relevant results.

Effective prompt engineering requires understanding the AI model's capabilities and limitations, and the nuances of language. By providing clear instructions, context, and examples within the prompt, developers can influence the AI to produce high-quality content, solve complex problems, or perform specific tasks. This approach enhances the AI's ability to comprehend and respond to various queries, making interactions more useful and engaging.

When building a Generative AI application, prompt engineering enables you to define a "persona" for your assistant. The persona instructs the LLM about interacting with Generative AI application users, describing how to handle inputs and create outputs. Creating a persona is accomplished using a system prompt that you add to the collection of messages used by the Azure OpenAI client and the underlying language model.

```python
# Define the system prompt that contains the assistant's persona.
system_prompt = """
You are an intelligent Generative AI application for Cosmic Works designed to help users manage and find bicycle-related products.
You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
If asked to apply a discount:
    - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
    - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
If asked to remove discounts from a category:
    - Remove any discounts applied to products in the specified category by setting the discount value to 0.
When asked to provide a list of products, you should:
    - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
"""

# Provide the Generative AI application with a persona using the system prompt.
messages = [{ "role": "system", "content": system_prompt }]
```

### Perform vector searches using Azure Cosmos DB for NoSQL from a Generative AI application

Integrating the vector search capabilities of Azure Cosmos DB for NoSQL into a Python-based Generative AI application significantly enhances the Generative AI application's ability to perform similarity searches efficiently. Vector search allows retrieving semantically similar items based on vector embeddings, making it ideal for Generative AI application' natural language processing tasks. By using vector search, a Generative AI application can quickly find and retrieve relevant information from large datasets, enriching its responses with accurate and contextually relevant data. This capability enhances the user experience by providing more precise and meaningful assistance, ultimately making the Generative AI application more intelligent and effective.

#### What is vector search?

Vector search is a technique that allows items to be found based on their data characteristics instead of exact matches on a specific property or field. Instead of requiring exact matches, vector search enables you to identify matches based on their vector representations. This technique is advantageous when performing similarity searches and is valuable in applications that require searching for information within large blocks of text.

![A flow diagram for performing vector search, showing an incoming message being vectorized and compared against a vector store, then being added to an LLM prompt for the generation of a completion response.](https://learn.microsoft.com/en-us/training/wwl-data-ai/build-generative-ai-applications-with-azure-cosmos-db-nosql/media/7-vector-search.png)

It plays a critical role in the implementation of the retrieval augmented generation (RAG) pattern frequently used by Generative AI applications by enabling secure and efficient integration of proprietary data into large language model (LLM) responses, ensuring that Generative AI applications can provide precise, domain-specific answers while maintaining the confidentiality and integrity of sensitive information.

#### Understand the steps involved in vector search

Executing vector searches with Azure Cosmos DB for NoSQL involves the following steps:

1. Create and store vector embeddings for the fields on which you want to perform similarity searches.
2. Specify the vector embedding paths in the container's vector embedding policy.
3. Include any desired vector indexes in the indexing policy for the container.
4. Populate the container with documents containing vector embeddings.
5. Generate embeddings representing the search query using Azure OpenAI or another service.
6. Run a query using the `VectorDistance` function to compare the similarity of the search query embeddings to those embeddings of the vectors stored in the Cosmos DB container.

#### Perform vector search using the VectorDistance function

The `VectorDistance` function in Azure Cosmos DB for NoSQL measures the similarity between two vectors by calculating their distance using metrics like cosine similarity, dot product, or Euclidean distance. This function is vital for applications that require quick and accurate similarity searches, such as those involving natural language processing or recommendation systems. By utilizing `VectorDistance`, you can efficiently handle high-dimensional vector queries, significantly improving the relevance and performance of your AI-driven applications.

Suppose, for example, you want to search for products about bike pedals by looking at the product description. First, you must generate embeddings for your query text. In this case, you might want to create embeddings for the query text "clip-in pedals." Providing the embedding for your search query text to the `VectorDistance` function in the vector search query, you can identify and retrieve products similar to your query, like the following query:

```sql
SELECT TOP 5 c.name, c.description, VectorDistance(c.embedding, [1,2,3]) AS SimilarityScore 
FROM c 
ORDER BY VectorDistance(c.embedding, [1,2,3])
```

The previous example shows a NoSQL query that projects the similarity score as the alias `SimilarityScore` and sorts it from most-similar to least-similar.

> ❗ You should always use a `TOP N` clause in `SELECT` queries when performing vector searches. Without it, the vector search will attempt to return many more matches, resulting in the query costing more RUs and having higher latency than necessary.

Implementing vector search from Python code for a Generative AI application would look similar to:

```python
# Create a Cosmos DB client
cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
# Load the database
database = cosmos_client.get_database_client(DATABASE_NAME)
# Retrieve the container
container = database.get_container_client(CONTAINER_NAME)

# Function for generating embeddings using Azure OpenAI's text-embedding-3-small model
def generate_embeddings(text: str):
    client = AzureOpenAI(
            api_version = AZURE_OPENAI_API_VERSION,
            azure_endpoint = AZURE_OPENAI_ENDPOINT,
            azure_ad_token_provider = token_provider
    )
    response = client.embeddings.create(input = text, model = "text-embedding-3-small")
    return response.data[0].embedding

def vector_search(query_embedding: list, num_results: int = 3, similarity_score: float = 0.25):
    """Search for similar product vectors in Azure Cosmos DB"""
    results = container.query_items(
        query = """
        SELECT TOP @num_results p.name, p.description, p.sku, p.price, p.discount, p.sale_price, VectorDistance(p.embedding, @query_embedding) AS similarity_score
        FROM Products p
        WHERE VectorDistance(p.embedding, @query_embedding) > @similarity_score
        ORDER BY VectorDistance(p.embedding, @query_embedding)
        """,
        parameters = [
            {"name": "@query_embedding", "value": query_embedding},
            {"name": "@num_results", "value": num_results},
            {"name": "@similarity_score", "value": similarity_score}
        ],
        enable_cross_partition_query = True
    )
    results = list(results)
    formatted_results = [{'similarity_score': result.pop('similarity_score'), 'product': result} for result in results]
    return formatted_results

# Get embeddings for search query text
query_embedding = generate_embeddings(query_text)

# Perform a vector search
results = vector_search(query_embedding, 3, 0.5)
```
### Build a copilot with Python and Azure Cosmos DB for NoSQL

By utilizing Python’s versatile programming capabilities and Azure Cosmos DB’s scalable NoSQL database and vector search capabilities, you can create powerful and efficient AI copilots, streamlining complex workflows.

In this lab, you will build a copilot using Python and Azure Cosmos DB for NoSQL, creating a backend API that will provide endpoints necessary for interacting with Azure services (Azure OpenAI and Azure Cosmos DB) and a frontend UI to facilitate user interaction with the copilot. The copilot will serve as an assistant for helping Cosmic Works users manage and find bicycle-related products. Specifically, the copilot will enable users to apply and remove discounts from categories of products, look up product categories to help inform users of what product types are available, and use vector search to perform similarity searches for products.

[![A high-level copilot architecture diagram, showing a UI developed in Python using Streamlit, a backend API written in Python, and interactions with Azure Cosmos DB and Azure OpenAI.](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/media/07-copilot-high-level-architecture-diagram.png)](https://microsoftlearning.github.io/dp-420-cosmos-db-dev/gen-ai/python/instructions/media/07-copilot-high-level-architecture-diagram.png)

Separating app functionality into a dedicated UI and backend API when creating a copilot in Python offers several benefits. Firstly, it enhances modularity and maintainability, allowing you to update the UI or backend independently without disrupting the other. Streamlit provides an intuitive and interactive interface that simplifies user interactions, while FastAPI ensures high-performance, asynchronous request handling and data processing. This separation also promotes scalability, as different components can be deployed across multiple servers, optimizing resource usage. Additionally, it enables better security practices, as the backend API can handle sensitive data and authentication separately, reducing the risk of exposing vulnerabilities in the UI layer. This approach leads to a more robust, efficient, and user-friendly application.

> 🛑 The previous exercises in this module are prerequisites for this lab. If you still need to complete any of those exercises, please finish them before continuing, as they provide the necessary infrastructure and starter code for this lab.

#### Construct a backend API

The backend API for the copilot enriches its abilities to handle intricate data, provide real-time insights, and connect seamlessly with diverse services, making interactions more dynamic and informative. To build the API for your copilot, you will use the FastAPI Python library. FastAPI is a modern, high-performance web framework designed to enable you to build APIs with Python based on standard Python type hints. By decoupling the copilot from the backend using this approach, you ensure greater flexibility, maintainability, and scalability, allowing the copilot to evolve independently from backend changes.

> 🛑 The backend API builds upon the code you added to the `main.py` file in the `python/07-build-copilot/api/app` folder in the previous exercise. If you have not yet finished the previous exercise, please complete it before continuing.

1. Using Visual Studio Code, open the folder into which you cloned the lab code repository for **Build copilots with Azure Cosmos DB** learning module.
    
2. In the **Explorer** pane within Visual Studio Code, browse to the **python/07-build-copilot/api/app** folder and open the `main.py` file found within it.
    
3. Add the following lines of code below the existing `import` statements at the top of the `main.py` file to bring in the libraries that will be used to perform asychronous actions using FastAPI:
    
    ```python
    from contextlib import asynccontextmanager
    from fastapi import FastAPI
    import json
    ```
    
4. To enable the `/chat` endpoint you will create to receive data in the request body, you will pass content in via a `CompletionRequest` object defined in the projects _models_ module. Update the `from models import Product` import statement at the top of the file to include the `CompletionRequest` class from the `models` module. The import statement should now look like this:
    
    ```python
    from models import Product, CompletionRequest
    ```
    
5. You will need the deployment name of the chat completion model you created in your Azure OpenAI Service. Create a variable at the bottom of the Azure OpenAI configuration variable block to provide this:
    
    ```python
    COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
    ```
    
    If your completion deployment name differs, update the value assigned to the variable accordingly.
    
6. The Azure Cosmos DB and Identity SDKs provide async methods for working with those services. Each of these classes will used in multiple functions in your API, so you will create global instances of each, allowing the same client to be shared across methods. Insert the following global variable declarations below the Cosmos DB configuration variables block:
    
    ```python
    # Create a global async Cosmos DB client
    cosmos_client = None
    # Create a global async Microsoft Entra ID RBAC credential
    credential = None
    ```
    
7. Delete the following lines of code from the file, as the functionality provided will be moved into the `lifespan` function you will define in the next step:
    
    ```python
    # Enable Microsoft Entra ID RBAC authentication
    credential = DefaultAzureCredential()
    ```
    
8. To create singleton instances of the `CosmosClient` and `DefaultAzureCredentail` classes, you will take advantage of the `lifespan` object in FastAPI: This method manages those classes through the lifecycle of the API app. Insert the following code to define the `lifespan`:
    
    ```python
    @asynccontextmanager
    async def lifespan(app: FastAPI):
        global cosmos_client
        global credential
        # Create an async Microsoft Entra ID RBAC credential
        credential = DefaultAzureCredential()
        # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
        cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
        yield
        await cosmos_client.close()
        await credential.close()
    ```
    
    In FastAPI, lifespan events are special operations that run at the beginning and end of the application’s life cycle. These operations execute before the app starts handling requests and after it stops, making them ideal for initializing and cleaning up resources that are used across the entire application and shared between requests. This approach ensures that necessary setup is completed before any requests are processed and that resources are properly managed when shutting down.
    
9. Create an instance of the FastAPI class using the following code. This should be inserted below the `lifespan` function:
    
    
    ```python
    app = FastAPI(lifespan=lifespan)
    ```
    
    By calling `FastAPI()`, you are initializing a new instance of the FastAPI application. This instance, referred to as `app`, will serve as the main entry point for your web application. Passing in the `lifespan` attaches the lifespan event handler to your app.
    
10. Next, stub out the endpoints for your API. The `api_status` method is attached to the root URL of your API and acts as a status message to show that the API is up and running correctly. You will build out the `/chat` endpoint later in this exercise. Insert the following code below the code for creating the Cosmos DB client, database and container:

    
    ```python
    @app.get("/")
    async def api_status():
        """Display a status message for the API"""
        return {"status": "ready"}
        
    @app.post('/chat')
    async def generate_chat_completion(request: CompletionRequest):
        """Generate a chat completion using the Azure OpenAI API."""
        raise NotImplementedError("The chat endpoint is not implemented yet.")
    ```
    
11. Overwrite the main guard block at the bottom of the file to start the `uvicorn` ASGI (Asynchronous Server Gateway Interface) web server when the file is run from the command line:
    
    ```python
    if __name__ == "__main__":
        import uvicorn
        uvicorn.run(app, host="0.0.0.0", port=8000)
    ```
    
12. Save the `main.py` file. It should now look like the following, including the `generate_embeddings` and `upsert_product` methods you added in the pervious exercise:
    
    ```python
    from openai import AsyncAzureOpenAI
    from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    from azure.cosmos.aio import CosmosClient
    from models import Product, CompletionRequest
    from contextlib import asynccontextmanager
    from fastapi import FastAPI
    import json
        
    # Azure OpenAI configuration
    AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
    AZURE_OPENAI_API_VERSION = "2024-10-21"
    EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
        
    # Azure Cosmos DB configuration
    AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
    DATABASE_NAME = "CosmicWorks"
    CONTAINER_NAME = "Products"
        
    # Create a global async Cosmos DB client
    cosmos_client = None
    # Create a global async Microsoft Entra ID RBAC credential
    credential = None
       
    @asynccontextmanager
    async def lifespan(app: FastAPI):
        global cosmos_client
        global credential
        # Create an async Microsoft Entra ID RBAC credential
        credential = DefaultAzureCredential()
        # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
        cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
        yield
        await cosmos_client.close()
        await credential.close()
        
    app = FastAPI(lifespan=lifespan)
        
    @app.get("/")
    async def api_status():
        return {"status": "ready"}
        
    @app.post('/chat')
    async def generate_chat_completion(request: CompletionRequest):
        """ Generate a chat completion using the Azure OpenAI API."""
        raise NotImplementedError("The chat endpoint is not implemented yet.")
        
    async def generate_embeddings(text: str):
        # Create Azure OpenAI client
        async with AsyncAzureOpenAI(
            api_version = AZURE_OPENAI_API_VERSION,
            azure_endpoint = AZURE_OPENAI_ENDPOINT,
            azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
        ) as client:
            response = await client.embeddings.create(
                input = text,
                model = EMBEDDING_DEPLOYMENT_NAME
            )
            return response.data[0].embedding
        
    async def upsert_product(product: Product):
        """Upserts the provided product to the Cosmos DB container."""
        # Create an async Cosmos DB client
        async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
            # Load the CosmicWorks database
            database = client.get_database_client(DATABASE_NAME)
            # Retrieve the product container
            container = database.get_container_client(CONTAINER_NAME)
            # Upsert the product
            await container.upsert_item(product)
        
    if __name__ == "__main__":
        import uvicorn
        uvicorn.run(app, host="0.0.0.0", port=8000)
    ```
    
13. To quickly test your API, open a new integrated terminal window in Visual Studio Code.
    
14. Ensure you are logged into Azure using the `az login` command. Running the following at the terminal prompt:
    
    ```bash
    az login
    ```
    
15. Complete the login process in your browser.
    
16. Change directories to `python/07-build-copilot` at the terminal prompt.
    
17. Ensure the integrated terminal window runs within your Python virtual environment by activating it using a command from the table below and selecting the appropriate command for your OS and shell.
    
    |Platform|Shell|Command to activate virtual environment|
    |---|---|---|
    |POSIX|bash/zsh|`source .venv/bin/activate`|
    ||fish|`source .venv/bin/activate.fish`|
    ||csh/tcsh|`source .venv/bin/activate.csh`|
    ||pwsh|`.venv/bin/Activate.ps1`|
    |Windows|cmd.exe|`.venv\Scripts\activate.bat`|
    ||PowerShell|`.venv\Scripts\Activate.ps1`|
    
18. At the terminal prompt, change directories to `api/app`, then execute the following command to run the FastAPI web app:
    
    ```bash
    uvicorn main:app
    ```
    
19. If one does not open automatically, launch a new web browser window or tab and go to [http://127.0.0.1:8000](http://127.0.0.1:8000/).
    
    A message of `{"status":"ready"}` in the browser window indicates your API is running.
    
20. Navigate to the Swagger UI for the API by appending `/docs` to the end of the URL: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs).
    
    > 📝 The Swagger UI is an interactive, web-based interface for exploring and testing API endpoints generated from OpenAPI specifications. It allows developers and users to visualize, interact with, and debug real-time API calls, enhancing usability and documentation.
    
21. Return to Visual Studio Code and stop the API app by pressing **CTRL+C** in the associated integrated terminal window.
    

#### Incorporate product data from Azure Cosmos DB

By leveraging data from Azure Cosmos DB, the copilot can streamline complex workflows and assist users in efficiently completing tasks. The copilot can update records and retrieve lookup values in real time, ensuring accurate and timely information. This capability enables the copilot to provide advanced interactions, enhancing users’ ability to quickly and precisely navigate and complete tasks.

Functions will allow the product management copilot to apply discounts to products within a category. These functions will be the mechanism through which the copilot retrieves and interacts with Cosmic Works product data from Azure Cosmos DB.

1. The copilot will use an async function named `apply_discount` to add and remove discounts and sale prices on products within a specified category. Insert the following function code below the `upsert_product` function near the bottom of the `main.py` file:
    
    ```python
    async def apply_discount(discount: float, product_category: str) -> str:
        """Apply a discount to products in the specified category."""
        # Load the CosmicWorks database
        database = cosmos_client.get_database_client(DATABASE_NAME)
        # Retrieve the product container
        container = database.get_container_client(CONTAINER_NAME)
        
        query_results = container.query_items(
            query = """
            SELECT * FROM Products p WHERE CONTAINS(LOWER(p.category_name), LOWER(@product_category))
            """,
            parameters = [
                {"name": "@product_category", "value": product_category}
            ]
        )
        
        # Apply the discount to the products
        async for item in query_results:
            item['discount'] = discount
            item['sale_price'] = item['price'] * (1 - discount) if discount > 0 else item['price']
            await container.upsert_item(item)
        
        return f"A {discount}% discount was successfully applied to {product_category}." if discount > 0 else f"Discounts on {product_category} removed successfully."
    ```
    
    This function performs a lookup in Azure Cosmos DB to pull all products within a category and apply the requested discount to those products. It also calculates the item’s sale price using the specified discount and inserts that into the database.
    
2. Next, you will add a second function named `get_category_names`, which the copilot will call to assist it in knowing what product categories are available when applying or removing discounts from products. Add the below method below the `apply_discount` function in the file:
    
    ```python
    async def get_category_names() -> list:
        """Retrieve the names of all product categories."""
        # Load the CosmicWorks database
        database = cosmos_client.get_database_client(DATABASE_NAME)
        # Retrieve the product container
        container = database.get_container_client(CONTAINER_NAME)
        # Get distinct product categories
        query_results = container.query_items(
            query = "SELECT DISTINCT VALUE p.category_name FROM Products p"
        )
        categories = []
        async for category in query_results:
            categories.append(category)
        return list(categories)
    ```
    
    The `get_category_names` function queries the `Products` container to retrieve a list of distinct category names from the database.
    
3. Save the `main.py` file.
    

#### Implement the chat endpoint

The `/chat` endpoint on the backend API serves as the interface through which the frontend UI interacts with Azure OpenAI models and internal Cosmic Works product data. This endpoint acts as the communication bridge, allowing UI input to be sent to the Azure OpenAI service, which then processes these inputs using sophisticated language models. The results are then returned to the front end, enabling real-time, intelligent conversations. By leveraging this setup, developers can ensure a seamless and responsive user experience while the backend handles the complex task of processing natural language and generating appropriate responses. This approach also supports scalability and maintainability by decoupling the front end from the underlying AI infrastructure.

1. Locate the `/chat` endpoint stub you added previously in the `main.py` file.
    
    ```python
    @app.post('/chat')
    async def generate_chat_completion(request: CompletionRequest):
        """Generate a chat completion using the Azure OpenAI API."""
        raise NotImplementedError("The chat endpoint is not implemented yet.")
    ```
    
    The function accepts a `CompletionRequest` as a parameter. Utilizing a class for the input parameter allows multiple properties to be passed into the API endpoint in the request body. The `CompletionRequest` class is defined within the _models_ module and includes user message, chat history, and max history properties. The chat history allows the copilot to reference previous aspects of the conversation with the user, so it maintains knowledge of the context of the entire discussion. The `max_history` property allows you to define the number of history messages should be passed into the context of the LLM. This enables you to control token usages for your prompt and avoid TPM limits on requests.
    
2. To start, delete the `raise NotImplementedError("The chat endpoint is not implemented yet.")` line from the function as you are beginning the process of implementing the endpoint.
    
3. The first thing you will do within the chat endpoint method is provide a system prompt. This prompt defines the copilots “persona,” dictacting how the copilot should interact with users, respond to questions, and leverage available functions to perform actions.
    
    ```python
    # Define the system prompt that contains the assistant's persona.
    system_prompt = """
    You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
    You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
    If asked to apply a discount:
        - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
        - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
    If asked to remove discounts from a category:
        - Remove any discounts applied to products in the specified category by setting the discount value to 0.
    """
    ```
    
4. Next, create an array of messages to send to the LLM, adding the system prompt, any messages in the chat history, and the incoming user message. This code should go directly below the system prompt declaration in the function:
    
    ```python
    # Provide the copilot with a persona using the system prompt.
    messages = [{"role": "system", "content": system_prompt }]
        
    # Add the chat history to the messages list
    for message in request.chat_history[-request.max_history:]:
        messages.append(message)
        
    # Add the current user message to the messages list
    messages.append({"role": "user", "content": request.message})
    ```
    
    The `messages` property encapsulates the ongoing conversation’s history. It includes the entire sequence of user inputs and the AI’s responses, which helps the model maintain context. By referencing this history, the AI can generate coherent and contextually relevant replies, ensuring that interactions remain fluid and dynamic. This property is crucial for enabling the AI to understand the flow and nuances of the conversation as it progresses.
    
5. To allow the copilot to use the functions you defined above for interacting with data from Azure Cosmos DB, you must define a collection of “tools.” The LLM will call these tools as part of its execution. Azure OpenAI uses function definitions to enable structured interactions between the AI and various tools or APIs. When a function is defined, it describes the operations it can perform, the necessary parameters, and any required inputs. To create an array of `tools`, provide the following code containing function definitions for the `apply_discount` and `get_category_names` methods you previously defined:
    
    ```python
    # Define function calling tools
    tools = [
        {
            "type": "function",
            "function": {
                "name": "apply_discount",
                "description": "Apply a discount to products in the specified category",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "discount": {"type": "number", "description": "The percent discount to apply."},
                        "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                    },
                    "required": ["discount", "product_category"]
                }
            }
        },
        {
            "type": "function",
            "function": {
                "name": "get_category_names",
                "description": "Retrieves the names of all product categories"
            }
        }
    ]
    ```
    
    By using function definitions, Azure OpenAI ensures that interactions between the AI and external systems are well-organized, secure, and efficient. This structured approach allows the AI to perform complex tasks seamlessly and reliably, enhancing its overall capabilities and user experience.
    
6. Create an async Azure OpenAI client for making requests to your chat completion model:
    
    ```python
    # Create Azure OpenAI client
    aoai_client = AsyncAzureOpenAI(
        api_version = AZURE_OPENAI_API_VERSION,
        azure_endpoint = AZURE_OPENAI_ENDPOINT,
        azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
    )
    ```
    
7. The chat endpoint will make two calls to Azure OpenAI to leverage function calling. The first provides the Azure OpenAI client access to the tools:
    
    ```python
    # First API call, providing the model to the defined functions
    response = await aoai_client.chat.completions.create(
        model = COMPLETION_DEPLOYMENT_NAME,
        messages = messages,
        tools = tools,
        tool_choice = "auto"
    )
        
    # Process the model's response and add it to the conversation history
    response_message = response.choices[0].message
    messages.append(response_message)
    ```
    
8. The response from this first call contains information from the LLM about what tools or functions it has determined are necessary to respond to the request. You must include code to process the function call outputs, inserting them into the conversation history so the LLM can use them to formulate a response over the data contained within those outputs:
    
    ```python
    # Handle function call outputs
    if response_message.tool_calls:
        for call in response_message.tool_calls:
            if call.function.name == "apply_discount":
                func_response = await apply_discount(**json.loads(call.function.arguments))
                messages.append(
                    {
                        "role": "tool",
                        "tool_call_id": call.id,
                        "name": call.function.name,
                        "content": func_response
                    }
                )
            elif call.function.name == "get_category_names":
                func_response = await get_category_names()
                messages.append(
                    {
                        "role": "tool",
                        "tool_call_id": call.id,
                        "name": call.function.name,
                        "content": json.dumps(func_response)
                    }
                )
    else:
        print("No function calls were made by the model.")
    ```
    
    Function calling in Azure OpenAI allows the seamless integration of external APIs or tools directly into your model’s output. When the model detects a relevant request, it constructs a JSON object with the necessary parameters, which you then execute. The result is returned to the model, enabling it to deliver a comprehensive final response enriched with external data.
    
9. To complete the request with the enriched data from Azure Cosmos DB, you need to send a second request to Azure OpenAI to generate a completion:
    
    ```python
    # Second API call, asking the model to generate a response
    final_response = await aoai_client.chat.completions.create(
        model = COMPLETION_DEPLOYMENT_NAME,
        messages = messages
    )
    ```
    
10. Finally, return the completion response to the UI:
    

```python
   return final_response.choices[0].message.content
```

1. Save the `main.py` file. The `/chat` endpoint’s `generate_chat_completion` method should look like this:


```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       # Define the system prompt that contains the assistant's persona.
       system_prompt = """
       You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
       You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
       If asked to apply a discount:
           - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
           - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
       If asked to remove discounts from a category:
           - Remove any discounts applied to products in the specified category by setting the discount value to 0.
       """
       # Provide the copilot with a persona using the system prompt.
       messages = [{ "role": "system", "content": system_prompt }]
    
       # Add the chat history to the messages list
       for message in request.chat_history[-request.max_history:]:
           messages.append(message)
    
       # Add the current user message to the messages list
       messages.append({"role": "user", "content": request.message})
    
       # Define function calling tools
       tools = [
           {
               "type": "function",
               "function": {
                   "name": "apply_discount",
                   "description": "Apply a discount to products in the specified category",
                   "parameters": {
                       "type": "object",
                       "properties": {
                           "discount": {"type": "number", "description": "The percent discount to apply."},
                           "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                       },
                       "required": ["discount", "product_category"]
                   }
               }
           },
           {
               "type": "function",
               "function": {
                   "name": "get_category_names",
                   "description": "Retrieves the names of all product categories"
               }
           }
       ]
       # Create Azure OpenAI client
       aoai_client = AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
    
       # First API call, providing the model to the defined functions
       response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages,
           tools = tools,
           tool_choice = "auto"
       )
    
       # Process the model's response
       response_message = response.choices[0].message
       messages.append(response_message)
    
       # Handle function call outputs
       if response_message.tool_calls:
           for call in response_message.tool_calls:
               if call.function.name == "apply_discount":
                   func_response = await apply_discount(**json.loads(call.function.arguments))
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": func_response
                       }
                   )
               elif call.function.name == "get_category_names":
                   func_response = await get_category_names()
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": json.dumps(func_response)
                       }
                   )
       else:
           print("No function calls were made by the model.")
    
       # Second API call, asking the model to generate a response
       final_response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages
       )
    
       return final_response.choices[0].message.content
```

#### Build a simple chat UI

The Streamlit UI provides a interface for users to interact with your copilot.

1. The UI will be defined using the `index.py` file located in the `python/07-build-copilot/ui` folder.
    
2. Open the `index.py` file and add the following import statements to the top of the file to get started:
    
    ```python
    import streamlit as st
    import requests
    ```
    
3. Configure the Streamlit page defined within the `index.py` file by adding the following line below the `import` statements:
    
    ```python
    st.set_page_config(page_title="Cosmic Works Copilot", layout="wide")
    ```
    
4. The UI will interact with the backend API by using the `requests` library to make calls to the `/chat` endpoint you defined on the API. You can encapsulate the API call in a method that expects the current user message and a list of messages from the chat history.
    
    ```python
    async def send_message_to_copilot(message: str, chat_history: list = []) -> str:
        """Send a message to the Copilot chat endpoint."""
        try:
            api_endpoint = "http://localhost:8000"
            request = {"message": message, "chat_history": chat_history}
            response = requests.post(f"{api_endpoint}/chat", json=request, timeout=60)
            return response.json()
        except Exception as e:
            st.error(f"An error occurred: {e}")
            return""
    ```
    
5. Define the `main` function, which is the entry point for calls into the application.
    
    ```python
    async def main():
        """Main function for the Cosmic Works Product Management Copilot UI."""
        
        st.write(
            """
            # Cosmic Works Product Management Copilot
            
            Welcome to Cosmic Works Product Management Copilot, a tool for managing and finding bicycle-related products in the Cosmic Works system.
            
            **Ask the copilot to apply or remove a discount on a category of products or to find products.**
            """
        )
        
        # Add a messages collection to the session state to maintain the chat history.
        if "messages" not in st.session_state:
            st.session_state.messages = []
        
        # Display message from the history on app rerun.
        for message in st.session_state.messages:
            with st.chat_message(message["role"]):
                st.markdown(message["content"])
        
        # React to user input
        if prompt := st.chat_input("What can I help you with today?"):
            with st. spinner("Awaiting the copilot's response to your message..."):
                # Display user message in chat message container
                with st.chat_message("user"):
                    st.markdown(prompt)
                    
                # Send the user message to the copilot API
                response = await send_message_to_copilot(prompt, st.session_state.messages)
        
                # Display assistant response in chat message container
                with st.chat_message("assistant"):
                    st.markdown(response)
                    
                # Add the current user message and assistant response messages to the chat history
                st.session_state.messages.append({"role": "user", "content": prompt})
                st.session_state.messages.append({"role": "assistant", "content": response})
    ```
    
6. Finally, add a **main guard** block at the end of the file:
    
    
    ```python
    if __name__ == "__main__":
        import asyncio
        asyncio.run(main())
    ```
    
7. Save the `index.py` file.
    

#### Test the copilot via the UI

1. Return to the integrated terminal window you opened in Visual Studio Code for the API project and enter the following to start the API app:
    
    ```bash
    uvicorn main:app
    ```
    
2. Open a new integrated terminal window, change directories to `python/07-build-copilot` to activate your Python environment, then change directories to the `ui` folder and run the following to start your UI app:
    
    ```bash
    python -m streamlit run index.py
    ```
    
3. If the UI does not open automatically in a browser window, launch a new browser tab or window and navigate to [http://localhost:8501](http://localhost:8501/) to open the UI.
    
4. At the chat prompt of the UI, enter “Apply discount” and send the message.
    
    Because you needed to provide the copilot with more details to act, the response should be a request for more information, such as providing the discount percentage you’d like to apply and the category of products to which the discount should be applied.
    
5. To understand what categories are available, ask the copilot to provide you with a list of product categories.
    
    The copilot will make a function call using the `get_category_names` function and enrich the conversation messages with those categories so it can respond accordingly.
    
6. You can also ask for a more specific set of categories, such as, “Provide me with a list of clothing-related categories.”
    
7. Next, ask the copilot to apply a 15% discount to all clothing products.
    
8. You can verify the pricing discount was applied by opening your Azure Cosmos DB account in the Azure portal, selecting the **Data Explorer**, and running a query against the `Products` container to view all products in the “clothing” category, such as:
    
    ```sql
    SELECT c.category_name, c.name, c.description, c.price, c.discount, c.sale_price FROM c
    WHERE CONTAINS(LOWER(c.category_name), "clothing")
    ```
    
    Observe that each item in the query results has a `discount` value of `0.15`, and the `sale_price` should be 15% less than the original `price`.
    
9. Return to Visual Studio Code and stop the API app by pressing **CTRL+C** in the terminal window running that app. You can leave the UI running.
    

#### Integrate vector search

So far, you have given the copilot the ability to perform actions to apply discounts to products, but it still has no knowledge of the products stored within the database. In this task, you will add vector search capabilities that will allow you to ask for products with certain qualities and find similar products within the database.

1. Return to the `main.py` file in the `api/app` folder and provide a method for performing vector searches against the `Products` container in your Azure Cosmos DB account. You can insert this method below the existing functions near the bottom of the file.
    
    ```python
    async def vector_search(query_embedding: list, num_results: int = 3, similarity_score: float = 0.25):
        """Search for similar product vectors in Azure Cosmos DB"""
        # Load the CosmicWorks database
        database = cosmos_client.get_database_client(DATABASE_NAME)
        # Retrieve the product container
        container = database.get_container_client(CONTAINER_NAME)
        
        query_results = container.query_items(
            query = """
            SELECT TOP @num_results p.name, p.description, p.sku, p.price, p.discount, p.sale_price, VectorDistance(p.embedding, @query_embedding) AS similarity_score
            FROM Products p
            WHERE VectorDistance(p.embedding, @query_embedding) > @similarity_score
            ORDER BY VectorDistance(p.embedding, @query_embedding)
            """,
            parameters = [
                {"name": "@query_embedding", "value": query_embedding},
                {"name": "@num_results", "value": num_results},
                {"name": "@similarity_score", "value": similarity_score}
            ]
        )
        similar_products = []
        async for result in query_results:
            similar_products.append(result)
        formatted_results = [{'similarity_score': product.pop('similarity_score'), 'product': product} for product in similar_products]
        return formatted_results
    ```
    
2. Next, create a method named `get_similar_products` that will serve as the function used by the LLM to perform vector searches against your database:
    
    ```python
    async def get_similar_products(message: str, num_results: int):
        """Retrieve similar products based on a user message."""
        # Vectorize the message
        embedding = await generate_embeddings(message)
        # Perform vector search against products in Cosmos DB
        similar_products = await vector_search(embedding, num_results=num_results)
        return similar_products
    ```
    
    The `get_similar_products` function makes asynchronous calls to the `vector_search` function you defined above, as well as the `generate_embeddings` function you created in the previous exercise. Embeddings are generated on the incoming user message to allow it to be compared to vectors stored in the database using the built-in `VectorDistance` function in Cosmos DB.
    
3. To allow the LLM to use the new functions, you must update the `tools` array you created earlier, adding a function definition for the `get_similar_products` method:
    
    ```json
    {
        "type": "function",
        "function": {
            "name": "get_similar_products",
            "description": "Retrieve similar products based on a user message.",
            "parameters": {
                "type": "object",
                "properties": {
                    "message": {"type": "string", "description": "The user's message looking for similar products"},
                    "num_results": {"type": "integer", "description": "The number of similar products to return"}
                },
                "required": ["message"]
            }
        }
    }
    ```
    
4. You must also add code to handle the new function’s output. Add the following `elif` condition to the code block that handles function call outputs:
    
    ```python
    elif call.function.name == "get_similar_products":
        func_response = await get_similar_products(**json.loads(call.function.arguments))
        messages.append(
            {
                "role": "tool",
                "tool_call_id": call.id,
                "name": call.function.name,
                "content": json.dumps(func_response)
            }
        )
    ```
    
    The completed block with now look like this:
    
    ```python
    # Handle function call outputs
    if response_message.tool_calls:
        for call in response_message.tool_calls:
            if call.function.name == "apply_discount":
                func_response = await apply_discount(**json.loads(call.function.arguments))
                messages.append(
                    {
                        "role": "tool",
                        "tool_call_id": call.id,
                        "name": call.function.name,
                        "content": func_response
                    }
                )
            elif call.function.name == "get_category_names":
                func_response = await get_category_names()
                messages.append(
                    {
                        "role": "tool",
                        "tool_call_id": call.id,
                        "name": call.function.name,
                        "content": json.dumps(func_response)
                    }
                )
            elif call.function.name == "get_similar_products":
                func_response = await get_similar_products(**json.loads(call.function.arguments))
                messages.append(
                    {
                        "role": "tool",
                        "tool_call_id": call.id,
                        "name": call.function.name,
                        "content": json.dumps(func_response)
                    }
                )
    else:
        print("No function calls were made by the model.")
    ```
    
5. Lastly, you need to update the system prompt definition to provide instructions on how to perform vector searches. Insert the following at the bottom of the `system_prompt`:
    
    ```plaintext
    When asked to provide a list of products, you should:
        - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
    ```
    
    The updated system prompt will be similar to:
    
    ```python
    system_prompt = """
    You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
    You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
    If asked to apply a discount:
        - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
        - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
    If asked to remove discounts from a category:
        - Remove any discounts applied to products in the specified category by setting the discount value to 0.
    When asked to provide a list of products, you should:
        - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
    """
    ```
    
6. Save the `main.py` file.
    

#### Test the vector search feature

1. Restart the API app by running the following in the open integrated terminal window for that app in Visual Studio Code:
    
    ```bash
    uvicorn main:app
    ```
    
2. The UI should still be running, but if you stopped it, return to the integrated terminal window for it and run:
    
    ```bash
    python -m streamlit run index.py
    ```
    
3. Return to the browser window running the UI, and at the chat prompt, enter the following:
    
    ```bash
    Tell me about the mountain bikes in stock
    ```
    
    This question will return a few products that match your search.
    
4. Try a few other searches, such as “Show me durable pedals,” “Provide a list of 5 stylish jerseys,” and “Give me details about all gloves suitable for warm weather riding.”
    
    For the last two queries, observe that the products contain the 15% discount and sale price you applied previously.

### Integrate LangChain orchestration to improve efficiency and code maintainability in a Python Generative AI application

Azure Cosmos DB seamlessly integrates with leading large language model (LLM) orchestration packages like Semantic Kernel and LangChain, enabling you to harness the power of advanced AI capabilities within your applications. These orchestration packages can streamline the management and use of LLMs, embedding models, and databases, making it even easier to develop advanced AI Generative AI applications.

#### Integrate LangChain orchestration

LangChain is a powerful tool that enhances the integration and coordination of multiple AI models and tools to create complex and dynamic AI applications. By using orchestration capabilities, LangChain allows you to seamlessly combine various language models, APIs, and custom components into a unified workflow. This orchestration ensures that each element works together efficiently, enabling the creation of sophisticated applications capable of performing various tasks, from natural language understanding and generation to information retrieval and data analysis.

LangChain's orchestration capabilities are beneficial when building a Generative AI application using Python and Azure Cosmos DB for NoSQL. Generative AI applications must often combine natural language processing (NLP) models, knowledge retrieval systems, and custom logic to provide accurate and contextually relevant responses. LangChain facilitates this process by orchestrating various NLP models and APIs, ensuring the Generative AI application can effectively understand and generate responses to user queries.

Moreover, integrating Azure Cosmos DB for NoSQL with LangChain provides a scalable and flexible database solution that can handle large volumes of data with low latency. The Cosmos DB Vector Search feature allows for high-performance retrieval of relevant information based on the semantic similarity of data, which is especially useful for NLP applications. This means the Generative AI application can perform sophisticated searches over large datasets, retrieving contextually relevant information for user queries.

LangChain's orchestration ensures that data from Azure Cosmos DB's vector search is seamlessly integrated with the AI models, enabling the Generative AI application to provide timely and accurate responses. Combining LangChain's orchestration and Cosmos DB's advanced search capabilities enhances the Generative AI application's ability to understand and interact with users more effectively.

#### Retrieval-augmented generation (RAG) with LangChain

Retrieval-augmented generation (RAG) is a pattern that combines retrieval and generation to enhance AI applications' performance and accuracy, making it a powerful tool when integrated with LangChain and Azure Cosmos DB for NoSQL's vector search feature. By using LangChain's orchestration capabilities, RAG can seamlessly combine the retrieval of relevant information with the generative power of AI models. Azure Cosmos DB's vector search feature is critical in this process by enabling high-performance retrieval of semantically similar data from large datasets. This retrieval process ensures that the Generative AI application can access and utilize the most relevant information quickly and efficiently. When a user poses a query, the RAG model can retrieve contextually appropriate data from Cosmos DB using vector search and then generate a comprehensive, coherent response based on that data. This combination of retrieval and generation significantly enhances the Generative AI application's ability to provide accurate, context-aware answers, leading to a more robust and user-friendly experience.

#### Understand function calling and tools in LangChain

Function calling in LangChain offers a more structured and flexible approach compared to using the Azure OpenAI client directly in Python. In LangChain, you can define and manage functions as modular components that are easily reusable and maintainable. This approach allows for more organized code, where each function encapsulates a specific task, reducing complexity and making the development process more efficient.

When using the Azure OpenAI client in Python, function calls are typically limited to direct API interactions. While you can still build complex workflows, it often requires more manual orchestration and handling of asynchronous operations, which can become cumbersome and harder to maintain as the application grows.

LangChain's tools play a crucial role in enhancing function calling. With a vast array of built-in tools and the ability to integrate external ones, LangChain allows you to create sophisticated pipelines where functions can call tools to perform specific operations, such as data retrieval, processing, or transformation. These tools can be configured to operate conditionally or in parallel, further optimizing the application's performance. Additionally, LangChain's tools simplify error handling and debugging by isolating functions and tools, making it easier to identify and resolve issues.

### Implement RAG with LangChain and Azure Cosmos DB for NoSQL Vector Search

LangChain’s orchestration capabilities bring a multitude of benefits over implementing your copilot’s LLM integration using the Azure OpenAI client directly. LangChain allows for more seamless integration with various data sources, including Azure Cosmos DB, enabling efficient vector search that enhances the retrieval process. LangChain offers robust tools for managing and optimizing workflows, making it easier to build complex applications with modular and reusable components. This flexibility not only simplifies development but also ensures scalability and maintainability.

In this lab, you will enhance your copilot by transitioning your API’s `/chat` endpoint from using the Azure OpenAI client to leveraging LangChain’s powerful orchestration capabilities. This shift will enable more efficient data retrieval and improved performance by integrating vector search functionality with Azure Cosmos DB for NoSQL. Whether you are looking to optimize your app’s information retrieval process or simply explore the potential of RAG, this module will guide you through the seamless conversion, demonstrating how LangChain can streamline and elevate your app’s capabilities. Let’s embark on this journey to unlock new efficiencies and insights with LangChain and Azure Cosmos DB!

> 🛑 The previous exercises in this module are prerequisites for this lab. If you still need to complete any of those exercises, please finish them before continuing, as they provide the necessary infrastructure and starter code for this lab.

#### Install the LangChain libraries

1. Using Visual Studio Code, open the folder into which you cloned the lab code repository for **Build copilots with Azure Cosmos DB** learning module.
    
2. In the **Explorer** pane within Visual Studio Code, browse to the **python/07-build-copilot** folder and open the `requirements.txt` file found within it.
    
3. Update the `requirements.txt` file to include the required LangChain libraries:
    
    ```
    langchain==0.3.9
    langchain-openai==0.2.11
    ```
    
4. Launch a new integrated terminal window in Visual Studio Code and change directories to `python/07-build-copilot`.
    
5. Ensure the integrated terminal window runs within your Python virtual environment by activating it using the appropriate command for your OS and shell from the following table:
    
    |Platform|Shell|Command to activate virtual environment|
    |---|---|---|
    |POSIX|bash/zsh|`source .venv/bin/activate`|
    ||fish|`source .venv/bin/activate.fish`|
    ||csh/tcsh|`source .venv/bin/activate.csh`|
    ||pwsh|`.venv/bin/Activate.ps1`|
    |Windows|cmd.exe|`.venv\Scripts\activate.bat`|
    ||PowerShell|`.venv\Scripts\Activate.ps1`|
    
6. Update your virtual environment with the LangChain libraries by executing the following command at the integrated terminal prompt:
    
    ```bash
    pip install -r requirements.txt
    ```
    
7. Close the integrated terminal.
    

#### Update the backend API

In the previous lab, you executed a RAG pattern using the Azure OpenAI client and data from Azure Cosmos DB. Now, you will update the backend API to use a LangChain agent with tools to perform the same actions.

Using LangChain to interact with language models deployed in your Azure OpenAI Service is somewhat simplier from a code standpoint…

1. Remove the `from openai import AzureOpenAI` import statement at the top of the `main.py` file. That client library is no longer needed, as all interactions with Azure OpenAI will go through LangChain-provided classes.
    
2. Delete the following import statements at the top of the `main.py` file, as they will no longer necessary:
    
    ```python
    from openai import AsyncAzureOpenAI
    import json
    ```
    

##### Update embedding endpoint

1. Import the `AzureOpenAIEmbeddings` class from the `langchain_openai` library by adding the following import statement at the top of the `main.py` file:
    
    ```python
    from langchain_openai import AzureOpenAIEmbeddings
    ```
    
2. Locate the `generate_embeddings` method in the file and overwrite it with the following, which uses the `AzureOpenAIEmbeddings` class to handle interactions with Azure OpenAI:
    
    ```python
    async def generate_embeddings(text: str):
        """Generates embeddings for the provided text."""
        # Use LangChain's Azure OpenAI Embeddings class
        azure_openai_embeddings = AzureOpenAIEmbeddings(
            azure_deployment = EMBEDDING_DEPLOYMENT_NAME,
            azure_endpoint = AZURE_OPENAI_ENDPOINT,
            azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
        )
        return await azure_openai_embeddings.aembed_query(text)
    ```
    
    The `AzureOpenAIEmbeddings` class provides an interface for interacting with the Azure OpenAI Embeddings API, returning a simplified response object containing only the generated vector.
    
##### Update chat endpoint

1. Update the `lanchain_openai` import statement to append the `AzureChatOpenAI` class:
    
    ```python
    from langchain_openai import AzureOpenAIEmbeddings, AzureChatOpenAI
    ```
    
2. Import the following additional LangChain objects that will be used when building out the revised `/chat` endpoint:
    
    ```python
    from langchain.agents import AgentExecutor, create_openai_functions_agent
    from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
    from langchain_core.tools import StructuredTool
    ```
    
3. The chat history will be injected into the copilot conversation differently using a LangChain agent, so delete the lines of code immediately following the `system_prompt` definition. The line you should delete are:
    
    ```python
    # Provide the copilot with a persona using the system prompt.
    messages = [{ "role": "system", "content": system_prompt }]
    
    # Add the chat history to the messages list
    for message in request.chat_history[-request.max_history:]:
        messages.append(message)
    
    # Add the current user message to the messages list
    messages.append({"role": "user", "content": request.message})
    ```
    
4. In place of the code you just deleted, define a `prompt` object using LangChain’s `ChatPromptTemplate` class:
    
    ```python
    prompt = ChatPromptTemplate.from_messages(
        [
            ("system", system_prompt),
            MessagesPlaceholder("chat_history", optional=True),
            ("user", "{input}"),
            MessagesPlaceholder("agent_scratchpad")
        ]
    )
    ```
    
    The `ChatPromptTemplate` is being created with several components in a specific order. Here’s how those peices fit together:
    
    - **System Message**: Uses the `system_prompt` to gives a persona to the copilot, providing instructions on how the assistant should behave and interact with users.
    - **Chat History**: Allows the `chat_history`, containing a list of past messages in the conversation, to be incorporated into the context over which the LLM is working.
    - **User Input**: The current user message.
    - **Agent Scratchpad**: Allows for intermediate notes or steps taken by the agent.
    
    The resulting prompt provides a structured input for the conversational AI agent, helping it to generate a response based on the given context.
    
5. Next, replace the `tools` array definition with the following, which uses LangChain’s `StructuredTool` class to extract function definitions into the proper format:
    
    ```python
    tools = [
        StructuredTool.from_function(coroutine=apply_discount),
        StructuredTool.from_function(coroutine=get_category_names),
        StructuredTool.from_function(coroutine=get_similar_products)
    ]
    ```
    
    The `StructuredTool.from_function` method in LangChain creates a tool from a given function, using the input parameters and the function’s docstring description. To use it with async methods, you specify pass the function name to the `coroutine` input parameter.
    
    In Python, a docstring (short for documentation string) is a special type of string used to document a function, method, class, or module. It provides a convenient way of associating documentation with Python code and is typically enclosed within triple quotes (“”” or ‘’’). Docstrings are placed immediately after the definition of the function (or method, class, or module) they document.
    
    Using this function automates the creation of the JSON function definitions you had to manually create using the Azure OpenAI client, simplifying the process of function calling.
    
6. Delete all of the code between the `tools` array definition you completed above and the `return` statement at the end of the function. Using the Azure OpenAI client, you had to make two calls the the language model. The first to allow it to determine what function calls, if any, it needs to make to augment the prompt, and the second to ask for a RAG completion. In between, you had to use code to inspect the response from the first call to determine if function calls were required, and then write code to “handle” calling those functions. You then had to insert the output of those function calls into the messages being sent to the LLM, so it could have the enriched prompt to reason of when formulating a completion response. LangChain greatly simplifies the process of calling an LLM using a RAG pattern, as you will see below. The code you should remove is:
    
    ```python
    # Create Azure OpenAI client
    aoai_client = AsyncAzureOpenAI(
        api_version = AZURE_OPENAI_API_VERSION,
        azure_endpoint = AZURE_OPENAI_ENDPOINT,
        azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
    )
    
    # First API call, providing the model to the defined functions
    response = await aoai_client.chat.completions.create(
        model = COMPLETION_DEPLOYMENT_NAME,
        messages = messages,
        tools = tools,
        tool_choice = "auto"
    )
    
    # Process the model's response
    response_message = response.choices[0].message
    messages.append(response_message)
    
    # Handle function call outputs
    if response_message.tool_calls:
        for call in response_message.tool_calls:
            if call.function.name == "apply_discount":
                func_response = await apply_discount(**json.loads(call.function.arguments))
                messages.append(
                    {
                        "role": "tool",
                        "tool_call_id": call.id,
                        "name": call.function.name,
                        "content": func_response
                    }
                )
            elif call.function.name == "get_category_names":
                func_response = await get_category_names()
                messages.append(
                    {
                        "role": "tool",
                        "tool_call_id": call.id,
                        "name": call.function.name,
                        "content": json.dumps(func_response)
                    }
                )
            elif call.function.name == "get_similar_products":
                func_response = await get_similar_products(**json.loads(call.function.arguments))
                messages.append(
                    {
                        "role": "tool",
                        "tool_call_id": call.id,
                        "name": call.function.name,
                        "content": json.dumps(func_response)
                    }
                )
    else:
        print("No function calls were made by the model.")
    
    # Second API call, asking the model to generate a response
    final_response = await aoai_client.chat.completions.create(
        model = COMPLETION_DEPLOYMENT_NAME,
        messages = messages
    )
    
    return final_response.choices[0].message.content
    ```
    
7. Working from just below the `tools` array definition, create a reference to the Azure OpenAI API using the `AzureChatOpenAI` class in LangChain:
    
    ```python
    # Connect to Azure OpenAI API
    azure_openai = AzureChatOpenAI(
        azure_deployment=COMPLETION_DEPLOYMENT_NAME,
        azure_endpoint=AZURE_OPENAI_ENDPOINT,
        azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
        api_version=AZURE_OPENAI_API_VERSION
    )
    ```
    
8. To allow your LangChain agent to interact with the functions you’ve defined, you will create an agent using the `create_openai_functions_agent` method, to which you will provide the `AzureChatOpenAI` objedt, `tools` array, and `ChatPromptTemplate` object:
    
    ```python
    agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
    ```
    
    The `create_openai_functions_agent` function in LangChain creates an agent that can call external functions to perform tasks using a specified language model and tools. This enables the integration of various services and functionalities into the agent’s workflow, providing flexibility and enhanced capabilities.
    
9. In LangChain, the `AgentExecutor` class is used to manage the execution flow of the agents, such as the one you created with the `create_openai_functions_agent` method. It handles the processing of inputs, the invocation of tools or models, and the handling of outputs. Use the below code to create an agent executor for your agent:
    
    ```python
    agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
    ```
    
    The `AgentExecutor` ensures that all the steps required to generate a response are executed in the correct order. It abstracts the complexities of execution for agents, providing an additional layer of functionality and structure, and making it easier to build, manage, and scale sophisticated agents.
    
10. You will use the agent executor’s `invoke` method to send the incoming user message to the LLM. You will also include the chat history. Insert the following code below the `agent_executor` definition:
    
    ```python
    completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
    ```
    
    The `input` and `chat_history` tokens were defined in the prompt object created using the `ChatPromptTemplate`. With the `invoke` method, these will be injected into the prompt, allowing the LLM to use that information when creating a response.
    
11. Finally, update the return statement to use the `output` of the agent’s completion object:
    
    ```python
    return completion["output"]
    ```
    
12. Save the `main.py` file. The updated `/chat` endpoint function should now look like this:
    
    ```python
    @app.post('/chat')
    async def generate_chat_completion(request: CompletionRequest):
        """Generate a chat completion using the Azure OpenAI API."""
        # Define the system prompt that contains the assistant's persona.
        system_prompt = """
        You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
        You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
        If asked to apply a discount:
            - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
            - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
        If asked to remove discounts from a category:
            - Remove any discounts applied to products in the specified category by setting the discount value to 0.
        When asked to provide a list of products, you should:
            - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
        """
        prompt = ChatPromptTemplate.from_messages(
            [
                ("system", system_prompt),
                MessagesPlaceholder("chat_history", optional=True),
                ("user", "{input}"),
                MessagesPlaceholder("agent_scratchpad")
            ]
        )
        
        # Define function calling tools
        tools = [
            StructuredTool.from_function(apply_discount),
            StructuredTool.from_function(get_category_names),
            StructuredTool.from_function(get_similar_products)
        ]
        
        # Connect to Azure OpenAI API
        azure_openai = AzureChatOpenAI(
            azure_deployment=COMPLETION_DEPLOYMENT_NAME,
            azure_endpoint=AZURE_OPENAI_ENDPOINT,
            azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
            api_version=AZURE_OPENAI_API_VERSION
        )
        
        agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
        agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
            
        completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
                
        return completion["output"]
    ```
    

#### Start the API and UI apps

1. To start the API, open a new integrated terminal window in Visual Studio Code.
    
2. Ensure you are logged into Azure using the `az login` command. Running the following at the terminal prompt:

    
    ```bash
    az login
    ```
    
3. Complete the login process in your browser.
    
4. Change directories to `python/07-build-copilot` at the terminal prompt.
    
5. Ensure the integrated terminal window runs within your Python virtual environment by activating it using a command from the table below and selecting the appropriate command for your OS and shell.
    
    |Platform|Shell|Command to activate virtual environment|
    |---|---|---|
    |POSIX|bash/zsh|`source .venv/bin/activate`|
    ||fish|`source .venv/bin/activate.fish`|
    ||csh/tcsh|`source .venv/bin/activate.csh`|
    ||pwsh|`.venv/bin/Activate.ps1`|
    |Windows|cmd.exe|`.venv\Scripts\activate.bat`|
    ||PowerShell|`.venv\Scripts\Activate.ps1`|
    
6. At the terminal prompt, change directories to `api/app`, then execute the following command to run the FastAPI web app:
    
    ```bash
    uvicorn main:app
    ```
    
7. Open a new integrated terminal window, change directories to `python/07-build-copilot` to activate your Python environment, then change directories to the `ui` folder and run the following to start your UI app:
    
    ```bash
    python -m streamlit run index.py
    ```
    
8. If the UI does not open automatically in a browser window, launch a new browser tab or window and navigate to [http://localhost:8501](http://localhost:8501/) to open the UI.
    

#### Test the copilot

1. Before sending messages into the UI, return to Visual Studio Code and select the integrated terminal window associated with the API app. Within this window, you will see the “verbose” ouptut generated by the LangChain agent executor, which provides insights into how LangChain is handling the requests you send in. Pay attention to the output in this window as you send in the below requests, checking back in after each call.
    
2. At the chat prompt in the UI, enter “Apply a discount” and send the message.
    
    You should receive a reply asking for the discount percentage you would like to appy, and for what product category.
    
3. Reply, “Gloves.”
    
    You will receive a response asking for what discount percentage would you like to apply to the “Gloves” category.
    
4. Send a message of “25%.”
    
    You should get a response of “A 25% discount has been successfully applied to all products in the “Gloves” category.”
    
5. Ask the copilot to “show me all gloves.”
    
    In the reply, you should see a list of all gloves in database, which will include the 25% discount price.
    
6. Finally, ask “What gloves are best cold weather riding?” to perform a vector search. This involves a function call to the `get_similar_items` method, which then calls both the `generate_embeddings` method you updated to use a LangChain implementation and the `vector_search` function.
    
7. Close the integrated terminal.
    
8. Close **Visual Studio Code**.