# Lab 8 - Detect Language

In this lab we are going to integrate language detection ability of cognitive services into our bot.

## Lab 8.1: Retrieve your Cognitive Services url and keys

1. Open the [Azure Portal](https://portal.azure.com)

1. Search for **Cognitive Services** using the portal search, select the cognitive services resource that is generic (aka, it contains all end points with the **kind** **CognitiveServices**).

1. Under **RESOURCE MANAGEMENT**, select the **Keys and Endpoint** tab and record the url and the key for the cognitive services resource

## Lab 8.2: Add language support to your bot

1. If not already open, open your **PictureBot** solution

1. Right-click the project and select **Manage Nuget Packages**

1. Select **Browse**

1. Search for **Azure.AI.TextAnalytics**, select it then select **Install**, then select **I Accept**

1. Open the **Startup.cs** file, add the following using statements:

    ```csharp
    using Azure.AI.TextAnalytics;
    using Azure;
    ```

1. Add the following code to the **ConfigureServices** method:

    ```csharp
    services.AddSingleton<TextAnalyticsClient>(sp =>
    {
        Uri cogsBaseUrl = new Uri(Configuration.GetSection("cogsBaseUrl")?.Value);
        string cogsKey = Configuration.GetSection("cogsKey")?.Value;

        var credentials = new AzureKeyCredential(cogsKey);
        return new TextAnalyticsClient(cogsBaseUrl, credentials);
    });
    ```

1. Open the **PictureBot.cs** file, add the following using statements:

    ```csharp
    using Azure.AI.TextAnalytics;
    ```

1. Add the following class variable:

    ```csharp
    private TextAnalyticsClient _textAnalyticsClient;
    ```

1. Modify the constructor to include the new `TextAnalyticsClient`:

    ```csharp
    public PictureBot(PictureBotAccessors accessors, LuisRecognizer recognizer, TextAnalyticsClient analyticsClient)
    ```

1. Inside the constructor, initialize the class variable:

    ```csharp
    _textAnalyticsClient = analyticsClient;
    ```

1. Navigate to the **OnTurnAsync** method and find the following line of code:

    ```csharp
    var utterance = turnContext.Activity.Text;
    var state = await _accessors.PictureState.GetAsync(turnContext, () => new PictureState());
    state.UtteranceList.Add(utterance);
    await _accessors.ConversationState.SaveChangesAsync(turnContext);
    ```

1. Add the following line of code after it

    ```csharp
    //Check the language
        DetectedLanguage detectedLanguage = _textAnalyticsClient.DetectLanguage(turnContext.Activity.Text);
        switch (detectedLanguage.Name)
        {
            case "English":
                break;
            default:
                //throw error
                await turnContext.SendActivityAsync($"I'm sorry, I can only understand English. [{detectedLanguage.Name}]");
                break;
        }
    ```

1. Everyting you have in the method after `switch` ends move to the `case "English"`. Finally your method should looks like following:

    ```csharp
    public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type is "message")
        {
            var utterance = turnContext.Activity.Text;
            var state = await _accessors.PictureState.GetAsync(turnContext,() => new PictureState());
            state.UtteranceList.Add(utterance);
            await _accessors.ConversationState.SaveChangesAsync(turnContext);

            //Check the language
            DetectedLanguage detectedLanguage = _textAnalyticsClient.DetectLanguage(turnContext.Activity.Text);
            switch (detectedLanguage.Name)
            {
                    case "English":
                        // Establish dialog context from the conversation state.
                        var dc = await _dialogs.CreateContextAsync(turnContext);
                        // Continue any current dialog.
                        var results = await dc.ContinueDialogAsync(cancellationToken);

                        // Every turn sends a response, so if no response was sent,
                        // then there no dialog is currently active.
                        if (!turnContext.Responded)
                        {
                            // Start the main dialog
                            await dc.BeginDialogAsync("mainDialog", null, cancellationToken);
                        }
                        break;
                    default:
                        //throw error
                        await turnContext.SendActivityAsync($"I'm sorry, I can only understand English. [{detectedLanguage.Name}]");
                        break;
            }
        }
    }
    ```


1. Open the **appsettings.json** file and ensure that your cognitive services settings are entered:

    ```csharp
    "cogsBaseUrl": "",
    "cogsKey" :  ""
    ```

1. Press **F5** to start your bot

1. Using the Bot Emulator, send in a few phrases and see what happens:

- Como Estes?
- Bon Jour!
- Привет
- Hello

## Going further

Since we have already introduced you to LUIS in previous labs, think about what changes you may need to make to support multiple languages using LUIS.  Some helpful articles:

- [Language and region support for LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-language-support)

## Resources

- [Example: Detect language with Text Analytics](https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-language-detection)
- [Quickstart: Text analytics client library for .NET](https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/quickstarts/csharp)

## Next Steps

- [Lab 09-01: Test Bot DirectLine](../Lab9-Test_Bots_DirectLine/01-Introduction.md)
