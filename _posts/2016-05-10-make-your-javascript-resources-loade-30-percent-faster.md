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

In my [last article](/javascript-binary-bundles) I wrote about my idea on how bundling javascript resources can save space and decrease the loading time. I have been playing with this idea for quite some time and today I decided to put it down for some serious testing.

Skipping the long technical part, [jbb](https://github.com/wavesoft/jbb) is a serialisation format for javascript structures. It was originally designed to be a replacement to the dozens of different 3D model formats out there, trying to outperform -or par with- them in loading time. 

So, trying to be fair, I decided to load meshes from a variety of mesh formats, encode them to `jbb` along with their resources (ex. images) and compare the size and loading time.

## Picking source files

I browsed through the [models in the three.js examples](https://github.com/mrdoob/three.js/tree/master/examples) and I picked a few candidates for the `jbb` encoder. I looked for the most common file formats and for already optimised file formats in order to make the competition harder. That said, I collected a few and grouped them in 5 different bundles:

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
            <ul>
                <li><code>house.wrl</code></li>
            </ul>
        </td>
        <td>
            VRML is a simple, yet frequently encountered format for 3D models. It should be dead simple for jbb to beat it.
        </td>
    </tr>
    <tr>
        <td><code>animated.jbbsrc</code></td>
        <td>532K</td>
        <td>
            <ul>
                <li><code>flamingo.js</code></li>
                <li><code>horse.js</code></li>
                <li><code>monster</code></li>
                <li><code>monster.jpg</code></li>
                <li><code>monster.js</code></li>
            </ul>
        </td>
        <td>
            These are some animated models with different complexity. They are encoded as THREE.js JSON objects, which is quite fast to parse.
        </td>
    </tr>
    <tr>
        <td><code>obj.jbbsrc</code></td>
        <td>548K</td>
        <td>
            <ul>
                <li><code>WaltHead_bin.bin</code></li>
                <li><code>WaltHead_bin.js</code></li>
            </ul>
        </td>
        <td>
            This is a quite big mesh file, in the widely used Wavefont .obj binary format. Looks like a challenging competitor.
        </td>
    </tr>
    <tr>
        <td><code>heavy.jbbsrcd</code></td>
        <td>3.3M</td>
        <td>
            <ul>
                <li><code>ben.js</code></li>
                <li><code>ben.utf8</code></li>
                <li><code>ben_dds.js</code></li>
                <li><code>dds/James_Body_Lores.dds</code></li>
                <li><code>dds/James_Eye_Green.dds</code></li>
                <li><code>dds/James_Eye_Inner_Green.dds</code></li>
                <li><code>dds/James_EyeLashBotTran.dds</code></li>
                <li><code>dds/James_EyeLashTopTran.dds</code></li>
                <li><code>dds/James_Face_Color_Hair_Lores.dds</code></li>
                <li><code>dds/James_Mouth_Gum_Lores.dds</code></li>
                <li><code>dds/James_Tongue_Lores.dds</code></li>
                <li><code>dds/MCasShoe1TEX_Lores.dds</code></li>
                <li><code>dds/MJeans1TEX_Lores.dds</code></li>
                <li><code>dds/MTshirt3TEX_Lores.dds</code></li>
                <li><code>dds/Nail_Hand_01_Lores.dds</code></li>
                <li><code>hand.jpg</code></li>
                <li><code>hand.js</code></li>
                <li><code>hand.utf8</code></li>
            </ul>
        </td>
        <td>
            The <code>ben.utf8</code> and <code>hand.utf8</code> are two havey meshes encoded in UTF8 format. This is another well-optimised format for storing high-density meshes and another difficult competitor for jbb.
        </td>
    </tr>
    <tr>
        <td><code>md2.jbbsrc</code></td>
        <td>4.0M</td>
        <td>
            <ul>
                <li><code>ratamahatta.md2</code></li>
                <li><code>w_bfg.md2</code></li>
                <li><code>w_blaster.md2</code></li>
                <li><code>w_chaingun.MD2</code></li>
                <li><code>w_glauncher.md2</code></li>
                <li><code>w_hyperblaster.md2</code></li>
                <li><code>w_machinegun.md2</code></li>
                <li><code>w_railgun.md2</code></li>
                <li><code>w_rlauncher.md2</code></li>
                <li><code>w_shotgun.md2</code></li>
                <li><code>w_sshotgun.md2</code></li>
                <li><code>weapon.md2</code></li>
                <li><code>skins/ctf_b.png</code></li>
                <li><code>skins/ctf_r.png</code></li>
                <li><code>skins/dead.png</code></li>
                <li><code>skins/gearwhore.png</code></li>
                <li><code>skins/ratamahatta.png</code></li>
                <li><code>skins/w_bfg.png</code></li>
                <li><code>skins/w_blaster.png</code></li>
                <li><code>skins/w_chaingun.png</code></li>
                <li><code>skins/w_glauncher.png</code></li>
                <li><code>skins/w_hyperblaster.png</code></li>
                <li><code>skins/w_machinegun.png</code></li>
                <li><code>skins/w_railgun.png</code></li>
                <li><code>skins/w_rlauncher.png</code></li>
                <li><code>skins/w_shotgun.png</code></li>
                <li><code>skins/w_sshotgun.png</code></li>
                <li><code>skins/weapon.png</code></li>
                <li><code>grasslight-big.jpg</code></li>
            </ul>
        </td>
        <td>
            This format is used by Quake II to encode animated models. It's a quite optimised file format, both for size and loading time. This also looks like another good competitor.
        </td>
    </tr>
</table>

## Size comparison

I used the JBB compiler to load the meshes using their individual loader and to encode them in a `.jbb` bundle. Since a jbb archive is quite sparse, it can easily benefit from a gzip compression. 

To be fair, I also decided to compress the source bundles in order to have a reference value.

Putting them in a table we see the following:

<table>
    <tr>
        <th>Bundle</th>
        <th>.jbbsrc Size</th>
        <th>.jbb Size</th>
        <th>Gzipped .jbbsrc Size</th>
        <th>Gzipped .jbb Size</th>
    </tr>
    <tr>
        <td><code>md2.jbbsrc</code></td>
        <td>4.0M</td>
        <td>4.4M</td>
        <th>3.8M</th>
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
        <th>320K</th>
        <th>480K</th>
    </th>
    <tr>
        <td><code>animated.jbbsrc</code></td>
        <td>532K</td>
        <td>404K</td>
        <th>248K</th>
        <th>268K</th>
    </th>
    <tr>
        <td><code>vrml.jbbsrc</code></td>
        <td>88K</td>
        <td>76K</td>
        <th>12K</th>
        <th>12K</th>
    </th>
</table>

<img src="/assets/images/jbb-plot-size.png" alt="Size comparison" />

Ok, so we don't have a clear winner. If we compare it against the compressed source bundle, `jbb` can even be bigger in be some cases. This means that if you just enable gzip compression in your webserver you are as good as `jbb`.

So let's have a look at the parsing time. 

## Speed comparison

I created a [mocha test file](https://github.com/wavesoft/jbb/blob/master/test/test-three.js) that executes the following steps:

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

Here we see something more interesting. Even though in one case `jbb` loads slightly slower than it's competitor, overall the average loading time of the `jbb` bundle is **30% to 35% faster**.

## A real-world example

Since in a real-world scenario the actual loading time depends both on the data being transferred and the parsing speed, I decided to create a [benchmarking website](https://cdn.rawgit.com/wavesoft/jbb-tests/7d71eb26378860d4059097f3ee6d41fb0940299b/jbb_test_timing.html) to put the `jbb` solution in a real test.

I started the measurements from my local installation. As expected, everything seems perfect, since there is no network penalty:

<img src="/assets/images/jbb-timing-local.png" alt="Local Timing" />

The moment however we switch to a hosted server, we start to see the real performance of the system:

<img src="/assets/images/jbb-timing-rawgit-https.png" alt="Rawgit Timing" />

Surprisingly enough, even with one bad case, the average loading time with `jbb` remains **around 30% better**.

## Conclusions

The JBB project has matured during the past year, and it can now [even integrate with your gulp](https://github.com/wavesoft/gulp-jbb) pipeline, bringing it one step closer to your project.

Even though new features are still planned, you can already benefit from JBB's performance and usability! You can start with the [JBB and THREE.js tutorial](https://github.com/wavesoft/jbb/blob/master/doc/Tutorial%201%20-%20THREEjs/Using%20with%20THREEjs.md).

I am looking forward to hearing from your experience with jbb.

Github: [https://github.com/wavesoft/jbb](https://github.com/wavesoft/jbb)
