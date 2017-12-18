# Open DACS and Open DACS Permission Server

## Introduction
Open DACS API and Open DACS Permission Server (ODPS) are two ways fo a developer to get a programmatic, application's view into DACS.  Developers often face tasks that require the programmatic "window" into DACS.  One very important use case is implementing on-behalf of entitlement checking: i.e., subscribing to a comprehensive set of market data from data source(s) once, and redistributing a subset of this market data to any number of entitled users.  The ultimate goal of this type of use cases is to redistribute market data in a fully exchange-compliant manner.  This is where both Open DACS API and Open DACS Permission Server can be of help.

#### Permissioning
The goal of permissioning is to control access to data by users. Using an entitlement system, such as Data Access Control System (DACS), user permission profiles can be defined.  Permission profiles define what instruments a user is allowed to access.  Using OpenDACS or Open DACS Permission Server, allowed(entitled) permissions can be verified, and if the requesting user is permissioned for content, then the content is be made available to the user.

#### Open DACS Overview
The DACS Authorization API enables a class of applications which require authorization checks to be performed outside the scope of TREP-embedded authorization checks.  It is a "tight" integration option.  Open DACS library is emdedded wihin a custom application, and the application is responsible for making the required entitlement check calls using the library.
![Open DACS With DACS]( https://github.com/TR-API-Samples/Article.OpenDACSandODPS/blob/master/OpenDACSWithDACS.gif)

#### ODPS Overview
The Thomson Reuters Open DACS Permission Server (ODPS) complements the existing Open DACS API offering. ODPS allows an application to perform DACS permissioning and generate DACS usage data without having to embed OpenDACS API in the application. Instead, to gain access to all Open DACS API functionality, the application can contact ODPS using the industry standard HTTP 1.1/REST protocol. This is an extrenal, or "loose" integration option, via HTTP request/response.

![ODPS With DACS](https://github.com/TR-API-Samples/Article.OpenDACSandODPS/blob/master/ODPSwithDACS%20.gif)

## Approach Considerations

#### Time and Effort to Implement
The time and effort that are required to integrate the entitlement check are often a very important consideration.  
Open DACS API is implemented with RFA API, and RFA-like code integration is required. The language choices are C++, Java.and C#. The build includes RFA libraries.
ODPS accepts HTTP requests, therefore, any language that supports HTTP requests will be an option.   The included examples of http clients are C, Java and Perl, but any language and enviroment supporting HTTP request/response can be utilized.

#### Coupling and Flexibility
OpenDACS application calls OpenDACS from within, i.e. the integration is tight, from within the application.  I.e. the developer/architect has a complete freedom in choosing if to implement, not to implement, or how to implement robustness, error handling and application-level reporting.
Let's note that the communication with DACS is going to be facilitated over DACS sink daemon, that can be local, or remote, but is an outside process with either of these choices. 
ODPS client calls ODPS via HTTP, issuing HTTP request and receiving HTTP response.  Integration is loose.  The functionality offered by ODPS is used, the calls to ODPS are integrated into the overall application design.  It's very common to use ODPS for ad-hoc testing, utilizing a browser to send HTTP requests and observe HTTP responses.

#### Mandatory Checks
In order to be compliant, the following checks have to be made:*User login, followed by user logout, confirms that a user is entitled, and user's maximum number allowed simultaneous connections is not exceeded.*Subscription test or PE test verifies that a user is entitled to certain items*Periodic repremission test verifies that entitlements for a specific user are still valid, and re-requests the entitlements is they are not.
There are two important principles to permissioning that application developer should uphold at all times in order for the application to be compliant:*Entitlements should be properly verified, prior to the release of data, as described above *Usage of data should be properly reflected within DACS

## Caching and Performance

#### DACS Lock Caching 
DACS Lock Caching is an important for both OpenDACS and ODPS applications.  OpenDACS applications often keeps a cache of locks/PEs, it's up to the application designer to implement this in order to minimize the unnecessary entitlement info retrieval from DACS.ODPS implements caching of locks/PEs out of the box, it is configurable, including "keep forever" but default keep time is 60 minutes.  
To clarify why this is useful, here is an example of ODPS cache:S,"IDN","TRI.N","L:0301116562c0",FALSES,"IDN","IBM.N","L:03011162c0",FALSES,”IDN”,”EUR=”,”P:&16,34,132”,FALSEWhere column 2 contains the item name and column 4 contains locks/PE lists.
This caching allows mapping item to PEs or lock without referencing infrastrcuture, performing actual subscription and triggering an actual usage.

#### User Profile Caching
Open DACS application and ODPS keep track of logged in users and their entitlements.  When user initially logs in, the profile, or what the user is entitled to access are retrieved from DACS. 
ODPS caches the profile, and within the cached period, subsequent checks use the cached info.
Open DACS app often implement caching of user permission profiles.

#### Custom Caching
Both kind of apps may choose to implement custom caches based on the specific requirements of an application.  OpenDACS app is more likely to impelement custom caching.

Content list command (getPEList Open DACS method and contentList command in ODPS) can be run per user to return the list of assigned PEs (for content-based services).  To clarify, here is an example of this information:

SUCCESS:1 2 3 4 6 7 8 18 20 62

This info can be cached.

Subscription list command (detSubscriptionList Open DACS method and subscriptionList command in ODPS) can be run per user to return a list of assigned subscriptions (for subject-based services) To clarify, here is an example of this information:

\*.O

Also, when item subscription is established, if it's content-based, PEs can be found in PROD_PERM FID.

The performance for subscription test on ODPS is roughly 20K operations per second.  Thus, running it on every subscription request is not efficient, for some applications it's not sufficcient (they require more frequent checks), and this command also generates dacs usage events.  Which makes caching a very popular approach.

## Service Type Considerations

How service is defined in DACS is very important when planning a programmatic compliant solution.  The three types of service definition in DACS are:
*Content-Based
*Subject-Based
*Service Level

#### Content-Based Service
Entitlement check is based on Permission Entitity or PE.  Therefore, the OpenDACS API functionality and ODPS requests based on PEs will be available for this type of service. 
The most common ODPS and OpenDACS actions are *content list retrieval per user, returning the complete list of PEs assigned to the user (getPEList OpenDACS method, contentList ODPS command)*permission entity check, (usually accepting mulitple PEs as input and returning positive or negative answer on each of them, peCheck ODPS command,implemented via getPEList when integrating with OpenDACS)*subscription test that is based on DACS lock or PE (subscriptionTest ODPS command, implemented via getPEList)

#### Subject-Based Serice
Entitlement check is based on item name that makes up "subject".  The entitlement checks available for this type of service are on subject, that is expressed as RIC name, for example, "TRI.N", "GOOG.O" or wild-carded RIC name convention,f or example "*.N"  
The most common ODPS and OpenDACS actions are: *subscription list (usually returning wild-card answers such as "\*.N", getSubscriptionList OpenDACS function and subscriptionList ODPS command) *subscription test (accepting instrument as input and returning positive or negative answer as output, checkSubscription OpenDACS method and subscriptionTest ODPS command)

#### Service Level Authorization
User can be either entitled or not entitled to service as a whole.  This is the simplest, but least flexible type of permissioning.

## Redundant Solution

ODPS installations often handle ODPS load-balancing as well as ODPS server failure by imploying network load-balancing solution.  There is configuration option to disable http listening port on failure, so that load-balancing controller will seize sending requests to the failed ODPS.
OpenDACS applications implement redundancy according to requirements of the specific app. Redundancy is planned and designed as part of the overall application design, it's not included within functionality available from API, its design is up to the application developer/architect. 

## Tools

#### DACS UI
DACS UI is a tool that may or may not be directly accessible to a developer.  This is a graphical user interface for DACS administration, used by DACS administrators/market data groups.  It can be used to verify the permissions assigned, prior to retrival by OpenDACS or ODPS.  If it's not directly available, and a confimration of permissions is required, one may request and get the info via DACS administrator who has the access to the DACS UI.

#### Web Browser
ODPS asseps HTTP requests, that can be originated through a web browser, by typing the request into browser's address bar.  For example:

"http://myodps:8088/command=cacheDump"

## References

+ Open DACS DEVELOPERS GUIDE
+ ODPS INSTALL AND DEVELOPERS GUIDE

## GlossaryAPI         

+ Application Programming InterfaceDACS        
+ Data Access Control SystemPE          
+ Permission EntityDACS lock   
+ Infrastructure authorization information for specific item is constructed from service id and operator id, and often includes the list of required PEs 
