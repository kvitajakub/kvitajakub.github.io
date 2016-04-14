---
layout:     post
date:       2016-04-14 06:00:00
author:     "Jakub Kvita"
comments:   true
title:      "Visualizations of RNN units"
subtitle:   "Diagrams of RNN unrolling, LSTM and GRU."
---

I am currently writing my thesis and, as a part of my text, I created some very nice vector diagrams I would like to share. To give credit, I was inspired by Christopher Olah's [Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/) post, but I wanted to make vector images of those, as PNG diagrams don't look very professional in PDFs. If you are not sure about the topic of this post, please visit the mentioned Christopher's page to get the basic knowledge about RNNs and LSTMs.

First of all, I created a visualization of RNN unrolling ([svg file]({{ site.baseurl }}/img/rnn-unrolled.svg)).
<div style="width:70%; margin-left:auto; margin-right:auto; margin-bottom:5px; margin-top:17px;">
    <img src="{{ site.baseurl }}/img/rnn-unrolled.png" alt="RNN Unrolling image">
<span class="caption text-muted">RNN as a neural network very deep in time.</span>
</div>

Then, LSTM with peephole connections ([svg file]({{ site.baseurl }}/img/lstm-peepholes.svg)). Peepholes can be easily removed.
<div style="width:60%; margin-left:auto; margin-right:auto; margin-bottom:5px; margin-top:17px;">
    <img src="{{ site.baseurl }}/img/lstm-peepholes.png" alt="A LSTM image">
<span class="caption text-muted">LSTM with peephole connections.</span>
</div>

And finally GRU ([svg file]({{ site.baseurl }}/img/gru.svg)).
<div style="width:60%; margin-left:auto; margin-right:auto; margin-bottom:5px; margin-top:17px;">
    <img src="{{ site.baseurl }}/img/gru.png" alt="A GRU image">
<span class="caption text-muted">Gated recurrent unit.</span>
</div>

I spent some time creating those diagrams, so feel free to save time and reuse them. If you will, send me a message, I will be glad somebody found this useful.
