---
title: MultiModal Machine Learning
author: Edgar
math: true
---
# The Power of Data

I'm sure I don't need to convince any reader of this about the unreasonable effectivness of data (cite?). 
The more good quality data you have, the better your discriminator is at classifying or the richer the examples
your generative model is able to provide. Yet for as good as a model may get, we are fundamentally limited by the
types of data we use to train a model. 

Let's do a little thought experiment, let's say I'm training a text classifer that is given a sentence, and returns
whether or not the person saying that sentence meant for the sentence to be sarcastic. So for example, our input this
time was `Wow, great food`. It's kind of hard to tell with certainty whether or not this was being sarcastic. If we had
a model trained on characters, maybe the capitlization or punctation might give us some clues (e.g `WOW! great food`),
but even then it's hard to tell. Maybe if we were given some more context: `This place sure is interesting. Wow, great food`.
This might help in general, but in this particular case it's still exceedingly hard to tell. So what do we do? If we have
a lot of text, maybe we can tell if this particular phrase has some different characteristics to other phrases in the 
document. In general though, this example doesn't have a  concrete way to determine sarcasticity just from the text. In fact
I'm willing to wager this exact problem (infering sarcasm from text) can't be solved reliably like this. When humans communicate
over text, we often have misunderstanding about the intentions behind text, so we tend to take a guess and ascribe a certain
feeling or tone when we're not sure what the real intent was.  So lets ammend the problem and the model a bit. 

Let's say that our training data isn't just text, it's text and a waveform of a person reading out the text. For the sake of 
argument, let's assume our training data is well segmented, that we have the same person reading all the text, and that the
text is a transcrption of the person actually saying the text. Now, when we recieve as input `Wow, great food`, we can also 
hear the way it was said. Depending where the emphasis was, the inflection, the pitch, and the puases of the speaker, we have
significantly more information to base our prediction off of! We may not be perfect, after all there exist people who speak 
in a flat, uniform way or whose speaking mannerisims are independant of their current sentiment, but in general having 
the actual speech would help a lot. We don't have to stop here though, if we could also use the persons facial expressions as 
they're speaking, this would make the problem even easier! Herein lies the promise of multiple modalities, different modalities 
may be able to represent different things that may not captured in other modalities. The more modalities we can intake, the better
a guess or the better representation we can generate about something. 

## Working with Multiple Modalities

This is where things starting getting a bit trickier. I can always say "Adding in more good quality is a good thing", but this ignores
the fact that actually dealing with multiple different types of data is difficult. How do you correctly combine images and text? There 
are more than a few reasonable ways to do it, but there are always trade-offs. Can you fit both your data and your model in memory,
do you process them the inputs jointly, or seperatly and then combine them later, is adding another modality even worth it or would
it be a lot more compute for marginal performance gains? It's case by case, but there are some general patterns than can help.

### Encoders
One of the more straightforward ways to handle multiple modalities at once it just have an encoder for each seperate modality and 
to have some way of combining them. Lets say for the sake of arguments that I have an encoder model $g_e()$ that generates an embedding for the input type e. For text, this might look something like an LLM, a Bi-LSTM, or even someting as simple as a combination of word vectors. Let's just assume we have some way to turn the input into some embedding $e_t \in \mathbb{R}^{n}$ .
So for our text imput, we have some encoder $g_t(): \text{Our text representation} \mapsto \mathbb{R}^{n}$.   
Similarly, if our other modality is images we want some encoder $g_i(): \text{Our image representation} \mapsto \mathbb{R}^{m}$, where $g_i()$ can be some model like a CNN for example.
So now we have the embeddings for each of the input modalities, so what do we do? We ostensibly want a single global embedding $$e_g \in \mathbb{R}^{g}$$ that we can 
use as an input to the model that solves the task(s) we care about, but how do we generate $e_g$? Let's state this a little more formally, we want some 
combing function $f()$ that takes as input each invidiual modality we have, and outputs a single global embedding we can mess with $e_g$. So the 
question now becomes, what the heck should $f()$ be?

#### A concatenation
One of the more straightforward ways to implement $f()$ would simply be to concatenate all of the input embeddings generating $e_g \in \mathbb{R}^{n+m}$. This 
is a fairly easy solution to implement and trivially lets us keep all of the information from each of the encoders.  

#### Limitations
There are a few limitations to this though, namely that each of the encodings is generated independatly and we don't have any explicit relationship between our different modalities. Consider the case where we have some text for our previous sarcasm task: `This place is so great`. Given the waveform for this phrase, which word(s) are the most important for us to consider when determing whether or not the speaker was being sarcastic? We probably care more about the latter half of this phrase, but our waveform encoder doesn't know what the text is at all, it's just encoding the given waveform. It might be a good idea to let the encoders pass information to each so that the waveform model can attend to differnt parts of the waveform conditioned on the given text for example. There are a lot of considerations to make for this (how do we pass information? Do we chain the encoders in a certain order or do we let information pass between all of them simultaneously? Etc.), but if done correctly this might let you make even richer embeddings than just having independant encoders who don't have any context of the other modalities while generating the embeddings. 

<!-- #### An Operation
You don't necessarily need to keep all of the embedding dimensions to generate the global embedding, we can multiply by some binary selection matrices, impose some sort of $L_1$ penatly over the carried weights, or even take the sum over all the input embeddings (taking care to reshape/broadcast as necessary) to generate a global embedding.  
-->

### Something Else
While having an encoder per each input modality allows you some granularity with respect to the types of models you use and the inductive biases of those models, we don't necessicarily need discrete encoders for each input modality. If we had a model that is general enough to work on both image and text, we can just concatenate our inputs and pass in a combined text + image input into our model and generate the final global embedding directly. Using models that operate [directly on the bytes of a file](https://huggingface.co/papers/2306.00238) for example would allow you a lot of freedom with respect to what type of modalities you accept without having to worry too much about how to treat and later combine each of the individual modalities. Having all the modalities input at once may even allow the model to recognize how differnt parts of the input affect what aspects of the other modalties should be attended to, assuming the model is able to handle all the data and context at once. 

## Using the embedding

Once we have the global embedding, it becomes a standard ML problem. We have a feature vector $e_g$, and some task we want to use it for so we can just use our standard methods for answering our problem, except that know we osentibly have more information about what we're trying to predict! 

## The types of things we can do

One of the more common multi modal settings is in image captioning. Given an image then generate a caption, or given a caption generate a possible image seem like natural tasks to propose but they can be fairly difficult. This isn't the only tasks these are used for though, I did some work early on multi modal methods for medical data. Consider having access to patients blood labs, ekg data, and doctors notes for each patient. Looking at each modality individually may let you do some tasks well, but given how complicated the human body can be looking at only one aspect of data usually gives you an incomplete view of the patient. Consider for example that we are trying to predict what infection a patient has contracted. If we have only blood lab data, it may be obvious that the patient has an infection, but the specific infection they have may be difficult if not impossible to predict from just the blood labs. Likewise if we have the patients medical history on file, we can see what infections they may previously have had and maybe if they have a predisposition to a certain type of infection, but if we don't have access to current blood labs, we won't know that the patient is currently suffering form an infection. This is a bit of a contrived example, but the processing of differnt types of modalites for certain tasks has potential to greatly increase what our models are able to do. 