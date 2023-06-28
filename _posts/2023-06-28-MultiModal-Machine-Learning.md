---
title: MultiModal Machine Learning
author: emarro
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

TODO:
    1. How to encode each modality
    2. How to combine the encodings
    3. What if we want a generative model

