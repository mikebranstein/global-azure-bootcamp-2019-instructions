## Deploying Custom Speech Service Endpoints

In this chapter, you'll learn how to:
- deploy your customized acoustic and language models and create a secured endpoint
- test the customized CSS endpoint using the web app you deployed earlier  

### Overview

Previously, you learned about the various data sets you need to train, test, and operationalize machine learning systems. Over the past 2 chapters, you created training and testing data sets, built customized acoustic and language models, then tested the customization accuracy.

The next step is to deploy your customization and test them with real-world data.

Let's get started!

### Deploying Custom Models

You've already done the hard work of building the customized models, so let's use them to create a deployment.

> **PREREQUISITES**
>
> Before you proceed, you'll need a customized acoustic and language model that have a *Succeeded* status. If your models are still training, wait a few more minutes, then check back when the models are ready.

<h4 class="exercise-start">
    <b>Exercise</b>: Deploying customized acoustic and language models
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, and navigate to *Deployments*. 

Click the *Create New* button to create a new deployment.

Complete the following fields:
- Locale: en-US
- Description: *blank*
- Name: Pokemon
- Subscription: *the one you created earlier*
- Base Model: Microsoft Search and Diction Model, then select the Pokemon models below

<img src="images/chapter5/deploy1.png" class="img-override" />

Click *Create* to deploy the models to production.

When the test run is saved, you'll navigate back to the *Deployments* page:

<img src="images/chapter5/deploy2.png" class="img-override" />

Note the *Status* of the test run is *NotStarted*. In a few moments, it will change to *Running*, then *Succeeded*.

The deployment will not take long (up to 1 minute). That's it! You've deployed your models.

This concludes the exercise. 

<div class="exercise-end"></div> 

### Exploring the Custom Speech Service Endpoint

Now that you've deployed a customized CSS endpoint, you can consume it in an application. But how?

Let's take a closer look at your deployment.

<h4 class="exercise-start">
    <b>Exercise</b>: Exploring a CSS endpoint deployment
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, and navigate to *Deployments*. 

Click the *Details* link next to your *Pokemon* deployment:

<img src="images/chapter5/deploy3.png" class="img-override" />

The deployment details page shows you a variety of details about your deployment. Scroll down to the *Endpoints* area:

<img src="images/chapter5/deploy4.png" class="img-override" />

The endpoints area shows a variety of URIs that you can use to access your customized deployment. You can interact with the CSS via a:
- HTTP REST API
- WebSockets, using a .NET/Android/iOS client library
- WebSockets, using a .NET service library
- WebSockets with support for the Speech Protocol/JavaScript WebSocket API

You'll notice that for each option, you have endpoints for short and long-form audio. In some cases, endpoints support punctuation detection.

Depending on the needs of your application, you may choose a different endpoint. I've used each of these previously and think it's good to walk through each at a high-level.

#### HTTP REST API

Use this option when you have a .wav file that you want to upload and get a single response back. You won't get real-time speech results back, but it works well when you want to do quick, bulk processing of a large collection of audio files. 

#### WebSockets, using a .NET/Android/iOS client library

If you need real-time processing of audio, when using a microphone that's embedded in software with a .NET app, Android app, or iOS app, this is the right choice for you. An important designation here is that you need to use this in conjunction with the SDKs/libraries provided by Microsoft. It's also important to note that these are intended for client-side applications, not a server-side process that will have a long life span. 

#### WebSockets, using a .NET service library

When you need a long-running server-side process to interact in real-time with the CSS in a .NET app, use these endpoints. You'll also need to use the .NET SDK built for this purpose.

#### WebSockets with support for the Speech Protocol/JavaScript WebSocket API

The last option is to interact with the CSS using a specific protocol called the Speech Protocol. This also has it's own SDK and API you need to adhere to when using these endpoints. 

> **The Speech Protocol**
>
> I consider the first three options a legacy way of interacting with the CSS. The 4th option (Speech Protocol) is the new and recommended way of interfacing with the CSS. Eventually the first 3 options will be deprecated and the Speech Protocol will be *the* way to interact. 
>
> Right now, support for the new Speech Protocol is limited to a JavaScript SDK, but if you need a C# version, you can roll your own. To learn more about the protocol, check out the official [protocol documentation](https://docs.microsoft.com/en-us/azure/cognitive-services/speech/api-reference-rest/websocketprotocol).

