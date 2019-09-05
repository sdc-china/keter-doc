---
title: "Repacking IDA Application"
category: installation
date: 2018-09-20 15:17:55
last_modified_at: 2019-07-25 21:34:00
order: 3
---

# Repacking IDA Application
***
### Properties Configuration


**Configure IDA properties** 
 
Edit [IDA_HOME]/conf/**ida.properties** under **conf** folder. This properties file contains three main sections.

**application configuration**
   
*  **spring.datasource.username** : Database connection username  
*  **spring.datasource.password** : Database connection password  
*  **spring.datasource.url** : Database connection URL
*  **server.host** :  IDA server IP or host name
*  **server.context-path** : default as  "/ida"  
*  **http.port** : Liberty http port defined in the server.xml
*  **https.port** : Liberty https port defined in the server.xml
*  **engineConfig.connectionTimeout** :  page load timeout
*  **engineConfig.waitTimeout** :  default wait timeout, used in waitTextPresent and waitElement commands    
*  **engineConfig.retryTimes** :  retry the failed command by the given times
*  **engineConfig.retryInterval** : the interval time  between the retry.The default is 3 seconds    
*  **engineConfig.dataDir** :  Temp folder to keep resouces like screenshot,checkstyle,twx etc  
*  **smtp.host** : SMTP host    
*  **smtp.port** : SMTP port    
*  **email.from** : Sender Email address 
*  **spring.activemq.broker-url** : IP and Port of embedded ActiveMQ. It is only needed when you integrate IDA with BPM DEF[1] to have the monitoring feature. By default this property is commented out. You can leave it as-is if you don't need the monitoring feature. If you need it, you can uncomment it and replace the localhost with the actual IP address of the IDA deployment server.
*  **deployment.timeout** : The timeout in seconds for REST API call to check if snapshot is installed on PS and set as default snapshot
*  **deployment.interval** : The interval in seconds for REST API call to check if snapshot is installed on PS and set as default snapshot
*  **jasypt.encryptor.password** : default value password  
*  **enable_case_history** : Enable save test case history function, default value is true.  
*  **max_case_history** : Set latest test case history max number of each test case, default value is 100.
*  **deployment.service.type**: Set the deployment service type to be WASADMIN or SSH.   
*  **truststore.location**: Set the IDA server's truststore file location.  

**Notes** 
If you want to use encrypt database password in the **spring.datasource.password**.You need to generate the encrypt password.There are two ways to generate it.The secret key is define in the **jasypt.encryptor.password**.
You can use [online web tool](https://www.devglan.com/online-tools/jasypt-online-encryption-decryption) to generate encrypted password.    

![][onlineencrypt]

[onlineencrypt]: ../images/install/online.png

Also you can use command line tool from [jasypt](http://www.jasypt.org/download.html) to generate.


![][offlineencrypt]

[offlineencrypt]: ../images/install/offline.png

Once you get the encrypt password,you need to change spring.datasource.password like below format.As a general rule, jasypt expects encrypted configuration parameters to appear surrounded by "ENC(...)".    
spring.datasource.password: ENC(Yasd1hzrpo05EdNTzlifQ22+8LUNAgcT)     

 
**checkstyle properties configuration**

*  **checkstyle.engine**: The checkstyle engine property can be EMBEDDED or ODM. By default, it's *EMBEDDED*. You need configure the following ODM server and rules configuration if you use ODM as checkstyle engine.
*  **resUser** : rule execution server username  
*  **resPassword** : rule execution server password  
*  **resUrl** : rule execution server url  
*  **resPort** : rule execution server port 
*  **restUrl** : The checkstyle rule ODM REST url

**monitoring properties configuration (For BPM Monitoring purpose)**
	
*  **event_type** : event types to be stored in IDA database  
*  **track_serviceflow** : default as true  

It is only needed when you want to configure monitoring[1] to have the monitoring feature. If you don't need it, you can leave them as-is.

#### Configure Database properties
You can uncomment DB2 property configuration and comment MySQL property configuration.

![][db2config]

[db2config]: ../images/install/db2configuration.png

#### Configure Deployment Service type
You can configure the deployment service type to be either WASADMIN or SSH for executing Pipeline steps including the Deployment step(Online or Offline Deployment) and the Scirpt step(to invoke BPM wasadmin commands such as AdminTask.BPMSetEnvironmentVariable).  If you set **deployment.service.type** to SSH, the steps are executed via SSH. If you set **deployment.service.type** to WASADMIN, the steps are executed using WAS Administration Thin Client APIs. In this case, you also need to set the value of **truststore.location** properly. It is the truststore file location configured on IDA server. Scroll down to see [**Install Trustore for WASADMIN deployment service**](../installation/installation-ida-installing.html#installing-on-liberty) for a Liberty config example.

To use WASADMIN deployment service type, follow **ADMINISTRATION** steps in [Add a BPM server with WASADMIN deployment service].

[Add a BPM server with WASADMIN deployment service]: ../administration/administration-bpm-configuration.html#add-a-bpm-server

**Notes**   

If you want your IDA support Latin alphabet in page,you need to add **useUnicode=yes&characterEncoding=UTF-8** at the end of jdbc url.

### Repacking IDA-web with configuration

The purpose of repackage  is to update your application-product.yaml in the war to make sure it loads the recent configuration.Once all the properties in [IDA_HOME]/conf/ida.properties file have been updated, you can then re-package the [IDA_HOME]/build/**ida-web.war** file.  

Before running the [IDA_HOME]/**package.bat** or [IDA_HOME]/**package.sh** you will need to set the JAVA_HOME environment variable and make ensure that [JAVA_HOME]/bin is included in the variable.  Once the environment variable is set , you can then execute the package shell command to repackage the **ida-web.war**.

![][package]

[package]: ../images/install/package.png
