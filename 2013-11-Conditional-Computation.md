

Conditional Computation Ideas
==============================

```
from:     Christopher Olah <christopherolah.co@gmail.com>
to:       Yoshua Bengio <yoshua.bengio@gmail.com>
date:     Sun, Nov 24, 2013 at 5:51 PM
subject:  Conditional Computation Ideas
```
</br>

Dear Yoshua,

I was reading your paper "Deep Learning of Representations: Looking
Forward" (which I've quite enjoyed, so far), and was inspired by the
"conditional computation" section (a term I had not previously been
familiar with) to write out some thoughts I had in this direction. (As
I'm not familiar with the relevant literature, its fairly likely that
these ideas aren't novel or don't work for some obvious reason. But I
couldn't find them on a first search!)

As we will be meeting in a little over 12 hours, there's probably
little reason for you to read this. But I figured I'd send it,
anyways. Again, just writing with an intended reader (resulting in me
explaining things more clearly) is valuable.

**Introduction**

Pretraining can be thought of us a form of surgery on neural networks.
For example, when we pretrain with an autoencoder, we can think of it
as: train an autoencoder, lop off the second half, and sew a new top
on. The successes of pretraining suggest it might be useful to
consider other types of surgery we can perform on neural networks.

(We are fortunate that surgery on artificial neural networks seems to
be much less dangerous and morally hazardous than surgery on
biological neural networks.)

In this email, I would like to explore some ideas for surgeries that
could be performed on a deep convolutional neural network trained on
ImageNet classification (as in Krizhevsky et al (2012)'s work). The
goal of these surgeries will be to produce networks that can produce
significantly better results without massively increasing the
computational cost of the network through conditionally compute
features.

In pretraining, the objective of the surgery is to inject good
features into the lower layers. Here, our objective will be to use the
original network to acquire a conditional structure for a new network,
constructing conditionally computed feature-roles which are only
computed in a particular set of scenarios we believe should share
useful features. For example, if we could extract a feature
corresponding to looking at a picture of a tree, we could train
certain features specifically to help us differentiate between species
of trees, that are only run when we look at a tree -- one imagines we
might get features differentiating between different shades of greens
and browns at low levels and, at higher levels, particular leaf
patterns, or bark textures. Of course, we may as well also use the
features we get from the original network, and they can help protect
us from over-fitting because they were trained on a much larger
dataset.

In these models, we do not have to train anything to decide whether or
not to gate a particular part of our network. Instead, we will train
features to be useful in particular circumstances we believe are
especially meaningful based on a previous model.

**Feature-Cluster Decision Trees**

One of the most striking parts of the Krizhesky paper, to me, was the
section where they look at images that have the smallest Euclidean
distance between their most abstract representations (pg. 8). It's
really amazing just how similar, to my human judgment, these images
are.

One could imagine applying a clustering algorithm to this
representation. One imagines it would give you clusters of images that
are similar in a very high level way, at a resolution defined by the
hyper-parameters for your clustering algorithm. Let's imagine we split
it into 10 clusters.

Now we have divided our dataset into 10 subsets which we have reason
to believe are likely to benefit in classification from different
features.

For each cluster, we train a new network, perhaps identical to the
original one, only on that cluster of data. Each layer of this new
network receives the activations of the previous layer, and also of
it's twin in the original network. This allows it to focus on learning
new features that are specific to the task at hand, instead of general
use features.

And, if we like, we can repeat again, by clustering the high level
representations of this new network. For each one, we train a new
network.

This forms a decision tree. To evaluate an example, we run it through
a network, but instead of classifying it there, we just decide which
network should get it next, and create some useful features. And
again, and again, until we reach the bottom of the tree.

Likely, you won't want to actually use the same network each level of
the tree. As the amount of data available for the network to train on
shrinks, this would leave you at horrible risk of over-fitting.
Instead, you probably want to reduce the number of new features with
each step down the tree. You may also want to distribute them more on
higher level features, which are probably more likely to benefit from
specialization.

This approach has very nice computational properties. Once you finish
training the first network, you can (in parallel) compute the features
and cluster for all your data in advance, and then train each new
network completely independently without any computational cost from
running the original network. After a few steps down the tree, your
work is trivially, massively parallel.

(An interesting interpretation of this: we are "zooming in" on
sections of image space, by progressively adding local features to
allow us to differentiate within clusters of similar images, spreading
them out.)

**Convolutional Activity Specialization**

In the previous section, we constructed features customized to
specific classes of images. But there's a great deal of variety within
a given image as to what features are useful in a given region. Trying
to detect hyena spots in the middle of a tree just isn't helpful, even
if there is a hyena standing beneath it. And often, significant
portions of the image will be so boring there isn't anything to be
gained from analyzing them in any depth at all.

The premise of our approach is that the activation of high-level
features in a deep convolutional architecture corresponds to
particular squares in the input image being "interesting" and that
which features activate allow us to differentiate between different
kinds of interestingness. For any given kind of interestingness, we
expect different features to be useful in understanding it...

The construction is pretty natural. For each high-level feature in the
original network, we create a number of convolutional features, on
each layer, that are only computed when the high-level feature fires.
They receive as inputs the lower-level specialized features and the
generic features computed by the original network.

The result should be features specialized to differentiate within
particular types of image regions, such as the leaves of a tree or the
fur of an animal, that only get run in regions that appear to be of
that nature at a first glance.

This technique could be used as the basis of a coarse to fine model,
helping scale deep convolutional networks to higher resolution images
by focusing computation on interesting regions of the image. If the
original network was trained on low resolution images, it could still
express information about the content of regions of a higher
resolution image, which one could then investigate in more detail.


**Unsupervised Variations**

There isn't anything about these models that is very architecture
specific. They don't even need to be supervised! I just originally
formulated the thoughts in terms of the Krizhevsky et al paper, and it
was much easier to explain with a specific network in mind.

In principle, the first model could be applied to any deep
autoencoder, and the second to any deep convolutional autoencoder.

Chris

</br>
</br>
</br>


Re: Conditional Computation Ideas
---------------------------------

```
from:     Yoshua Bengio <yoshua.bengio@gmail.com>
to:       Christopher Olah <christopherolah.co@gmail.com>
date:     Sun, Nov 24, 2013 at 7:39 PM
subject:  Re: Conditional Computation Ideas
```
</br>

> Dear Yoshua,
>
> I was reading your paper "Deep Learning of Representations: Looking
> Forward" (which I've quite enjoyed, so far), and was inspired by the
> "conditional computation" section (a term I had not previously been
> familiar with)

Yes, I made it up (it exists in another context, programming languages).

> to write out some thoughts I had in this direction. (As
> I'm not familiar with the relevant literature, its fairly likely that
> these ideas aren't novel or don't work for some obvious reason. But I
> couldn't find them on a first search!)

It’s one of the most active areas of research in the lab currently, and large
companies like Google, Microsoft or Facebook are very interested in this.

> In this email, I would like to explore some ideas for surgeries that
> could be performed on a deep convolutional neural network trained on
> ImageNet classification (as in Krizhevsky et al (2012)'s work). The
> goal of these surgeries will be to produce networks that can produce
> significantly better results without massively increasing the
> computational cost of the network through conditionally compute
> features.

Sounds like an interesting plan.

>
> In pretraining, the objective of the surgery is to inject good
> features into the lower layers. Here, our objective will be to use the
> original network to acquire a conditional structure for a new network,
> constructing conditionally computed feature-roles which are only
> computed in a particular set of scenarios we believe should share
> useful features. For example, if we could extract a feature
> corresponding to looking at a picture of a tree, we could train
> certain features specifically to help us differentiate between species
> of trees, that are only run when we look at a tree -- one imagines we
> might get features differentiating between different shades of greens
> and browns at low levels and, at higher levels, particular leaf
> patterns, or bark textures. Of course, we may as well also use the
> features we get from the original network, and they can help protect
> us from over-fitting because they were trained on a much larger
> dataset.

Yes, that is how we see it too.

>
> In these models, we do not have to train anything to decide whether or
> not to gate a particular part of our network. Instead, we will train
> features to be useful in particular circumstances we believe are
> especially meaningful based on a previous model.

Isn’t that the same?

>
> **Feature-Cluster Decision Trees**
>
> One of the most striking parts of the Krizhesky paper, to me, was the
> section where they look at images that have the smallest Euclidean
> distance between their most abstract representations (pg. 8). It's
> really amazing just how similar, to my human judgment, these images
> are.
>
> One could imagine applying a clustering algorithm to this
> representation. One imagines it would give you clusters of images that
> are similar in a very high level way, at a resolution defined by the
> hyper-parameters for your clustering algorithm. Let's imagine we split
> it into 10 clusters.

Ok.

>
> Now we have divided our dataset into 10 subsets which we have reason
> to believe are likely to benefit in classification from different
> features.

Yes. We have been exploring something related but different. I’ll tell you tomorrow.

>
> For each cluster, we train a new network, perhaps identical to the
> original one, only on that cluster of data. Each layer of this new
> network receives the activations of the previous layer, and also of
> it's twin in the original network. This allows it to focus on learning
> new features that are specific to the task at hand, instead of general
> use features.

Interesting idea.

>
> And, if we like, we can repeat again, by clustering the high level
> representations of this new network. For each one, we train a new
> network.

Cool.

>
> This forms a decision tree. To evaluate an example, we run it through
> a network, but instead of classifying it there, we just decide which
> network should get it next, and create some useful features. And
> again, and again, until we reach the bottom of the tree.

We have been exploring other kinds of tree-structured networks,
but I like your approach better.

>
> Likely, you won't want to actually use the same network each level of
> the tree. As the amount of data available for the network to train on
> shrinks, this would leave you at horrible risk of over-fitting.
> Instead, you probably want to reduce the number of new features with
> each step down the tree. You may also want to distribute them more on
> higher level features, which are probably more likely to benefit from
> specialization.

Yeah, it’s plausible.

>
> This approach has very nice computational properties. Once you finish
> training the first network, you can (in parallel) compute the features
> and cluster for all your data in advance, and then train each new
> network completely independently without any computational cost from
> running the original network. After a few steps down the tree, your
> work is trivially, massively parallel.

Indeed.

>
> (An interesting interpretation of this: we are "zooming in" on
> sections of image space, by progressively adding local features to
> allow us to differentiate within clusters of similar images, spreading
> them out.)

I really like your idea.

This seems reasonable, although the actual computational gain
needs to be evaluated.

>
> **Unsupervised Variations**
>
> There isn't anything about these models that is very architecture
> specific. They don't even need to be supervised! I just originally
> formulated the thoughts in terms of the Krizhevsky et al paper, and it
> was much easier to explain with a specific network in mind.
>
> In principle, the first model could be applied to any deep
> autoencoder, and the second to any deep convolutional auto encoder.

Right.

See you tomorrow,

— Yoshua


