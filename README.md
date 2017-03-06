# Splunk Add-on For Azure Monitor Logs

This add-on was built in accordance with the guidelines on this page:<br/>
[How to work with modular inputs in the Splunk SDK for JavaScript](http://dev.splunk.com/view/javascript-sdk/SP-CAAAEXM)

It consumes Diagnostic Logs according to the techniques defined by Azure Monitor, which provides highly granular and real-time monitoring data for any Azure resource, and passes those selected by the user's configuration along to Splunk. It uses a Key Vault to store Event Hub credentials. Those credentials are retrieved by a Service Principal in the Reader role of the subscription. 

I'll refer to Splunk Add-on for Azure Monitor Logs as 'the add-on' further down in this text.

Here are a few resources if you want to learn more:<br/>
* [Overview of Azure Monitor](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-overview)
* [Overview of Azure Diagnostic Logs](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)

## Installation
In both Windows and Linux cases, it's easiest if you first clone the repo into your personal space.

1. Go to Manage Apps in the Splunk Web interface.
2. Click the button to "Install app from file"
3. Select "TA-Azure_monitor_logs_0_9_1.spl" from deployment folder of the repo directory on your PC
4. Restart Splunk <br/>

### After restarting Splunk
In the Apps manager screen of the Splunk Web UI you should now see "Azure Monitor Logs". In Settings / Data Inputs you should see in the Local Inputs list "Azure Monitor Logs". Create a new instance of the add-on and supply the needed inputs according to the Inputs section below. I name my instances according to the subscription that I'm monitoring with it, but you may have other naming standards.

### What's an Azure AD Service Principal and where can I get one?
See here: [Use portal to create Active Directory application and service principal that can access resources](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal)<br/>

### Azure Configuration Steps

1. Assign your service principal the Reader role (at least) in the subscription containing your Key Vault. In the portal, this is done by displaying the Subscriptions blade, then the subscription blade of the one containing your Key Vault, then Access Control (IAM). 
2. Create a secret in your Key Vault. The normally optional Content Type field must be filled in with the name of the Event Hub's SAS key policy and the value must be filled in with the SAS key value.
3. In the Access Policies of the Key Vault, add the Service Principal and give it GET secrets permissions.
4. For each resource that you want to monitor, find the Monitoring section of its management blade in the portal, click the Diagnostic Logs button and fill in the details. This can also be done with PowerShell, the Azure CLI, or a program using the REST api.

## Inputs: (all are required)

| Input Name | Example | Notes |
|------------|---------|-------|
| SPNTenantID | `XXXX88bf-XXXX-41af-XXXX-XXXXd011XXXX` | your Azure AD tenant id |
| SPNName | `0edc5126-XXXX-4f47-bd78-XXXXXXXXXXXX` | your Service Principal Application ID |
| SPNPassword | `7lqd4scZWpxjBe6dQyYBY2bFjk+8jio9iOvCv65gf9w=` | your Service Principal password |
| eventHubNamespace | `myEventHubNamespace` | the namespace of the event hub receiving logs |
| vaultName | `myKeyVault` | Name of the key vault containing your secrets |
| secretName | `mySecret` | Name of the secret containing your event hub SAS credentials |
| secretVersion | `secretVersion` | Version of the secret containing your event hub SAS credentials |

Click the "More Settings" box and provide the following: (required)
* Interval = 60
* Set sourcetype = manual
* Source Type = azureMonitorLogs
* Index = main

These are the values that I use on my test system and of course you may have other standards for index and sourcetype values. The add-on takes no dependency on the values you use.

## How the add-on works
The add-on assumes that you have resources in a subscription that you want to monitor. These resources must be configured to send Diagnostic Logs to an Event Hub.<br/>

### How it works, step-by-step (The add-on is invoked once per minute by Splunk Enterprise)
* Get an authentication token for Key Vault secrets.
* Get the secret named in the Input Parameters.
* Create connections to Azure Monitor hubs in the Event Hub Namespace.
* Listen to the connections and pass any new messages along to Splunk.
* If there's 5 seconds of silence across all hubs, terminate.
* Splunk calls the add-on periodically, according to the Interval value above.

## Logging

Logs are generated by Logger. Errors and Warnings will appear in `splunkd.log` which is located at `$SPLUNK_HOME/var/log/splunk`. Most logs are in the debug log level. To change the log level, in the Splunk Web UI go to Settings / Server Settings / Server Logging. Search for ExecProcessor in the very long list of log channels. Change its value to whatever you want and save. It'll reset to INFO upon restarting Splunk unless you change it permanently via other means.

# Contributing

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
