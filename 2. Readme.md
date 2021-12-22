
# Handson 2 : Refaktor dari Aplikasi Monolith ke microservices
Arsitektur diagram : 

![image](https://user-images.githubusercontent.com/23251706/146799179-164fc285-48a5-4fc2-b0d9-ee39cd2cdff6.png)



### Drone delivery before
```console
public class PackageProcessor : IPackageProcessor
    {
        public Task<PackageGen> CreatePackageAsync(PackageInfo packageInfo)
        {
            //Uses common data store e.g. SQL Azure tables
            Utility.DoWork(100);
            return Task.FromResult(new PackageGen { Id = packageInfo.PackageId });
        }
    }
```

### Drone delivery after
Azure CLI
```console
public class PackageServiceCaller : IPackageProcessor
    {
        private readonly HttpClient httpClient;

        public static string FunctionCode { get; set; }

        public PackageServiceCaller(HttpClient httpClient)
        {
            this.httpClient = httpClient;
        }

        public async Task<PackageGen> CreatePackageAsync(PackageInfo packageInfo)
        {
            var result = await httpClient.PutAsJsonAsync($"{packageInfo.PackageId}?code={FunctionCode}", packageInfo);
            result.EnsureSuccessStatusCode();

            return new PackageGen { Id = packageInfo.PackageId };
        }
    }
```

### PackageServiceFunction
Azure CLI
```console
public static class PackageServiceFunction
    {
        [FunctionName("PackageServiceFunction")]
        public static Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "put", Route = "packages/{id}")] HttpRequest req,
            string id, ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            //Uses common data store e.g. SQL Azure tables
            Utility.DoWork(100);
            return Task.FromResult((IActionResult)new CreatedResult("http://example.com", null));
        }
    }
```



### 1. Deploy Azure function \ function app
Bash
```console
APPSERVICENAME="$(az webapp list \
                    --resource-group [sandbox resource group] \
                    --query '[].name' \
                    --output tsv)"
FUNCTIONAPPNAME="$(az functionapp list \
                    --resource-group [sandbox resource group] \
                    --query '[].name' \
                    --output tsv)"
```



### 2. Set variable app service
Run the following command to configure the app service to run a build as part of the deployment.

```console
APPSERVICENAME="$(az webapp list \
                    --resource-group [sandbox resource group] \
                    --query '[].name' \
                    --output tsv)"

```

### 3. Build dan zip code aplikasi untuk function app
Azure CLI
```console
cd ~/mslearn-microservices-architecture/src/after
dotnet build ./PackageService/PackageService.csproj -c Release
cd PackageService/bin/Release/netcoreapp2.2
zip -r PackageService.zip .
```


### 4. Push Code ke Function App
Azure CLI
```console
az functionapp deployment source config-zip \
    --resource-group [sandbox resource group] \
    --name $FUNCTIONAPPNAME \
    --src PackageService.zip
```


### 5. Deploy microservices Drone delivery 
```console
RESOURCEGROUPID=$(az group show \
                    --resource-group [sandbox resource group] \
                    --query id \
                    --output tsv)
FUNCTIONCODE=$(az rest \
                    --method post \
                    --query default \
                    --output tsv \
                    --uri "https://management.azure.com$RESOURCEGROUPID/providers/Microsoft.Web/sites/$FUNCTIONAPPNAME/functions/PackageServiceFunction/listKeys?api-version=2018-02-01")
echo "FunctionName - $FUNCTIONAPPNAME"
echo "FunctionCode - $FUNCTIONCODE"
```

### 6. Edit Appsettings.json
In the code editor, replace the values PackageServiceUri and PackageServiceFunctionCode. In PackageServiceUri, replace <FunctionName> with the name of your function app.

In PackageServiceFunctionCode, replace the <FunctionCode> with the function code you retrieved. Your appsettings.json file should look similar to this:
Azure CLI
```console
cd ~/mslearn-microservices-architecture/src/after
code ./DroneDelivery-after/appsettings.json
```
JSON
```console
{
    "Logging": {
    "LogLevel": {
        "Default": "Warning"
    }
    },
    "AllowedHosts": "*",
    "PackageServiceUri": "https://packageservicefunction-abc.azurewebsites.net/api/packages/",
    "PackageServiceFunctionCode": "SvrbiyhjXJUdTPXrkcUtY6bQaUf7OXQjWvnM0Gq63hFUhbH2vn6qYA=="
}
```
Press Ctrl+S to save the file, and then Ctrl+Q to close the code editor.

### 7. Deploy updated aplikasi ke app service
Azure CLI
```console
zip -r DroneDelivery-after.zip . -x \*/obj/\* \*/bin/\*
az webapp deployment source config-zip \
    --resource-group [sandbox resource group] \
    --name $APPSERVICENAME \
    --src DroneDelivery-after.zip
```
Screenshot of the Drone Delivery website Microservices.

![image](https://user-images.githubusercontent.com/23251706/146800369-23e392e1-f390-4182-b7e5-adf0b5c19eb4.png)
