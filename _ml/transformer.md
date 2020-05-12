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
img: /assets/img/transformer2.jpg
wikipedia-url: https://en.wikipedia.org/wiki/Transformer_(machine_learning_model)
---

### Intro

Jay Alammar put together [an excellent explanation](http://jalammar.github.io/illustrated-transformer/) of the workings of the Transformer. This article is great--it goes into <i>graphic</i> detail, following with the life cycle of a single input unto its output in another language. Make sure to watch the animated gif's of time-lapse of the decoder.

Transformers have subsequently been used in other domains like [vision](https://arxiv.org/abs/1502.03044):

<img src="/assets/img/xu2015-fig6b.png" width=400/>

### The Diagram

I wanted to add my two cents to what has already been written. Personally, I benefit most from very detailed, complete diagrams, which is one reason I feel that the [original paper](https://arxiv.org/abs/1706.03762) is a bit dense. Even The illustrated transformer takes 30 minutes to get through despite it being one of the clearest explanations I have found. So, I put this diagram together to illucidate what is going on, but in a single picture, digestible as quickly as you can look through a single picture. Hopefully you too find it helpful.

<img src="/assets/img/transformer.jpg" width=900/>

N.B. You will need a bit of background, though, e.g. read some of the original paper, know about sequence to sequence and word embeddings, etc. to get what is going on.

Like in the paper, input starts in the lower left. A sentence of length n is first word-embedded to a matrix of size n x 512. Next, it computes the output of the encoder phase. Then, The decoder phase begins and builds up an encoded memory of its outputs, Y (also it outputs words after each decoder pass.

Aren't all-in-one diagrams great?

### Update 1:
There is a possibly simpler/better way to feed outputs back into the decoder than I have drawn. In the drawing I have the raw outputs being re-encoded, which is essentially computing
<br/>
<br/>
p( y(t) | x(1), x(2), ..., x(n), y(1), y(2), ..., y(t-1) )
<br/>
<br/>
This puts an unnecessary dependence on the output. In addition, if we build Y from outputs only, we lose alot of hidden state information. While this approach could be used to remove a dependency from hidden-to-hidden states and allow for faster training via teacher forcing, it was not the intention. For this reason, it would be wiser to feed the final encoded state rather than the output back into the Y matrix. This would remove the dependency on the outputs:
<br/>
<br/>
p( y(t) | x(1), x(2), ..., x(n) )
<br/>
<br/>
Having said that, neither the Illustrated Transformer nor the Annotated Transformer do this: both are auto-regressive w.r.t. the output. I am not sure why this is the case. Possibly since the transformer itself uses self-attention, then previous hidden embedding contains no more useful information than the outputs. Or possibly since the same infrastructure is used to encode/embed the input words, it makes sense to use it in the same manner for the decoder.

### Update 2:

[The Annotated Transformer](https://nlp.seas.harvard.edu/2018/04/03/attention.html), published by Harvard, is another excellent, detailed explanation of the inner working of the Transformer. This notebook includes explanation, drawings, and working code, weaved into a single, flowing dialogue. Very nice!

However, I still prefer my single drawing :)
