# File Management

    
File Management helps you to manage your Files. It helps you to Store, Manage, Retrieve/fetch and Delete Files in very easy and efficient way. 
*  It will help you to Upload your file either Securely or normally.
*  It will do a virus/malware check before uploading it onto actual storage. 
* It will also reduce your effort of writing a repetitive code everywhere.
* This system helps you to upload different format of files like document (docx, doc, Ppt, Pdf, Text), Images (jpg, png) and Package (.zip).
* files are internally converted into different categories based on it’s FileType  
There are 3 different kind of FileTypes:
   1) Document
   2) Image
   3) Package.  
* We restricted the different formats of documents that we can upload based on it’s type.
  1)  If your file is of type **Document** then you can upload only those files that belongs to **Document** category like docx, doc, Ppt, Pdf, Text etc.
  1) If your file is of type **Image** then you can upload only those files that belongs to **Image** category like jpg, jpeg, png etc.

## Features/Functionalities

* File Upload 
*   Encyrpt files (Secured)
*	File Scan for Virus/Malware Detection 
*	File Download / View
*	File Delete
*	Converting Document in Different Formats 
*	File Upload Restriction base on it's File Types
*	Upload Multiple files at a time 

---


## Azure Storage

#### Storage Structure 

Below, We have mentioned storage structure in Azure Blob storage.

![storage structure](https://demographicstorage.blob.core.windows.net/public/folderStructureAzure.png)

---

##  Detailed Description 

### File Scan for Virus/Malware Detection

* A **file-infecting virus** is a type of malware that infects executable files with the intent to cause permanent damage or make them unusable. A file-infecting virus overwrites code or inserts infected code into a executable file.  
* If a user uploads an infected file to the website, you will be essentially distributing that infected file to any of your other potential users.
* To protect our App from the infected files we have created a feature called **File Scan/Malware Detection**, which will scan each file before uploading it on actual azure storage.  

If the uploaded file contains any virus or theft then it will not allow you to upload that malicious file. instead, it will give an error while you upload a file.
* you can also enable or disable this feature via appsetting.json file.
* To add or Enable this feature first you need to run the **clamav Image** locally. 
   * you will able to find this Image on server `10.10.1.16:4000`. 
* To add this feature First you need to add configuration setting for virus scanner services into your appsetting.json file. it is shown as below :

if you are running it **locally** then serverAddress is localhost. but if you are running it on any server then serverAddress will be the Address of that server.  

```
"VirusScanner": {
    "Enabled": true,
    "ServerAddress": "localhost",
    "Port": 3310
  } 

```
here,  
properties :   
* Enabled -  you can enable or disble virus scanner by setting it as true or false.
* ServerAddress and port are set accordingto the server on which your virus scanner (clamav Image) is running.

---

### File upload  

You can upload your file in 2 different ways.
1) Upload your file at temporary location first and then when it's final, you can save it at actual location.  
2) Directly upload your file at actual location.

you can also upload your files either securely or normally.

----
If you are uploading your file at temporary location then follow below steps:  
**_Step 1 :_** create a FileController which is inherited from FileController of FileSerivce. and also create a constuctor for the same as shown here.  
``` 
[Route("[Area]/[controller]")]
public class FileController : Exxat.Services.File.Controller.FileController
{  
      public FileController(IRootConfiguration configuration, ILoggerFactory loggerFactory, 
                            IInternalFileService internalfileService, IFileService fileService) 
        : base(configuration, loggerFactory, internalfileService, fileService)
        {

        }
}
```    
base  FileController's File upload method will internally handle the temporary fileupload. 

**_Step 2 :_**  If you are using **Postman** then follow below steps:
1. Specify the Url as       
`Uri/area/controller? `   
for ex. `localhost:5900/student/File`.  
2. Specify the parameters in the params.
    1) description : here you can specify information related to your file it is kind of notes for the particular file.   
    2) fileType : here you can specify either Image, Document or Package based on it's uploading type.  
below I have mentions all types with it's supported values.    
        1)  **Document :**  docx, doc, Ppt, Pdf, Text etc. 
        2)  **Image :** jpg, jpeg, png etc.
        3)  **Package :** zip.  

    > _Note : the first letter in the value of filetype is capital._  

    3) filepath : here you can specify the path/folder under which you want to store you document in blob storage.
    4) isSecured : here it is a boolean value you can specify either true or false.
        -  true : file will be encrypted first and then get stored into blob.
        -  false :file will be stored normally. 
3. Specify the TenantId in the header.

**_Step 3 :_** Now in the code you change the following things: 
* Generally we are using this process in Forms where we have to attach an image or document with the other formfields. and we save it in our controller by calling `SaveFilesAsync()` method as below.  
    1) Inject the file service into your serives.\
     ex. 
    ```
         private readonly IFileService _fileService;
         private readonly IDemographicRepository _demographicRepository;

         public DemographicService(IRootConfiguration rootConfiguration,ILoggerFactory loggerFactory, 
                IBusinessScopeFactory businessScopeFactory, IFileService fileService, 
                IDemographicRepository demographicRepository)  
                : base(rootConfiguration,loggerFactory, businessScopeFactory, demographicRepository)
                {
                    _demographicRepository = demographicRepository;
                    _fileService = fileService;
                }
   ```
    2) Call following method in the Update/Add method of your service from where you are saving your data.
    
    ``` 
         _fileService.SaveFilesAsync(FileCollectionKey,student.FileDescriptions);

    ```   
