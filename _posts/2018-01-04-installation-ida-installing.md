---
title: "Installing IDA Application"
category: installation
date: 2018-09-21 15:17:55
last_modified_at: 2019-07-25 21:37:00
order: 4
---

# Installing IDA Application
***
## Installing IDA Application Components
There are three components for IDA application we need install and configure, included (1) *IDA web application*, (2) *IDA Browser Plugin* and (3) *IDA BPM toolkit*.


# Step 1: Installing IDA Web Application

IDA Web Application can be installed on WebSphere Application Server (WAS), liberty or Docker. First, let's introduce the way to install IDA on liberty.

## Installing on Liberty


**Create a Liberty server manually** 

You can create a server from the command line.

* Unzip the liberty installation package.Open a command line, then change directory to the wlp/bin directory.
Where path_to_liberty is the location you installed Liberty on your operating system.  

``` 
cd path_to_liberty/wlp/bin
``` 

* Run the following command to create a server. If you do not specify a server name, defaultServer is used.
Where server_name is the name you want to give your server.  

*Windows*

``` 
server create server_name  
``` 
*Linux*

``` 
./server create server_name  
``` 
server_name must use only Unicode alphanumeric (for example, 0-9, a-z, A-Z), underscore (_), dash (-), plus (+), and period (.) characters. The name cannot begin with a dash or period. Your file system, operating system, or compressed file directory might impose more restrictions.

If the server is created successfully, you receive message: Server server_name created.

	
**Configure WAS Liberty**  

* Edit **server.xml** from *wlp/usr/servers/servername* folder.  Please ensure both **httpPort** and **httpsPort** are unique and not same with BPM server port.If found port conflict，pls change the  **httpPort** and **httpsPort** address.  

```    
 <!-- Enable features -->
    <featureManager>
        <feature>servlet-3.1</feature>
		<feature>ssl-1.0</feature>
        <feature>websocket-1.1</feature>
    </featureManager>

    <!-- This template enables security. To get the full use of all the capabilities, a keystore and user registry are required. -->
    
    <!-- For the keystore, default keys are generated and stored in a keystore. To provide the keystore password, generate an 
         encoded password using bin/securityUtility encode and add it below in the password attribute of the keyStore element. 
         Then uncomment the keyStore element. -->
    <!--
    <keyStore password=""/> 
    -->
    <webContainer invokeFlushAfterService="false"/>
    
    <!--For a user registry configuration, configure your user registry. For example, configure a basic user registry using the
        basicRegistry element. Specify your own user name below in the name attribute of the user element. For the password, 
        generate an encoded password using bin/securityUtility encode and add it in the password attribute of the user element. 
        Then uncomment the user element. -->
    <basicRegistry id="basic" realm="BasicRealm"> 
        <!-- <user name="yourUserName" password="" />  --> 
    </basicRegistry>
    
    <!-- To access this server from a remote client add a host attribute to the following element, e.g. host="*" -->
    <httpEndpoint id="defaultHttpEndpoint"
				  host="*"
                  httpPort="9081"
                  httpsPort="9443" />
				  

                  
    <!-- Automatically expand WAR files and EAR files -->
    <applicationManager autoExpand="true" startTimeout="360s" stopTimeout="120s"/> 
	<application type="war" id="ida" name="ida" location="${server.config.dir}/apps/ida-web.war">
		<classloader delegation="parentLast" />
    </application>
	
	<keyStore id="defaultKeyStore" password="idaAdmin" />

```  
    

* Copy **IDA-web.war** application into the /usr/servers/*yourservername*/apps directory.

* Start Liberty server and visit the url like http://serverip:port/ida (port is defined in the server.xml).

For example:  
In Liberty installation bin folder you can use below command to start the server.
**server start default** (default is your server name).

**Install Trustore for [WASADMIN] deployment service**

[WASADMIN]: ../installation/installation-ida-repacking.html#configure-deployment-service-type 

