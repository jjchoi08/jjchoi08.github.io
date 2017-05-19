---
layout: single
title: The Mural Networks- Street Art Recognition App
---

For my final project at Metis, I wanted to utilize that I learned for the final lecture- Deep Learning. So I first reproduced cat & dog image recognition model with Convolusional Neural Networks using Keras as a practice. Keras is a super useful Python wrapper for Tensorflow and Theano and it was perfect for beginner in Deep Learning like me. 


You can learn more about the cat & dog recognition Kaggle competition and its state-or-art algorithms from winners <a href="https://www.kaggle.com/c/dogs-vs-cats-redux-kernels-edition"> here </a>.

I wanted to utilize cat & dog recognition technique into something that interests me or something useful that has business value. Also, I wanted to create something unique that's related to New York City. Since I am a huge fan of street art, I decided to create an app that recognizes patterns of street art and predicts the artists. Street artists have very unique style in their artworks. They use repetitive pattern or color so the viewers can easily determine the artist. So I thought this could be an interesting classification problem where I could utilize computer vision using CNN. Sound cool enough?

My first step was gathering the image data. I follow number of street artists on Instagram so I decided to scrape images from their account. There's this really cool tool called instagram-scraper where you could just enter the user name and it saves all of the user's media into your machine. I will not disclose the names of the street artists.

You can find my presentation <a href="https://github.com/jjchoi08/DSProj/blob/master/proj_kojac.pdf"> HERE </a>.

So with 5000+ images, I was able to create something that works. It was definitely tough. Image recognition isn't something that I can master in few weeks. However, I was able to pull if off and create a model that predicts few number of artists (thanks to GPU unit on AWS). 

The next step was a web app. My ultimate goal was to create and iOS App and test it on the street. But as an initial product, I created a Python Flask App that does the job and named it The Mural Networks (Mural + Neural Networks). Here is a demo video that I used for my final presentation. 

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/J6RCjJNd1mU)

It only runs in my local machine yet. However, I am planning to fine-tune it after learning more about Deep Learning. Stay tuned for a test version!

Lastly, here is a tweet from Metis from my presentation day.

[![IMAGE ALT TEXT HERE]