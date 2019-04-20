## Building a LUIS App

In this chapter, you'll learn:
- how to build intents
- how to create entities and map them to intent phrases
- how phrase lists can help increase accuracy

### Overview

In the last chapter you learned about the basics of LUIS: intents, utterances, entities. You also learned how we'll be using LUIS and how it will integrate into our speech-to-text pipeline.

I don't want to delay us too much, so let's dive right in by creating your first LUIS app.

### Creating a LUIS app

<h4 class="exercise-start">
    <b>Exercise</b>: Creating a LUIS App
</h4>

Log in to the LUIS portal at [https://www.luis.ai](https://www.luis.ai), and navigate to the *My Apps* area:

<img src="images/chapter7/luis1.png" class="img-override" />

This page lists the various LUIS apps you have created.

Click the *Create new app* button. Name it *Pokemon*:

<img src="images/chapter7/luis2.png" class="img-override" />

After creating the app, you'll be redirected to the *Intent* page automatically:

<img src="images/chapter7/luis3.png" class="img-override" />

There's not much more to creating the initial LUIS app, so let's continue on with defining your app intents.

This concludes the exercise. 

<div class="exercise-end"></div>

### Creating Intents

You'll recall that LUIS intents are represent actions the user wants to perform. The intent is a purpose or goal expressed in a user's input, such as booking a flight, paying a bill, or finding a news article. In our case, we'll be creating intents like sitting down, scratching, jumping, etc.

<h4 class="exercise-start">
    <b>Exercise</b>: Creating LUIS intents
</h4>

Log in to the LUIS portal at [https://www.luis.ai](https://www.luis.ai), and navigate to the *Intents* area by first navigating to the *My Apps* areas, drilling down into the *Pokemon* app:

<img src="images/chapter7/luis3.png" class="img-override" />

This page lists the various intents you have created. You'll notice that a *None* intent was automatically created for you. All LUIS apps start with this intent - you *should not* delete it. The *None* intent is used as a default fall-back intent for your LUIS apps. 

#### Finding the intents

You'll be creating a variety of intents in the *Pokemon* app. To help you, we've already defined the intents for you. In the code you downloaded from Github, locate the *language-understanding-data* folder. It contains a file named *luis-utterances.md*:

<img src="images/chapter7/luis4.png" class="img-override" />

Inside the markdown file, you'll find an intent followed by a series of utterances. For example:

```
# Sit
- ash's best friend should sit down
- sit pikachu
- sit on the floor pikachu
- have a seat meowth
- meowth please sit on the ground
```

In the snippet above, the heading `Sit` is the name of an intent you'll create, followed by utterances that align to the intent.

LUIS is like the Custom Speech Service, as it needs to *learn* the types of phrases (or utterances) that map to an intent. 

Referencing the *luis-utterances.md* file, create intents for each intent listed in the document.

Follow along below to create the first intent, then rinse and repeat for the remaining intents.

#### Adding an intent

To add an intent, click the *Create new intent* button:

<img src="images/chapter7/luis5.png" class="img-override" />

Name the intent *Sit*, as referenced in the *luis-utterances.md* file:

<img src="images/chapter7/luis6.png" class="img-override" />

On the next screen, the *Sit* intent will be listed at the top. In the text box below the intent, enter in the related utterances from the *luis-utterances.md* file. After each utterance, press *Enter* to save the utterance.

<img src="images/chapter7/intent.gif" class="img-override" />

You'll notice that the LUIS portal associated the utterance with the intent each time you press *Enter*.

When you've finished entering the associated utterances, you can scroll down to review and modify them:

<img src="images/chapter7/luis7.png" class="img-override" />

You'll also notice each utterance has a drop down next to it allow you to re-associate it with a different intent, if you made a mistake.

When you're finished entering the utterances for this intent, navigate back to the list of intents by clicking the *Intents* link on the left:

<img src="images/chapter7/luis8.png" class="img-override" />

#### Finish adding the remaining intents

Proceed to add the remaining intents listed in the *luis-utterances.md* file. When you're finished, your list of intents should look like these:

<img src="images/chapter7/luis9.png" class="img-override" />

This concludes the exercise. 

<div class="exercise-end"></div>

### Adding Entities

After adding intents to your LUIS app, it's time to add entities. As you'll recall, an entity represents detailed information that is relevant in an utterance. For example, in the utterance "Jigglypuff, stop singing", "Jigglypuff" is a Pokemon. By recognizing and labeling the entities that are mentioned in the user’s utterance, LUIS helps you choose the specific action to take to answer a user's request.

#### Additional entity information

There are a few important things about entities that are relevant, but we hadn't covered them yet. 

First, entities are optional but highly recommended.

While intents are required, entities are optional. You do not need to create entities for every concept in your app, but only for those required for the app to take action.

For example, as you begin to develop a machine learning pipeline that integrates LUIS, you may not have a need to identify details/entities to act upon. So, when starting off, don't add them. Then, as your app matures, you can slowly add entities. 

Second, entities are shared across intents. They don't belong to any single intent. Intents and entities can be semantically associated but it is not an exclusive relationship. This allows you to have a detail/entity be applicable across various intents. 

In the LUIS app you're building today, we'll be defining a *Pokemon* entity that can identify a Pokemon by name. Because each of our intents typically involves a particular Pokemon, the *Pokemon* entity will be shared across intents. This means that an intent/entity combination gives us a unique action to perform. 

For example, the "Jigglypuff, sing." utterance yields the intent of *Sing* with an identified *Pokemon* entity of type *Jigglypuff*. 

#### Types of entities

LUIS has a variety of entity types, and each has a specific use. There are too many to dive into here, but I encourage you to learn more by reading the [LUIS documentation]()https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-entity-types#types-of-entities.

Now that you know about entities, let's add the *Pokemon* entity.

<h4 class="exercise-start">
    <b>Exercise</b>: Creating LUIS intents
</h4>

Log in to the LUIS portal at [https://www.luis.ai](https://www.luis.ai), and navigate to the *Entities* area by first navigating to the *My Apps* areas, drilling down into the *Pokemon* app:

<img src="images/chapter7/ent1.png" class="img-override" />

This page lists the entities you have defined.

Click the *Create new entity* button to create the *Pokemon* entity. Select *List* as the entity type:

<img src="images/chapter7/ent2.png" class="img-override" />

> **List Entities**
>
> A list entity is a fixed list of values. Each value is itself a list of synonyms or other forms the value may take. For example, a list entity named PacificStates include the values Washington, Oregon, California. The Washington value then includes both "Washington" and the abbreviation "WA".

After clicking the *Done* button, you're redirected to the *Pokemon* entity detail page. Add the following values:
- Pikachu
- Jigglypuff
- Meowth

After adding these Pokemon, add synonyms for each, as shown below.

<img src="images/chapter7/ent3.png" class="img-override" />

By adding these synonyms, you train LUIS to recognize the synonyms as the entity list value. For example, *bubble pokemon* will be recognized as a *Pokemon* entity, with *Jigglypuff* as the specific type.

#### Validating entity mapping

When you've finished adding the entities and synonyms, refresh your browser.

To validate that the entities are being properly recognized, navigate back to the *Intents* page, then open the detail page for an intent.

You'll notice that text in each utterance is now replaced with a generic intent *Pokemon* block. You can use the *Entities view* toggle switch to see how utterance text maps to an entity:

<img src="images/chapter7/entity.gif" class="img-override" />

This concludes the exercise. 

<div class="exercise-end"></div>

### Increasing Accuracy with Phrase Lists

Now that you've added intents and entities, let's explore a quick way to improve the accuracy of your LUIS app with phrase lists.

#### What is a phrase list?

A phrase list includes a group of values (words or phrases) that belong to the same class and must be treated similarly (for example, names of cities or products). What LUIS learns about one of them is automatically applied to the others as well. This is not a white list of matched words.

In other words, a phrase list can help you better identify an intent or entity in a more dynamic manner.

#### How to use phrase lists

For example, in a travel agent app, you can create a phrase list named "Cities" that contains the values London, Paris, and Cairo. If you label one of these values as an entity, LUIS learns to recognize the others.

A phrase list may be interchangeable or non-interchangeable. An interchangeable phrase list is for values that are synonyms, and a non-interchangeable phrase list is intended for values that aren't synonyms but are similar in another way.

There are two rules of thumb to keep in mind when using phrase lists:

1. Use phrase lists for terms that LUIS has difficulty recognizing. Phrase lists are a good way to tune the performance of your LUIS app. If your app has trouble classifying some utterances as the correct intent, or recognizing some entities, think about whether the utterances contain unusual words, or words that might be ambiguous in meaning. These words are good candidates to include in a phrase list feature.

2. Use phrase lists for rare, proprietary, and foreign words.
LUIS may be unable to recognize rare and proprietary words, as well as foreign words (outside of the culture of the app), and therefore they should be added to a phrase list feature. This phrase list should be marked non-interchangeable, to indicate that the set of rare words form a class that LUIS should learn to recognize, but they are not synonyms or interchangeable with each other.

> **A note on using phrase lists**
>
> A phrase list feature is not an instruction to LUIS to perform strict matching or always label all terms in the phrase list exactly the same. It is simply a hint. For example, you could have a phrase list that indicates that "Patti" and "Selma" are names, but LUIS can still use contextual information to recognize that they mean something different in "make a reservation for 2 at patti's diner for dinner" and "give me driving directions to selma, georgia".

#### When to use phrase lists instead of list entities

After learning about phrase lists, you may be confused on whether to use these, or to use a list entity.

With a phrase list, LUIS can still take context into account and generalize to identify items that are similar to, but not an exact match, as items in a list. If you need your LUIS app to be able to generalize and identify new items in a category, it's better to use a phrase list. When you want to be able to recognize new instances of an entity, like a meeting scheduler that should recognize the names of new contacts, or an inventory app that should recognize new products, use another type of machine learned entity such as a simple or hierarchical entity. Then create a phrase list of words and phrases. This list guides LUIS to recognize examples of the entity by adding additional significance to the value of those words.

A list entity explicitly defines every value an entity can take, and only identifies values that match exactly. A list entity may be appropriate for an app in which all instances of an entity are known and don't change often, like the food items on a restaurant menu that changes infrequently.

> **LUIS Best practice**
>
> After the model's first iteration, add a phrase list feature that has domain-specific words and phrases. This feature helps LUIS adapt to the domain-specific vocabulary, and learn it fast.

<h4 class="exercise-start">
    <b>Exercise</b>: Creating phrase lists to increase accuracy
</h4>

Log in to the LUIS portal at [https://www.luis.ai](https://www.luis.ai), and navigate to the *My Apps* areas, drilling down into the *Pokemon* app.

Click on the *Phrase lists* navigation option:

<img src="images/chapter7/phrase1.png" class="img-override" />

Click the *Create new phrase list* button to create a phrase list.

You'll be using the phrase list feature to create a phrase list for each of our intents. This is a good idea because our intents like *Sit* and *Jump* are fairly generic, and you can describe each of these intents in a variety of ways. For example, "sit down" and "grab a chair" are different utterances that have the same intent: *Sit*.

Add a phrase list named *Sit*, and either type in a variety of synonyms for sit, or use the *Recommend* option to help identify a list of words that are related to sitting:

<img src="images/chapter7/phrase.gif" class="img-override" />

I added the following options to my phrase lists, feel free to create your own, add more, subtract some, but be sure to create a phrase list for each intent:

- Sit: sit, lie, rest, be seated, sits, sitting, stay, relax, take a seat, to stay 
- Wave: wave, signal, waves, sign, gesticulate 
- Sing: sing, singing, sings, chant, sang 
- Jump: jump, skip, hop, jumping, jumps, bounce, leap, jumped 
- Wink: wink, smile, grin, twinkle, winks 
- Scratch: scratch, scrape, rub, scratching, graze, scratches, scratched
- ActAngry: angry, annoyed, enraged, frustrated, angered, irate, angers, sad, irritated, fuming 

When you're finished, your phrase lists should look like this:

<img src="images/chapter7/phrase2.png" class="img-override" />

This concludes the exercise. 

<div class="exercise-end"></div>

That's it - you've created intents and entities to help train your LUIS app to recognize Pokemon and an action you want the Pokemon to perform. You've also added phrase lists to help train your app to respond and identify intents more efficiently.

In the next chapter, you'll train, test, and publish your LUIS app.


