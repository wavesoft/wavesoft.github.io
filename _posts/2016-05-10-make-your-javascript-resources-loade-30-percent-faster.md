---
layout: post
cover: false
subclass: 'post tag-speeches'
navigation: True
disqus: True
title:  "Making your javascript resources load 30% faster"
date:   2016-05-10 18:48:01 +0100
categories: javascript three.js 3d jbb
logo: 'assets/images/wavesoft.png'
cover: 'assets/images/cover13.jpg'
---

In my [last article](/javascript-binary-bundles) I wrote about my idea on how bundling javascript resources can save space and decrease the loading time. I have been working with this idea for quite some time and today I decided to put it down for some serious testing.

Skipping the long technical part, [jbb](https://github.com/wavesoft/jbb) is a serialisation format for javascript structures. It was originally designed to be a replacement to the dozens of different 3D model formats out there, trying to outperform - or par with - them in loading time. The operation principle is simple: You load your resources with node.js in the same way you do in the browser and you pack the them in a bundle. This will also bundle referred DOM elements, such as images, videos, sounds or scripts, making this format ideal for any kind of run-time resources. 

For my tests, I decided to focus on the 3D models since they are the most demanding. I selected a variety of meshes in different formats, I encoded them to `jbb` and I compared the bundle size and loading speed. I was pleased to find out that in most of the cases, `.jbb` can **speed-up the loading time up to 30%!**

## Picking source files

I browsed through the models in the [three.js examples](https://github.com/mrdoob/three.js/tree/master/examples) and I hand-picked a few candidates for the `jbb` encoder. I looked for the most wide-used file formats and for already optimised file formats in order to make the competition harder.

I then grouped them together and created five different [source bundles](https://github.com/wavesoft/jbb#creating-a-bundle), that can be tested individually. Each source bundle consists of an index file and the source model files in their original format.

The following table illustrates the bundles and their contents. Click on the number of files for more details:

<table>
    <tr>
        <th>Bundle</th>
        <th>Size</th>
        <th>Files</th>
        <th>Reason</th>
    </tr>
    <tr>
        <td><code>vrml.jbbsrc</code></td>
        <td>88K</td>
        <td>
            <a href="https://github.com/wavesoft/jbb-tests/tree/master/bundles/vrml.jbbsrc" target="_blank"> 2 </a>
        </td>
        <td>
            VRML is a simple, yet frequently encountered format for 3D models. It should be dead simple for jbb to beat it.
        </td>
    </tr>
    <tr>
        <td><code>animated.jbbsrc</code></td>
        <td>532K</td>
        <td>
            <a href="https://github.com/wavesoft/jbb-tests/tree/master/bundles/animated.jbbsrc" target="_blank"> 6 </a>
        </td>
        <td>
            These are some animated models with different complexity. They are encoded as THREE.js JSON objects, which is quite fast to parse.
        </td>
    </tr>
    <tr>
        <td><code>obj.jbbsrc</code></td>
        <td>548K</td>
        <td>
            <a href="https://github.com/wavesoft/jbb-tests/tree/master/bundles/obj.jbbsrc" target="_blank"> 3 </a>
        </td>
        <td>
            This is a quite big mesh file, in the widely used Wavefont .obj binary format. Looks like a challenging competitor.
        </td>
    </tr>
    <tr>
        <td><code>heavy.jbbsrcd</code></td>
        <td>3.3M</td>
        <td>
            <a href="https://github.com/wavesoft/jbb-tests/tree/master/bundles/heavy.jbbsrc" target="_blank"> 19 </a>
        </td>
        <td>
            The <code>ben.utf8</code> and <code>hand.utf8</code> are two havey meshes encoded in UTF8 format. This is another well-optimised format for storing high-density meshes and another difficult competitor for jbb.
        </td>
    </tr>
    <tr>
        <td><code>md2.jbbsrc</code></td>
        <td>4.0M</td>
        <td>
            <a href="https://github.com/wavesoft/jbb-tests/tree/master/bundles/md2.jbbsrc" target="_blank"> 30 </a>
        </td>
        <td>
            This format is used by Quake II to encode animated models. It's a quite optimised file format, both for size and loading time. This also looks like another good competitor.
        </td>
    </tr>
</table>

## Size comparison

I used the JBB compiler to load the meshes using their individual loader and to encode them in a `.jbb` bundle. In addition, I compressed the bundle, since a jbb archive is quite sparse, and it can easily benefit from it. I used GZip compression at maximum level, since we can get it for free from the browser. 

To be fair, I also decided to compress the source bundles in order to have a reference value. I compressed each file individually, since this is how the would be served.

The following table summarises the results:

<table>
    <tr>
        <th>Bundle</th>
        <th>.jbbsrc Size</th>
        <th>.jbb Size</th>
        <th>GZipped .jbbsrc Size</th>
        <th>GZipped .jbb Size</th>
    </tr>
    <tr>
        <td><code>md2.jbbsrc</code></td>
        <td>4.0M</td>
        <td>4.4M</td>
        <th>3.9M</th>
        <th>3.8M</th>
    </th>
    <tr>
        <td><code>heavy.jbbsrcd</code></td>
        <td>3.3M</td>
        <td>2.2M</td>
        <th>2.0M</th>
        <th>1.4M</th>
    </th>
    <tr>
        <td><code>obj.jbbsrc</code></td>
        <td>548K</td>
        <td>736K</td>
        <th>328K</th>
        <th>480K</th>
    </th>
    <tr>
        <td><code>animated.jbbsrc</code></td>
        <td>532K</td>
        <td>404K</td>
        <th>260K</th>
        <th>268K</th>
    </th>
    <tr>
        <td><code>vrml.jbbsrc</code></td>
        <td>88K</td>
        <td>76K</td>
        <th>16K</th>
        <th>12K</th>
    </th>
</table>

<img src="/assets/images/jbb-plot-size.png" alt="Size comparison" />

We can definitely see that the compressed version of `.jbb` is always smaller than the source files, however that's not always the case when comparing to the compressed version of them.

At least we can see that `.jbb` is not introducing any penalty for the next tests. So let's have a look at the parsing time next.

## Speed comparison

I created a [mocha test file](https://github.com/wavesoft/jbb/blob/master/test/test-three.js) that executes the following steps for every bundle:

 * Load the source bundle
 * Encode the javascript structures to a `jbb` bundle
 * Load the `jbb` bundle 
 * Compare the loading times

The following table illustrates the different loading times:

<table>
    <tr>
        <th>Bundle</th>
        <th>.jbbsrc Time</th>
        <th>.jbb Time</th>
    </tr>
    <tr>
        <td><code>md2.jbbsrc</code></td>
        <td>147ms</td>
        <td>103ms</td>
    </th>
    <tr>
        <td><code>heavy.jbbsrcd</code></td>
        <td>185ms</td>
        <td>39ms</td>
    </th>
    <tr>
        <td><code>obj.jbbsrc</code></td>
        <td>86ms</td>
        <td>116ms</td>
    </th>
    <tr>
        <td><code>animated.jbbsrc</code></td>
        <td>122ms</td>
        <td>39ms</td>
    </th>
    <tr>
        <td><code>vrml.jbbsrc</code></td>
        <td>178ms</td>
        <td>155ms</td>
    </th>
</table>

<img src="/assets/images/jbb-plot-speed.png" alt="Speed comparison" />

Here we see something more interesting. Even though in one case `jbb` loads slightly slower than it's competitor, the overall average loading time of the `jbb` bundle is **30% to 35% faster**.

Not to get excited yet, since this is an ideal scenario on my development machine. Let's see how it behaves in the real world...

## A real-world example

Since in a real-world scenario the actual loading time depends both on the data being transferred and the parsing speed, I decided to create a [benchmarking website](https://cdn.rawgit.com/wavesoft/jbb-tests/7d71eb26378860d4059097f3ee6d41fb0940299b/jbb_test_timing.html) to put the `jbb` solution in a real test.

I started the measurements from my local installation. As expected, everything seems perfect, since there is no network penalty:

<img src="/assets/images/jbb-timing-local.png" alt="Local Timing" />

The moment however we switch to a hosted server, we start to see the real performance of the system:

<img src="/assets/images/jbb-timing-rawgit-https.png" alt="Rawgit Timing" />

Surprisingly enough, even with one bad case, the average loading time with `jbb` remains **around 30% better**.

## Conclusions

I have intentionally picked some corner case for the `.jbb` format, but I might have missed some other real-world scenarios. Nonetheless, with what I observed so far, it looks that `.jbb` is a quite promising bundling format.

The JBB project has matured during the past year, and it can now [even integrate with your gulp](https://github.com/wavesoft/gulp-jbb) pipeline, bringing it one step closer to your project.

Even though more optimisations and new features are still under development, you can already benefit from JBB's performance and bundling convenience! If you are curious, you can start with the [JBB and THREE.js tutorial](https://github.com/wavesoft/jbb/blob/master/doc/Tutorial%201%20-%20THREEjs/Using%20with%20THREEjs.md).

I am looking forward to hearing from your experience with jbb. And of course your complaints and objections, since this is a live project!

Github: [https://github.com/wavesoft/jbb](https://github.com/wavesoft/jbb)
