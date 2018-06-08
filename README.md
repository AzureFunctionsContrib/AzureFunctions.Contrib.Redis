# Redis bindings for Azure Functions

 Allows a function to interact with Redis. Following operations are currently supported: 
- add/insert item to lists
- set a key
- increment a key value

Other operations with Redis can be achieved using the ```[RedisDatabase]``` attribute which will resolve a IDatabase.

## Installation
Install package AzureFunctions.Contrib.Redis

```bash
dotnet package add AzureFunctions.Contrib.Redis
```
```PS
Install-Package AzureFunctions.Contrib.Redis
```

## Examples
### Simple key value set

```CSharp
/// <summary>
/// Sets a value in Redis
/// </summary>
/// <param name="req"></param>
/// <param name="redisItem"></param>
/// <param name="log"></param>
/// <returns></returns>
[FunctionName(nameof(SetValueInRedis))]
public static IActionResult SetValueInRedis(
    [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
    [RedisOutput(Connection = "%redis_connectionstring%", Key = "%redis_setvalueinredis_key%")]  out RedisOutput redisItem,
    TraceWriter log)
{
    string requestBody = new StreamReader(req.Body).ReadToEnd();

    redisItem = new RedisItem()
    {
        TextValue = requestBody
    };

    return new OkResult();
}
```

### Append value to a list

```CSharp
/// <summary>
/// Appends a value to a redis list
/// </summary>
/// <param name="req"></param>
/// <param name="redisItem"></param>
/// <param name="log"></param>
/// <returns></returns>
[FunctionName(nameof(AppendToListInRedis))]
public static IActionResult AppendToListInRedis(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
    [RedisOutput(Connection = "%redis_connectionstring%", Key = "myList", Operation = RedisOutputOperation.ListRightPush)]  out RedisOutput redisItem,
    TraceWriter log)
{
    string itemValue = new StreamReader(req.Body).ReadToEnd();
    if (string.IsNullOrEmpty(itemValue))
        itemValue = req.Query["value"].ToString();

    redisItem = new RedisOutput()
    {
        TextValue = itemValue
    };

    return new OkResult();
}
}
```

### Retrieve a list
 
```CSharp
/// <summary>
/// Retrieve the current value of a list in Redis
/// </summary>
/// <param name="req"></param>
/// <param name="redisItem"></param>
/// <param name="log"></param>
/// <returns></returns>
[FunctionName(nameof(RetrieveList))]
public static async Task<IActionResult> RetrieveList(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req,
    [RedisDatabase(Connection = "%redis_connectionstring%")] IDatabase db,
    TraceWriter log)
{
    var key = req.Query["key"].ToString();
    if (string.IsNullOrEmpty(key))
        return new BadRequestObjectResult("Please pass a key on the query string or in the request body");

    var list = await db.ListRangeAsync(key);
    var resultList = new List<string>();

    if (list != null)
    {
        foreach (var srcItem in list)
        {
            if (!srcItem.IsNullOrEmpty)
                resultList.Add((string)srcItem);
        }
    }
            
    return new OkObjectResult(resultList);
}
```