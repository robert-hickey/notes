Pre-Page Exeuction: Request Architecture
========================================
> The architecture explained in these notes is based on IIS Integrated Mode.

All inbound requests are handled by the http.sys process, which routes the request to the correct handler. It uses the Windows Activation Service (WAS) to identify configuration data from the applicationHost.config file. This identifies the application pool and other site configuration settings. WAS then hands this information off to the World Wide Web Publishing Services (W3SVC) which is responsible managing the HTTP protocol and performance counters. It is also responsible for keeping http.sys up-to-date on configuration changes. Finally, W3SVC hands the request back to http.sys, which adds it to the request queue.

The Worker Process (w3wp.exe) is responsible for handling the requests from the queue. While being processed, the request passes through a number of stages known as the Integrated Pipeline, which consists of the stages detailed below:

* __BeginRequest__
* __AuthenticateRequest__
* __MapRequestHandler__ - decides which handler will execute the request, and is based on the Handler Mapping Configuration in IIS.
* __AcquireRequestState__
* __ExecuteRequestHandler__ - where the request is executed using the handler determined by the MapRequestHandler event.
* __EndRequest__

Managed .NET HttpModules can subscribe to the various pipeline events. Modules are executed in the order in which they are registered with IIS. The ManagedEngine (a native module) kicks in when the first managed module needs to be executed, and handles the entire ASP.NET integration. For the first request, it will create an instance of the ApplicationManager class (which corresponds to the AppDomain) which will instantiate the HttpApplication (global.asax). The HttpApplication fires the same event(s) that the integrated pipeline fired, at which point the modules you have registered in code will handle event. Existing instances will be used for subsequent requests. 

Finally, once the pipeline has finished processing the request, it is handed back to http.sys which serves the generated response back to the client. 

IIS Classic Mode (and the advantages of integrated mode)
----------------------------------------------------
