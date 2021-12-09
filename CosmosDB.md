# CosmosDB
## CosmosDB classes
[CosmosDB classes](CosmosdbClasses.md)

## SQL API
The SQL API calls the items as documents which come inside a collection.

```c#
private static async Task CreateNewItem()
{
    using (CosmosClient cosmos_client = new CosmosClient(endpoint, accountkeys))
    {
        Database db_conn = cosmos_client.GetDatabase(database);
        Container container_conn = db_conn.GetContainer(containername);
        customer obj = new customer(1, "John", "Miami");
        obj.id = Guid.NewGuid().ToString();

        ItemResponse<customer> response = await container_conn.CreateItemAsync(obj);
        Console.WriteLine("Request charge is {0}", response.RequestCharge);
        Console.WriteLine("Customer added");
    }
}

private static async Task ReadItem()
{
    using (CosmosClient cosmos_client = new CosmosClient(endpoint, accountkeys))
    {
        Database db_conn = cosmos_client.GetDatabase(database);
        Container container_conn = db_conn.GetContainer(containername);
        string cosmos_sql = "select c.customerid,c.customername,c.city from c";
        QueryDefinition query = new QueryDefinition(cosmos_sql);

        FeedIterator<customer> iterator_obj = container_conn.GetItemQueryIterator<customer>(cosmos_sql);
        while (iterator_obj.HasMoreResults)
        {
            FeedResponse<customer> customer_obj = await iterator_obj.ReadNextAsync();
            foreach (customer obj in customer_obj)
            {
                Console.WriteLine("Customer id is {0}", obj.customerid);
                Console.WriteLine("Customer name is {0}", obj.customername);
                Console.WriteLine("Customer city is {0}", obj.city);
            }
        }
    }
} 
private static async Task ReplaceItem()
{
    using (CosmosClient cosmos_client = new CosmosClient(endpoint, accountkeys))
    {
	Database db_conn = cosmos_client.GetDatabase(database);
	Container container_conn = db_conn.GetContainer(containername);

	PartitionKey pk = new PartitionKey("Miami");
	string id = "0e31ed86-9824-4fe6-a6de-a7853348f344";

	ItemResponse<customer> response = await container_conn.ReadItemAsync<customer>(id, pk);
	customer customer_obj = response.Resource;

	customer_obj.customername = "James";
	response = await container_conn.ReplaceItemAsync<customer>(customer_obj, id, pk);
	Console.WriteLine("Item updated");
    }
}

private static async Task DeleteItem()
{
    using (CosmosClient cosmos_client = new CosmosClient(endpoint, accountkeys))
    {
	Database db_conn = cosmos_client.GetDatabase(database);
	Container container_conn = db_conn.GetContainer(containername);
	PartitionKey pk = new PartitionKey("Miami");
	string id = "0e31ed86-9824-4fe6-a6de-a7853348f344";
	
	ItemResponse<customer> response = await container_conn.DeleteItemAsync<customer>(id, pk);
	Console.WriteLine("Item deleted");
    }
    }
}
```

## Trigger

```js
function Append(){
	var context=getContext();
	var request=context.getRequest();
	var document=request.getBody();

	document["orders"]=0;
	request.setBody(document);
}
```

```c#
using (CosmosClient cosmos_client = new CosmosClient(endpoint, accountkeys))
{
    Database db_conn = cosmos_client.GetDatabase(database);
    Container container_conn = db_conn.GetContainer(containername);
    customer obj = new customer(5, "David", "New York");
    ItemResponse<customer> response = await container_conn.CreateItemAsync(obj,null,
        new ItemRequestOptions { PreTriggers = new List<string> { "AppendDocument" } });
    Console.WriteLine("Request charge is {0}", response.RequestCharge);
    Console.WriteLine("Customer added");
}
```

## Stored procedure
```js
using (CosmosClient cosmos_client = new CosmosClient(endpoint, accountkeys))
{
    Database db_conn = cosmos_client.GetDatabase(database);
    Container container_conn = db_conn.GetContainer(containername);
    var stored_procedures = container_conn.Scripts;
    PartitionKey key = new PartitionKey(string.Empty);
    var response = await stored_procedures.ExecuteStoredProcedureAsync<string>("demo", key, null);
    Console.WriteLine(response.Resource);
}
```
## Trigger
- Pre
- Post


## CosmosDB Table API

The Table API provides the same query functionality as Azure Table storage.
In the case of Table API, it is called as Rows. 

Each table defines:
- PartitionKey
- RowKey

