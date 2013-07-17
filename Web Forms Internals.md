# Pre-Page Exeuction: Request Architecture
> The architecture explained in these notes is based on IIS Integrated Mode.

All inbound requests are handled by the http.sys process, which routes the request to the correct handler. It uses the Windows Activation Service (WAS) to identify configuration data from the applicationHost.config file. This identifies the application pool and other site configuration settings. WAS then hands this information off to the World Wide Web Publishing Services (W3SVC) which is responsible managing the HTTP protocol and performance counters. It is also responsible for keeping http.sys up-to-date on configuration changes. Finally, W3SVC hands the request back to http.sys, which adds it to the request queue.

The Worker Process (w3wp.exe) is responsible for handling the requests from the queue. While being processed, the request passes through a number of stages known as the Integrated Pipeline, which consists of the stages detailed below:

* __BeginRequest__
* __AuthenticateRequest__
* __MapRequestHandler__ - decides which handler will execute the request, and is based on the Handler Mapping Configuration in IIS.
* __AcquireRequestState__
* __ExecuteRequestHandler__ - where the request is executed using the handler determined by the MapRequestHandler event. Relies on the PageHandlerFactory to map an inbound request to an existing *.aspx page.
* __EndRequest__

Managed .NET HttpModules can subscribe to the various pipeline events. Modules are executed in the order in which they are registered with IIS. The ManagedEngine (a native module) kicks in when the first managed module needs to be executed, and handles the entire ASP.NET integration. For the first request, it will create an instance of the ApplicationManager class (which corresponds to the AppDomain) which will instantiate the HttpApplication (global.asax), HttpRuntime and HttpContext. The HttpApplication fires the same event(s) that the integrated pipeline fired, at which point the modules you have registered in code will handle event. Existing instances will be used for subsequent requests. 

Finally, once the pipeline has finished processing the request, it is handed back to http.sys which serves the generated response back to the client. 

### Http Module Configuration

Modules that indicate an Entry Type of 'Inherited' are configured at the applicationHost.config file level (in the globalModules section). This means any change will affect every application running under this IIS instance. To prevent this, you can set the entry type to 'Local' and configure them in your web/app.confg.  Note that the order in which these modules are defined in configuration are the order in which they are executed!

### IIS Classic Mode (and the advantages of integrated mode)

* improved performance, no longer need to load an ISAPI extension to handle .net code
* managed modules can be used to process non ASP.NET files (such as images). 

# Introducing ViewState
> ViewState is critical to understanding the page life-cycle. It is much more than a variable that persists state across postbacks. 

ViewState is a property of System.Web.UI.Control that has a key/value pair indexer. It is of type System.Web.UI.StateBag. The StateBag class has the ability to track changes, which can be enabled by calling "TrackViewState()". A tracked member is said to be "Dirty" (IsItemDirty()). Tracking is by default off, but once turned on it cannot be turned off. The key thing to know in relation to ViewState is that items will be marked as 'Dirty' during the Page Life Cycle.

### ViewState Basics

* Technique used by ASP.NET to remember change of state between multiple requests
* Saved in __VIEWSTATE hidden HTML field
* Two serialized data types are stored in __VIEWSTATE:
    * _Programmatic_ (not Static!) - change of page and control state (i.e. "Dirty" items)
    *  Explicit data stored using ViewState[""] indexer.
* ASP.NET Server Controls use ViewState to store property values
    * ex. Text property of TextBox control
    * public virtual string Text{ get { return (string) this.ViewState["Text"] ?? string.Empty; } 
    * This rule should be applied when developing your own server controls!
* Note: this does NOT mean that values are stored in __VIEWSTATE . Only state changes are stored in __VIEWSTATE, not design-time data.

# The Page Lifecycle and ViewState





