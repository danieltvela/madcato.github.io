---
layout:     post
title:      "Intuition about AI models"
date:       2021-08-23 14:56:00
author:     "Daniel Vela"
og_image: "/img/post-bg-07.jpg"
locale:       en
lang-ref:   intuition-about-ia-models
---

It is easy to notice that the AI/ML models behave like regression functions: coming from a group of samples, a function is calculated that tends to represent all those samples, minimizing the error.

<img style = "margin-left: auto; margin-right: auto;" src = "https://upload.wikimedia.org/wikipedia/commons/5/53/Linear_least_squares_example2.png" alt = "drawing" width = "250" />

Given a set of samples *{x, y}*, the model learns to return results similar to what it has learned. For each *x*, its corresponding *y* is calculated. **It's a basic indexation system**.

In recent years, advances in AI algorithms, –as well as improvements in hardware capacity–, have allowed us to apply these regression techniques, not only to discrete numerical values, but also to other types of more complex, such as images, texts, audios, any digital medium.

Examples of these uses are the **GPT-2**, **BERT**, to generate texts; **DeepSpeech** for voice recognition; **DALL•E** to generate images from texts; and many more.

<img style = "margin-left: auto; margin-right: auto;" src = "https://upload.wikimedia.org/wikipedia/commons/a/a3/DALL-E_sample.png" width = "150" />

There are another type of AI models in which the task, rather than generating data, is about performing a **classification**: either the detection of objects or the recognition of the feeling of a text, as an example.

In these "classifying models", we can intuit that, what the model does internally is to find the characteristics that differentiate one input from another. **But, what do the generating algorithms have inside the model?**

Some studies have been dedicated to trying to represent what is inside these models: what is hidden among the millions of parameters that make up a model? In these studies, it has been found that among the nodes of the model are the images and texts with which it was trained, all mixed up.

<img style = "margin-left: auto; margin-right: auto;" src = "https://upload.wikimedia.org/wikipedia/commons/0/00/Multi-Layer_Neural_Network-Vector-Blank.svg" width = "150" />

Remembering that these models are just regressions of sophisticated functions, **how is it possible that from an input, a complete object arises?** Intuition makes us think that these **models act as an addressing system similar to indexing systems**, like those inside databases, but where **the files that store the indexing information are also the files that store that data**. That is, it is like a **database with a self-indexed storage format**. Interesting.

This makes me wonder if it would be possible to create generative models from algorithms typical of database systems and other more traditional systems within computational systems. Or better yet, could we mix ML algorithms with other algorithms and computer structures? I don't know how to answer this question yet, but I think it is key to be able to develop intelligent systems that allow us to take advantage of 100% of the information systems: **AI interacting files, processors, networks, is the key to developing intelligent agents**.
