## Building Custom Speech Service data sets 

In this chapter, you'll learn:
- The difference between training and testing data sets
- How acoustic data sets are built 
- That language data sets help the Custom Speech Service (CSS) understand the likelihood of certain words and phrases
- That pronunciation data sets can help with simple word and and syllable replacements

### Understanding Machine Learning Data Sets

At the core of every artificial intelligence (or machine learning) problem is data. And that data is used in various capacities to train, build, and test the systems you develop. Because data is so critical to machine learning endeavors, you'll need to learn about the different ways data is used.

> **Thank you, StackExchange**
>
> This next section was adapted from a [StackExchange post](https://stats.stackexchange.com/questions/19048/what-is-the-difference-between-test-set-and-validation-set). Thank you to all that contributed, as you said it better than I could have.

#### Training and Test Data Sets

In many machine learning processes, you need two types of data sets:

- In one data set (your *gold standard*) you have the input data together with correct/expected output, This data set is usually duly prepared either by humans or by collecting some data in semi-automated way. But it is important that you have the expected output for every data row here, because you need to feed the machine learning algorithms the *expected*, or *correct* results for it to learn. This data set is often referred to as the *training data set*. 

- In the other data set, you collect the data you are going to apply your model to. In many cases this is the data where you are interested for the output of your model and thus you don't have any "expected" output here yet. This is often real-world data. 

With these two data sets, the machine learning process adheres to a standard 3-phase process:

1. Training phase: you present your data from your "gold standard" (or *training* data set) and train your model, by pairing the input with expected output. Often you split your entire training data set into two pieces. Approximately 70% of the training data is used for training, and 30% reserved for validation/testing. The 30% reserved data is often referred to as *test* data. The result of this phase is a trained model.

2. Validation/Test phase: to estimate how well your trained model has been trained, you pass in the reserved 30% of your testing data and evaluate it's accuracy. 

3. Application phase: now you apply your trained model to the real-world data and get the results. Since you normally don't have any reference value in this type of data, you can only speculate about the quality of your model output using the results of your validation phase. You perform additional accuracy tests.

> **Separation of Training, Test, and Real-World Data Sets**
>
> An easy mistake to make with your training, test, and real-world data sets is overlapping data (or reusing data from one set in another). Imagine that you training a model to answer true/false questions using a series of 10 questions and answers. After the model is trained, you use the same 10 questions to evaluate how well the model performs. Ideally, it should perform 100%, but you don't know how well it *really* performs because you tested with the training data. The only true test is to use other real-world questions, then re-evaluate its performance.

#### Applying Machine Learning Data Set Concepts to the Custom Speech Service

Now that you know about the different types of data, you'll be creating training data sets for acoustic, language, and pronunciation data, then testing acoustic data. 

> **Acoustic, Language, and Pronunciation**
>
> Don't worry if you don't know the difference between these 3 types of data the CSS uses, you'll be learning about it next.

### Acoustic Training data sets

In a previous chapter, you learned about acoustic models. 

> **Acoustic Model**
>
> The acoustic model is a classifier that labels short fragments of audio into one of a number of phonemes, or sound units, in a given language. For example, the word “speech” is comprised of four phonemes “s p iy ch”. 

To build acoustic models, you need acoustic data sets. An acoustic data set consists of two parts: 

1. a set of audio files containing the speech data
2. a file containing the transcriptions of all audio files

#### Audio File Format and Recommendations

To build testing acoustic audio data for the Custom Speech Service, you should adhere to the following guidelines:

- All audio files in the data set should be stored in the WAV (RIFF) audio format.
- The audio must have a sampling rate of 8 kHz or 16 kHz and the sample values should be stored as uncompressed PCM 16-bit signed integers (shorts).
- Only single channel (mono) audio files are supported.
- The audio files must be between 100ms and 1 minute in length. Each audio file should ideally start and end with at least 100ms of silence, and somewhere between 500ms and 1 second is common.
- If you have background noise in your data, it is recommended to also have some examples with longer segments of silence, e.g. a few seconds, in your data, before and/or after the speech content.
- Each audio file should consist of a single utterance, e.g. a single sentence for dictation, a single query, or a single turn of a dialog system.
- Each audio file to in the data set should have a unique filename and the extension “wav”.
- The set of audio files should be placed in a single folder without subdirectories and the entire set of audio files should be packaged as a single ZIP file archive.

> **Holy Audio Requirements, Batman!**
>
> Yeah. This is a lot to take in. Don't worry. I've already built the audio files for you. We'll take a look in a bit.

#### Audio File Transcriptions

The second component of acoustic data is a text file containing transcripts of each audio file. 

The transcriptions for all WAV files should be contained in a single plain-text file. Each line of the transcription file should have the name of one of the audio files, followed by the corresponding transcription. The file name and transcription should be separated by a tab (\t). Each line must end with a line feed and new line character (\r\n).

For example:

```
speech01.wav    speech recognition is awesome
speech02.wav    the quick brown fox jumped all over the place
speech03.wav    the lazy dog was not amused
```

The transcriptions should be text-normalized so they can be processed by the system. However, there are some very important normalizations that must be done by the user prior to uploading the data to the Custom Speech Service. The [normalization rules](https://docs.microsoft.com/en-us/azure/cognitive-services/custom-speech-service/customspeech-how-to-topics/cognitive-services-custom-speech-transcription-guidelines) are too lengthy to cover here, so you should check them out on your own. It may seem like a lot at first, but i've found it fairly straight-forward and I was quickly able to learn and apply them regularly.

#### Prepping your acoustic data for the CSS portal

In the source code you downloaded from Github, you'll find the training audio files and an audio transcript of the files in the *custom-speech-service-data/training* folder:

<img src="images/chapter3/acoustic-training-data.png" class="img-override" />

> **Pokemon!**
>
> You may have noticed the file names of the acoustic data are Pokemon. My son and I have recently started to play Pokemon the Card Game together, so I thought this would be a fun way (and topic) to teach you about speech recognition. After all, Pokemon names *are* difficult to pronounce, and are a domain-specific language of their own. They're a perfect match for the capabilities of the Custom Speech Service.

Let's get started by uploading an acoustic data set to the CSS portal.

<h4 class="exercise-start">
    <b>Exercise</b>: Uploading an acoustic data set to the CSS portal
</h4>

Start by locating the acoustic .wav audio files. Select the 17 audio files, zip them up, and name the zip file *training-utterances.zip*.

<img src="images/chapter3/training-utterances.gif" class="img-override" />

Next, navigate to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>.

Click the *Sign In* link in the upper right and sign in with your Azure portal subscription login.

After logging in, click on the *Custom Speech* navigation option and navigate to *Adpatation Data*:

<img src="images/chapter3/adaptation-data.png" class="img-override" />

At the top of the *Adaptation Data* page, will be an area for *Acoustic Datasets*. 

<img src="images/chapter3/import1.png" class="img-override" />

Click the *Import* button and complete the following fields:
- Name: Pokemon - Acoustic Data - Training
- Description *blank*
- Locale: en-US
- Transcriptions file (.txt): upload the *training-utterances.txt* file
- Audio files (.zip): upload the *training-utterances.zip* file you created earlier

<img src="images/chapter3/import-acoustic-data.png" class="img-override" />

Click *Import* to upload the acoustic data and build the data set.

When the data is uploaded, you'll navigate back to the *Acoustic Datasets* page and your data set will be displayed in the grid:

<img src="images/chapter3/import3.png" class="img-override" />

Note the *Status* of the acoustic dataset is *NotStarted*. In a few moments, it will change to *Running*:

<img src="images/chapter3/import4.png" class="img-override" />

When you upload acoustic data, the CSS will analyze the data, check it for errors, and ensure the transcription file matches the uploaded audio filenames. There are a variety of other checks that are performed that aren't important, but it's good to know that there is some post-processing that needs to occur before you can use the acoustic data set.

When the CSS finishes analyzing and validating the acoustic data, the *Status* will change to *Succeeded*:

<img src="images/chapter3/import5.png" class="img-override" />

Congratulations! You've created your first acoustic data set. We'll be using it later in this chapter.

> **Curious? ...and Challenge #1**
>
> If you're wondering what the audio files sound like, don't hesitate to download them to your computer and play them. Just remember that playing the audio files on the VM we've created for the workshop probably won't work, so you'll have to download the files to your actual computer.
>
> If you're in the mood for a challenge, augment the training data by adding your own audio files. I've found the open-source software [Audacity](https://www.audacityteam.org/) to be a great tool for recording, editing, and exporting audio files in the right format. I suggest creating a few sample audio utterances relating to your favorite Pokemon (or try [Charizard](https://wiki.kidzsearch.com/wiki/Charizard)).
>
> <img src="images/chapter3/charizard.png" class="img-small" />
>
> If you do add to the acoustic data set, don't forget to transcribe your audio and add the transcription to the *training-utterances.txt* file!

This concludes the exercise. 

<div class="exercise-end"></div> 

### Language Training data sets

Now that you've created an acoustic data set, let's build a language data set. As you'll recall from a previous chapter, language models and language data sets teach the CSS the likelihood of encountering certain words or phrases. 

> **Language Model**
>
> The language model is a probability distribution over sequences of words. The language model helps the system decide among sequences of words that sound similar, based on the likelihood of the word sequences themselves. For example, “recognize speech” and “wreck a nice beach” sound alike but the first hypothesis is far more likely to occur, and therefore will be assigned a higher score by the language model.

#### Language Data Sets

To create a custom language data set for your application, you need to provide a list of example utterances to the system, for example:

- "pikachu please sit down"
- "don't sing jigglypuff you'll put me to sleep"
- "meowth put away those sharp claws"

The sentences do not need to be complete sentences or grammatically correct, and should accurately reflect the spoken input you expect the system to encounter in deployment. These examples should reflect both the style and content of the task the users will perform with your application.

The language model data should be written in plain-text file using either the US-ASCII or UTF-8, depending of the locale. For en-US, both encodings are supported. The text file should contain one example (sentence, utterance, or query) per line.

If you wish some sentences to have a higher weight (importance), you can add it several times to your data. A good number of repetitions is between 10 - 100. If you normalize it to 100 you can weight sentence relative to this easily.

> **More Rules!**
>
> Don't worry about these rules for now, because we've already assembled a collection of utterances appropriate for our needs today.

Before we get started, take a look at the utterances in the *training-language-model-data.txt* file. Here's a short excerpt:

```
ash's best friend should sit down
sit pikachu
sit on the floor pikachu
have a seat meowth
meowth please sit on the ground
i'd like to see ash's best friend act angry
get really mad pikachu
``` 

You'll notice that this is a collection of commands. This is of importance and significance. Later in the workshop, you'll be using the Language Understanding (LUIS) service to analyze the intent of spoken commands. So, it makes sense that the language model we'll be building contains commands.

#### Creating a Language Data Set

Now that you know what is in a language data set, let's head over to the CSS portal and create one.

<h4 class="exercise-start">
    <b>Exercise</b>: Uploading a language data set to the CSS portal
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, then navigate back to the *Adaptation Data* page.

Scroll down past the *Acoustic Datasets* area, and you'll find the *Language Datasets* area:

<img src="images/chapter3/lang1.png" class="img-override" />

Click the *Import* button and complete the following fields:
- Name: Pokemon - Language Data - Training
- Description *blank*
- Locale: en-US
- Language data file (.txt): upload the *training-language-model-data.txt* file

<img src="images/chapter3/lang2.png" class="img-override" />

Click *Import* to upload the language data and build the data set.

When the data is uploaded, you'll navigate back to the *Language Datasets* page and your data set will be displayed in the grid:

<img src="images/chapter3/lang3.png" class="img-override" />

Note the *Status* of the language data set is *NotStarted*. In a few moments, it will change to *Running*, the *Succeeded*, just like the acoustic data set did.

Congratulations! You've created your first language data set. We'll be using it later in this chapter.

> **Challenge #2**
>
> Just like you did for the acoustic data set, feel free to augment the utterances I built. I suggest continuing to create utterances related to the Pokemon you added in the last challenge.

This concludes the exercise. 

<div class="exercise-end"></div> 

### Pronunciation Training data sets

Now that you've created acoustic and language data sets, you could be ready to move on to designing the models for each. But, there's another customization you can provide that helps to train you model in a special way: pronunciation data.

Custom pronunciation enables users to define the phonetic form and display of a word or term. It is useful for handling customized terms, such as product names or acronyms. All you need is a pronunciation file (a simple .txt file).

Here's how it works. In a single .txt file, you can enter several custom pronunciation entries. The structure is as follows:

```
Display form <Tab>(\t) Spoken form <Newline>(\r\n)
```

#### Requirements for the spoken form

The spoken form must be lowercase, which can be forced during the import. No tab in either the spoken form or the display form is permitted. There might, however, be more forbidden characters in the display form (for example, ~ and ^).

Each .txt file can have several entries. For example, see the following screenshot:

<img src="images/chapter3/custom-speech-pronunciation-file.png" class="img-override" />

The spoken form is the phonetic sequence of the display form. It is composed of letters, words, or syllables. Currently, there is no further guidance or set of standards to help you formulate the spoken form.

#### When to use Pronunciation data

I've found it useful to use pronunciation in a variety of circumstances. In the above example, pronunciation helps transform *see three pee oh* to *C3PO*. I've also used it in the past to transform *a t and t* to *AT&T*, and *microsoft dot com* to *Microsoft.com*.

#### Adding a Pronunciation Data Set

For your final data set, you'll create a pronunciation data set. Let's get to it!

<h4 class="exercise-start">
    <b>Exercise</b>: Uploading a pronunciation data set to the CSS portal
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, then navigate back to the *Adaptation Data* page.

Scroll down past the *Language Datasets* area, and you'll find the *Pronunciation Datasets* area:

<img src="images/chapter3/pro1.png" class="img-override" />

Click the *Import* button and complete the following fields:
- Name: Pokemon - Pronunciation Data - Training
- Description *blank*
- Locale: en-US
- Language data file (.txt): upload the *training-pronunciation-data.txt* file

<img src="images/chapter3/pro2.png" class="img-override" />

Click *Import* to upload the pronunciation data and build the data set.

When the data is uploaded, you'll navigate back to the *Pronunciation Datasets* page and your data set will be displayed in the grid:

<img src="images/chapter3/pro3.png" class="img-override" />

Note the *Status* of the language data set is *NotStarted*. In a few moments, it will change to *Running*, the *Succeeded*, just like the acoustic data set did.

Congratulations! You've created your first pronunciation data set. We'll be using it in the next chapter.

> **Challenge #3**
>
> I bet you can't guess what this challenge is about... This is a more difficult challenge, probably. That's because you don't really know about your problem domain yet. Typically, you add pronunciation data sets once you know more about your problem domain that you're trying to train for. But, if you think you can add something to what we already have, go for it!

This concludes the exercise. 

<div class="exercise-end"></div> 

### Acoustic Testing data sets

You'll recall earlier in this chapter that there are multiple types of data sets we'll need: training, testing, and real-world. 

So far, you've created 3 training data sets: acoustic, language, and pronunciation. Next, you'll need to create a testing data set.

#### Testing Data Sets are Acoustic Data Sets

Here's a secret - testing data sets for the CSS *are* acoustic data sets. And here's why. Think about it - an acoustic data set provides audio files, with transcriptions of the audio file content. As a result, an acoustic data set is ideal for testing because it includes audio files, and their actual content.

Now, we have to be a bit careful, because it's easy to confuse your training and testing data sets because they are both acoustic data sets. So, as we create a second acoustic data set, we'll be sure to name it properly - with *testing* in it's name. 

<h4 class="exercise-start">
    <b>Exercise</b>: Creating a testing acoustic data set
</h4>

Start by locating the testing files we included in the workshop files. You'll find 6 .wav audio files in the *custom-speech-service-data/testing* folder:

<img src="images/chapter3/files.png" class="img-override" />

Select the 6 audio files, zip them up, and name the zip file *testing-utterances.zip*.

<img src="images/chapter3/testing-utterances.gif" class="img-override" />

Next, navigate to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, and navigate to *Adpatation Data*.

Click the *Import* button by *Acoustic Datasets* and complete the following fields:
- Name: Pokemon - Acoustic Data - Testing
- Description *blank*
- Locale: en-US
- Transcriptions file (.txt): upload the *testing-acoustic-model-data.txt* file
- Audio files (.zip): upload the *testing-utterances.zip* file you created earlier

<img src="images/chapter3/testing2.png" class="img-override" />

Click *Import* to upload the acoustic data and build the data set.

When the data is uploaded, you'll navigate back to the *Acoustic Datasets* page and your data set will be displayed in the grid:

<img src="images/chapter3/testing3.png" class="img-override" />

Note the *Status* of the acoustic dataset is *NotStarted*. In a few moments, it will change to *Running*, then *Succeeded*.

Congratulations! You've created your testing acoustic data set. 

> **Challenge #4**
>
> Yes. Again. Feel free to augment the testing data set you just created. Remember - don't overlap training/testing data, and make the data similar enough. For example, if you added *Charizard* to your training data sets, it would be a good idea to test for *Charizard*. Likewise, if you didn't add another pokemon, like *Chespin*, you shouldn't expect the CSS to magically recognize it.
>
> <img src="images/chapter3/chespin.jpeg" class="img-small" />

This concludes the exercise. 

<div class="exercise-end"></div> 

Phew! That was a long chapter! But, you learned quite a bit, like:
- the importance of separating training data from testing data
- that acoustic data is a combination of .wav files and normalized text transcripts
- pronunciation data sets can help your CSS models interpret multi-word phrases into an abbreviation - like C3PO and AT&T

