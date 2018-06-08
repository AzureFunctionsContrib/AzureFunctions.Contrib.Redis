# Redis bindings for Azure Functions.
 Allows a function to interact with Redis. Following operations are currently supported: 
- add/insert item to lists
- set a key
- increment a key value

Reading or more complex operations can be achieved by using the ```[RedisDatabase]``` attribute which will resolve a IDatabase.

## Installation
Install package AzureFunctions.Contrib.Redis

```bash
dotnet package add AzureFunctions.Contrib.Redis
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