_terms :_  
* **FileCollectionKey :** When there are multiple Document pages of the same Document then that group of Documents can be identify by the same FileCollectionKey.  
  
---

If you are uploading your file **directly to the actual storage** than you have to follow below steps:

**_Step 1 :_** IF your FileController is not exists as mention in above method's Step 1 process then you need to create a new Controller for File Operations as we mentioned in step 1 of above method.  


> _**Note :** fileService and logger are created and initalized in base FileController itself so you can directly use that you don't nead to declare and initialize it again._




**_Step 2 :_** create a method which will call the UploadAsync() method of fileService. which is shown in following example code snippet by the following line : ` List<Guid> fileKeys = await _fileService.UploadAsync(blobFiles);` it will return a filekeys that you have uploaded.

``` 
 [HttpPost("[action]")]
 public async Task<ActionResult> Upload([FromForm] List<IFormFile> files)
 {
      var blobFiles = new BlobFiles()
      {
           Description = "Creating sample upload",
           FileCollectionKey = Guid.NewGuid(),
           FilePath = "sampleblobs",
           FileType = Services.File.Configuration.Settings.FileType.Document,
           Files = files
       };
       List<Guid> fileKeys = await _fileService.UploadAsync(blobFiles);
       return Ok(fileKeys);
  }
```

---
### File Download/View

you can download your file based on FileKey as shown in below steps.

**_Step 1 :_** IF your FileController is not exists as mention in above method's Step 1 process then you need to create a new Controller for File Operations as we mentioned in step 1 of above method.  

>_**Note :** fileService and logger are created and initalized in base FileController itself so you can directly use that you don't nead to declare and initialize it again._



**_Step 2 :_** create a method which will call the DownloadAsync() method of fileService. which is shown in following example code snippet by the following line : `var file = await _fileService.DownloadAsync(fileKey, Services.File.Configuration.Settings.FileOutputFormat.Original);` it will return a file that you have uploaded.
here second argument accepts enum value of FileOutputFormat. means which type of outputformat do you want.  
**Types of OutputFormats :**
* FileOutputFormat.Original - it will return original file.
* FileOutputFormat.PdfFormat - it will return a file that is viewer compatible. generally PDF files are browser compatible and you cansee it in browser.
* FileOutputFormat.ThumbnailSmall - it will return a small thumbnail Image.
* FileOutputFormat.ThumbnailMedium - it will return a medium thumbnail Image.
* FileOutputFormat.ThumbnailLarge - it will return a large thumbnail Image.

IF you want to Download/View a Document category file then `FileOutputFormat.Original and PdfFormat` can be used as FileOutPutFormat based on you requirement.  
IF you want to Download/View a Image category file then `FileOutputFormat.Original, ThumbnailSmall, ThumbnailMedium, ThumbnailLarge` can be used as FileOutPutFormat based on you requirement.
 
``` 
 [HttpGet("[action]")]
 public async Task<ActionResult> Download()
 {
    Guid fileKey;
    Guid.TryParse("c2f75836-47db-4fd7-ac06-d4c14f89e6d6", out fileKey);
    var file = 
    await _fileService.DownloadAsync(fileKey,Services.File.Configuration.Settings.FileOutputFormat.Original);
    return new FileContentResult(file.Content.ToArray(), file.ContentType);
 }
```

---
### File Delete

you can Delete your file based on **FileKey or FileCollectionKey** as shown in below steps.

**_Step 1 :_** IF your FileController is not exists as mention in above method's Step 1 process then you need to create a new Controller for File Operations as we mentioned in step 1 of above method.  

> _**Note :** fileService and logger are created and initalized in base FileController itself so you can directly use that you don't nead to declare and initialize it again._


**_Step 2 :_** create a method which will call the DeleteAsync() method of fileService. it will Delete a actual file i.e. store in Azure storage as well as it will delete File's matadata i.e store in Cosmos DB.
the code snippet for deleting a file is shown as follow.`var result = _fileService.DeleteAsync(key,isFileKey)`.

In DeleteAsync() function,  
* The first argument specifies the **Key** it can be either **FileKey or FileCollectionKey** based on which File/Files you want to Delete. it can be distinguished by second argument.
* The Second argument specifies the **isFileKey** it is a boolean value.
   * If **isFileKey = true** means Key specify in 1st argument is Key.
   * If **isFileKey = false** means Key specify in 1st argument is FileCollectionKey.  

default value for isFileKey is false.

---

### Find FileDescriptions

* you can find list of FileDescriptions based on FileCollectionKey.
* you can use following line of code for using FindFileDescriptions of FileService's method. v 
```
List<FileDescription> fileDescriptions = _fileService.FindFileDescriptions(fileColletionKey);
```  

---
