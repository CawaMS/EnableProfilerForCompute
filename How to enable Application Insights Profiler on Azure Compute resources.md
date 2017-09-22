# How to enable Application Insights Profiler on Azure Compute resources

This walkthrough demonstrates how to enable Application Insights Profiler on an ASP.NET applications hosted by Azure Compute resources. The examples include support for Virtual Machine, Virtual Machine Scale Sets, Services Fabric, Cloud Services, and applications hosted on on-prem servers.

The following will be presented in the walkthrough:
* Overview
* Enable the Profiler on Azure Virtual Machines v2
* Enable the Profiler on Virtual Machine Scale Sets
* Enable the Profiler on Service Fabric applications
* Enable the Profiler on Cloud Services applications
* Enable the Profiler on classic Azure Virtual Machines
* Enable the Profiler on on-premise servers
* Troubleshooting
* Feedback


## Overview

The diagram below illustrates how the Profiler works for Azure Compute resources. It uses Azure Virtual Machine as an example.

![Overview][Overview]

The Diagnostics Agent component is what we need to install on the Azure Compute resources so Application Insights Profiler can collect necessary information to process and display on Azure portal. The rest of the walkthrough intends to provide guidance for how to install and configure the diagnostics agent to enable the Application Insights Profiler.

### Prerequisites for the Walkthrough

* Download the deployment template that installs the Profiler agents on the VMs or Scale Sets.

    [WindowsVirtualMachine.json](./WindowsVirtualMachine.json) | [WindowsVirtualMachineScaleSet.json](./WindowsVirtualMachineScaleSet.json)
* An Application Insights instance enabled for profiling. Check https://docs.microsoft.com/en-us/azure/application-insights/app-insights-profiler#enable-the-profiler to see how to do that.
* .NET framework >= 4.6.1 installed in the target Azure Compute resource.


## Enable the Profiler on Azure Virtual Machines v2
These steps will be explained in this section:
* Create Resource Group in your Azure subscription
* Create an Application Insights resource in the Resource group
* Apply Application Insights Instrumentation Key in the Azure Resource Manager template
* Create Azure VM to host the web application
* Configure Web Deploy on the VM
* Install Azure Application Insights SDK to your project
* Publish the project to Azure VM
* Enable the Profiler on Application Insight
* Add an Availability Test to your application
* View your performance data

### Create Resource Group in your Azure subscription
Create a resource group in your Azure subscription. Below is an example for doing so in using PowerShell script:

```
New-AzureRmResourceGroup -Name "Replace_With_Resource_Group_Name" -Location "Replace_With_Resource_Group_Location"
```

### Create an Application Insights resource in the Resource group

![Create Application Insights][Create-AppInsights]

### Apply Application Insights Instrumentation Key in the Azure Resource Manager template
If you haven't downloaded the template yet, download the template from below. [WindowsVirtualMachine.json](./WindowsVirtualMachine.json)

![Find AI Key][Find-AI-Key]

![Replace Template Value][Replace-TemplateValue]

### Create Azure VM to host the web application
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

### Enable the Profiler on Application Insight
Go to your Application Insights Performance blade. Click on the Configure icon and Enable the Profiler

![Enable Profiler step 1][Enable-Profiler1]

![Enable Profiler step 2][Enable-Profiler2]

### Add an Availability Test to your application
Browse to the Application Insights resource you created earlier. Go to the Availability blade and add a performance test that sends web requests to your application URL. This way we can collect some sample data to be displayed in the Application Insights Profiler

![Add Performance Test][Add-Test]

### View your performance data

Wait for a 10-15 minutes for the Profiler to collect and analyze the data. Then go to the Performance blade in your AI resource and view how your application was performing when it's under load.

![View Performance][View-AIPerformance]

Clicking on the icon under Examples with open the Trace View blade.

![Trace View][TraceView]


### What to Add If You Have an Existing Azure Resource Manager (ARM) Template for VM or VMSS

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

