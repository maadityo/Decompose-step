
# Handson : Deploy Aplikasi Monolith di App Service
Arsitektur diagram : 

![image](https://user-images.githubusercontent.com/23251706/146795722-6a7bf249-f742-4820-ba26-061a1b20b637.png)


### 0. Create Resource Group
```console
az group create --resource-group RG-POS-Demo --location southeastasia
```

### 1. Create Resources
Azure CLI
```console
az deployment group create \
    --resource-group [RG_NAME] \
    --template-uri https://raw.githubusercontent.com/MicrosoftDocs/mslearn-microservices-architecture/master/deployment/azuredeploy.json
```
Now that we have the resources created, let's deploy the application. First, run the following command to pull down the source code from the sample repository.

### 2. Deploy Aplikasi
Azure CLI
```console
git clone https://github.com/MicrosoftDocs/mslearn-microservices-architecture.git ~/mslearn-microservices-architecture
cd ~/mslearn-microservices-architecture/src/before
Run the following command to zip up the application code, which we use to deploy to the app service.
```
### 3. Unzip code yang akan dipakai
Bash
```console
zip -r DroneDelivery-before.zip .
Run the following command to set a variable with the name of your app service.
```

### 4. Set variable app service
Run the following command to configure the app service to run a build as part of the deployment.

```console
APPSERVICENAME="$(az webapp list \
                    --resource-group [sandbox resource group] \
                    --query '[].name' \
                    --output tsv)"

```

### 5. Configure app service
Azure CLI
```console
az webapp config appsettings set \
    --resource-group [sandbox resource group] \
    --name $APPSERVICENAME \
    --settings SCM_DO_BUILD_DURING_DEPLOYMENT=true
```
And now, run the following command to deploy the application to App Service. This deployment takes a few minutes to finish.

### 6. Deploy aplikasi ke app service
Azure CLI

```console
az webapp deployment source config-zip \
    --resource-group [sandbox resource group] \
    --name $APPSERVICENAME \
    --src DroneDelivery-before.zip
```
After the deployment finishes, confirm that the deployment was successful by visiting the website of your app service. Run the following command to get the URL, and select it to open the page.

### 7. Pastikan App terdeploy dengan sempurna dan tampilan berikut akan muncul
```console
echo https://$(az webapp config hostname list \
                --resource-group [sandbox resource group] \
                --webapp-name $APPSERVICENAME \
                --query [].name \
                --output tsv)
```
Screenshot of the Drone Delivery website.

![image](https://user-images.githubusercontent.com/23251706/146795592-814d2be1-0d1f-4439-9bb4-9a5a6c507bb7.png)
