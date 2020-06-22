# Lab 2 - Implement Computer Vision

## Introduction

We're going to build an end-to-end application that allows you to pull in your own pictures, use Cognitive Services to obtain a caption and some tags about the images. In later labs, we will build a Bot Framework bot using LUIS to allow easy, targeted querying of such images.

## Lab 2.0: Objectives

In this lab, you will:

- Learn about the various Cognitive Services APIs
- Understand how to configure your apps to call Cognitive Services
- Build an application that calls various Cognitive Services APIs (specifically Computer Vision) in .NET applications

While there is a focus on Cognitive Services, you will also leverage Visual Studio 2019.

> **Note** if you have not already, follow the directions for creating your Azure account, Cognitive Services, and getting your api keys in [Lab1-Technical_Requirements.md](../Lab1-Technical_Requirements/02-Technical_Requirements.md).

## Lab 2.1: Architecture

We will build a simple C# application that allows you to ingest pictures from your local drive, then invoke the [Computer Vision API](https://www.microsoft.com/cognitive-services/en-us/computer-vision-api) to analyze the images and obtain tags and a description.

In the continuation of this lab throughout the course, we'll show you how to query your data, and then build a [Bot Framework](https://dev.botframework.com/) bot to query it. Finally, we'll extend this bot with [LUIS](https://www.microsoft.com/cognitive-services/en-us/language-understanding-intelligent-service-luis) to automatically derive intent from your queries and use those to direct your searches intelligently.

![Architecture Diagram](../images/AI_Immersion_Arch.png)

## Lab 2.2: Resources

There are some directories in the [main](./) github repo folder:

- **sample_images**: Some sample images to use in testing your implementation of Cognitive Services.

- **code**: In here, there are two directories. Each folder contains a solution (.sln) that has several different projects for the lab.

  - **Starter**: A starter project, which you can use if you want to learn about creating the code that is used in the project.

  - **Finished**: A finished project that you will make use of to implement computer vision and work with the images in this lab.

## Lab 2.3: Image Processing

### Cognitive Services

Cognitive Services can be used to infuse your apps, websites and bots with algorithms to see, hear, speak, understand, and interpret your user needs through natural methods of communication.

There are five main categories for the available Cognitive Services:

- **Vision**: Image-processing algorithms to identify, caption and moderate your pictures
- **Knowledge**: Map complex information and data in order to solve tasks such as intelligent recommendations and semantic search
- **Language**: Allow your apps to process natural language with pre-built scripts, evaluate sentiment and learn how to recognize what users want
- **Speech**: Convert spoken audio into text, use voice for verification, or add speaker recognition to your app
- **Search**: Add Bing Search APIs to your apps and harness the ability to comb billions of webpages, images, videos, and news with a single API call

You can browse all of the specific APIs in the [Services Directory](https://azure.microsoft.com/en-us/services/cognitive-services/directory/).

Let's talk about how we're going to call Cognitive Services in our application by reviewing the sample code in the Finished project.

### Image Processing Library

1. Open the **code/Finished/ImageProcessing.sln** solution

Within your `ImageProcessing` solution you'll find the `ProcessingLibrary` project. It serves as a wrapper around several services. This specific PCL contains some helper classes (in the ServiceHelpers folder) for accessing the Computer Vision API and an "ImageInsights" class to encapsulate the results.

![Processing Library PCL](../images/ProcessingLibrary.png)

You should be able to pick up this portable class library and drop it in your other projects that involve Cognitive Services (some modification will be required depending on which Cognitive Services you want to use).

**ProcessingLibrary: Service Helpers**

Service helpers can be used to make your life easier when you're developing your app. One of the key things that service helpers do is provide the ability to detect when the API calls return a call-rate-exceeded error and automatically retry the call (after some delay). They also help with bringing in methods, handling exceptions, and handling the keys.

You can find additional service helpers for some of the other Cognitive Services within the [Intelligent Kiosk sample application](https://github.com/Microsoft/Cognitive-Samples-IntelligentKiosk/tree/master/Kiosk/ServiceHelpers). Utilizing these resources makes it easy to add and remove the service helpers in your future projects as needed.

**ProcessingLibrary: The "ImageInsights" class**

1. In the **ProcessingLibrary** project, navigate to the **ImageInsights.cs** file.

You can see that there are properties for `Caption` and `Tags` from the images, as well as a unique `ImageId`. "ImageInsights" collects  the information from the Computer Vision API.

Now let's take a step back for a minute. It isn't quite as simple as creating the "ImageInsights" class and copying over some methods/error handling from service helpers. We still have to call the API and process the images somewhere. For the purpose of this lab, we are going to walk through `ImageProcessor.cs`to understand how it is being used. In future projects, feel free to add this class to your PCL and start from there (it will need modification depending what Cognitive Services you are calling and what you are processing - images, text, voice, etc.).

## Lab 2.4: Review `ImageProcessor.cs`

1. Navigate to **ImageProcessor.cs** within `ProcessingLibrary`.

1. Note the following [`using` directives](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/using-directive) **to the top** of the class, above the namespace:

```csharp
using System;
using System.IO;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.ProjectOxford.Vision;
using ServiceHelpers;
```

[Project Oxford](https://blogs.technet.microsoft.com/machinelearning/tag/project-oxford/) was the project where many Cognitive Services got their start. As you can see, the NuGet Packages were even labeled under Project Oxford. In this scenario, we'll call `Microsoft.ProjectOxford.Vision` for the Computer Vision API. Additionally, we'll reference our service helpers (remember, these will make our lives easier). You'll have to reference different packages depending on which Cognitive Services you're leveraging in your application.

1. In **ImageProcessor.cs** we start by utliziling a method to process the image, `ProcessImageAsync`. The code will utilize asynchronous processing because it will utilize services to perform the actions.

```csharp
public static async Task<ImageInsights> ProcessImageAsync(Func<Task<Stream>> imageStreamCallback, string imageId)
{
	// Set up an array that we'll fill in over the course of the processor:
  VisualFeature[] DefaultVisualFeaturesList = new VisualFeature[] { VisualFeature.Tags, VisualFeature.Description };

  // Call the Computer Vision service and store the results in imageAnalysisResult:
  var imageAnalysisResult = await VisionServiceHelper.AnalyzeImageAsync(imageStreamCallback, DefaultVisualFeaturesList);

  // Create an entry in ImageInsights:
  ImageInsights result = new ImageInsights
  {
  	ImageId = imageId,
    Caption = imageAnalysisResult.Description.Captions[0].Text,
    Tags = imageAnalysisResult.Tags.Select(t => t.Name).ToArray()
  };

  // Return results:
  return result;
}
```

In the above code, we use `Func<Task<Stream>>` because we want to make sure we can process the image multiple times (once for each service that needs it), so we have a Func that can hand us back a way to get the stream. Since getting a stream is usually an async operation, rather than the Func handing back the stream itself, it hands back a task that allows us to do so in an async fashion.

In `ImageProcessor.cs`, within the `ProcessImageAsync` method, we set up a [static array](https://stackoverflow.com/questions/4594850/definition-of-static-arrays) that we'll fill in throughout the processor. As you can see, these are the main attributes we want to call for `ImageInsights.cs`.

1. Next, we want to call the Cognitive Service (specifically Computer Vision) and put the results in `imageAnalysisResult`.

1. We use the code below to call the Computer Vision API (with the help of `VisionServiceHelper.cs`) and store the results in `imageAnalysisResult`. Near the bottom of `VisionServiceHelper.cs`, you will want to review the available methods for you to call (`RunTaskWithAutoRetryOnQuotaLimitExceededError`, `DescribeAsync`, `AnalyzeImageAsync`, `RecognizeTextAsyncYou`). You will use the AnalyzeImageAsync method in order to return the visual features.

```csharp
var imageAnalysisResult = await VisionServiceHelper.AnalyzeImageAsync(imageStreamCallback, DefaultVisualFeaturesList);
```

Now that we've called the Computer Vision service, we want to create an entry in "ImageInsights" with only the following results: ImageId, Caption, and Tags (you can confirm this by revisiting `ImageInsights.cs`).

1. The following code below accomplishes this.

```csharp
ImageInsights result = new ImageInsights
{
    ImageId = imageId,
    Caption = imageAnalysisResult.Description.Captions[0].Text,
    Tags = imageAnalysisResult.Tags.Select(t => t.Name).ToArray()
};
```

So now we have the caption and tags that we need from the Computer Vision API, and each image's result (with imageId) is stored in "ImageInsights".

1. Lastly, we need to close out the method by using the following line at the end of the method:

```csharp
return result;
```

1. In order to use this application, we need to build the project, press **Ctrl-Shift-B**, of select the **Build** menu and choose **Build Solution**.

1. Work with your instructor to fix any errors.

### Exploring Cosmos DB	

Azure Cosmos DB is Microsoft's resilient NoSQL PaaS solution and is incredibly useful for storing loosely structured data like we have with our image metadata results. There are other possible choices (Azure Table Storage, SQL Server), but Cosmos DB gives us the flexibility to evolve our schema freely (like adding data for new services), query it easily, and can be quickly integrated into Azure Cognitive Search (which we'll do in a later lab).	

## Lab 2.5 (optional): Understanding CosmosDBHelper	

Cosmos DB is not a focus of this lab, but if you're interested in what's going on - here are some highlights from the code we will be using:	
1. Navigate to the `CosmosDBHelper.cs` class in the `ImageStorageLibrary` project. Review the code and the comments. Many of the implementations used can be found in the [Getting Started guide](https://docs.microsoft.com/en-us/azure/cosmos-db/documentdb-get-started).	
1. Go to the `TestCLI` project's `Util.cs` file and review  the `ImageMetadata` class (code and comments). This is where we turn the `ImageInsights` we retrieve from Cognitive Services into appropriate Metadata to be stored into Cosmos DB.	
- Finally, look in `Program.cs` in `TestCLI` and at  `ProcessDirectoryAsync`. First, we check if the image and metadata have already been uploaded - we can use `CosmosDBHelper` to find the document by ID and to return `null` if the document doesn't exist. Next, if we've set `forceUpdate` or the image hasn't been processed before, we'll call the Cognitive Services using `ImageProcessor` from the `ProcessingLibrary` and retrieve the `ImageInsights`, which we add to our current `ImageMetadata`.  	
- Once all of that is complete, we can store our image - first the actual image into Blob Storage using our `BlobStorageHelper` instance, and then the `ImageMetadata` into Cosmos DB using our `CosmosDBHelper` instance. If the document already existed (based on our previous check), we should update the existing document. Otherwise, we should be creating a new one.	

## Lab 2.6: Loading Images using TestCLI	

We will implement the main processing and storage code as a command-line/console application because this allows you to concentrate on the processing code without having to worry about event loops, forms, or any other UX related distractions. Feel free to add your own UX later.	
1. In the **TestCLI** project, open the **settings.json** file	
1. Add your specific environment settings from [Lab1-Technical_Requirements.md](../Lab1-Technical_Requirements/02-Technical_Requirements.md)	

> **Note** the url for cognitive services should end with **/vision/v1.0** for the project oxford apis.  For example `https://westus2.api.cognitive.microsoft.com/vision/v1.0`.	

1. If you have not already done so, compile the project	
1. Open a command prompt and navigate to the build directory for the **TestCLI** project.  It should something like **{GitHubDir}\Lab2-Implement_Computer_Vision\code\Starter\TestCLI**.

> **NOTE** Do not navigate to the debug directory	

1. Run command **dotnet run**	
```cmd	
Usage:  [options]	
Options:	
-force            Use to force update even if file has already been added.	
-settings         The settings file (optional, will use embedded resource settings.json if not set)	
-process          The directory to process	
-query            The query to run	
-? | -h | --help  Show help information	
```	
By default, it will load your settings from `settings.json` (it builds it into the `.exe`), but you can provide your own using the `-settings` flag. To load images (and their metadata from Cognitive Services) into your cloud storage, you can just tell _TestCLI_ to `-process` your image directory as follows:	
```cmd	
dotnet run -- -process <%GitHubDir%>\AI-100-Design-Implement-Azure-AISol\Lab2-Implement_Computer_Vision\sample_images	
```	
> **Note** Replace the <%GitHubDir%> value with the folder where you cloned the repository.	
Once it's done processing, you can query against your Cosmos DB directly using _TestCLI_ as follows:	
```cmd	
dotnet run -- -query "select * from images"	
```	
Take some time to look through the sample images (you can find them in /sample_images) and compare the images to the results in your application.	
> **Note** You can also browse the results in the CosmosDb resource in Azure.  Open the resource, then select **Data Explorer**.  Expand the **metadata** database, then select the **items** node.  You will see several json documents that contains your results.


## Credits

This lab was modified from this [Cognitive Services Tutorial](https://github.com/noodlefrenzy/CognitiveServicesTutorial).

## Resources

- [Computer Vision API](https://www.microsoft.com/cognitive-services/en-us/computer-vision-api)
- [Bot Framework](https://dev.botframework.com/)
- [Services Directory](https://azure.microsoft.com/en-us/services/cognitive-services/directory/)
- [Portable Class Library (PCL)](https://docs.microsoft.com/en-us/dotnet/standard/cross-platform/cross-platform-development-with-the-portable-class-library)
- [Intelligent Kiosk sample application](https://github.com/Microsoft/Cognitive-Samples-IntelligentKiosk/tree/master/Kiosk/ServiceHelpers)

## Next Steps

- [Lab 03-01: Basic Filter Bot](../Lab3-Basic_Filter_Bot/01-Introduction.md)