## Enable the Profiler on Virtual Machine Scale Sets
Download the [WindowsVirtualMachineScaleSet.json](./WindowsVirtualMachineScaleSet.json) template to see how to enable the Profiler. Apply the same changes in a VM template to VMSS diagnostics extension resource.
You have to make sure each instance in the Scale Set has access to Internet, so the Profiler Agent can send the collected samples to Application Insights to be analyzed and displayed.

## Enable the Profiler on Service Fabric applications
Currently enabling the Profiler on Service Fabric applications requires the following:
1. Provision the Service Fabric Cluster have the WAD extension that installs the Profiler agent
2. Install Application Insights SDK in the project and configure AI Key
3. Add application code to instrument telemetry

### Provision the Service Fabric Cluster have the WAD extension that installs the Profiler agent
A Service Fabric cluster can be secure or non-secure. A typical having the Gateway cluster to be non-secure so it doesn't require certificate to access it, together with one or more secure clusters that hosts the business logics and handles the data. The Profiler can be enabled on both secure and non-secure Service Fabric clusters. The walkthrough uses non-secure cluster as example to explain what changes are required to enable the Profiler. The same changes are applicable to a secure cluster.

Download the [ServiceFabricCluster.json](./ServiceFabricCluster.json). Same as for VMs and VMSS, replace the Application Insights Key with your AI Key:

```
"publisher": "AIP.Diagnostics.Test",
                 "settings": {
                   "WadCfg": {
                     "SinksConfig": {
                       "Sink": [
                         {
                           "name": "MyApplicationInsightsProfilerSinkVMSS",
                           "ApplicationInsightsProfiler": "[Application_Insights_Key]"
                         }
                       ]
                     },
```

Deploy the template using PowerShell script:
```
Login-AzureRmAccount
New-AzureRmResourceGroup -Name [Your_Resource_Group_Name] -Location [Your_Resource_Group_Location] -Verbose -Force
New-AzureRmResourceGroupDeployment -Name [Choose_An_Arbitrary_Name] -ResourceGroupName [Your_Resource_Group_Name] -TemplateFile [Path_To_Your_Template]

```

### Install Application Insights SDK in the project and configure AI Key
Install Application Insights SDK from NuGet Package. Make sure you install a stable version 2.3 or later. [Microsoft.ApplicationInsights.Web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/)
Please refer to [Using Service Fabric with Application Insights](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/blob/dev/appinsights/ApplicationInsights.md) for configuring the Application Insights in your projects.

### Add application code to instrument telemetry
For any piece of code you want to instrument, add a using statement around it. For example, the RunAsync method below is doing some work, and the telemetryClient class will instrument the operation once it starts. The event needs a unique name across the application.

```
protected override async Task RunAsync(CancellationToken cancellationToken)
       {
           // TODO: Replace the following sample code with your own logic
           //       or remove this RunAsync override if it's not needed in your service.

           while (true)
           {
               using( var operation = telemetryClient.StartOperation<RequestTelemetry>("[Insert_Event_Unique_Name]"))
               {
                   cancellationToken.ThrowIfCancellationRequested();

                   ++this.iterations;

                   ServiceEventSource.Current.ServiceMessage(this.Context, "Working-{0}", this.iterations);

                   await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
               }

           }
       }
```

Deploy your application to the Service Fabric cluster. Wait for the app to run for 10 minutes. For better effect you can run a load test on the app. Go to the Application Insights portal Performance blade, you should see Examples of profiling traces showing up.

