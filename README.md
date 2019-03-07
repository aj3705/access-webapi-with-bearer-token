# Accessing Azure AD protected Web APIs using bearer token (OAuth 2.0 client credentials grant flow)

## Summary
This article shows how to invoke an Azure AD protected Web API from any client application (native or web) using [OAuth 2.0 client credentials grant flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow).
Below are the key charateristics of the Web API. 

- Web API is deployed to Azure App Service
- Web API is protected by Azure AD Authentication

The [OAuth 2.0 client credentials grant flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow) as shown below below involves aquiring a bearer token from Azure AD token service and then invoking the Web API with that token.This method can be used by any client (native or web) to access the Web API from anywhere.

![client credentials grant flow](/images/ccgf.PNG)

This document provides step by step instructions for doing the following

- build a simple Web API with ASP.Net Core
- deploy the Web API to Azure App Service
- enable Azure AD authentication for the Web API
- successfully invoke the API from Postman using the bearer token obtained from Azure AD.

## Prerequisites
1. [Install Git](https://git-scm.com/)
1. [Install .Net Core](https://dotnet.microsoft.com/learn/dotnet/hello-world-tutorial/intro)
1. [Install Postman for windows](https://www.getpostman.com/downloads/)
1. **optional:** [Install visual studio code](https://code.visualstudio.com/Download)
1. **optional:** [Install python](https://www.python.org/downloads/)

# Step by step instructions

## 1. Build a simple Web API with ASP.Net Core

First make sure you have successfully installed .Net Core and Git on your desktop. Then open a command propmt either directly or from visual studio code (Terminal > new Terminal). It's perfectly fine if you chose not to install visual studio code, just open the command prompt by typing "cmd" in the search box in the lowerleft corner. 
 
### 1.1. Run the following command in the command shell to create a Web API starter project
```msdos
dotnet new webapi -o RetailApi
```
*The preceding command uses an ASP.NET Core project template, aliased as webapi, to scaffold a C#-based starter web API project. A directory named RetailApi is created that contains an ASP.NET Core project targeting .NET Core. The project name matches the directory name.*

### 1.2. Run the following command in command shell to change directory to the newly created RetailApi folder
```msdos
cd ./RetailApi
```
*Make sure the following files and directories are created Controllers/ , Program.csm RetailApi.csproj, Startup.cs*

### 1.3. Run the following command in command shell to build and test the API

```msdos
dotnet run
```
*The preceding command will start the Web API locally The Web API will be hosted at both ```http://localhost:5000``` and ```https://localhost:5001```*

### 1.4. Verify the API  
Open a browser and type ```https://localhost:5001/api/values```. You should see the following

![browser output](/images/retailapibrowseroutput.PNG)

alternatively,  you can also use the following to verify the output from your Web API

```curl
curl -k -s https://localhost:5001/api/values | python -m json.tool
```

please visit [Build a web API with ASP.NET Core](https://docs.microsoft.com/en-us/learn/modules/build-web-api-net-core/) on microsoft learn for detailed instructions on building a simple ASP.NET Code web API. **Disclaimer**: Some of the above steps are a copy-paste from the preceeding link.
    
## 2.Deploy the Web API to Azure App Service

### 2.1. Create a Web App in Azure
Follow the instructions [here](https://docs.microsoft.com/en-us/learn/modules/host-a-web-app-with-azure-app-service/2-create-a-web-app-in-the-azure-portal) to create a Web App with any name for example , "WebApp3".

### 2.2. Enable local git and automated deployment for the Web App you created

#### 2.2.1. Click on Deployment Center > Local Git > Continue > App Service Kudu build server >continue > finish

![deployment center2](/images/deploymentcenter.PNG)
   
![deployment center2](/images/deploymentcenter2.PNG)
   
![deployment center3](/images/deploymentcenter3.PNG)
  
   
#### 2.2.2. Create deployment credentials for your Web App. Go to your Web App > Click on Deployment Center > Click on Deployment Credentials     > User Cedentials > Enter a user name and password and click on save. 
   
   ![deployment credentials](/images/deploymentcredentials.PNG)
    
    
#### 2.2.3 Note down the git clone uri
   
   ![git clone](/images/deploymenturl.PNG)
   
### 2.3. Initializ,stage and commit all your applicatin files to your local git repo on your desktop

#### 2.3.1 Change directories to the "RetailApi" folder (created in step 1: Build a simple Web API...) in your command shell and type the following command 

   ```msdos
   git init
   ```
   you should see an output similar to the following
   ```msdos
   Initialized empty Git repository in C:/RetailApi/.git/
   ```
   
#### 2.3.2 Stage application files: Type the followng command in your command shell

   ```
   git add . 
   ```
   The command above adds all files, represented by the ".", to the staging state of Git.

#### 2.3.3 Commit your code to local git

   ```
   git commit -m "Initial Commit" 
   ```
### 2.4 Add Remote for the local Git Repo  to connect the local git to Azure git

#### 2.4.1 Copy the git clone uri from step 2.2.3 and execute the following command in your command shell
  
   ```msdos
   git remote add origin  https://retailapixxx.scm.azurewebsites.net:443/retailapixxx.git
   ```
#### 2.4.2 Verify Remote Git is added successfully
   
   ```
   git remote -v
   ```
   you should see output similar to the following
   ```
   origin  https://retailapixxx.scm.azurewebsites.net:443/retailapixxx.git (fetch)
   origin  https://retailapixxx.scm.azurewebsites.net:443/retailapixxx.git (push)
   ```
### 2.5 Push local code to Azure

Execute the following command from your command shell

```
 git push origin master
```
You will be prompted for a username and password. Enter the username and password you created in step 2.2.2 : create deployment credentials for your Web App

### 2.6 Verify code is uploaded to Azure

Go to Azure portal > navigate to your Web App > click on Deployment Center. You will see the first commit that you have on your local machine is now uploaded to Azure.

![deployment center](/images/deploymentcenter-code.PNG)


### 2.7 Verify API in Azure

Naviagte to your Web App in Azure portal > click on overview > note down the url of your Web App
![web app url](/images/webapp-url.PNG)

Open a browser and type [your-web-app-url]/api/values, for example ```https://retailapixxx.azurewebsites.net/api/values```

![azure api](/images/azure-browser-output.PNG)

Congragulations !! You have successfully deployed your Web API to Azure Web Apps.

## 3.Enable Azure AD authentication for the Web API

In this step you will enable Azure AD authentication for your Web API. Once authentication is enabled, your Web API can not be accessed without user credentials.

### 3.1 Navigate to your Web App in Azure Portal and perform the following

#### 3.1.1 Click on "Authentication / Authorization" in the left navigation pane
![auth authorization](/images/auth-authorization.PNG)

#### 3.1.2 Click on the "On" button under App Service Authentication
![auth authorization](/images/auth2.PNG)

#### 3.1.3 Select "Login with Azure Active Directory" in the "Action to take when request is not authenticated" drop down and click on "Azure Active Directory" under "Authentication Providers"
![auth authorization](/images/auth3.PNG)

#### 3.1.2 Click on "Express" > click on "Creat New AD App" and click "OK"
![auth authorization](/images/auth4-2.PNG)

This will create a new AD App for your Web API and turn on Authentication for your Web API. Now go back to the browser and try to access the url for the Web APP. You will be prompted to enter the crendentials for the Web API.

![auth authorization](/images/auth5.PNG)


## 4.Invoke the Web API from Postman using the bearer token obtained from Azure AD

When you access your Web API from the browser, Azure AD automatically directs you to the log-in page where the user can enter their credentials. However in order to access the Web API from an external client application, you will need to perform some addtional steps to access the API. This section walks you through those steps and will show you how to make an API call successfully from Postman.

### 4.1 Get the Azure AD Application Id for your Web API

Navigate to your Web API > click on Authentication / Authorization > Click on Authenticaon Providers / Azure Active Directory >Click on Manage Application >  note down the Application ID

![web api app id](/images/ad-appid.PNG)

### 4.2 Create an Azure AD Application for the Web API client

#### 4.2.1 create client app
Navigate to your Azure portal > click on All Services(in the left pane) > click on Identity > click on Azure Active Directory > click on App registrations > Click on New application registration > enter Application name , leave the application type as "Web App / API" , enter the sign-on url as http://localhost (thid doesnt have to be real) > click on create

![web api app id](/images/ad-appid.PNG)


![client app](/images/client-app.PNG)


#### 4.2.2 Get  Azure AD application ID for the client Application

Navigate to your Azure AD in the portal > click on App registrations > click on the Azure AD client application you just created and note down the application id.

![client app id](/images/client-appid.PNG)


#### 4.2.3 Get client secret for the client Application

Navigate to your client App in Azure AD > click on settings > click on Keys > Enter a key name and pick a duration > click on save. 

![client secret step1](/images/client-secret.PNG)

This will generate a new secret. You will have to note down the secret before you navigate away from this page. This key will be displayed only once.

![client secret step2](/images/client-secret-generated.PNG)

#### 4.2.4 Get the Azure AD token URL

Navigate to Azure AD in your Azure Portal > Click on App Registrations > click on Endpoints. Copy the Token URL.

![token url](/images/token-endpoint.PNG)

#### 4.2.5 Get the bearer token

Open Postman for windows > click on New Request > enter the required values (create collection if needed) > click save.

![new postman request](/images/new-postman-req.PNG)

Click the dropdown and select "POST". enter the token url copied in step 4.2.4 in the URL field. 
Click on Body and select the x-www-form-urlencoded radio button. Enter the following key value pairs in the form fields as shown in the screenshot below
key: grant_type , value: "client_credentials"
key: client_id, value: Azure AD Applciation id of the client application from step 4.2.2
key: resource, value : Azure AD Application id of the Web API from step 4.1
key: client_secret, value: client secret of the client application from step 4.2.3

Click on Send. You should get the Bearer token in the response as shown below
![bearer token](/images/postman-gettoken.png)

Copy the bearer token returned in the response

![bearer token 2](/images/bearer-token-2.PNG)

###  4.2.6 Make an API call to your Web API
Open a new Request in Postman > Now select "Get" from the drop down > Enter the Web API URL (eg.https://retailapixxx.azurewebsites.net/api/values) in the url field

Click on "Authorization" and select "Bearer Token" from the "TYPE" dropdown. Enter the bearer token copied in the previous step.
![web api request 1](/images/postman-webapi-1.PNG)

Click on Send. You should see the response from the Web API.
![web api request 2](/images/postman-webapi-2.PNG)


Congragulations !! you have successfully made an API call using a bearer token to an Azure AD protected Web API. 


