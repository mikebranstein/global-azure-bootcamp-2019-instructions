## Introduction to the Custom Speech Service

In this chapter you'll learn about the Custom Speech Service, how to provision one in the Azure Portal, and how to link your subscription to the Custom Speech Service portal.

> **Abbreviation** 
>
> To save some time, you may see me refer to the Custom Speech Service as CSS. I know it can be confusing, especially if you're a web developer. But, let's pretend for a day that you're not, and use CSS in a different way. Thanks!

### Overview

The Custom Speech Service enables you to create a customized speech-to-text platform that meets the needs of your business. With the service, you create customized language models and acoustic models tailored to your application and your users. By uploading your specific speech and/or text data to the Custom Speech Service, you can create custom models that can be used in conjunction with Microsoft’s existing state-of-the-art speech models. With these capabilities, you're able to filter out common background noise, adjust for localized dialects, and train the speech service to recognize non-standard/obscure words and phrases (like "Pokemon", scientific terms, and technical jargon).

For example, if you’re adding voice interaction to a mobile phone, tablet or PC app, you can create a custom language model that can be combined with Microsoft’s acoustic model to create a speech-to-text endpoint designed especially for your app. If your application is designed for use in a particular environment or by a particular user population, you can also create and deploy a custom acoustic model with this service.

#### How do speech recognition systems work?

Before you get started, it's important to understand how speech recognition systems work.

Speech recognition systems are composed of several components that work together. Two of the most important components are the acoustic model and the language model.

> **Acoustic Model**
>
> The acoustic model is a classifier that labels short fragments of audio into one of a number of phonemes, or sound units, in a given language. For example, the word “speech” is comprised of four phonemes “s p iy ch”. These classifications are made on the order of 100 times per second.

> **Phoneme**
>
> In short, a sound unit. Any of the perceptually distinct units of sound in a specified language that distinguish one word from another, for example p, b, d, and t in the English words pad, pat, bad, and bat.

> **Language Model**
>
> The language model is a probability distribution over sequences of words. The language model helps the system decide among sequences of words that sound similar, based on the likelihood of the word sequences themselves. For example, “recognize speech” and “wreck a nice beach” sound alike but the first hypothesis is far more likely to occur, and therefore will be assigned a higher score by the language model.

Both the acoustic and language models are statistical models learned from training data. As a result, they perform best when the speech they encounter when used in applications is similar to the data observed during training. The acoustic and language models in the Microsoft Speech-To-Text engine have been trained on an enormous collection of speech and text and provide state-of-the-art performance for the most common usage scenarios, such as interacting with Cortana on your smart phone, tablet or PC, searching the web by voice or dictating text messages to a friend.

> **Credits**
>
> This section was borrowed from Microsoft's [official documentation](https://docs.microsoft.com/en-us/azure/cognitive-services/Custom-Speech-Service/cognitive-services-custom-speech-home#what-is-the-custom-speech-service). Thank you!

#### Using the Custom Speech Service, Acoustic Models, and Language Models

Throughout the next several chapters, you'll be building acoustic and language models. Don't worry if you don't understand everything right now, because you'll be learning as you go.

> **Bing Speech API**
>
> Microsoft has another speech-to-text service in Azure called the Bing Speech API. This API is like the Custom Speech Service, but it cannot be customized. I like to think of the Bing Speech API as a v1 product, and the Custom Speech Service as a v2 product. Both are highly capable, but when I need to account for background noise, custom words, etc. I choose the Custom Speech Service.

### Provisioning in Azure

Now that you know what the Custom Speech Service can do, let's start using it! You'll start by creating a Custom Speech Service instance in the Azure portal. 

<h4 class="exercise-start">
    <b>Exercise</b>: Creating a Custom Speech Service Instance
</h4>

Start by jumping back to the Azure portal, and create a new resource by clicking the *Create a resource* button.

Search for *Custom Speech Service*:

<img src="images/chapter2/css-search.png" class="img-override" />

Fill out the required parameters as you create an instance:
- Name: *workshop-css*, or something similar
- Subscription
- Location: *West US*
- Pricing tier: *F0*
- Resource group: the resource group you created earlier

<img src="images/chapter2/css-create.png" class="img-override" />

> **West US Location**
>
> Normally, I recommend you keep resources in the same region, but the Custom Speech Service is in preview right now, so it's only available in West US.

When the Custom Speech Service instance is provisioned, it will appear in your resource group:

<img src="images/chapter2/css-resource-group.png" class="img-override" />

The final step is to navigate to the Custom Speech Service instance by clicking on it. 

Locate the *Keys* area and take note of *KEY 1*:

<img src="images/chapter2/css-keys.png" class="img-override" />

You'll need this key in the next step, so don't forget it.

This concludes the exercise. 

<div class="exercise-end"></div>

### Linking your Subscription on the CSS Web Portal

There's not much you can do with the Custom Speech Service in the Azure portal because the service is still in preview. Instead, a separate portal exists to perform customizations and work with the service. In the next section, you'll be introduced to the Custom Speech Service portal.

<h4 class="exercise-start">
    <b>Exercise</b>: Linking your CSS subscription to the CSS portal
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>.

Click the *Sign In* link in the upper right and sign in with your Azure portal subscription login.

After logging in, click on your name the upper right, and select the *Subscriptions* option below it:

<img src="images/chapter2/portal-sub.png" class="img-override" />

#### Subscriptions

The *Subscriptions* page shows all of your connected CSS subscriptions. 

<img src="images/chapter2/portal-sub-page.png" class="img-override" />

Click the *Connect existing subscription* button. Add the CSS subscription you just created in the Azure portal. Give it a name and enter *KEY 1* from the Azure portal.

<img src="images/chapter2/sub-add.png" class="img-override" />

You should see the subscription appear on the subscriptions page.

This concludes the exercise. 

<div class="exercise-end"></div>

That's it. In the next chapter, you'll start to use the CSS by creating various data sets for training and testing.
