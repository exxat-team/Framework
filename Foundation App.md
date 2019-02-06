# Foundation-App

Foundation app is an front end **base layer architecture and design standards** for Angular Apps Team. It represents a set of guidelines to **keep the project organised and maintainable**.The goal is to divide the application into functional layers and define responsibilities and restrictions for each one of them.The theory behind is framework agnostic and can be applied in any framework.  
The Foundation-App architecture is developed in **Angular-7**.  
Foundation-App contains features and Functionalities that are common to many pages like Document service, Notification Service, Logging services, Authentication service, Security service and so on. This services are very important for any application. Foundation-App has generalized some features and functionalities and implemented it at base layer so that the focus of other developers are on actual functionality.  
It provides guidelines and suggestions to decouple certain parts of your application in order to make it **more flexible and maintainable**. Like backend micro-service architecture it will help you to create **micro-frontend** architecture.
  
---
## Architectural Design

This Architectural design imposes some basic features for our entire front-end Application.

* **Modular design**  -  This means that we should be able to plug in/out our modules easily when needed, make each part of the app testable and enable multiple team members to work together on the project. 
* **Unidirectional data flow** - Angular will organize components in a hierarchical tree, which means that components can have a parent and children. and The **main rule is that actions go up and data flows down.**
![Unidirectional data flow](https://demographicstorage.blob.core.windows.net/public/DataFlow.png)
* **Predictable state management** - A state is a javascript object which holds the application data structure. Here we can store the data needed to display to the user like information about logged in user,information about TenantId etc. 
* **Communication layer for async requests** - Async services are group of modules each responsible for handling different types of communication to the external world.
* **Decoupled presentational layer from core layer** -  Our modules should have single responsibilities and do only one thing. where, **Core layer** contains application core logic e.g. data manipulation, communication with outer world etc. and **Presentation layer** fulfills the whole picture of one feature module, because all related parts are in one place.

---
## Architecture

![Frontend Architecture](https://demographicstorage.blob.core.windows.net/public/FrontendArchitecture.png)

---
## Features/Functionality


* Foundation Layout (Base Layout)
  * Foundation Core Module Layer
  * Foundation Core Facade Layer
  * Presentation Module
  * Sandbox Layer
* Well Design Folder Structure
* Asynchronous Services
  * HTTP Services
* Utility Services
  * Translation Services (Internationalization)
  * Configuration Services 
  * Notification Services
  * Logging Services
* Shared Components
* State Management
  * Fire Store services
  * Event Services
* Publisher Subscriber Methodologies

---

## Detailed Descriptions

### Foundation Layout (Base Layout)

Base Layout in any application helps to achieve a consistent and organized design. A good base layer design saves a lots of time and man efforts. It also reduces the redoing things for developers for many features and functionalities.

We’ll focus on application key building blocks and will give an example where to put it in the code.  
**Async services** are group of modules each responsible for handling different types of communication to the external world.  
**State management** consists of the pieces related to the temporary storage and state manipulation.  
**Application (Foundation) core facade** is an abstract class which holds common logic of the application core API. It includes functions that every Sandbox will inherit e.g. for getting the certain piece of the application state etc.  
**Sandbox**  is a service which extends application core facade and exposes streams of state  and connections to the async services.

Let’s do a quick recap of what’s going on here and how the communication is channeled through the **presentational modules** and **application core**.

1) Each presentational module subscribes, through it’s own sandbox, to events published by the application core
2) UI module calls one of the sandbox methods which triggers a corresponding process in  async services
3) Async service translates the message into suitable format and sends the request to the outer world (e.g. server request)
4) Asynchronous response from the server is translated by the application core into javascript object and forwarded further to the sandbox as a stream of data.
5) Each presentational module subscribed to that event gets notified through its sandbox

---
### Folder Structure
Now that we have a clearer picture of the overall design, we can see how to organize all of that in the code. We can start by creating a folder structure. It will help us visualize the problem and make it easier to start the development of each module. In practice there’s no clear cut between presentational and core layer and very often we need to mix them together because of practical reasons. We will organize our code into three main groups:
* **Presentational features** - logical units which represent rounded, standalone and reusable pieces of code. 
* **Shared features** - modules used through the entire application (e.g. async services, data management, utility services, configuration…). These parts represent the backbone of our app. 
* **features** - All base level features are created here.e.g. File-Upload. for uploading file you just need to use that file upload selector with required properties and it will upload a file. same way other base level functionality are created here.

