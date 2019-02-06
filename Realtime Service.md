# Realtime Service
### What is Realtime Service?
Realtime service enable users to receive information as soon as it is published by its authors, rather than requiring that they check a source periodically for updates. It guarantees immediate updates to all connected users.

These are few application scenarios where realtime updates are important,
1. Bulk email or An admin sends an email to all of his/her students and want to track the status of each email. (whether it's sent, failed or bounced).
2. To track progress of inter-service communication.

### Terminology
- ***Hub*** : Hub enables you to make remote procedure calls (RPCs) from a server to connected clients and from clients to the server. In server code, you define methods that can be called by clients, and you call methods that run on the client.
- ***Hub Connection*** : It is a Web-Socket connection of Server Hub with Client Application.
- ***Group Name*** : Used to categories messages. Hub conncetion is created based on group name.

![Realtime Service](https://demographicstorage.blob.core.windows.net/public/realtime_service.png)

### How to Use
***Server Side Setup***
1. Install ***Exxat.Services.Foundation.Background.Jobs*** nuget package from Exxat Nuget Package Source.
2. Create Hub
    - Create a new class as SampleHub
    - Inherit SampleHub from ***HubBase*** class using ***Exxat.Services.Foundation.Background.Hubs*** 
    - Provide DI of IRootConfiguration service to base class
    SampleHub code will look like :
        ```
        using Exxat.Service.Foundation.Infrastructure.Core;
        using Exxat.Services.Foundation.Background.Hubs;
        namespace Exxat.Services.Example.API.Hubs
        {
            public class SampleHub : HubBase
            {
                public SampleHub(IRootConfiguration rootConfiguration) : base(rootConfiguration)
                { 
                    //This is sample Hub
                }
            }
        }
        ```
3. Map Hub with URL in Configure() Method of `Startup.cs` of your project, This URL will be used to create Hub Connection with Client.
    ```
    app.UseAzureSignalR(routes => {
        routes.MapHub<SampleHub>("/hub/sample");
    });
    ```
4. Set Azure SignalR connection string in `appsettings.json` as :
    ```
    "Azure": {
        "SignalR": {
          "ConnectionString": "//Azure SignalR connection string here"
        }
      }
    ```
5. Configure required services for Foundation.Background.Jobs as follows : 
    ```
    Exxat.Services.Foundation.Background.Jobs
    .Configuration.IocContainerConfiguration.ConfigureService(services, configuration);
    ```
6. Inject IRealtimeService in your service
    ```
        private readonly IRealtimeService<SampleHub> _realtimeService;
        public JobLoggerService(IRealtimeService<SampleHub> realtimeService)
        {
            _realtimeService = realtimeService;
        }
    ```
    and use it as following to send messages to connected clients : 
    ```
        _realtimeService.SendToGroupAsync(message,"groupname");
    ```
    That's all, we have configured server side things.
    
***Client Side Setup***
1. Import **RealtimeService** from **@exxat/foundation**
2. Inject RealtimeService in constructor of your Component or Service.
    ```
    export class SampleComponent {
      constructor(private realtimeService : RealtimeService){
            // RealtimeService is injected
      }
    }
    ```
3. Start connection with backend Hub
    ```
    realtimeService.start(groupName);
    ```
4. RealtimeService has an EventEmitter (messageReceived) which is emitted when a meesage is received from server. To get received message subscibe to this EventEmiiter.
    For example :
    ```
    realtimeService.messageReceived.subscribe((message) => {
      console.log(JSON.stringify(message));
    });
    ```
5. Set hub connection string in `development.json` as follows :
    ```
    {
      "realtime": {
        "connectionString": "//hub connection string ex. http://localhost:5000/hub/sample"
      }
    }
    ```
> **Note :** This connection string is same URL that we have mapped in third step of server side setup.  

### Troubleshoot
We have tried to cover everything in this document; However, you might still face some problems. Here is the list of common errors with their solution.

|Error|Solution|
|----|----|
|Failed to complete negotiation with the server | Check Hub connection string is correct in development.json and your server application is running |
|Message is not received by client| Make sure messages are sent to same group with which client is created. Client connection with Hub is created based on group name. Client will receive messages sent to that group only. Also TenantId needs to be present in browser's local storage |

That's all of Realtime service. Happy coding and keep smiling, you deserve a smile today :-)
