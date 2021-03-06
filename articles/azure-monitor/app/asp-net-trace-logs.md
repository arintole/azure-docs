---
title: Explore .NET trace logs in Application Insights
description: Search logs generated with Trace, NLog, or Log4Net.
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm

ms.assetid: 0c2a084f-6e71-467b-a6aa-4ab222f17153
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 02/19/2019
ms.author: mbullwin

---
# Explore .NET/.NET Core trace logs in Application Insights

If you use ILogger, NLog, log4Net, or System.Diagnostics.Trace for diagnostic tracing in your ASP.NET/ASP.NET Core application, you can have your logs sent to [Azure Application Insights][start], where you can explore and search them. Your logs will be merged with the other telemetry coming from your application, so that you can identify the traces associated with servicing each user request, and correlate them with other events and exception reports.

> [!NOTE]
> Do you need the log capture module? It's a useful adapter for 3rd-party loggers, but if you aren't already using NLog, log4Net or System.Diagnostics.Trace, consider just calling [Application Insights TrackTrace()](../../azure-monitor/app/api-custom-events-metrics.md#tracktrace) directly.
>
>

## Install logging on your app
Install your chosen logging framework in your project. This should result in an entry in app.config or web.config.

```XML
    <configuration>
      <system.diagnostics>
    <trace autoflush="true" indentsize="0">
      <listeners>
        <add name="myAppInsightsListener" type="Microsoft.ApplicationInsights.TraceListener.ApplicationInsightsTraceListener, Microsoft.ApplicationInsights.TraceListener" />
      </listeners>
    </trace>
  </system.diagnostics>
   </configuration>
```

## Configure Application Insights to collect logs
**[Add Application Insights to your project](../../azure-monitor/app/asp-net.md)** if you haven't done that yet. You'll see an option to include the log collector.

Or **Configure Application Insights** by right-clicking your project in Solution Explorer. Select the option to **Configure trace collection**.

*No Application Insights menu or log collector option?* Try [Troubleshooting](#troubleshooting).

## Manual installation
Use this method if your project type isn't supported by the Application Insights installer (for example a Windows desktop project).

1. If you plan to use log4Net or NLog, install it in your project.
2. In Solution Explorer, right-click your project and choose **Manage NuGet Packages**.
3. Search for "Application Insights"
4. Select one of the following packages:

   - For ILogger: [Microsoft.Extensions.Logging.ApplicationInsights](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights/)
[![Nuget](https://img.shields.io/nuget/vpre/Microsoft.Extensions.Logging.ApplicationInsights.svg)](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights/)
   - For NLog: [Microsoft.ApplicationInsights.NLogTarget](https://www.nuget.org/packages/Microsoft.ApplicationInsights.NLogTarget/)
[![Nuget](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.NLogTarget.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.NLogTarget/)
   - For Log4Net: [Microsoft.ApplicationInsights.Log4NetAppender](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Log4NetAppender/)
[![Nuget](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.Log4NetAppender.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Log4NetAppender/)
   - For System.Diagnostics: [Microsoft.ApplicationInsights.TraceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.TraceListener/)
[![Nuget](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.TraceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.TraceListener/)
   - [Microsoft.ApplicationInsights.DiagnosticSourceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener/)
[![Nuget](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.DiagnosticSourceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener/)
   - [Microsoft.ApplicationInsights.EtwCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector/)
[![Nuget](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.EtwCollector.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector/)
   - [Microsoft.ApplicationInsights.EventSourceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener/)
[![Nuget](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.EventSourceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener/)

The NuGet package installs the necessary assemblies, and where applicable modifies the web.config or app.config.

## ILogger

For examples of using the Application Insights ILogger implementation with Console applications and ASP.NET Core check out this [article](ilogger.md).

## Insert diagnostic log calls
If you use System.Diagnostics.Trace, a typical call would be:

    System.Diagnostics.Trace.TraceWarning("Slow response - database01");

If you prefer log4net or NLog:

    logger.Warn("Slow response - database01");

## Using EventSource events
You can configure [System.Diagnostics.Tracing.EventSource](https://msdn.microsoft.com/library/system.diagnostics.tracing.eventsource.aspx) events to be sent to Application Insights as traces. First, install the `Microsoft.ApplicationInsights.EventSourceListener` NuGet package. Then edit `TelemetryModules` section of the [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) file.

```xml
    <Add Type="Microsoft.ApplicationInsights.EventSourceListener.EventSourceTelemetryModule, Microsoft.ApplicationInsights.EventSourceListener">
      <Sources>
        <Add Name="MyCompany" Level="Verbose" />
      </Sources>
    </Add>
```

For each source, you can set the following parameters:
 * `Name` specifies the name of the EventSource to collect.
 * `Level` specifies the logging level to collect. Can be one of `Critical`, `Error`, `Informational`, `LogAlways`, `Verbose`, `Warning`.
 * `Keywords` (Optional) specifies the integer value of keywords combinations to use.

## Using DiagnosticSource events
You can configure [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md) events to be sent to Application Insights as traces. First, install the [`Microsoft.ApplicationInsights.DiagnosticSourceListener`](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener) NuGet package. Then edit the `TelemetryModules` section of the [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) file.

```xml
    <Add Type="Microsoft.ApplicationInsights.DiagnosticSourceListener.DiagnosticSourceTelemetryModule, Microsoft.ApplicationInsights.DiagnosticSourceListener">
      <Sources>
        <Add Name="MyDiagnosticSourceName" />
      </Sources>
    </Add>
```

For each DiagnosticSource you want to trace, add an entry with the `Name` attribute  set to the name of your DiagnosticSource.

## Using ETW events
You can configure ETW events to be sent to Application Insights as traces. First, install the `Microsoft.ApplicationInsights.EtwCollector` NuGet package. Then edit `TelemetryModules` section of the [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) file.

> [!NOTE] 
> ETW events can only be collected if the process hosting the SDK is running under an identity that is a member of "Performance Log Users" or Administrators.

```xml
    <Add Type="Microsoft.ApplicationInsights.EtwCollector.EtwCollectorTelemetryModule, Microsoft.ApplicationInsights.EtwCollector">
      <Sources>
        <Add ProviderName="MyCompanyEventSourceName" Level="Verbose" />
      </Sources>
    </Add>
```

For each source, you can set the following parameters:
 * `ProviderName` is the name of the ETW provider to collect.
 * `ProviderGuid` specifies the GUID of the ETW provider to collect, can be used instead of `ProviderName`.
 * `Level` sets the logging level to collect. Can be one of `Critical`, `Error`, `Informational`, `LogAlways`, `Verbose`, `Warning`.
 * `Keywords` (Optional) sets the integer value of keyword combinations to use.

## Using the Trace API directly
You can call the Application Insights trace API directly. The logging adapters use this API.

For example:

    var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
    telemetry.TrackTrace("Slow response - database01");

An advantage of TrackTrace is that you can put relatively long data in the message. For example, you could encode POST data there.

In addition, you can add a severity level to your message. And, like other telemetry, you can add property values that you can use to help filter or search for different sets of traces. For example:

    var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
    telemetry.TrackTrace("Slow database response",
                   SeverityLevel.Warning,
                   new Dictionary<string,string> { {"database", db.ID} });

This would enable you, in [Search][diagnostic], to easily filter out all the messages of a particular severity level relating to a particular database.

## Explore your logs
Run your app, either in debug mode or deploy it live.

In your app's overview blade in [the Application Insights portal][portal], choose [Search][diagnostic].

You can, for example:

* Filter on log traces, or on items with specific properties
* Inspect a specific item in detail.
* Find other telemetry relating to the same user request (that is, with the same OperationId)
* Save the configuration of this page as a Favorite

> [!NOTE]
> **Sampling.** If your application sends a lot of data and you are using the Application Insights SDK for ASP.NET version 2.0.0-beta3 or later, the adaptive sampling feature may operate and send only a percentage of your telemetry. [Learn more about sampling.](../../azure-monitor/app/sampling.md)
>
>

## Next steps
[Diagnose failures and exceptions in ASP.NET][exceptions]

[Learn more about Search][diagnostic].

## Troubleshooting
### How do I do this for Java?
Use the [Java log adapters](../../azure-monitor/app/java-trace-logs.md).

### There's no Application Insights option on the project context menu
* Check that Application Insights tools are installed on this development machine. In Visual Studio menu Tools, Extensions and Updates, look for Application Insights Tools. If it isn't in the Installed tab, open the Online tab and install it.
* This might be a type of project not supported by Application Insights tools. Use [manual installation](#manual-installation).

### No log adapter option in the configuration tool
* You need to install the logging framework first.
* If you're using System.Diagnostics.Trace, make sure you [configured it in `web.config`](https://msdn.microsoft.com/library/system.diagnostics.eventlogtracelistener.aspx).
* Have you got the latest version of Application Insights? In Visual Studio **Tools** menu, choose **Extensions and Updates**, and open the **Updates** tab. If Developer Analytics tools are there, click to update it.

### <a name="emptykey"></a>I get an error "Instrumentation key cannot be empty"
Looks like you installed the logging adapter Nuget package without installing Application Insights.

In Solution Explorer, right-click `ApplicationInsights.config` and choose **Update Application Insights**. You'll get a dialog that invites you to sign in to Azure and either create an Application Insights resource, or reuse an existing one. That should fix it.

### I can see traces in diagnostic search, but not the other events
It can sometimes take a while for all the events and requests to get through the pipeline.

### <a name="limits"></a>How much data is retained?
Several factors impact the amount of data retained. See the [limits](../../azure-monitor/app/api-custom-events-metrics.md#limits) section of the customer event metrics page for more information. 

### I'm not seeing some of the log entries that I expect
If your application sends a lot of data and you are using the Application Insights SDK for ASP.NET version 2.0.0-beta3 or later, the adaptive sampling feature may operate and send only a percentage of your telemetry. [Learn more about sampling.](../../azure-monitor/app/sampling.md)

## <a name="add"></a>Next steps
* [Set up availability and responsiveness tests][availability]
* [Troubleshooting][qna]

<!--Link references-->

[availability]: ../../azure-monitor/app/monitor-web-app-availability.md
[diagnostic]: ../../azure-monitor/app/diagnostic-search.md
[exceptions]: asp-net-exceptions.md
[portal]: https://portal.azure.com/
[qna]: ../../azure-monitor/app/troubleshoot-faq.md
[start]: ../../azure-monitor/app/app-insights-overview.md