![Foundation App Folder Structure](https://demographicstorage.blob.core.windows.net/public/Folder_structure.png)

---
### Asynchronous Services
Async services are a collection of modules responsible for different types of communication. Their responsibility is to prepare the data in corresponding format, establish the communication with associated communication protocol and translate the response to application friendly format. Let’s explain how to implement an http service since it’s the most common one.

The goal of the http layer, besides the ones mentioned above,  is to add headers, manage the request methods, intercept requests, receive the responses, parse them and handle the various types of errors without writing it all over again through the application.

There’s one more requirement. When using the http layer it would be very nice to have rest-like interface. This is very useful because we usually got used to rest api services on the servers and another reason is that they are very self descriptive. The goal is to have a very tiny http client with rest-like methods which we can call from the sandbox.  

Example Code snippet : 
```
@Injectable()
export class ProductsApiClient extends HttpService {
  @PUT<Product>("/products/{id}")
  public updateProductById(@Path("id") id: number, @Body product: Product): Observable<any> { return null; };
}
```
Let’s recap what we did here.  
**@PUT** decorator sets the request method to PUT type with targeted api endpoint “/products/{id}”.generic parameter with @Put decorator is a return type. It indicates type of response from the corresponding HTTP request.  **@Path** decorator sets the given id parameter in the url. @Body decorator specifies the data, of type Product,to be sent to the server.

All possible Decorators are as follow:

* Method decorators :  
  * @GET() - HTTP get request. used for fetching data from the server via API. 
  * @PUT() - HTTP put request. used for updating existing data into particular entity. 
  * @POST() -  HTTP Post request. used for creating/adding new data into particular entity. 
  * @DELETE() - HTTP Delete request. used for deleting existing data from particular entity. 
  * @HEAD() - HTTP head request. used for fetching MetaInformation about particular entity without it's body.
* parameter decorators :
  * @Path() - used for Path variable of a method's url.
  * @Body() -  used for Body of a REST method.
  * @Query() - used for Query value of a method's url
  * @Header() - used for Custom header of a REST method.

---
### Foundation Core Module

We can call it the root module as well and it’s located in **foundation/foundation.module.ts** file. It describes how the application parts fit together and it’s also the entry point used for launching the application. The main tasks for the root module are:
* **Imports** all other modules we want to plug in the application.
* **Provides** services we want to expose globally inside the application and instantiate only once.
* **Declares** the application’s root component.
* **Bootstraps** the root component that Angular creates and inserts it into the index.html host web page.

```
@NgModule({
  imports: [
    SimpleNotificationsModule.forRoot(),
    UtilityModule,
    HttpClientModule,
    HttpServiceModule,
    BrowserAnimationsModule,
    NoopAnimationsModule,
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useFactory: HttpLoaderFactory,
        deps: [HttpClient]
      },
      isolate: true
    }),
    FoundationRoutingModule,
    AngularFireModule.initializeApp(firestoreConfig),
    AngularFirestoreModule,
    EventsModule,
    UIModule,
    FileModule
  ],
  declarations: [FoundationComponent],
  providers: [
     ConfigService,
    {
      provide: APP_INITIALIZER,
      useFactory: configServiceFactory,
      deps: [ConfigService],
      multi: true
    },
    HttpResponseHandler,
    FirestoreService
  ],
  exports: [FoundationComponent,UIModule,FileModule]
})

export class FoundationModule {
  constructor(injector: Injector) {
    setAppInjector(injector);
  }
}
```

---
### Foundation Core Facade
Application core facade is represented as a sandbox. It is an abstract class which holds common logic of the application core API. We placed it in **foundation/src/lib/shared/sandbox/base.sandbox.ts** file.

---
### Sandbox
Sandbox is a service which extends application core facade and exposes streams of state  and connections to the async services. It acts as a mediator and a facade for each presentational module with some extra logic, like serving needed piece of state from the store, providing necessary async services to the UI components, dispatching events.

> _**Note:** Be careful with subscribing to many events because we need to unsubscribe from them as well to avoid multiple subscriptions when user comes to the same page over again. This can lead to memory leaks._  

---
### Utility Services
#### Translation Service (Internationalization)
We have clients from multiple countries and multiple domains. so our app must also support multiple languages for visibility purpose. Users can also change the language at any time. they have to select the particular language from the header's language selector drop-down.
mapping of particular language with the keywords used in the application are stored as a key-value pair in a json file named by that language's code. which is store in **i18n** folder in a project.


following is an example code snippet of file en.json (english).

```
{
  "student": {
    "StudentKey": "Student Key",
    "FirstName": "First Name",
    "LastName": "Last Name",
    "DateOfBirth": "Date of Birth",
    "Actions": "Actions",
    "StudentId": "Student Id",
    "TenantId": "Tenant Id",
    "AddNew": "Add New",

    "StudentNotifications":{
      "SuccessfullyAdded":"Student is added successfully",
      "SuccessfullyUpdated":"Student is updated successfully"
      }
  }
}

```
These files contains mapping based on it's category. as shown in above snippet  if you are trying to map for Confirm box then you define key as **ConfirmDialog** and Object of Key-value pair as value.

#### Configuration Services
All application level configurations are store in the Configuration file. Base level configurations like base Api Url, possible localization's code, it's name and it's culture, Notifications configurations etc. are store in the form of json format inside this Configuration file.  
In addition to that Environment level configurations are also store in configuration file.
Some Configurations are differ at different environments. so to provide this level of abstraction we are creating different configurations file for each environment. those files you can create/found at **foundation/config/{environment name}.json**  location. and for selecting perticular environment as well as the Envirnment level configurations are store at **foundation/config/env.json**.
 
#### Notification Services
This service is used for getting different kinds of notifications to the users. Notification is very important part in any applications. Notifications are of different types. 
generally we have implemented toast message with a specified amount of time for showing Notifications.  

**How to call Notification service in your code ?**  
_**Step 1 :**_ First you have Inherit your Component form the **BaseComponent**.  
code snippets: 
```
export class StudentFormComponent extends BaseComponent implements OnInit {
    ...
      // code logic 
    ...
}
```
This BaseComponent is an abstract class. and it already has created an object of **NotificationService**.

_**Step 2 :**_ After inheriting BaseComponent you can use the **displayNotification()** method of notification service.
  **displayNotification()** is declare as below.
```
  displayNotification(messageTranslationCode: string, type?: string, titleTranslationCode?: string)

```
Here,  
* messageTranslationCode - this is the code that you have specify in the language's json file under i18n folder. 
* type - this is an optional parameter. this is indicating the type of notification. based on it's type the notification will be shown. if you not specify anything here the default style of the notification will be display. all possible types are shown below :
  * error
  * success
  * alert
  * default 
* titleTranslationCode - this is also the code that you have specify in the language's json file under i18n folder. for title.  
example code snippets:

```
 this.notificationService.displayNotification("student.StudentNotifications.SuccessfullyAdded","success");
```

detailed explanation of Notification types are :  
* **Success Notifications** - generally these notifications are used for showing the messages that are return with success messages. If Data are added/updated/deleted successfully. If Documents are uploaded/Updated/Deleted successfully. If mails are sent successfully then we used this success Notifications.
* **Error Notifications** - generally these notifications are used for showing the messages that are return with error messages. If Data are not added/updated/deleted successfully. If Documents are not uploaded/Updated/Deleted successfully. If mails are not sent successfully then we used this Error Notifications.
* **Warning Notifications** - generally these notifications are used for showing the messages that are return with warning messages. It is also used for alert messages.
* **Other Notifications** - IF you want to show normal notification message then you can use default Notification.


* configurations of Notifications' settings are store in {Environment_name}.json file. settings like Timeout period of notification,weather you want to show progress-bar or not, pause Notification time on Hover, position of toast message etc.

 
#### Logging Services
This service is mainly used for maintaining the logs of the systems.   
Logs are generally used for keeping track of the systems.  
Logs are very important for the developers who are developing particular functionality. logs helps them to find out particular errors.
  
---
### Shared Components
Shared components are presentational elements stored in application core layer. Does that sound weird? Well, the application core can hold the presentational elements repeatedly used in the project and provide them to presentational modules to be included as snippets in the template. Each of these components belongs to a module that declares and exports it. This module will include other dependency modules required by the components to work properly. Let’s say we need to translate some text with TranslatePipe, we have to include the translate module.

We can also have smart components here. Now, it’s the architectural decision where to put them. We can create a separate module for containers so we can easily distinguish the difference between them. It would also be easier to find them in the code. Another approach is to place them in presentation layer but in that case we’ll break the rule of organizing presentational modules by feature.If you remember the folder structure, the available options are **foundation/src/lib/shared/components** folder.

Let’s say we are building a business app which contains of a dashboard and login page. Login page will have a login form with some simple background. Once the user gets signed in he/she will be redirected to dashboard. Dashboard page will contain a sidebar navigation menu, header with user’s image, link to sign out and a list with the most recent added products. The requirement is that every other page contains the same layout, with the sidebar and the header.

We can already see that we’ll need a place to handle all this commonly used logic. We can handle this in several ways and one of them is to create a smart layout component which will delegate it’s child events (from navbar and header) to it’s own sandbox. Another way is to let the root app component do this. We’ll go with the approach of creating a separate container module which will hold all reusable containers in the app. This way every presentational root component (except login) will be wrapped inside the layout container and automatically will inherit sidebar and header. This way the login component can be styled separately without the common layout.

---
### State Management
We are not going to go too deep into each piece of the state management layer.  
There are two things we’ll concentrate on:
* How to organize the store
* Handling async actions with Events 

#### Store
Like a traditional database represents the point of record for an application, your store can be thought of as a client side ‘single source of truth’, or database. By adhering to the 1 store contract when designing your application, a snapshot of store at any point will supply a complete representation of relevant application state. This becomes extremely powerful when it comes to reasoning about user interaction, debugging, and in the context of Angular 2, performance.

![Store](https://demographicstorage.blob.core.windows.net/public/store.png)

For managing client's state at client side we are using **fire store service.** 
Information like user's preferred language for the application view and other temporary storage.
 
we have also created Events for dynamically loading component into reference area. that will be explained shortly in Publisher-subscriber methodologies.

---

### Publisher Subscriber Methodologies

This technique is used in inter-Component Communication. When we want to load any component dynamically at any particular reference area then we can use this technique. There are lots of other methods available for inter-component communications like using Directives, using Services or using Properties. but in each case the Component between which the communication is happen is static. so we tried to build a service at framework level via which anyone can dynamically load a component and pass any kind of data between two component in a very easy way. For that we have used Event services. 

Before explaining Event Service first you need to understand the following terminologies: 
* Workarea : this is the actual working area. the main Component (i.e. Student List Component, Product List Component, etc) are loaded first into the workarea. 
* reference area : this is the area over which dependent/Child Components are loaded. This component are loaded based on particular event called from the main Component. drawer is an example of reference area.
the dependent/Child components (i.e. student Form Component, Product Form Component, etc) are loaded into the drawer.

for example, 
When you load the page for showing the list of all students(i.e StudentList Component) then it is loaded into Workarea. Now you want to edit any particular student's details (StudentForm Component) then you click on edit button besides that student. that will open a drawer i.e reference area. and inside that the Student Form Component is loaded.
same way, if you want to view the Details of any particular student then you click on that student from the student list and the drawer (i.e. reference area) will be opened and that student's details (StudentDetail Component) will be loaded into that drawer. So here in reference area we are deciding dynamically what to load.

#### Usage : 

_**Step 1 :**_ First you have Inherit your Component form the **BaseComponent**.  
code snippets: 
```
export class StudentFormComponent extends BaseComponent implements OnInit {
    ...
      // code logic 
    ...
}
```
This BaseComponent is an abstract class. it has definitions for publishEvent(), subscribeEvent(), updateRefArea(), updateWorkArea(), workAreaUpdateHandler(). 


#### publishEvent() :
If you want to Publish any event for passing data between two components then you can publish that event via this method.
 this is used for publishing any event that you want to use for passing data between multiple components.

**_Declaration :_** 
```
publishEvent(eventName: string, data: EventItem): void;
```
Here, 
* eventName - Name of your Event to which you want to publish.
* data - this is of type **EventItem**. and **EventItem** contains payload data.


#### subscribeEvent() : 
This is used for subscribing any event.
 
> **Notes :** remember that subscribe Event Name must be similar to Published Event Name for which you want to subscribe. 

**_Declaration :_** 
```
subscribeEvent(eventName: string, callback: (value: any) => void, error?: (error: any) => void, complete?: () => void): void;
```
Here, 
* eventName - Name of your Event to which you want to subscribe. 
* callback - here you have to pass an callback function which you want to call after subscription.
* error - this is an optional parameter. here you can pass a error handling function.

#### updateRefArea() : 
This method is called when you want to update the content of the drawer. here you have to pass a payload data of type **EventItem**.

**_Declaration :_** 
```
updateRefArea(data: EventItem): void;
```
> **Remember :** wherever you are using publisher, subscriber methods, the type of data will always be of **EventItem**. 

#### updateWorkArea() : 
this method is called when you want to update the working area. when any changes done in reference area and you want to update work area base on that then you can you this method updateWorkArea().
 
**_Declaration :_** 
```
 updateWorkArea(data: EventItem): void;
```
Here, 
* data - it is of type EventItem. and contains information regarding the contents which you want to update.

#### workAreaUpdateHandler()
generally this method is initialized/Subscribed into ngoninit() method.
Whenever any Event is published after making changes in the reference area then this subscription is used. it is used for refreshing the work area without refreshing the browser page.

**_Declaration :_** 
```
workAreaUpdateHandler(callback: (value: any) => void): void;
```
Here, 
* callback - here you have to pass an callback function which you want to call after subscription.

#### Usage Scenarios :

_**Scenarios 1 :**_ Usage of updateRefArea() method.

After inheriting from the BaseComponent all above methods are available to you.  
Remember the Student Scenario I have mentioned earlier. there area 3 Components 1) StudentList Component 2) StudentDetail Component and 3) studentForm Component
after clicking on any row of studentList component if you want to load studentDetail component then onClick() method, you have to call the updateRefArea() with payload as studentDetailCompoenent's information.    
Code snippet : 
```
onRowClick(student: Student) {
    let obj: UIState = { componentType: StudentDetailsComponent, data: student.id };
    this.updateRefArea({ payload: obj });
  }
```
here,
* obj is of type **UIState**.  
UIState contains information regarding,  
 **componentType** (i.e component that you want to dynamically load) and **data** (i.e data for which you want to load particular component).  
and lastly, pass this obj as payload in updateRefArea().

**_Scenarios 2,3 :_** Usage of workAreaUpdate() and workAreaUpdateHandler().

In the student scenario if you want to edit firstname of any student then you will click on edit button against that student. it will load the student form component in reference area. now you will do whatever changes you want to do and then you will click on save which will save your changes into actual entity but at the same time you want to update the studentList Component which is in workarea.   
this scenario is taken care by workareaupdate() and workAreaUpdateHandler() methods. you will first publish an event via updateWorkArea() method and the subscription is already registered as workAreaUpdateHandler().    
in workAreaUpdateHandler() method's callback function you have to pass the function that is loading the studentList.

Code snippet for workAreaUpdate() : 
```
Update(id: number) {
   
   this.observer = observe(this.student);
   var patch = generate(this.observer);
   this.studentSandbox.updateStudent(id, patch).subscribe((resp: any) => {
       ...
        this.updateWorkArea(null);
       ...
      });
    }
```

Code snippet for workAreaUpdateHandler() : 
```
ngOnInit() {
    ... // your other logic
    this.workAreaUpdateHandler((event) => this.getAllStudents())
    ... // your other logic
  }
```

---