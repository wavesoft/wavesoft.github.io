---
layout: post
cover: false
subclass: 'post tag-speeches'
navigation: True
title:  "CSS Only Web Goodies"
date:   2015-12-10 18:00:31 +0100
categories: html css
logo: 'assets/images/wavesoft.png'
cover: 'assets/images/cover9.jpg'
---

We are a few days from Christmas and at CERN we are preparing for our christmas issues of various posts and websites. What a better way to celebrate with some nice animations! (And what a nice opportunity to explore CSS and it's powerful new features!)

I liked how our websites came out, and therefore I decided to share with you my code. But let's have a quick look.

## Christmas Presents

Christmas is all about presets! And since for our first special issue, we wanted to show a different story for each one of the contributors, what a better way to show them as in a christmas present!

Our original idea was to have a box with a number, counting down to christmas. Each box will have a letter inside, that will be shown to the user when the mouse is moved over.

<img src="/assets/images/story-boxes.png" alt="Story Boxes" />

Having a look on the latest CSS3 3D transformations, I figured out that the simplest we could do is to form a 3D box with 

This test demonstrates the use of CSS3 and 3D transitions in order to create any number of present-like boxes. Each box, when hovered, will pop a card from inside.

Each box is individually placed in the container using a javascript function.

Get them on github: [http://wavesoft.github.io/web-goodies](http://wavesoft.github.io/web-goodies)
