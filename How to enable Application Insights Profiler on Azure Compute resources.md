# How to enable Application Insights Profiler on Azure Compute resources

This walkthrough demonstrates how to enable Application Insights Profiler on an ASP.NET applications hosted by Azure Compute resources. The examples include Virtual Machine and Virtual Machine Scale Sets.

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
Create a resource group in your Azure subscription. Below is an example for doing so in using PowerShell script:

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
New-AzureRmResourceGroupDeployment -ResourceGroupName "Replace_With_Resource_Group_Name" -TemplateFile .\WindowsVirtualMachine.json -adminUsername "Replace_With_your_user_name" -adminPassword $password -dnsNameForPublicIP "Replace_WIth_your_DNS_Name" -Verbose
```

After the script executes successfully, you should find a VM named *MyWindowsVM* in your resource group.

### Configure Web Deploy on the VM
Make sure **Web Deploy** is enabled on your VM so your can publish your web application from Visual Studio.
Here is a quick way to install Web Deploy on a VM manually via WebPI:[Installing and Configuring Web Deploy on IIS 8.0 or Later](https://docs.microsoft.com/en-us/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later)
Here is an example for how to automate installing Web Deploy using Azure Resource Manager template:
[Create, configure and deploy Web Application to an Azure VM](https://azure.microsoft.com/en-us/resources/templates/201-web-app-vm-dsc/)

If you are deploying an ASP.NET MVC application, you will need to go to Server Manager, **Add Roles and Features | Web Server (IIS) | Web Server | Application Development** and enable ASP.NET 4.5 on your server.
![Add ASP.NET][Enable-ASPNET]

### Install Azure Application Insights SDK to your project
* Open your ASP.NET web application in Visual Studio
* Right click on the project and select **Add | Connected Services**
* Choose Application Insights
* Follow the instructions on the page. Select the Application Insights resource you have created earlier
* Click the **Register** button


### Publish the project to Azure VM
There are several ways to publish an application to an Azure VM. One way is to do so in Visual Studio 2017.
Right click on the project, select 'Publish...'. Select Azure Virtual Machine as the publish target and follow the steps to finish the publish process.

![Publish-FromVS][Publish-AzureVM]

Run some load test against your application. you should be able to see results in the Application Insights instance portal webpage.
Check https://docs.microsoft.com/en-us/azure/application-insights/app-insights-profiler#viewing-profiler-data for more details.

## Enable the Profiler
Go to your Application Insights Performance blade. Click on the Configure icon and Enable the Profiler

![Enable Profiler step 1][Enable-Profiler1]

![Enable Profiler step 2][Enable-Profiler2]

## Add an Availability Test to your application
Browse to the Application Insights resource you created earlier. Go to the Availability blade and add a performance test that sends web requests to your application URL. This way we can collect some sample data to be displayed in the Application Insights Profiler

![Add Performance Test][Add-Test]

## View your performance data

Wait for a 10-15 minutes for the Profiler to collect and analyze the data. Then go to the Performance blade in your AI resource and view how your application was performing when it's under load.

![View Performance][View-AIPerformance]

Clicking on the icon under Examples with open the Trace View blade.

![Trace View][TraceView]


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
[Enable-ASPNET]:./media/EnableProfilerForCompute/AddASPNET45.png
[Enable-Profiler1]:./media/EnableProfilerForCompute/EnableProfiler1.png
[Enable-Profiler2]:./media/EnableProfilerForCompute/EnableProfiler2.png
[Add-Test]:./media/EnableProfilerForCompute/AvailabilityTest.png
[View-AIPerformance]:./media/EnableProfilerForCompute/AIPerformance.png
[TraceView]:./media/EnableProfilerForCompute/TraceView.png
