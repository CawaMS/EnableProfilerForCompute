# How to enable Application Insights Profiler on Azure Compute resources

If you have an application installed in an Azure Compute resource (e.g.: a simple VM)
and you'd like to start using Application Insights Profiler on it, this will guide you.


## What you'll need:

* The current deployment template for the Azure Compute resource where Application Insights Profiler will be installed.
* An Application Insights instance enabled for profiling. Check https://docs.microsoft.com/en-us/azure/application-insights/app-insights-profiler#enable-the-profiler to see how to do that.
* .NET framework >= 4.6.1 installed in the target Azure Compute resource.


## How to do it:

* Locate the Windows Azure Diagnostics (WAD) resource declaration in your deployment template.
  * Create one if you don't have it yet (check how it's done in the full example).
  * You can update the template from the Azure Resource website (https://resources.azure.com).
* Modify publisher from "Microsoft.Azure.Diagnostics" to "AIP.Diagnostics.Test".
* Use typeHandlerVersion as "0.0".
* Make sure to have autoUpgradeMinorVersion set to true.
* Add the new ApplicationInsightsProfiler sink instance in WadCfg settings object, following the example below.

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


## What to expect:

After deploying a WAD resource definition containing the ApplicationInsightsProfiler sink,
you should be able to see results in the Application Insights instance portal webpage.
Check https://docs.microsoft.com/en-us/azure/application-insights/app-insights-profiler#viewing-profiler-data for more on that.


## Troubleshooting:

https://docs.microsoft.com/en-us/azure/application-insights/app-insights-profiler#a-idtroubleshootingatroubleshooting


## Full Example
[WindowsVirtualMachine.json](https://wadexample.blob.core.windows.net/wadexample/WindowsVirtualMachine.json)