> **Rolling your Own Speech Protocol Client**
>
> Don't do this, and I speak from experience. I've done it. It's not easy, the documentation isn't great, and unless you have a lot of experience writing web socket protocol code in C#, this can be really time consuming and difficult. I lost a month of my life to this. The end result was pretty cool. I didn't have an option of waiting for Microsoft's team to implement the C# SDK, but you probably will. 

#### Subscription Key and Endpoint URL

Ok, sorry for the tangent. Let's get back to the deployment. You'll need to keep track of a few pieces of data:

<img src="images/chapter5/deploy4.png" class="img-override" />

First, take note of the *Subscription Key* at the top. Second, you'll need the web socket protocol base URL from the *WebSocket with the Speech Protocol/JavaScript WebSocket API* (wss://610e08d3ae4b4d4eb7d45dcf2e877698.api.cris.ai) for my deployment. 

> **Don't Use MY Endpoint Base URL**
>
> Please don't copy my endpoint base URL. If you do, you'll get errors later on. Please copy your own.

With these two values copied/saved, you're ready to move on to testing the endpoint.

This concludes the exercise. 

<div class="exercise-end"></div>


### Testing the CSS Endpoint

Now, let's return to the web app you deployed to Azure earlier and test your Custom Speech Service deployment. 

<h4 class="exercise-start">
    <b>Exercise</b>: Testing a CSS endpoint deployment
</h4>

> **Don't use your VM for this exercise**
>
> It's important that you don't use your VM for this exercise, because you'll be using your computer's microphone. This just doesn't work well through a remote desktop connection.

Start by navigating to your deployed Azure web site. My URL was [http://workshopwebapp.azurewebsites.net/](http://workshopwebapp.azurewebsites.net/). 

After the page loads, paste your deployed CSS endpoint base URL into the *Endpoint* text box, and the subscription key into the *Subscription Key* text box:

<img src="images/chapter5/deploy5.png" class="img-override" />

Ignore the LUIS-related fields, change the *Recognition Mode* drop down to *Dictation*, and *Format* to *Detailed Result*:

<img src="images/chapter5/deploy6.png" class="img-override" />

Press the *Start* button and start speaking. The page may ask to access your microphone, and as you speak, the site will submit your speech to the CSS endpoint you created and return incremental speech results in real-time.

Try speaking the phrase, "Pikachu is a cool pokemon.":

<img src="images/chapter5/speech.gif" class="img-override" />

Now, that's cool! As you speak, you'll see incremental results returned to you browser and displayed in the *Current hypothesis* area. Then, when the CSS recognizes the end of your utterance, it returns JSON-formatted result:

```json
{
   "RecognitionStatus": "Success",
   "Offset": 0,
   "Duration": 26300000,
   "NBest": [
      {
         "Confidence": 0.923154,
         "Lexical": "pikachu is a cool pokemon",
         "ITN": "pikachu is a cool Pokémon",
         "MaskedITN": "pikachu is a cool Pokémon",
         "Display": "Pikachu is a cool Pokémon."
      }
   ]
}
{
   "RecognitionStatus": "EndOfDictation",
   "Offset": 54210000,
   "Duration": 0
}
```

The way this CSS endpoint works is that each time an utterance is detected, a JSON object is returned with `"RecognitionStatus": "Success"`. Inside, it tracks the audio millisecond count that was sent, based on the *Offset* and *Duration*, meaning that at audio millisecond 0, the system detected an utterance beginning, and an utterance ending after 26300000 milliseconds.

The CSS also returns the speech hypothesis in a variety of formats. The most meaningful is the `"Display": "Pikachu is a cool Pokémon."` result, which is the official transcription with a confidence % of 92.3154%.

Pretty cool.

Go ahead an try a few more phrases.

#### Diving into the web page code

We're not going to dive into the JavaScript code that manages interacting with the Speech Protocol WebSocket endpoint. It's *really* complicated. You're welcome to dive into the details on your own, but it's out of scope for us today.

> **Challenge #6**
>
> Now that you have a full training to testing to real-world testing methodology for the CSS, try to stump your trained model. Then, return back to your data sets, models, and deployments. Update all of them and attempt to retrain the system to address the shortcomings you identified. 

This concludes the exercise. 

<div class="exercise-end"></div>

In this chapter, you learned:
- how to test your trained models with real-world data 
- that a CSS deployment deploy various endpoints that are used in different scenarios


