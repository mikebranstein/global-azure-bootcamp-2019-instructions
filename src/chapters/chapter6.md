## Introduction to Language Understanding (LUIS)

In this chapter, you'll learn:
- how Azure's Language Understanding (LUIS) service can identify the intent of text
- how to provision a LUIS subscription and it to the LUIS web portal

### Overview

In the past chapters, you've been focused on building a customized speech recognition engine with the CSS. Now that you are equipped with the knowledge to build these on your own, we'll turn our attention to analyzing the results your CSS generates.

In most machine learning and speech-to-text projects, using a single product (like CSS) isn't common. Instead, you'll often chain the results of one service to the input of another. This process is referred to as building a machine learning *pipeline*. 

In the final chapters of this workshop, you'll be expanding your speech-to-text pipeline by adding intent analysis with Language Understanding (LUIS).

#### About Language Understanding (LUIS)

Language Understanding (LUIS) allows your application to understand what a person wants in their own words. LUIS uses machine learning to allow developers to build applications that can receive user input in natural language and extract meaning from it. A client application that converses with the user can pass user input to a LUIS app and receive relevant, detailed information back.

#### What is a LUIS app?

A LUIS app is a domain-specific language model designed by you and tailored to your needs. You can start with a prebuilt domain model, build your own, or blend pieces of a prebuilt domain with your own custom information.

A model starts with a list of general user intentions such as "Book Flight" or "Contact Help Desk." Once the intentions are identified, you supply example phrases called utterances for the intents. Then you label the utterances with any specific details you want LUIS to pull out of the utterance.

Prebuilt domain models include all these pieces for you and are a great way to start using LUIS quickly.

After the model is designed, trained, and published, it is ready to receive and process utterances. The LUIS app receives the utterance as an HTTP request and responds with extracted user intentions. Your client application sends the utterance and receives LUIS's evaluation as a JSON object. Your client app can then take appropriate action.

<img src="images/chapter6/luis-overview-process.png" class="img-override" />

#### Key LUIS Concepts

**Intents** 

An intent represents actions the user wants to perform. The intent is a purpose or goal expressed in a user's input, such as booking a flight, paying a bill, or finding a news article. You define and name intents that correspond to these actions. A travel app may define an intent named "BookFlight."

**Utterances**

An utterance is text input from the user that your app needs to understand. It may be a sentence, like "Book a ticket to Paris", or a fragment of a sentence, like "Booking" or "Paris flight." Utterances aren't always well-formed, and there can be many utterance variations for a particular intent.

**Entities**

An entity represents detailed information that is relevant in the utterance. For example, in the utterance "Book a ticket to Paris", "Paris" is a location. By recognizing and labeling the entities that are mentioned in the user’s utterance, LUIS helps you choose the specific action to take to answer a user's request.

| Intent | &nbsp;&nbsp;&nbsp;&nbsp; | Sample User Utterance | &nbsp;&nbsp;&nbsp;&nbsp; | Entities |
| :----- | ------ | :-------------------- | ------ | :------- | 
| BookFlight | &nbsp; | "Book a flight to Seattle?" | &nbsp; | Seattle |
| StoreHoursAndLocation | &nbsp; | "When does your store open?" | &nbsp; | open |
| ScheduleMeeting | &nbsp; | "Schedule a meeting at 1pm with Bob in Distribution" | &nbsp; | 1pm, Bob |

#### How you'll use LUIS

Now that you know a little bit about LUIS, let's see how it'll be used in conjunction with CSS. 

When you're finished integrating LUIS into your solution, you'll be able to speak commands into the web site you published, and ask a variety of Pokemon (Pikachu, Jigglypuff, Meowth, etc.) to perform a variety of actions (sit, jump, scratch, sing, etc.).

LUIS will be used to take the output of the CSS endpoint you created and identify the intent (the action) and entity (the Pokemon). Then, the web site will parse the LUIS response and act accordingly by displaying the appropriate image at the bottom of the page. 

### Provisioning in Azure

Before we can use LUIS, we'll need to provision a LUIS subscription in the Azure portal. Let's get started!

<h4 class="exercise-start">
    <b>Exercise</b>: Creating a LUIS Subscription
</h4>

Start by jumping back to the Azure portal, and create a new resource by clicking the *Create a resource* button.

Search for *Language Understanding*:

<img src="images/chapter6/luis1.png" class="img-override" />

Fill out the required parameters as you create an instance:
- Name: *workshop-luis*, or something similar
- Subscription: *your Azure subscription*
- Location: *East US*
- Pricing tier: *F0*
- Resource group: the resource group you created earlier

<img src="images/chapter6/luis2.png" class="img-override" />

When the LUIS subscription is provisioned, it will appear in your resource group:

<img src="images/chapter6/luis3.png" class="img-override" />

The final step is to navigate to the LUIS subscription by clicking on it. 

Locate the *Keys* area and take note of *KEY 1*:

<img src="images/chapter6/luis4.png" class="img-override" />

You'll need this key in the next chapter, so don't forget it.

This concludes the exercise. 

<div class="exercise-end"></div>

### Linking your Subscription on the LUIS Web Portal

There's not much you can do with LUIS in the Azure portal because it has a separate portal (like the Custom Speech Service). 

The process for linking your LUIS subscription in the LUIS portal is a bit different than linking your CSS subscription. As a result, we'll revisit this in a later chapter. 

For now, hold on to your subscription key.