## Enable the Profiler on Cloud Services applications
This walkthrough uses [Visual Studio 2017](https://www.visualstudio.com/)

### Prerequisites
1. Download the [Contoso University Cloud Services app](https://github.com/CawaMS/ContosoUniversityCS)
2.  Download scripts from [here](https://github.com/weng5e/WADProfilerEnabling/blob/master/TestNameSpace/CloudService.zip)

### Deploy the app to Azure
1. Sign-in to [Azure portal](http://portal.azure.com)
2. Create an Azure Cloud Services resource
3. Create an Azure SQL database
4. Create an Azure Storage account
5. In the **ContosoAdsWeb** project, open **Web.Release.config** file, add the following section. Replace the ContosoAdsContext connection string with your Azure SQL database connection string.
```
<connectionStrings>
    <add name="ContosoAdsContext" connectionString="{connectionstring}"
    providerName="System.Data.SqlClient" xdt:Transform="SetAttributes" xdt:Locator="Match(name)"/>
</connectionStrings>
```
6. In **ContosoAdsCloudService project**, open **ServiceConfiguration.Cloud.cscfg**, in Role element with name equals ContosoAdsWorker, change the value of **ContosoAdsDbConnectionString** to your Azure SQL database connection string.
7. In **ServiceConfiguration.Cloud.cscfg**, change the **"Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString"** and **"StorageConnectionString"** configuration settings under **ContosoAdsWeb** and **ContosoAdsWorker** to be the value of your Azure Storage connection string.
8. Publish your application to Azure. You can skip the Diagnostics settings page in the publish wizard for now as Application Insights will be configured later.

### Enable Application Insights on the application
Application Insights SDK needs to be added to the Web role and Worker role projects.

#### Add Application Insights to Web role projects
1. Double click on the Connected Services node in the **ContosoAdsWeb** project.
2. Choose **Monitoring with Application Insights**.
3. Follow instructions to add an Application Insights resource to your app.

#### Add Application Insights to Worker role projects
1. Create an Application Insights resource on [Azure portal](http://portal.azure.com).
2. Right click on **ContosoAdsWorker**
3. Select **Manage NuGet Packages...**
4. Search for **Microsoft.ApplicationInsights**
5. Install the latest version to your worker role projects
6. Unlike Web role projects, you need to add some application code to instrument requests made by the worker role. Open **WorkerRole.cs** file.
7. Import the following libraries:
```
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.DataContracts;
```
8. Suppose you are instrumenting some calls in the Run() method. Add the following to the method:
```
  TelemetryConfiguration.Active.InstrumentationKey = "[your_application_insights_key]";
```
9. Add code to instrument the function you want to collect data for. For example if you want to instrument the ProcessQueueMessage() function, you will add code around the function in a similar way to the following:
```
using (var operation = client.StartOperation<RequestTelemetry>("[unique_name_for_your_operation]"))
{
    ProcessQueueMessage(msg);
}
```
10. Redeploy your application to Azure with these changes

#### Enable Application Insights Profiler on the app
1. In the scripts you downloaded from [here](https://github.com/weng5e/WADProfilerEnabling/blob/master/TestNameSpace/CloudService.zip), open **EnableWADExtensionWithProfiler-CloudService.ps1**
2. edit the variables $subscriptionId, $serviceName, $storageAccountName, $storageAccountKey. The Storage Account is required by WAD to save collected data. You can use the Storage account you created earlier.
3. Open the xml file **wad-settings-with-profiler.xml** from the scripts you downloaded. Change the value of **ApplicationInsightsProfiler** element to your Application Insights account's instrumentation key
4. Run the script **EnableWADExtensionWithProfiler-CloudService.ps1**. This will ask you to sign in to your Azure Account. Wait for a few minutes for the script to finish.
5. Enable the Profiler by navigating to the Application Insights resource on Azure portal, browse to the performance blade, and click on the purple banner or click on **Configure** widget to go to the Profiler configuration page. Click Enable button.
If you are using the Preview version of the Performance blade, click on the Profiler widget on the top right hand side of the Performance blade, then click Enable to enable the Profiler
6. Generate some traffic to your app. You can run a performance and load test to your app.
7. Navigate to the Performance blade again. You should be able to see the Profiler traces showing up in the Operations table. If you are using the Preview performance blade, you will see a button labelled **Profiler Traces** in the Take Actions section on the bottom left corner of the blade.

## Enable the Profiler on classic Azure Virtual Machines
[TODO]

## Enable the Profiler on on-premise servers
We have no plan to officially support Profiler for on-premise servers. If you are interested in experimenting this scenario, you can download scripts from [here](./Profiler_installonprem.zip). We will not be responsible for maintaining these scripts.

## Troubleshooting

https://docs.microsoft.com/en-us/azure/application-insights/app-insights-profiler#a-idtroubleshootingatroubleshooting

## Feedback
Email serviceprofilerhelp@microsoft.com for questions or feedback

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
