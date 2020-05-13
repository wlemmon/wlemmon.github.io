---
object-id: transformer
title: Transformers
category: ML
class: 
school: 
featured: false
featured-priority:
listing-priority: 4
blurb: A Drawing is All You Need.
img: /assets/img/transformer/transformer2.jpg
wikipedia-url: https://en.wikipedia.org/wiki/Transformer_(machine_learning_model)
---

### Intro

Jay Alammar put together [an excellent explanation](http://jalammar.github.io/illustrated-transformer/) of the workings of the Transformer. This article is great--it goes into <i>graphic</i> detail, following with the life cycle of a single input unto its output in another language. Make sure to watch the animated gif's of time-lapse of the decoder.

Transformers have subsequently been used in other domains like [vision](https://arxiv.org/abs/1502.03044):

<img src="/assets/img/transformer/xu2015-fig6b.png" width="400"/>

### The Diagram

I wanted to add my two cents to what has already been written. Personally, I benefit most from very detailed, complete diagrams, which is one reason I feel that the [original paper](https://arxiv.org/abs/1706.03762) is a bit dense. Even The illustrated transformer takes 30 minutes to get through despite it being one of the clearest explanations I have found. So, I put this diagram together to illucidate what is going on, but in a single picture, digestible as quickly as you can look through a single picture. Hopefully you too find it helpful.

<img src="/assets/img/transformer/transformer.jpg" width="900"/>

N.B. You will need a bit of background, though, e.g. read some of the original paper, know about sequence to sequence and word embeddings, etc. to get what is going on.

Like in the paper, input starts in the lower left. A sentence of length n is first word-embedded to a matrix of size n x 512. Next, it computes the output of the encoder phase. Then, The decoder phase begins and builds up an encoded memory of its outputs, Y (also it outputs words after each decoder pass.

Aren't all-in-one diagrams great?

### Q&A:
* Why pass the emission word back into the decoder instead of the output embedding <i>used</i> to create the emission?
> By passing the emission word <i>y</i>, the transformer models the following:
> <pre>
> p( y(t) | x(1), x(2), ..., x(n), y(1), y(2), ..., y(t-1) )
> </pre>
> And this is what we want. If we only pass the output embedding, it models this instead:
> <pre>
> p( y(t) | x(1), x(2), ..., x(n) )
> </pre>
> In the first case, the transformer creates a sentence that is consistent with the previous words we have chosen. The model does not choose the words--it only outputs a probability distribution over words for each timestep with softmax. From this a maximum likelihood (or weighted probability) word is chosen. Without the previous words, the output at each time would be unintelligible.
* Could the output embedding also be passed back into the Y input during decoding, like with RNNs?
> RNNs h(t) are conditioned on both h(t-1) and y(t-1) because baked into the model is the conditional independence assumption / markov stability that y(t) can be fully explained by y(t-1) and h(t-1):
> <pre>
> p( y(t) | x(1), x(2), ... x(t), y(1), y(2), ..., y(t-1)) 
    = p( y(t) | x(1), x(2), ... x(t), h(t-1), y(t-1))
    = p( y(t) | h(t-1), y(t-1)) (in the case with encoder/decoders.
> </pre>
> <img src="/assets/img/transformer/rnn.jpg" width="500px"/>
>
> (portion of RNN computation graph. Deep Learning (Ian J. Goodfellow, Yoshua Bengio and Aaron Courville), MIT Press, 2016.)
>
> <img src="/assets/img/transformer/encoderdecoder.jpg" width="500px"/>
>
> (encoder/decoder computation graph. Deep Learning (Ian J. Goodfellow, Yoshua Bengio and Aaron Courville), MIT Press, 2016.)
>
> RNNs need the h(t-1) since it summarizes the previous outputs. Its the feeding back in of y(t) that allows us to keep a running history of the current sentence being output.
>
> Transformer probabilities look more like this:
> <pre>
> p( y(t) | x(1), x(2), ... x(t), y(1), y(2), ..., y(t-1))
> </pre>
> There is no conditional independence state folding. Transformers still have a fixed-size weight matrix which can be thought of as replacing the hidden state, and thus are also lossy, but transformers do not constrain the model with a markov conditional independence simplification.
>
> Feeding h(t-1) back into the transformer's decoder might not hurt. Having said that, neither the Illustrated Transformer nor the Annotated Transformer do this: both are auto-regressive w.r.t. the outputs only. Since the transformer itself uses self-attention, then previous hidden embedding likely contains no more useful information than the output sequence, which is likely the main reason it is not done. Also, possibly since the same infrastructure is used to encode/embed the input words, it makes sense to use it in the same manner for the decoder.

### Update:

[The Annotated Transformer](https://nlp.seas.harvard.edu/2018/04/03/attention.html), published by Harvard, is another excellent, detailed explanation of the inner working of the Transformer. This notebook includes explanation, drawings, and working code, weaved into a single, flowing dialogue. Very nice!

However, I still prefer my single drawing :)
