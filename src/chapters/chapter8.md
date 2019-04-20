## Publishing and Testing LUIS Apps

In this chapter, you'll learn how to:
- train your LUIS app
- test a trained app
- publish a trained LUIS app

### Overview

The concept of training a LUIS app is like training Custom Speech Service apps. So far, you've created a training data set including intents and entities, so the next step is to train your app. Then you'll test the trained app, and publish for consumption.

In a real-world scenario, you'll incrementally advance the functionality of a LUIS app. For example, you'll start with a few intents and entities, then as your app matures (or when you identify a deficiency), you'll add new intents and entities, re-train, and publish a new version of the LUIS app.

Let's jump into the training process.

### Training LUIS

Training a LUIS app is easy. Seriously. I don't like to refer to machine learning concepts as *easy*, but all you do is click a button. 

<h4 class="exercise-start">
    <b>Exercise</b>: Training your LUIS app 
</h4>

Log in to the LUIS portal at [https://www.luis.ai](https://www.luis.ai), and navigate to the *My Apps* areas, drilling down into the *Pokemon* app.

In the upper-right corner, you'll see a *Train* button. Visually, the button will appear with a red dot when there are changes in the LUIS app that can be trained. Hovering over the button also indicates that it can untrained changes.

Click the *Train* button to train your LUIS app, integrating intents and entities into the trained app model:

<img src="images/chapter8/training.gif" class="img-override" />

When training is finished, the *Train* button has a green dot:

<img src="images/chapter8/train1.png" class="img-override" />

That's it. It was easy.

This concludes the exercise. 

<div class="exercise-end"></div>

### Testing LUIS

Now that your LUIS app is trained, you can test it. The LUIS portal offers a cool testing utility embedded, so it makes testing your app quick.

Let's test a few phrases that we may expect our app should handle.

<h4 class="exercise-start">
    <b>Exercise</b>: Testing your LUIS app 
</h4>

Log in to the LUIS portal at [https://www.luis.ai](https://www.luis.ai), and navigate to the *My Apps* areas, drilling down into the *Pokemon* app.

In the upper-right corner, you'll see a *Test* button. 

<img src="images/chapter8/test1.png" class="img-override" />

Click the *Test* button and a sidebar will pop out. In the textbox, enter in several utterances and see how your trained LUIS app performs. 

Try these utterances:
- Pikachu sit down.
- Jigglypuff jump.
- Stop being so loud Jigglypuff.

<img src="images/chapter8/test.gif" class="img-override" />

That's it for testing in the LUIS portal. 

This concludes the exercise. 

<div class="exercise-end"></div>

With a tested LUIS app that you're satisfied with, let's move on to publishing your LUIS app.

### Publishing a LUIS App

Let's jump right into publishing.

<h4 class="exercise-start">
    <b>Exercise</b>: Publishing a LUIS app 
</h4>

Log in to the LUIS portal at [https://www.luis.ai](https://www.luis.ai), and navigate to the *My Apps* areas, drilling down into the *Pokemon* app.

In the upper-right corner, you'll see a *PUBLISH* link. Click it: 

<img src="images/chapter8/pub1.png" class="img-override" />

On the publishing page, there are a few things you can do:
- select whether you publish your trained LUIS app to a staging slot (for testing) or a production slot
- choose the timezone of the published app
- have LUIS return the all intent analysis results or only the highest ranking value
- incorporate Bing spell checker to auto-correct words submitted
- add resource / subscription keys

> **Intent Analysis Results**
>
> As you're getting started with LUIS, I recommend you have your LUIS apps return all of the identified intents, because it can aid in your debugging of complex intents and models. When you become more comfortable with LUIS and have a greater confidence in it's results, you'll want to disable returning ALL results. This is also especially important on lower-bandwidth apps (like mobile).

> **Bing spell checking**
>
> In many cases, I like to enable Bing spell checker, because I cannot guarantee that text coming into LUIS is valid. But, in your Pokemon app, you should not enable Bing spell checker because you're not guaranteed the CSS speech-to-text pipeline will return valid words, plus you don't know if *Pikachu* will be replaced by something else by Bing spell checker. 
>
> A good rule of thumb: if you're using a speech-domain that could have strange words present, don't use Bing spell checker. 

We'll be publishing directly to production, so select *Production* and *Eastern* time zones. 

Before you click the *Publish to Production* button, scroll down a bit further and click the *Add Key* button.

<img src="images/chapter8/pub3.png" class="img-override" />

Select your Azure tenant, subscription, and the LUIS subscription you created in an earlier chapter and add the key.

This adds your LUIS subscription to the LUIS app you just created and allows you to deploy your app to that subscription.

After add the LUIS subscription, you can click the *Publish to Production* button:

<img src="images/chapter8/pub2.png" class="img-override" />

It will take a few minutes for LUIS to publish your app to the LUIS subscriptions you entered.

When it's finished, take note of two values below: the LUIS endpoint base URL and the Key String. For my LUIS app deployment, mine are:
- LUIS endpoint base URL: https://eastus.api.cognitive.microsoft.com/luis/v2.0/apps/6f0e678b-212c-4d4e-b0cc-967cb57f0a3a
Key String: 4a580409############ad344e6f25 (some obscured)

You'll need these values in the final chapter to test it all out!

This concludes the exercise. 

<div class="exercise-end"></div>


