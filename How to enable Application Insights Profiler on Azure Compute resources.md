# How to enable Application Insights Profiler on Azure Compute resources

This walkthrough demonstrates how to enable Application Insights Profiler on applications hosted by Azure Compute resources. The examples include Virtual Machine and Virtual Machine Scale Sets.

## Overview

The diagram below illustrates how the Profiler works for Azure Compute resources. It uses Azure Virtual Machine as an example.

![Overview][Overview]

The Diagnostics Agent component is what we need to install on the Azure Compute resources so Application Insights Profiler can collect necessary information to process and display on Azure portal. The rest of the walkthrough intends to provide guidance for how to install and configure the diagnostics agent to enable the Application Insights Profiler.

## Prerequisites

* Download the deployment template that installs the Profiler agents on the VMs.

    [WindowsVirtualMachine.json](https://wadexample.blob.core.windows.net/wadexample/WindowsVirtualMachine.json) | [WindowsVirtualMachineScaleSet.json](https://wadexample.blob.core.windows.net/wadexample/WindowsVirtualMachineScaleSet.json)
* An Application Insights instance enabled for profiling. Check https://docs.microsoft.com/en-us/azure/application-insights/app-insights-profiler#enable-the-profiler to see how to do that.
* .NET framework >= 4.6.1 installed in the target Azure Compute resource.


## Enable the Profiler on Azure Compute resources

### Create Resource Group in your Azure subscription
Open Create a resource group in your Azure subscription. Below is an example for doing so in using PowerShell script:

```
New-AzureRmResourceGroup -Name "Replace_With_Resource_Group_Name" -Location "Replace_With_Resource_Group_Location"
```

### Create an Application Insights resource in the Resource group

![Create Application Insights][Create-AppInsights]

### Apply Application Insights Instrumentation Key in the Azure Resource Manager template
If you haven't downloaded the template yet, download the template from below. [WindowsVirtualMachine.json](https://wadexample.blob.core.windows.net/wadexample/WindowsVirtualMachine.json)

![Find AI Key][Find-AI-Key]

![Replace Template Value][Replace-TemplateValue]

### Deploy the template
* Create a secure string to save the password
```
$password = ConvertTo-SecureString -String "Replace_With_Your_Password" -AsPlainText -Force
```

* Deploy ARM template
Change directory in PowerShell console to the folder that contains your ARM template. Run the following command to deploy the template

```
New-AzureRmResourceGroupDeployment -ResourceGroupName AI-ComputeWebApp -TemplateFile .\WindowsVirtualMachine.json -adminUsername "Replcae_With_your_user_name" -adminPassword $password -dnsNameForPublicIP "Replace_WIth_your_DNS_Name" -Verbose
```

After the script executes successfully, you should find a VM named *MyWindowsVM* in your resource group.

### Configure Web Deploy on the VM
Make sure **Web Deploy** is enabled on your VM so your can publish your web application from Visual Studio.
Here is a quick way to install Web Deploy on a VM manually via WebPI:[Installing and Configuring Web Deploy on IIS 8.0 or Later](https://docs.microsoft.com/en-us/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later)
Here is an example for how to automate installing Web Deploy using Azure Resource Manager template:
[Create, configure and deploy Web Application to an Azure VM](https://azure.microsoft.com/en-us/resources/templates/201-web-app-vm-dsc/)

### Publish the project to Azure VM
There are several ways to publish an application to an Azure VM. One way is to do so in Visual Studio 2017.
Right click on the project, select 'Publish...'. Select Azure Virtual Machine as the publish target and follow the steps to finish the publish process.

![Publish-FromVS][Publish-AzureVM]

Run some load test against your application. you should be able to see results in the Application Insights instance portal webpage.
Check https://docs.microsoft.com/en-us/azure/application-insights/app-insights-profiler#viewing-profiler-data for more details.




## What to add if you have an existing VM template

1. Locate the Windows Azure Diagnostics (WAD) resource declaration in your deployment template.
  * Create one if you don't have it yet (check how it's done in the full example).
  * You can update the template from the Azure Resource website (https://resources.azure.com).
2. Modify publisher from "Microsoft.Azure.Diagnostics" to "AIP.Diagnostics.Test".
3. Use typeHandlerVersion as "0.0".
4. Make sure to have autoUpgradeMinorVersion set to true.
5. Add the new ApplicationInsightsProfiler sink instance in WadCfg settings object, following the example below.

```
"resources": [
        {
          "type": "extensions",
          "name": "Microsoft.Insights.VMDiagnosticsSettings",
          "apiVersion": "2016-03-30",
          "properties": {
            "publisher": "AIP.Diagnostics.Test",
            "type": "IaaSDiagnostics",
            "typeHandlerVersion": "0.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
              "WadCfg": {
                "SinksConfig": {
                  "Sink": [
                    {
                      "name": "Give a descriptive short name. E.g.: MyApplicationInsightsProfilerSink",
                      "ApplicationInsightsProfiler": "Enter the Application Insights instance instrumentation key guid here"
                    }
                  ]
                },
                "DiagnosticMonitorConfiguration": {
                    ...
                }
                ...
              }
              ...
            }
            ...
          }
          ...
]
```

## Working with Virtual Machine Scale Set
Download the [WindowsVirtualMachineScaleSet.json](https://wadexample.blob.core.windows.net/wadexample/WindowsVirtualMachineScaleSet.json) template to see how to enable the Profiler. You have to make sure each instance in the Scale Set has access to Internet, so the Profiler Agent can send the collected samples to Application Insights to be analyzed and displayed.


## Troubleshooting:

https://docs.microsoft.com/en-us/azure/application-insights/app-insights-profiler#a-idtroubleshootingatroubleshooting

[Overview]:./media/EnableProfilerForCompute/overview.png
[Create-AppInsights]:./media/EnableProfilerForCompute/CreateAI.png
[Find-AI-Key]: ./media/EnableProfilerForCompute/CopyAIKey.png
[Replace-TemplateValue]:./media/EnableProfilerForCompute/CopyAIKeyToTemplate.png
[Publish-AzureVM]:./media/EnableProfilerForCompute/PublishToVM.png
