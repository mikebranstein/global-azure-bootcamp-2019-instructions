## Integrating LUIS into Your Apps

In the final chapter of the workshop, you'll learn:
- how to integrate LUIS into an app

### Overview

With a published app, you're ready to get LUIS into the Speech Recognition website you deployed to Azure earlier in the workshop.

Before you jump into testing, let's learn how to access LUIS endpoints.

#### LUIS REST API

LUIS is exposed as a REST API endpoint and can be accessed right from your web browser, or through your favorite REST API testing software. I like to use [Postman](https://www.getpostman.com/).

Let's test out your LUIS endpoint.

<h4 class="exercise-start">
    <b>Exercise</b>: Testing out your LUIS endpoint with Postman
</h4>

Start by downloading and installing [Postman](https://www.getpostman.com/) on the Azure VM you've been using today.

After Postman is installed, close out of all the windows/popups it shows on startup, and you should see a screen like this:

<img src="images/chapter9/postman1.png" class="img-override" />

Navigate back to the LUIS portal and copy the URL next to your LUIS subscription on the *Publish* page of your *Pokemon* app:

<img src="images/chapter9/copy-link.gif" class="img-override" />

Paste the link into the HTTP GET textbox in Postman, then click the *Params* button. Next to the *q* query string parameter, type in *pikachu sit down on the ground*, then click the *Send* button:

<img src="images/chapter9/send.gif" class="img-override" />

The HTTP response will be displayed in the *Body* area below. My response is included below. I'll let you examine the HTTP response from LUIS on your own.

```json
{
    "query": "pikachu sit down on the ground",
    "topScoringIntent": {
        "intent": "Sit",
        "score": 0.956479847
    },
    "intents": [
        {
            "intent": "Sit",
            "score": 0.956479847
        },
        {
            "intent": "ActAngry",
            "score": 0.0324947946
        },
        {
            "intent": "Sing",
            "score": 0.00468421541
        },
        {
            "intent": "Wave",
            "score": 0.00439580763
        },
        {
            "intent": "None",
            "score": 0.00238293572
        },
        {
            "intent": "Scratch",
            "score": 0.00228275615
        },
        {
            "intent": "Jump",
            "score": 0.00204163464
        },
        {
            "intent": "Wink",
            "score": 0.00131020416
        }
    ],
    "entities": [
        {
            "entity": "pikachu",
            "type": "Pokemon",
            "startIndex": 0,
            "endIndex": 6,
            "resolution": {
                "values": [
                    "Pikachu"
                ]
            }
        }
    ]
}
```

As you can see, LUIS returns the query we passed *pikachu sit down on the ground*, and all of the intents it believes are applicable, with associated confidence percentages. It also selects the top-ranking intent and displays it separately. Finally, LUIS returns any entities it identifies, and indicates where in the query the entity occurred.

Pretty cool, and easy to use. 

This concludes the exercise. 

<div class="exercise-end"></div>

Now that you understand how to send data to LUIS via an HTTP GET and what the output of a LUIS request looks like, let's jump into the web site you published to Azure.

### Testing LUIS through the Speech Recognition web app

<h4 class="exercise-start">
    <b>Exercise</b>: Testing LUIS in your web app
</h4>

Return to your laptop, not the VM you've bee using. Navigate to the Speech recognition web app you published to Azure earlier. My URL was [https://workshopwebapp.azurewebsites.net/](https://workshopwebapp.azurewebsites.net/).

This time, enter your CSS endpoint URL, CSS Subscription key, check the LUIS checkbox, and add your LUIS endpoint base URL, and LUIS subscription key.

<img src="images/chapter9/site1.png" class="img-override" />

> **LUIS Endpoint base URL**
>
> The LUIS endpoint URL should be everything before the query string in the LUIS endpoint URL. For example, mine is https://eastus.api.cognitive.microsoft.com/luis/v2.0/apps/6f0e678b-212c-4d4e-b0cc-967cb57f0a3a. Do not include the query string.

With LUIS enabled on the web site, test it out. Using the *Start* and *Stop* buttons, speak a few phrases:
- pikachu sit down
- jigglypuff don't sing that loud, i'm trying to sleep

Check out the results!

<img src="images/chapter9/speak.gif" class="img-override" />

Sweet!

Now, how does it all work? Well, I'll let you check out the JavaScript code on your own in the MVC app. Here's a hint: check out the `UpdateRecognizedPhrase` JavaScript function in the Index.cshtml view.

This concludes the exercise. 

<div class="exercise-end"></div>

Wow. So, we have a full end-to-end solution for performing speech-to-text analysis, identifying the intent of the text, and reacting to the identified intent. Congratulations on sticking it out.

I have a few more things to share, so you're not finished yet.

> **Final Challenge**
>
> Starting from the beginning with the Custom Speech Service (acoustic & language data sets, models, and deployments), add some more Pokemon to the mix. Then update the LUIS intents, entities, and phrase lists. Finally, add more images to the web site and deploy it all to Azure. 

> **Super Double Secret Probation Challenge**
>
> As you can see, I desperately need help styling the web site that's part of this workshop. Submit a PR to my Github repo! Please. Someone. 

### Increasing LUIS Accuracy

Earlier in the workshop, you learned how to use phrase lists to increase the accuracy of your LUIS app. But that's not the only way. LUIS has adaptive learning capabilities and the ability to update the trained LUIS model by validating the generated intent and entity hypothesis of queries it has processed.

We're not going to cover this in the workshop, but you should check out the *Review endpoint utterances* area of your *Pokemon* app. 

<img src="images/chapter9/review.png" class="img-override" />


That's officially the end of the workshop. Thank you for your time! I hope you enjoyed learning some of the advanced artificial intelligence offerings on Azure!

Go forth and build some modern apps!