* Copy [IDA_HOME]/conf/**truststore.jks** to *wlp/usr/servers/servername* folder or generate your own truststore file using tools like keytool.
```
cd PATH/TO/JAVA/BIN
keytool -genkey -keyalg RSA -alias selfsigned -keystore wlp/usr/servers/servername/truststore.jks -storepass testingweb -validity 360 -keysize 2048
```

* Edit **server.xml** from *wlp/usr/servers/servername* folder to include the trustore ssl configuration as below:
```
    <ssl id="defaultSSLConfig" trustStoreRef="idaTrustStore"/>

    <keyStore id="idaTrustStore" 
        location="${server.config.dir}/truststore.jks" 
        type="JKS" 
        password="testingweb" />
```


### Customizing the Liberty environment with jvm.optioins - optional
Customize JVM options by using jvm.options files.

* Create a text file named jvm.options. Copy it to path_to_liberty/wlp/usr/servers/*yourservername* directory.      

**Update heap size setting**

Add below to jvm.options. e.g set the maximum heap size to 1024m

* -Xmx1024m

If the heap size is not big enough, IDA checkstyle may crash with out-of-memory exception throwed, increase the heap size and restart server can fix this issue.

**Support http proxy**

We might need proxy server to visit the application,the proxy settings can be passd to the runtime via the JAVA_OPTS environment variable.
* Add following lines to jvm.options based on your acutal proxy setting. You can change https to http as well.    
-Dhttps.proxyHost=host     
-Dhttps.proxyPort=port     
-Dhttps=proxyUser=user     
-Dhttps.proxyPassword=your password  

[1]: ../installation/installation-integrate-def.html

 with remote testing automation framework based on Selenium Grid.

**Notes**

Below is the reference link for how to setup selenium grid.It includes the detail parameter setting explanation.   
- [Selenium Grid Setup Guidance](https://github.com/SeleniumHQ/selenium/wiki/Grid2)  

## Installing on WAS V9

**Check the WAS version** 

If the WAS version is **9.0** (not 9.0.0.7, 9.0.0.8, 9.0.0.9), it may occur some problems when install the IDA web application, so fisrtly, we should check the WAS version. 
1. Login the WAS console, in the **Welcome** part, you can see the version of WAS. 
![][wasversion]
2. If the WAS version is 9.0, you should install the Fix Packs, here are the available fix packs:
- [9.0.0.7: WebSphere Application Server traditional V9.0 Fix Pack 7](http://www-01.ibm.com/support/docview.wss?uid=swg24044620)
- [9.0.0.8: WebSphere Application Server traditional V9.0 Fix Pack 8](http://www-01.ibm.com/support/docview.wss?uid=swg24044965)

3. When finished the fix packs installation, the version will changed to 9.0.0.7, 9.0.0.8 or 9.0.0.9.

**Deploy a New Application on WAS**

After finishing the installation of the fix packs, the next step is to deploy the IDA war to WAS.

1. Login to the WebSphere Integrated Solutions Console as an administrator (URL: https://host:port/ibm/console/login.do?action=secure).

2. In left navigation bar, click the **New Application>>New Enterprise Application**.

   ![][wasappnew]

3. In the **Path to the new application** section, check the **Local file system** and select the ida-web.war in your local file system. When the war package is uploaded, click **Next** button.

   ![][wasselectapp]  

4. Choose the **Fast Path** option.  click **Next** button.

5. Now the current page is used to specify options for installing enterprise application and modules. In step 1, you can change the application name, click **Next** button after changing the application name. 

   ![][waschangeappname]

6. There is nothing to change in step 2 and step3. And step 4 is used to configure values for contexts root in web modules, we should set the **Context Root** as **/ida** as shown below.

   ![][wassetcontextroot]

7. There is nothing to change in step 5. In step 6, click **finish** button and wait for the server to complete the installation of IDA web application. When finished, click the **WebSphere enterprise application** in left navigation bar, you can see that the IDA web application is in Enterprise Applications table. 

   ![][wasapplist]

**Confige the Class Loader Order** 

1. Click the link of the **ida-web** in the table and go to the app confiugration page. 

2. Click the **Class loading and update detection** link as shown below.

   ![][wasclassloadlink]

3. Change the class loader order to **Classes loaded with local class loader first (parent last)**.

   ![][wasclassloadorder]

4. Then go back to the configuration page, and then click the **Manage Modules** link.

   ![][wasmanagemodules]

5. Click the link of **ida-web.war**, in the configuration page, change the class loader order to  **Classes loaded with local class loader first (parent last)**.

   ![][wasmoduleclassloadorder]

6. Save the changes and start the IDA web applicaiton. when the status of the IDA web applicaiton changes to **started**, you can visit the url like http://serverip:port/ida to access IDA web application.

   ![][wasstartapp]

