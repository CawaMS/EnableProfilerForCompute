# How to augment Windows Azure Diagnostics (WAD) deployment template with Application Insights Profiler

## You'll need:

- The current deployment template matching your resource group state.
- The instrumentation key guid for the Application Insights instace you want to activate profiler on
    (this matches the key used by the application hosted in your VM that you're interested on).


## How to do it:

	•  Locate the windows Azure Diagnostics (WAD) resource declaration in your deployment template
		○ You can update the template from the Azure Resource website.
	•  Modify publisher from "Microsoft.Azure.Diagnostics" to "AIP.Diagnostics.Test"
	•  Use typeHandlerVersion as "0.0"
	•  Make sure to have autoUpgradeMinorVersion set to true
	•  Add the new ApplicationInsightsProfiler sink instance following the example:

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
                      "ApplicationInsightsProfiler": "Enter your Application Insights instrumentation key guid"
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

## Full Examples
[WindowsVirtualMachine.json](https://wadexample.blob.core.windows.net/wadexample/WindowsVirtualMachine.json)
