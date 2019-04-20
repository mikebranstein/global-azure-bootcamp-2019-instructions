## Custom Speech Service Models

In this chapter, you'll learn how to:
- perform testing on Microsoft's base models
- train acoustic and language models
- demonstrate that your hard work of creating custom models pays off by dramatically increasing the accuracy of Microsoft's base models

### Overview

After creating Custom Speech Service (CSS) data sets, you need to instruct CSS to train models based on these data sets. 

Training acoustic and language models is easy to do in the CSS portal - point and click. But, before we do that, we'll take a pit stop and establish a baseline accuracy of the CSS capabilities using Microsoft's base models.

> **Base Models - What Are They?**
>
> The CSS comes with several pre-trained acoustic and language models. In fact, there are different models for conversations and search/diction. If you stop to think about it, this makes a lot of sense. We tend to speak differently when we converse with others, as compared to dictating text or speaking search terms for a search engine. See below for the base models Microsoft provides.
>
> <img src="images/chapter4/model1.png" class="img-override" />

### Testing the Accuracy of the Base Models

To understand the effect our CSS customization will have, it's important to establish a baseline accuracy for the CSS service against our testing data set.

Let's get started and see how the CSS does against some of these Pokemon names ;-)

<h4 class="exercise-start">
    <b>Exercise</b>: Performing an accuracy test on the Microsoft base model
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, and navigate to *Accuracy Tests*. 

<img src="images/chapter4/test1.png" class="img-override" />

This page shows the status of the past and ongoing accuracy tests performed by the CSS.

Click the *Create New* button to begin a test against an acoustic and language model.

Complete the following fields:
- Locale: en-US
- Subscription: *the one you created earlier*
- Base Model: Microsoft Search and Diction Model, then select the base acoustic and base language models below
- Acoustic Data: Pokemon - Acoustic Data - Testing

<img src="images/chapter4/test2.png" class="img-override" />

Click *Create* to begin the test run.

When the test run is saved, you'll navigate back to the *Accuracy Test Results* page:

<img src="images/chapter4/test3.png" class="img-override" />

Note the *Status* of the test run is *NotStarted*. In a few moments, it will change to *Running*, then *Succeeded*.

The test run may take some time to execute (up to 10 minutes). So, it's a good time to take a short break. Check back in 5.

<div style="padding-left: 20px;"> <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/SEXXES5v59o?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> </div>

Hi, welcome back. I wish you had just won $1700. Will you settle for a lousy accuracy test?

So, let's check back in on the accuracy test.

<img src="images/chapter4/test4.png" class="img-override" />

Ugh! 45% word error rate - not good.