```c#
class Customer :TableEntity
    {
    public string customername { get; set; }
    public Customer(string _customername, string _city,string _customerid)
    {
        PartitionKey = _city;
        RowKey = _customerid;
        customername = _customername;
    }
}
```

```c#
static async Task NewItem()
{
    CloudStorageAccount p_account = CloudStorageAccount.Parse(connection_string);

    CloudTableClient p_tableclient = p_account.CreateCloudTableClient();

    CloudTable p_table = p_tableclient.GetTableReference("Customer");

    Customer obj = new Customer("2", "James", "New York");
    TableOperation p_operation = TableOperation.Insert(obj);
    TableResult response = await p_table.ExecuteAsync(p_operation);

    Console.WriteLine("Entity added");
}

static async Task ReadItem()
{
    CloudStorageAccount p_account = CloudStorageAccount.Parse(connection_string);

    CloudTableClient p_tableclient = p_account.CreateCloudTableClient();

    CloudTable p_table = p_tableclient.GetTableReference("Customer");

    string partition_key = "1";
    string rowkey = "John";

    TableOperation p_operation = TableOperation.Retrieve<Customer>(partition_key, rowkey);
    TableResult response = await p_table.ExecuteAsync(p_operation);

    Customer return_obj = (Customer)response.Result;

    Console.WriteLine("Customer ID is {0}", return_obj.PartitionKey);
    Console.WriteLine("Customer Name is {0}", return_obj.RowKey);
    Console.WriteLine("Customer City is {0}", return_obj.city);
}

static async Task UpdateItem()
{
    CloudStorageAccount p_account = CloudStorageAccount.Parse(connection_string);

    CloudTableClient p_tableclient = p_account.CreateCloudTableClient();

    CloudTable p_table = p_tableclient.GetTableReference("Customer");

    string partition_key = "1";
    string rowkey = "John";

    Customer updated_obj = new Customer(partition_key, rowkey, "Chicago");

    TableOperation p_operation = TableOperation.InsertOrReplace(updated_obj);
    TableResult response = await p_table.ExecuteAsync(p_operation);
    Console.WriteLine("Entity updated");
}

static async Task DeleteItem()
{
    CloudStorageAccount p_account = CloudStorageAccount.Parse(connection_string);

    CloudTableClient p_tableclient = p_account.CreateCloudTableClient();

    CloudTable p_table = p_tableclient.GetTableReference("Customer");

    string partition_key = "1";
    string rowkey = "John";

    TableOperation p_operation = TableOperation.Retrieve<Customer>(partition_key, rowkey);
    TableResult response = await p_table.ExecuteAsync(p_operation);

    Customer return_obj = (Customer)response.Result;


    TableOperation p_delete = TableOperation.Delete(return_obj);

    response = await p_table.ExecuteAsync(p_delete);
    Console.WriteLine("Entity deleted");
}
```

## CLI

```bash
uniqueId=$RANDOM
resourceGroupName="Group-$uniqueId"
location='westus2'
accountName="cosmos-$uniqueId" #needs to be lower case
databaseName='database1'
containerName='container1'

# Create a resource group
az group create -n $resourceGroupName -l $location

# Create a Cosmos account for SQL API
az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --default-consistency-level Eventual \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --locations regionName='East US 2' failoverPriority=1 isZoneRedundant=False

# --kind The type of Cosmos DB database account to create (GlobalDocumentDB, MongoDB, Parse)
# default value: GlobalDocumentDB

# Create a SQL API database
az cosmosdb sql database create \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName
# add regional failover for the account.
az cosmosdb update \
    -n $accountName \
    -g $resourceGroupName \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \

# Define the index policy for the container, include spatial and composite indexes
idxpolicy=$(cat << EOF 
{
    "indexingMode": "consistent", 
    "includedPaths": [
        {"path": "/*"}
    ],
    "excludedPaths": [
        { "path": "/headquarters/employees/?"}
    ],
    "spatialIndexes": [
        {"path": "/*", "types": ["Point"]}
    ],
    "compositeIndexes":[
        [
            { "path":"/name", "order":"ascending" },
            { "path":"/age", "order":"descending" }
        ]
    ]
}
EOF
)
# Persist index policy to json file
echo "$idxpolicy" > "idxpolicy-$uniqueId.json"

# Create a SQL API container
az cosmosdb sql container create \
    -a $accountName \
    -g $resourceGroupName \
    -d $databaseName \
    -n $containerName \
    -p '/zipcode' \
    --throughput 400 \
    --idx @idxpolicy-$uniqueId.json

```