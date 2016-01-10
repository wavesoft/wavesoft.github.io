---
layout: post
cover: false
subclass: 'post tag-speeches'
navigation: True
disqus: True
title:  "CSS Only Web Goodies"
date:   2015-12-10 18:00:31 +0100
categories: html css
logo: 'assets/images/wavesoft.png'
cover: 'assets/images/cover9.jpg'
---

We are a few days from Christmas and at CERN we are preparing for our christmas issues of various posts and websites. What a better way to celebrate with some nice animations! (And what a nice opportunity to explore CSS and it's powerful new features!)

I liked how our websites came out, and therefore I decided to share with you my code. If you want to jump-in directly to the examples, click here : [http://wavesoft.github.io/web-goodies](http://wavesoft.github.io/web-goodies)

## Christmas Presents

Christmas is all about presets! And since for our first special issue, we wanted to show a different story for each one of the contributors, what a better way to show them as in a christmas present!

Our original idea was to have a box with a number, counting down to christmas. Each box will have a letter inside, that will be shown to the user when the mouse is moved over.

<img src="/assets/images/story-boxes.png" alt="Story Boxes" />

Having a look on the latest CSS3 3D transformations, I figured out that the simplest we could do is to form a cube with 3D transformations. This means creating one face for each side, plus one face for the card.

However, when I had a quick look, I realised that it looks better if I have another texture from within the box. That said, too many faces needs to be created... time for javascript!

The following function takes care of creating our faces:

```javascript
    ...
    // Iterate over sides and faces
    var sides = ['f','b'];
    var faces = ['fr','bk','rt','lt', 'bt'];
    // First over sides...
    for (s=0;s < sides.length;s++){
        // Then over faces..
        for (f=0;f < faces.length;f++){
            var face = document.createElement('figure');
            face.className = sides[s] + ' ' + faces[f];
            cube.appendChild(face);
            // Also create a number item on the front face
            if ((s==0) && (f==0) && (faceText!=null)) {
                var num = document.createElement('div');
                num.className = 'num';
                num.innerHTML = faceText;
                face.appendChild(num);
            }
        }
    }
```

With a bit of a CSS magic that I am not going to go into details right now ([but you are free to browse it yourself](https://github.com/wavesoft/web-goodies/blob/master/christmas-boxes/res/christmas-cubes.less)) the christmas magic comes to life!

Even though we are using javascript to set-up our scene, the animation and transitions are pure CSS.

You can have a look on the first example [here](http://wavesoft.github.io/web-goodies/christmas-boxes/).

## Snowflakes

For our other issue, we wanted to have a nice background animation with some winter theme. However, again I wanted to avoid adding any javascript logic. 

The solution came this time using a simple CSS keyframe animation over a predefined path. Initially, I thought that something like this might look quite bad, however if you use many snowflakes and you use a different starting time for each one, you can have a very nice, winter-y animation!

You can have a look on the second example [here](http://wavesoft.github.io/web-goodies/let-it-snow/).