> **Word Error Rate (WER)**
>
> 'WER' (Word Error Rate) and 'Word Accuracy' are the best measurements to take when comparing two utterances, these are typically values in % and are derived by comparing a reference transcript with the speech-to-text generated transcript (or hypothesis) for the audio. In our case, the reference transcript is the transcript file we supplied for the testing data set, and the speech-to-text generated transcript (or hypothesis) is what the CSS generated when it processed the 6 audio files in the testing dataset.
>
> The algorithm used is called the Levenshtein distance, it is calculated by aligning the reference with hypothesis and counting the words that are Insertions, Deletions, and Substitutions. 
>
> In general, WER is fairly complex to calculate. We won't dive much deeper, but if you're interested in learning more, check out [this website](http://blog.voicebase.com/how-to-compare-speech-engine-accuracy). 

Well, 45% error rate is still pretty high. Let's explore the results of the accuracy test.

Click the *Details* link to learn more. At the bottom of the page, you'll find the detailed transcription (we provided that) and the hypothesis (decoder output).

<img src="images/chapter4/test5.png" class="img-override" />

You should notice several mis-interpretations, as the CSS had trouble with:
- Pikachu
- Meowth
- Jigglypuff
- Wink at me

What's interesting is that aside from the Pokemon names, the CSS did a pretty good job. It got confused a bit about winking, but perhaps I didn't annunciate very well in the test files. We'll see later on.

Another thing to note is our testing data set is *SMALL*. Really small. In fact, there are only ~30 words in the entire data set. That's really too small, and for each word missed, we add ~3% word error rate. In a production system, we'd want hundreds of utterances, and thousands of words in a testing data set. So, keep that in mind for future endeavors.

This concludes the exercise. 

<div class="exercise-end"></div>  

I know we can do better then 45%, and we will as we build our own acoustic and language models. 

### Acoustic Models

Let's get started building an acoustic model based on our acoustic data set we uploaded earlier.

<h4 class="exercise-start">
    <b>Exercise</b>: Training an acoustic model
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, and navigate to *Acoustic Models*. 

<img src="images/chapter4/acoustic-model1.png" class="img-override" />

This page shows the various acoustic models you've trained for the CSS.

Click the *Create New* button and complete the following fields:
- Locale: en-US
- Name: Pokemon - Acoustic Model
- Description: *blank*
- Base Acoustic Model: Microsoft Search and Diction Model
- Acoustic Data: Pokemon - Acoustic Data - Training
- Subscription: *your subscription*
- Accuracy Testing: *unchecked*, b/c we've already run a test

<img src="images/chapter4/acoustic-model2.png" class="img-override" />

Click *Create* to train the model.

When the model is saved, you'll navigate back to the *Acoustic Models* page:

<img src="images/chapter4/acoustic-model3.png" class="img-override" />

Note the *Status* of the test run is *NotStarted*. In a few moments, it will change to *Running*, then *Succeeded*.

The test run may take some time to execute (up to 10 minutes). So, it's a good time to take a short break. Check back in another 10.

<div style="padding-left: 20px;"> <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/25ShHY0LMpE?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> </div>

Hi, welcome back. My son *loves* these videos. 

So, let's check back in on the model training:

<img src="images/chapter4/acoustic-model4.png" class="img-override" />

Excellent, it's finished.

This concludes the exercise. 

<div class="exercise-end"></div>  

There's not much more to do with the acoustic model, so let's do the same with our language data set and train a language model

### Language Models

Training language models is just like training acoustic models, so let's dive in.

<h4 class="exercise-start">
    <b>Exercise</b>: Training a language model
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, and navigate to *Language Models*. 

<img src="images/chapter4/lang-model1.png" class="img-override" />

This page shows the various language models you've trained for the CSS.

Click the *Create New* button and complete the following fields:
- Locale: en-US
- Name: Pokemon - Language Model
- Description: *blank*
- Base Language Model: Microsoft Search and Diction Model
- Language Data: Pokemon - Language Data - Training
- Pronunciation Data: Pokemon - Pronunciation Data - Training
- Subscription: *your subscription*
- Accuracy Testing: *unchecked*, b/c we've already run a test

<img src="images/chapter4/lang-model2.png" class="img-override" />

Click *Create* to train the model.

When the model is saved, you'll navigate back to the *Language Models* page:

<img src="images/chapter4/lang-model3.png" class="img-override" />

Note the *Status* of the test run is *NotStarted*. In a few moments, it will change to *Running*, then *Succeeded*.

The training process may take some time to execute (up to 10 minutes). So, it's a good time to take yet another short break. Check back in another 5.

<div style="padding-left: 20px;"> <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/8_lfxPI5ObM?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> </div>

Welcome back, again. This video was for me. I *love* Tesla. Hopefully, I'll get one someday. Someday soon.

So, let's check back in on the model training:

<img src="images/chapter4/lang-model4.png" class="img-override" />

Excellent, it's finished.

This concludes the exercise. 

<div class="exercise-end"></div>  

### Testing the Trained Models

Now that you've built an acoustic model and language model that customizes the base models, let's test them! The original WER was 45%, so I think we can do better. 

<h4 class="exercise-start">
    <b>Exercise</b>: Performing an accuracy test on your trained models
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, and navigate to *Accuracy Tests*. 

Click the *Create New* button to begin a test against an acoustic and language model.

Complete the following fields:
- Locale: en-US
- Subscription: *the one you created earlier*
- Base Model: Microsoft Search and Diction Model, then select the Pokemon models below
- Acoustic Data: Pokemon - Acoustic Data - Testing

<img src="images/chapter4/test6.png" class="img-override" />

Click *Create* to begin the test run.

When the test run is saved, you'll navigate back to the *Accuracy Test Results* page:

<img src="images/chapter4/test7.png" class="img-override" />

Note the *Status* of the test run is *NotStarted*. In a few moments, it will change to *Running*, then *Succeeded*.

The test run may take some time to execute (up to 10 minutes). So, it's a good time to take a short break. Check back in 2.

<div style="padding-left: 20px;"> <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/A0FZIwabctw?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> </div>

This one was for everyone. And it's amazing.

So, let's check back in on the accuracy test.

<img src="images/chapter4/test8.png" class="img-override" />

Sweet! Look at that - 6% WER. I'm ok with that (for now). Feel free to explore the details of the accuracy test to learn more.

> **Challenge #5**
>
> Try to get the accuracy test WER down to 0%. Enough said.

This concludes the exercise. 

<div class="exercise-end"></div> 

In this chapter, you learned:
- why it's important to test Microsoft's base model to establish a baseline accuracy
- how to create acoustic and language models
- how to improve CSS accuracy by building customized models 

