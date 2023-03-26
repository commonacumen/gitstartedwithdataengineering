# gitstartedwithdataengineering
Use this repository to git (get) started with data engineering tasks using Azure Data Diabetes datasetFactory and Azure Databricks

## Step 1 Clone or Fork this GitHub Repo to get the datasets and code

```
git clone https://github.com/commonacumen/gitstartedwithdataengineering.git
```

The datasets are from [Diabetes dataset on Microsoft.com](https://learn.microsoft.com/en-us/azure/open-datasets/dataset-diabetes?tabs=pyspark) orginally from [Original dataset description](https://www4.stat.ncsu.edu/~boos/var.select/diabetes.html) and [Orginal data file](https://www4.stat.ncsu.edu/~boos/var.select/diabetes.tab.txt) and a ageband dataset created by me.

These datasets have been included in the data folder in this GitHub Repo [Datasets Here](https://github.com/commonacumen/gitstartedwithdataengineering/tree/main/data)

## Step 2 Create an Azure Data Factory pipeline from local template to copy and transform datasets using ADF

Download [ADF Template zip](https://github.com/commonacumen/gitstartedwithdataengineering/tree/main/code/adfTemplates) or find it in your cloned GitHub Repo.

![adftemplatezip](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adftemplatezip.png)

Open up the ADF deployed by the ARM template.  Select Pipeline > Import from pipeline template 

![adfplfromtemplate](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adfplfromtemplate.png)

Click on the zip file and click Open

![adfOpenLocalTemplate](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adfOpenLocalTemplate.png)

It should look like this

![adftemplateUserinputs](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adftemplateUserinputs.png)

Select +New in the first User input and create an HTTP Linked Service.

![adfHttpLinkedService](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adfHttpLinkedService.png)

Make sure the url is using raw

`https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/data/`

Select +New in the second User input and create an Azure Data Lake Storage Gen2 Linked Service 

![adfAdlsLinkedService](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adfAdlsLinkedService.png)

Then click on Use this template

![adfAllUserinputs.png](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adfAllUserinputs.png)

It should look like this when it is imported

![adfTemplateImported](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adfTemplateImported.png)

## Step 3 Debug the DiabetesCopyAndTransformDataToDelta Pipeline 

Click on Debug, and Click OK

Once the pipeline runs successfully it should look like this

![adfSuccessfulRun](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adfSuccessfulRun.png)

Check that the files have been created in Storage using [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) or the Azure Portal in the browser.  The files should be in silver container at a path like `diabetes/adfdelta/`

![adfFileInStorage](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adfFileInStorage.png)

You can now save the pipeline by clicking on Publish all

## Step 4 Import, configure, and run the Databricks notebook

### Requirements

- Databricks Runtime 8.3 or above when you create your cluster

- Setup Permissions to ADLS Gen2

- Secrets in Key vault

*Steps*

### Import the Databricks notebook

Open up you Databricks workspace and navigate to your user, select the dropdown and select import

![adbworkspace](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images//adbworkspace.png)

Import from file if you cloned the repo locally or enter the URL `https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/code/notebooks/ConnectToDeltaOnADLS.ipynb` to the Notebook in GitHub Repo [ConnectToDeltaOnADLS.ipynb](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/code/notebooks/ConnectToDeltaOnADLS.ipynb) and click Import

![adbnotebookimport](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbnotebookimport.png)

You should now have a notebook that looks like this:

![adbnotebook](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbnotebook.png)

Change the value of the adlsAccountName = "<EnterStorageAccountNameHere>" in cell one to the ADLS AccountName of in your deployment

In my chase my deployment has a Storage account name of `cdcacceleredfkdd3zynq6k` so the first row of the cell would read:

```
adlsAccountName = "cdcacceleredfkdd3zynq6k"
```

![adbrgservices](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbrgservices.png)

![adbadlsacctname](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbadlsacctname.png)

### Configure Service Principal and Permissions

*Create a Service principal* [Reference](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal#register-an-application-with-azure-ad-and-create-a-service-principal)

Create an [Azure Active Directory app and service principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal) in the form of client ID and client secret.

1. Sign in to your Azure Account through the Azure portal.

2. Select Azure Active Directory.

3. Select App registrations.

![adbappreg](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbappreg.png)

4. Select New registration.

Name the application something like `databricksSrvPrin`. Select a supported account type, which determines who can use the application. After setting the values, select Register.

Note that it is a good idea to name the application with something unique to you like your email alias (darsch in my case) because other might use similar names like `databricksSrvPrin`.

![adbregister](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbregister.png)

5. Copy the Directory (tenant) ID and store it to use to create an application secret.

6. Copy the Application (client) ID and store it to use to create an application secret.

![adbappids](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbappids.png)

**Assign Role Permissions** 

7. At storage account level assign this app service pricipal the following role to the storage account in which the input path resides:
    
    `Storage Blob Data Contributor` to access storage

![adbstorageiam](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbstorageiam.png)

**Create a new application secret**

- Select Azure Active Directory.

- From App registrations in Azure AD, select your application.

- Select Certificates & secrets.

- Select Client secrets -> New client secret.

- Provide a description `AppSecret` of the secret, and a duration. When done, select Add.

![adbappsecret](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbappsecret.png)

After saving the client secret, the value of the client secret is displayed. Copy this value because you won't be able to retrieve the key later. You will provide the key value with the application ID to sign in as the application. Store the key value where your application can retrieve it.

![adbappsecretval](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbappsecretval.png)

### Deploy a Key Vault and setup secrets

Create a Key Vault in the Resource group by clicking Create

Search for `Key vault`

![adbkvsearch](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images//adbkvsearch.png)

Click Create

![adbkvcreate](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbkvcreate.png)

Create the Key Vault in the same Resource group and Region as you other resource deployed. Click Review and Create and then click Create

![adbrevcreate](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbrevcreate.png)

You should now have a Key vault in your resources

![adbrgwithkv](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbrgwithkv.png)

Open up you Key vault and add the appsecret created above

Choose Secrets and click Generate/Import

![adbkvsecretgen](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbkvsecretgen.png)

Enter you secret Name and paste in the app secret you created earlier, set activation date and click Create

![adbcreatesecret](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbcreatesecret.png)

It should look like this:

![adbfirstsecret](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbfirstsecret.png)

*Create the rest of the secrets you need for the notebook*

Create the rest of the secrets in cell 4 of the notebook.  The secret names are at the end of each line after EnterDatabrickSecretScopeHere

```
SubscriptionID = dbutils.secrets.get("<EnterDatabrickSecretScopeHere>","SubscriptionID")
DirectoryID = dbutils.secrets.get("<EnterDatabrickSecretScopeHere>","DirectoryID")
ServicePrincipalAppID = dbutils.secrets.get("<EnterDatabrickSecretScopeHere>","ServicePrincipalAppID")
ServicePrincipalSecret = dbutils.secrets.get("<EnterDatabrickSecretScopeHere>","AppSecret")
ResourceGroup = dbutils.secrets.get("<EnterDatabrickSecretScopeHere>","ResourceGroup")
BlobConnectionKey = dbutils.secrets.get("<EnterDatabrickSecretScopeHere>","Adls2-KeySecret")
```

```
#Secret Names#

SubscriptionID

DirectoryID

ServicePrincipalAppID

AppSecret (already created above)

ResourceGroup

Adls2-KeySecret
```

The Adls2-KeySecret is created using the storage account key

![secrets](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/secrets.png)


**Create an Azure Key Vault-backed secret scope using the UI** [Reference](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes#create-an-azure-key-vault-backed-secret-scope-using-the-ui)

Verify that you have Contributor permission on the Azure Key Vault instance that you want to use to back the secret scope.

Go to https://databricks-instance/#secrets/createScope. This URL is case sensitive; The "S" in scope in createScope must be uppercase.

https://databricks-instance/#secrets/createScope

In my case `https://adb-1558951773184856.16.azuredatabricks.net/#secrets/createScope`

You can find the databricks-instance in the URL of your workspace

![adbinstance](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbinstance.png)

Enter Scope Name: I choose something like `databricksSrvPrin` which is what I used in the notebook

Manage Principal:  `All Users`

DNS Name: `https://xxxxxx.vault.azure.net/` Find in the properties of Key vault under Vault URI

Resource ID: Find in the properties of the Key vault.  Looks something like this: 

```
/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/databricks-rg/providers/Microsoft.KeyVault/vaults/databricksKV
```
![adbsecretResID](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbsecretResID.png)

Click Create

![adbsecretscope](https://raw.githubusercontent.com/commonacumen/gitstartedwithdataengineering/main/images/adbsecretscope.png)


### Create a Databricks Cluster and attach to notebook

Create a cluster using the Runtime 8.3 or above

Enter Cluster Name, Runtime Version, Set Terminate after, Min Workers, Max Workers and click Create Cluster

![adbcreatecluster](https://raw.githubusercontent.com/Azure/Azure-DataFactory/main/SamplesV2/ChangeDataCapture/images/adbcreatecluster.png)

### Add the Scopes into Cells 2 and 4

Change the value of "<EnterDatabrickSecretScopeHere>" in cell 2 and 4 to the Scope name you created earlier.

In my chase `demo-autoloader` so the 2 cell would read:

```
spark.conf.set(
    "fs.azure.account.key." + adlsAccountName + ".dfs.core.windows.net",
    dbutils.secrets.get(scope="demo-autoloader",key="Adls2-KeySecret"))
```

The 4 cell would read:

```
SubscriptionID = dbutils.secrets.get("demo-autoloader","SubscriptionID")
DirectoryID = dbutils.secrets.get("demo-autoloader","DirectoryID")
ServicePrincipalAppID = dbutils.secrets.get("demo-autoloader","ServicePrincipalAppID")
ServicePrincipalSecret = dbutils.secrets.get("demo-autoloader","AppSecret")
ResourceGroup = dbutils.secrets.get("demo-autoloader","ResourceGroup")
BlobConnectionKey = dbutils.secrets.get("demo-autoloader","Adls2-KeySecret")
```

### Run the notebook one cell at a time (at least the first time)

Once the cluster is started you will be able to run the code in the cells

Click on Run Cell

![adbcruncell](https://raw.githubusercontent.com/Azure/Azure-DataFactory/main/SamplesV2/ChangeDataCapture/images/adbcruncell.png)

Do this for the next cell down etc.

You can skip cell 6 the first time because nothing has been mounted.  You may get an error like this:

![adbunmount](https://raw.githubusercontent.com/Azure/Azure-DataFactory/main/SamplesV2/ChangeDataCapture/images/adbunmount.png)
