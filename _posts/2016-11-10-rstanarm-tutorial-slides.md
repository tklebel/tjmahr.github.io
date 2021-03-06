---
title: "Slides from my RStanARM tutorial"
excerpt: "Trial by :fire:"
tags:
  - rstanarm
  - bayesian
---

Back in September, I gave a tutorial on [RStanARM](https://cran.rstudio.com/web/packages/rstanarm/) to the [Madison R user's group](https://www.meetup.com/MadR-Madison-R-Programming-UseRs-Group/). As I did for [my magrittr tutorial](https://github.com/tjmahr/MadR_Pipelines), I broke the content down into slide decks. They were:

- [How I got into Bayesian statistics](http://rpubs.com/tjmahr/rep-crisis)
- [Some intuition-building about Bayes theorem](http://rpubs.com/tjmahr/bayes-theorem)
- [Tour of RStanARM](http://rpubs.com/tjmahr/rstanarm-tour)
- [Where to learn more about Bayesian statistics](http://rpubs.com/tjmahr/bayes-learn-more)

The source code and supporting materials are [on Github](https://github.com/tjmahr/MadR_RStanARM).


Observations (training data) 
-------------------------------------------------------------------------------

The intuition-building section was the most challenging and rewarding, because I
had to brush up on Bayesian statistics well enough to informally, hand-wavily 
teach about it to a crowd of R users. Like, I have good sense of how to fit 
these models and interpret them in practice, but there's a gulf between 
understanding something and teaching about it. It was a bit of trial by fire
:fire:.

One thing I did was work through a toy Bayesian updating demo. What's the mean of
some IQ scores, assuming a standard deviation of 15 and a flat prior over a 
reasonable range values? Cue some plots of how the distribution of probabilities
update as new data is observed.

<img src="/figs/2016-11-10-rstanarm-tutorial-slides/iq-00-data-1.png" title="A frame of my Bayesian updating animation" alt="A frame of my Bayesian updating animation" width="50%" /><img src="/figs/2016-11-10-rstanarm-tutorial-slides/iq-01-data-1.png" title="A frame of my Bayesian updating animation" alt="A frame of my Bayesian updating animation" width="50%" /><img src="/figs/2016-11-10-rstanarm-tutorial-slides/iq-02-data-1.png" title="A frame of my Bayesian updating animation" alt="A frame of my Bayesian updating animation" width="50%" /><img src="/figs/2016-11-10-rstanarm-tutorial-slides/iq-03-data-1.png" title="A frame of my Bayesian updating animation" alt="A frame of my Bayesian updating animation" width="50%" />

_See how the beliefs are updated? See how we retain uncertainty around that most
likely value?_ And so on.

Naturally, [I animated the thing](/figs/2016-11-10-rstanarm-tutorial-slides/simple-updating.gif)---I'll take any excuse to use [gganimate](https://github.com/dgrtwo/gganimate). 

Someone asked a good question about what advantages these models have over 
classical ones. I find the models more intuitive[^1], because posterior
probabilities are post-data probabilities. I also find them more flexible. For
example, I can use a _t_-distribution for my error terms---thick tails! If I
write the thing in Stan, I can incorporate measurement error into the model. If
I put my head down and work really hard, I could even fit one of [those gorgeous
Gaussian process 
models](https://matthewdharris.com/2016/05/16/gaussian-process-hyperparameter-estimation/).
We can fit vanilla regression models or get really, really fancy, but it all
kind of emerges nicely from the general framework of writing out priors and a
likelihood definition.


[^1]: But I was taught the classical models first... I sometimes think that these models are only more intuitive because this is my second bite at the apple. This learning came more easily because the first time I learned regression, I was a total novice and had to learn everything. I had learn to about _t_-test, reductions in variance, collinearity, and what interactions do. Here, I can build off of that prior learning. Maybe if I learn everything again---as what? everything as a neural network?---it will be even more intuitive. 