## Installing on Docker platform
Refer to [IDA-ondocker](https://github.com/sdc-china/IDA-ondocker) for deployment steps.

# Step 2: Installing IDA BPM Toolkit
The testing capability can only start exposed Business Process, Human Services and AJAX Services.  If you wish to directly test other services such as system services, integration services or business processes which are not exposed then you need to install the IDA Toolkit.

1. Import the IDA_Toolkit - 8.6.0_v1.1.twx from IDA toolkit folder into the proces center.
2. Add the IDA toolkit as a this toolkit dependency for within your process application.
3. Right click the "IDA Utility" service flow and copy the item to your process app.  

   ![][toolkit]
4. Make sure the service is installed in your process app.  
   ![][service]
   
# Step 3: Installing IDA Browser Plug-in

### Chrome plugin
- Open the url <a href="https://chrome.google.com/webstore/search/IDA%20IBM" target="_blank">https://chrome.google.com/webstore/search/IDA%20IBM</a>
- Click "Add to Chrome" button to install plug-in

### Firefox plugin
- Download firefox plugin [ida-1.35-fx.xpi](https://github.com/sdc-china/IDA-plugin/raw/master/firefox/ida-1.35-fx.xpi)
- Drag the "ida-1.35-fx.xpi" file into firefox window
- Click "Add" button

**Plug-in Configuration**

If you want to use the checkstyle and codereivew feature on web PD, then you need to set the IDA url and user credentials for the plug-in options. 
the IDA URL: https://9.30.255.220:9443/IDA   
the username: the IDA login username 
the password: the IDA login password.   

   ![][IDAOption]

**Notes**

If you want to install chrome plug-in offline,you can use online https://chrome-extension-downloader.com/  tools,then enter the url
https://chrome.google.com/webstore/detail/ida/mjfjiglcnojlicbkomcoohndhpceflbp to download crx ,then install crx.  

[toolkit]: ../images/install/toolkit.png 
[service]: ../images/install/service.png 
[IDA]: ../images/install/IDA.png 
[firefox]: ../images/install/firefox.png
[seleniumGrid]: ../images/install/seleniumGrid.png
[webDriver]: ../images/install/webdriver.png
[IDAOption]: ../images/install/IDAOption.png
[wasversion]: ../images/install/wasversion.png 
[wasappnew]: ../images/install/wasappnew.png 
[wasselectapp]: ../images/install/wasselectapp.png 
[waschangeappname]: ../images/install/waschangeappname.png 
[wassetcontextroot]: ../images/install/wassetcontextroot.png 
[wasapplist]: ../images/install/wasapplist.png 
[wasclassloadlink]: ../images/install/wasclassloadlink.png 
[wasclassloadorder]: ../images/install/wasclassloadorder.png 
[wasmanagemodules]: ../images/install/wasmanagemodules.png 
[wasmoduleclassloadorder]: ../images/install/wasmoduleclassloadorder.png 
[wasstartapp]: ../images/install/wasstartapp.png 

### Self-Signed SSL Certificates Installation

The IDA recorder plugin can't support website with self sign certification by default. In this case, a warning like this:
   
![][error]
   
   This warning will block the recording of test case. To resolve this problem, we need to make the browsers to accept self-signed certificate.    
   
#### FireFox - Add a Security Exception

1. In FireFox, go to Tools -> Options.

    ![][tool]

2. Click the **Privaty & Security** tab,  then the **View Certificates** button.

    ![][security_tab]
    
3. Go to the **Services** tab and press the **Add Exception** button.
    
    ![][servers_tab]
    
4.  Enter the host and port in **Add Security Exception** dialog, press  **Get Certificate** button, check the box near the bottom **Permanently store this exception** and press **Confirm Security Exception** .

     ![][add_security]
     
    From this point on, FireFox won't show SSL-related errors. when visiting the website, it will like this:
    
    ![][success]
    
#### Chrome - Visit in unsafe mode

Chrome browsers can save your data for a short time, and the warning page will not appear and block recording if you visit the testing website in unsafe mode before recording.

1. Click **ADVANCE** in warning page.

    ![][chrome_error]
    
2. Click **Proceed to 9.30.160.68(unsafe)**.

    ![][proceed]
     



   
   [error]: ../images/install/installation_self_signed_sertificates_error.png 
   [tool]: ../images/install/installation_self_signed_sertificates_tool.png 
   [security_tab]: ../images/install/installation_self_signed_sertificates_security_tab.png
   [servers_tab]: ../images/install/installation_self_signed_sertificates_servers_tab.png
   [add_security]: ../images/install/installation_self_signed_sertificates_add_security.png
   [success]: ../images/install/installation_self_signed_sertificates_success.png 
   [chrome_error]: ../images/install/installation_self_signed_sertificates_chrome_error.png
   [proceed]: ../images/install/installation_self_signed_sertificates_proceed.png
  




