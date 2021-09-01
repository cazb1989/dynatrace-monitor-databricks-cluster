## Install Dynatrace OneAgent for Azure Databricks

This instructions can help you to configure the monitoring for Azure Databricks Cluster.

### Requeriments

- Azure Databricks (Premium version)
- SaaS tenant ID (https://abc1234.live.dynatrace.com)
- PaaS Token: Generated in Dynatrace
- Key Vault
- Secret Configurated with the Paas Token (Use "databricksecret" name)

### Configuren the Secret in Databricks Scope

1. Replace this URL with the Databrick Workspace
https://<azure-databricks-service-url>#secrets/createScope
  
  You can find this URL in the Azure Databricks Service with the name "URL" value
  
2. Complete the fields with Scope Name (use "secretScopeKeyVaultDemo"), Key Vault DNS Name and Resource ID (you can find this one in Key Vault properties)

![image](https://user-images.githubusercontent.com/63391165/131609924-673c2948-297c-4ae8-823a-7b33c593fc91.png)

3. Click in Create button 
  
### Create the Notebook and configure variables
  
4. Create a Notebook with Python Language
5. Copy this code:

```markdown

#Install Dynatrace OneAgent on Azure Databricks Cluster
#Description: This script can help to install Dynatrace on infra-only mode to get infrastructure visibility

#Setting up variables
import requests
print('Setting up variables')
oa_pass_token_databricks = dbutils.secrets.get(scope = "secretScopeKeyVaultDemo", key = "databricksecret")
dt_tenant = 'XXXXXX.live.dynatrace.com' # Example: abc1234.live.dynatrace.com

#Beginning file download with requests
print('Beginning file download with requests')
url = 'https://'+str(dt_tenant)+'/api/v1/deployment/installer/agent/unix/default/latest?Api-Token='+str(oa_pass_token_databricks) +'&arch=x86&flavor=default'
print(url)
r = requests.get(url)
print(r)

with open('/dbfs/dynatrace/oneagent.sh', 'wb') as f:
    f.write(r.content)
#Validation file downloaded
if r.status_code == 200 :
  print('File downloaded successfully')
else :
  print('Failed download')

#List OneAgent downloaded:
print('List OneAgent downloaded')
dbutils.fs.ls("dbfs:/dynatrace/oneagent.sh")

#Writing Script to Install Dynatrace OneAgent infra-only mode
dbutils.fs.put("/dynatrace/init-oneagent-6.sh","""
#!/bin/bash
chmod u+x /dbfs/dynatrace/oneagent.sh
wait
/bin/sh /dbfs/dynatrace/oneagent.sh --set-infra-only=true --set-host-group=DATABRICKS""", True)

#List element uploaded
print('Next Step: Configure this script uploaded in DBFS:/ as Init Script in Cluster Advanced Options')
dbutils.fs.ls("dbfs:/dynatrace/init-oneagent-6.sh")

```
### Replace the variables values:
In case you do not used the variables names for Scope Databricks and Secret Name you can replace these values in the code.
Replace 'secretScopeKeyVaultDemo' for the name of Scope Name created in the step
Replace 'databricksecret' for the name of the Secret.
Aditionally, replace the 'dt_tenant' value with your Dynatrace tenant token.
(Optional) Replace DATABRICKS in '--set-host-group' parameters.
  
### Run the code
In this step your Cluster need to be started and remember is only to download the agent and generate the init script.
6. Run the cell and see the result below. Your download should be successful
![image](https://user-images.githubusercontent.com/63391165/131608953-0810540e-061e-452b-94be-38e11070cc04.png)

7. Copy the full dbfs path of the init script.

### Configure the Init Advanced Options
Using the init script generated, set up the Init Script in Advanced Options from the Cluster.
8. Go to the Cluster Settings
9. Click Terminate (If it is running) and then Edit
10. Configure de dbfs path copied into the init Script:

![image](https://user-images.githubusercontent.com/63391165/131608595-8d4529af-3f22-4742-8619-a954b03da79f.png)

11. Confirm adn Start the Cluster
12. Review the Event Logs, you should see a log with "Starting Init Script execution" and "Finished Init Scripts execution."

![image](https://user-images.githubusercontent.com/63391165/131608732-1b0e2702-4ad0-4a1f-9980-d351beabee12.png)

13. Check the host monitored in Dynatrace
![image](https://user-images.githubusercontent.com/63391165/131611796-364c170c-7975-45cc-a569-28d9548acd0f.png)
