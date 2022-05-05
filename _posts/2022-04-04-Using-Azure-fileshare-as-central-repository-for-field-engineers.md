---
layout: post
title:  "Azure storage as a central repository for field engineers"
author: sergeype
tags: [ azure, fileshare, filestorage, azcopy, sync, onecustomerstory ]
image: assets/users/sergeyperus/AzCopy.jpg
featured: false
hidden: false

---

#onecustomerstory

Our customers often don't have an Internet connection. We need to update their software. 
We have field engineers who can reach every customer by foot and do a maintenance. 
But they must have a recent copy of all files from the central fileshare

## Solution

1) Create an [Azure Blob storage](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=azure-portal)

2) Set permissions for Storage Account:

    *Storage Blob Data Contributor* for Write and Read access
    *Storage Blob Data Reader* for READ only access

We can use 3 types of *security principal*:

- User identity.

    A user identity is any user that has an identity in Azure AD. It's the easiest security principal to authorize. It runs with a user interaction, but only once, unless he or she does not authorize at least once in 90 days

- Service Principal.

    A service principal is better suited for scripts that run on-premises/outside of Azure.
    It can use a Password-based or Certificate-based authentication

    Create Service Principal with [Powershell](https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azps-7.5.0#create-a-service-principal) or [Portal](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal#register-an-application-with-azure-ad-and-create-a-service-principal)

- Managed Identity.

    A [managed identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) is better suited for scripts that run from an Azure Virtual Machine (VM)
    Here are some of the benefits of using managed identities:

        You don't need to manage credentials. Credentials aren’t even accessible to you.
        You can use managed identities to authenticate to any resource that supports Azure AD authentication, including your own applications.
        Managed identities can be used without any additional cost.
        
3) Use a command-line utility [AzCopy](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10)

I'd note here, only Blob storage currently supports Azure AD method of authorization, while File Storage supports only a Shared Access Signature (SAS) token.

Example of the script:

	

	$tenantid = "10868dd3-36ce-4e6e-bf97-XXXXXXXXXXXX"
	$localpath = "d:\1"
	$azureblob = "https://azcopyblobshare.blob.core.windows.net/container1"


	# Authorization
	azcopy login --tenant-id=$tenantid 	#user identity

	# Onprem to Azure - sync local changes to Azure
	azcopy sync $localpath  $azureblob --recursive --delete-destination=true

	# Azure to Onprem
	azcopy sync $azureblob $localpath --recursive --delete-destination=true



You can find an example of a service principal authorization  [here](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-authorize-azure-active-directory#authorize-a-service-principal-by-using-a-client-secret)

## Security

Keep in mind default security options:

*Secure transfer (HTTPS) is **required**

*Blob public access is **enabled**. I'd recommend to disable that

*Storage account key access is **enabled**. I'd recommend to disable it and use only AD auth

*Minimum TLS version is **1.2**. Windows 7 client will not be able to connect

*Data is **encrypted by default** using Microsoft-managed keys


You can add [Defender for Storage](https://docs.microsoft.com/en-us/azure/defender-for-cloud/defender-for-storage-introduction) - additional Azure-native layer of security intelligence that detects unusual and potentially harmful attempts to access or exploit your storage accounts. It uses advanced threat detection capabilities and [Microsoft Threat Intelligence](https://go.microsoft.com/fwlink/?linkid=2128684) data to provide contextual security alerts. Those alerts also include steps to mitigate the detected threats and prevent future attacks.

Apply Backup, soft delete and blob versioning as you need.


## Cost

This is the one of the most interesting part of any Azure solution. And one of the most challenging one.

Azure Storage bills based on your storage account usage. All objects in a storage account are billed together as a group. Storage costs are calculated according to the following factors:

- Region refers to the geographical region in which your account is based.
- Account type refers to the type of storage account you're using.
- Access tier refers to the data usage pattern you’ve specified for your general-purpose v2 or Blob Storage account.
- Capacity refers to how much of your storage account allotment you're using to store data.
- Redundancy determines how many copies of your data are maintained at one time, and in what locations.
- Transactions refer to all read and write operations to Azure Storage.
- Data egress refers to any data transferred out of an Azure region. When the data in your storage account is accessed by an application that isn’t  	running in the same region, you're charged for data egress.

The [Azure Storage pricing page](https://azure.microsoft.com/pricing/details/storage) provides detailed pricing information based on account type, storage capacity, replication, and transactions. The [Data Transfers pricing details](https://azure.microsoft.com/pricing/details/data-transfers) provides detailed pricing information for data egress. You can use the [Azure Storage pricing calculator](https://azure.microsoft.com/pricing/calculator/?scenario=data-management) to help estimate your costs.

Let’s make an assumption:

- Out local fileshare has 500 Gb of data. 

- Every month our engineers will download 110Gb of data altogether.

- LRS Local redundancy is good enough for our purpose (*Durability 99.9999999%, three copies within a single region. Protects again disk, node, rack failure. Superior to dual-parity RAID*)

- Standard Hot tier meets our requirements for performance

- We will use UK South datacenter

**Capacity** - 500 Gb will cost us £7.68 per month. It's almost impossible to predict the amount and cost of operations. But in general, with correctly chosen tier, there will be a relatively small charge for it. 

**Data egress** - 110 Gb will cost us £0.64 per month. First 100 Gb each month is free, and per £0.06 each Gb thereafter 

## Benefits

So, we end up with less than **£10 per month**. Our engineers **don't need a VPN connection** to securely download/upload data. Our storage is **99.9999999% durable** and we have **120 Gbps egress bandwidth** by default. 
We have both **AD authentication** for users and **password/certificate** for applications

Do not hesitate to ask questions in comments if any!

