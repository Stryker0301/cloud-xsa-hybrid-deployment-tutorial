### CPL166 
# Exercise 5 - Deploying your Multi-Target Application to Cloud Platform
In this exercise, we will take the previously developed application and deploy it to SAP Cloud Platform.
<br><hr><br>

## Step 1 - Adjust configuration files
The MTA descriptor file and the xs-security configuration require some smaller extension to work within the cloud environment.  

Because of the quota restrictions of the trial account that we are using in this session, we need to set a memory limit for the 'cpl166js' and 'cpl166ui' module. The module 'cpl166db' does not need to be restricted as the type `com.sap.xs.hdi` already includes an implicit memory limit to 256MB.

Open the mta.yaml file in the editor and select 'cpl166js' module. Add a new parameter with the key 'memory' and the value `256M`.

<img src="img/memory.png" alt="Memory Setting"/>
<br>
Add the same parameter for the module 'cpl166ui'.  
  
  Now we need to add some advanced security configuration. Therefore select the module 'cpl166js' and add a property with the key `SAP_JWT_TRUST_ACL` and the value `[{"clientid":"sb-cpl166js", "identityzone": "*"}]`:
  
  <img src="img/sap_jwt_trust_acl.png" alt="SAP JWT TRUST ACL setting"/>
  <br>
  
  Now select the module 'cpl166ui' and add a property with the key `TENANT_HOST_PATTERN` and the value   
  `^(.*)-trial-dev-cpl166ui.cfapps.eu10.hana.ondemand.com`:
  
  <img src="img/tenant-host-pattern.png" alt="TENTANT HOST PATTERN setting"/>
  <br>  
  <br>
  
  Because the hana service is named `hanatrial` in the trial organization, we need to specify this additional mapping in the mta.yaml. Therefore open the mta.yaml and for the module 'cpl166db' under Resources for 'hdi-container' add an additional parameter with the key `service` and `hanatrial`.
  
  Now, click Save. Afterwards please open the mta.yaml file with the code editor and check that for the two files 'cpl166js' and 'cpl166ui' the value 256M was saved. If not, please correct it manually and save again. Unfortunately, there was a bug in some versions, which caused the graphical editor to remove the M during the save.
  
  
  
  
  <hr>
  <br>
  
## Step 2 - Create the MTA archive
For the cloud deployment we need to create an mta archive. Right-click on the project-name 'CPL166MTA' and select 'Build'. Wait for the builder to finish.  

<img src="img/build_mtar.png" alt="Build mtar">

<br>

A new folder called 'mta_archives' has appeared. Unfold it by clicking on the folder. It contains a folder called 'CPL166MTA'. Unfold this folder as well. Inside you will find the newly created mta archive with the name 'cpl166mta_0.0.1.mtar'.  
Export the archive to your local disk by right-clicking and selecting 'Export'. This will start the download in the browser.  
  
<img src="img/export_mtar.png" alt="Export mtar">
<br>  
<hr>
<br>

## Step 3 - Deploy your application to Cloud Foundry
In this step we will deploy our new application to Cloud Foundry. To do so, you will require the cf CLI. In case you have not installed it yet, you can find installation instructions [here](https://github.com/cloudfoundry/cli#downloads).

On your Laptop, open the Windows Explorer and navigate to the location where you have downloaded the applications mta archive in Step 2. Usually this is the 'Downloads' folder.  
Open a command window by right-clicking in the Explorer and selecting 'Open Command Window Here'.  


<img src="img/open-cmd.png" alt="Open cmd" />
<br>


First we need to set the correct api endpoint of the SAP Cloud Platform Cloud Foundry Trial installation by entering one of the following commands (depending on your region) and confirm by pressing enter. Choose the first one with 'eu10' if your Cloud Foundry trial account is in the region 'Europe (Frankfurt) CF/AWS' or choose the second one with 'us10' if your Cloud Foundry trial account is in the region 'US East (VA) CF/AWS'.

```
cf api https://api.cf.eu10.hana.ondemand.com

cf api https://api.cf.us10.hana.ondemand.com
```
<br>

After a few seconds, the command will come back with a success message.

<img src="img/cf-api.png" alt="cf api success" />
<br>

Now we need to login. Enter the following command and confirm by pressing enter.

```
cf login
```
<br>
The prompt will ask you for your credentials, enter the email and password and confirm each step by pressing enter.

<img src="img/cf-login.png" alt="cf login" />
<br>

With the login done, we can start the application deployment. Therefore enter the deploy command as follows and confirm by pressing enter.
If you do not have the "cf deploy" command available, follow the instructions at [SAP/cf-mta-plugin](https://github.com/SAP/cf-mta-plugin#download-and-installation) to install the CF CLI plugin, which provides this functionality.

```
cf deploy cpl166mta_2.0.0.mtar
```
<br>

## Step 5 - Create a Role Collection and assign it to your user
Similar to the previous exercise, we now need to create a role collection and assign it to our user in order to be authorized to access our new application.  

Firstly we need to logon once to our uaa tenant to make our user known. Navigate to the url below depending on your trial account location. You need to replace `<p-username>` with your username. 

```
https://<p-username>trial.authentication.eu10.hana.ondemand.com

https://<p-username>trial.authentication.us10.hana.ondemand.com
```

Login with your credentials and then switch back to the SAP Cloud Platform Cockpit.

<img src="img/first-uaa-login.png" alt="uaa login page"/>
<br>

Open the "Security" menu item and navigate to Role Collections page. Create a new collection called 'cpl166_collection'
<img src="img/trial-create-rolecollection.png" alt="Trust Configuration page"/>
<br>

Click on the new collection 'cpl166_collection' and assign both 'cpl166View' and 'cpl166Create' roles from the deployed application using the dropdown lists
<img src="img/trial-add-role.png" alt="Trust Configuration page"/>
<br>

Navigate to the Trust Configuration page.

<img src="img/trust-configuration.png" alt="Trust Configuration page" />
<br>

Click on 'SAP ID Service'. This brings you to another view called 'Role Collection Assignment'.

<img src="img/role-collection-assignment.png" alt="Role Collection Assignment page" />
<br>

Enter your username in the search field and click on 'Show Assignments'. This enables the button 'Add Assignment'. Click on the button. In the popup select the previously created role collection and confirm by clicking 'Add Assignment'.

<img src="img/add-assignment.png" alt="Add Assignment" />
<br>

After deployment and role collection configuration you can copy the url of the cpl166ui application as shown after the deploy command and open it in your browser to test it in the cloud. The command `cf apps` will show you all deployed application microservices again. Alternatively or additionally, you can now also go back to your SAP Cloud Platform cockpit, refresh and you will see your deployed applications there. Under the application microservice cpl166ui you will see the same URL which you can also open directly from there. 
