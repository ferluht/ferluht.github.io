---
title: "Drawing generative NFT mushrooms with Three.js 🍄"
date: 2022-02-16
image: /assets/imgs/shrooms_main.jpg
layout: post
---

<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.11.0/styles/default.min.css">
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.11.0/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

<p align="center">
<img src="/assets/imgs/shrooms_main.jpg" 
	 alt="Otinium caseubbacula - one of the generative mushroom specimens" width="100%">
<i>Otinium caseubbacula - one of the generative mushroom specimens</i>
</p>

## Part 1: Intro

In this post, I’ll try to give a brief but complete overview of what generative art is, how it is connected to NFTs, and how one can start making generative things on a blockchain. I’ll try to answer all these questions based on my personal experience of making and releasing a NFT collection of generative mushrooms written in javascript.

### Background

I love to code unusual things just for fun. During the New Year holidays, I was spammed so hard by news about NFTs that I finally decided to try to make something creative in this paradigm. I was never excited by the idea of uploading JPEGs onto a blockchain, but the possibility of onchain generative art grabbed my attention. 

Briefly, the idea behind it is to make some token generator that gives you a unique art object each time you “mint” it (in actuality, call a method in the blockchain which spends some of your money on its execution and also gives some money to the artist). ***Definitely, there is some magic in the feeling that your transaction generates a unique object which will be stored into the blockchain forever, isn’t it?*** 

There are some art platforms that exploit this idea, the most famous of them is [artblocks.io](artblocks.io). But as it is has a lot of bureaucracy to enter and also it is built on the Ethereum blockchain, which still uses proof-of-work and has a very high gas price, I decided to try myself on a more democratic, cheap, and eco-friendly platform - [fxhash.xyz](fxhash.xyz)

### What is generative NFT artwork?

All the generative NFTs are basically webpages that draw something on the canvas using either vanilla javascript or some third-party libraries. Taking a stab at classification, from my perspective I’d broadly divide all generative NFTs into 3 categories: abstract math artworks, concrete procedural artworks, and variative hand-drawn artworks. The first class utilizes some mathematical concepts to generate an abstract image: there may be some fractals, attractors, cellular automatons, etc. Procedural arts are trying to describe some concrete things using parametrizations. And the third class is usually simple randomization of some pre-drawn parts of the image.

Also, there are some experimental and interactive works, even [modular synthesizers](https://www.fxhash.xyz/generative/7123) and [games](https://www.fxhash.xyz/generative/8588), but these are much rarer.

<p align="center">
<img src="/assets/imgs/3_types_of_nft.jpg" 
	 alt="Left to right examples of math, procedural and variative artworks. Credits: ciphrd, zancan, littlesilver" width="100%">
<i>Examples of math, procedural and variative artworks. Credits: <a href="https://www.fxhash.xyz/u/ciphrd">ciphrd</a>, <a href="https://www.fxhash.xyz/u/zancan">zancan</a>, <a href="https://www.fxhash.xyz/u/Littlesilver">littlesilver</a></i>
</p>

So what we will do during this article is to describe a procedural model of a mushroom and randomize it using the transaction hash. Combined with an artistic vision, composition, and stylization this gives us what’s called a generative NFT artwork.

## Part 2: drawing a mushroom 🍄 

Ok, let’s end up with all that philosophy and move on to the technical part. This project was made entirely using [Three.js](https://threejs.org/) library, which has a reasonably [simple and well-documented API](https://threejs.org/docs/index.html#manual/en/introduction/Creating-a-scene).

### Stipe

Basically, a stipe can be parametrized as a closed contour extrusion along some spline (let’s call it base spline). To create the base spline I used [CatmullRomCurve3](https://threejs.org/docs/#api/en/extras/curves/CatmullRomCurve3) class from three js. Then, I created the geometry vertex-by-vertex by moving another closed shape along the base spline and finally connected those vertices with faces. I used [BufferGeometry](https://threejs.org/docs/#api/en/core/BufferGeometry) for that purpose.

<details>
<summary><b>Stipe generation code</b></summary>

<pre><code class="javascript">
stipe_vSegments = 30; // vertical resolution
stipe_rSegments = 20; // angular resolution
stipe_points = []; // vertices
stipe_indices = []; // face indices

stipe_shape = new THREE.CatmullRomCurve3( ... , closed=false );

function stipe_radius(a, t) { ... }

for (var t = 0; t < 1; t += 1 / stipe_vSegments) {
  // stipe profile curve
  var curve = new THREE.CatmullRomCurve3( [
    new THREE.Vector3( 0, 0, stipe_radius(0, t)),
    new THREE.Vector3( stipe_radius(Math.PI / 2, t), 0, 0 ),
    new THREE.Vector3( 0, 0, -stipe_radius(Math.PI, t)),
    new THREE.Vector3( -stipe_radius(Math.PI * 1.5, t), 0, 0 ),
  ], closed=true, curveType='catmullrom', tension=0.75);

  var profile_points = curve.getPoints( stipe_rSegments );

  for (var i = 0; i < profile_points.length; i++) {
  	stipe_points.push(profile_points[i].x, profile_points[i].y, profile_points[i].z);
  }
}

// <- here you need to compute indices of faces
// and then create a BufferGeometry
var stipe = new THREE.BufferGeometry();
stipe.setAttribute('position', new THREE.BufferAttribute(new Float32Array(stipe_points), 3));
stipe.setIndex(stipe_indices);
stipe.computeVertexNormals();
</code></pre>
</details>

<br>

<p align="center">
<img src="/assets/imgs/stipe.jpg" 
	 alt="Stages of a stipe generation: spline, vertices, faces" width="100%">
<i>Stages of a stipe generation: spline, vertices, faces</i>
</p>

### Stipe noise

To be more natural, the stipe surface may somehow vary along its height. I defined stipe radius as a function of the angle and relative height of the point on the base spline. Then a slight amount of noise is added to the radius value depending on these parameters.

<details>
<summary><b>Stipe noise code</b></summary>

<pre><code class="javascript">
base_radius = 1; // mean radius
noise_c = 2; // higher this - higher the deformations

// stipe radius as a function of angle and relative position
function stipe_radius(a, t) {
	return base_radius + (1 - t)*(1 + Math.random())*noise_c;
}
</code></pre>
</details>

<br>

<p align="center">
<img src="/assets/imgs/stipe_noise.jpg" 
	 alt="Stipe noise variations" width="100%">
<i>Stipe noise variations</i>
</p>

### Cap

Cap can also be parameterized as a spline (let’s also call it a base spline) rotating around the top of the stipe. Let’s name the surface spawned by this rotation a base surface. Then base surface will be defined as a function of the position of a point on the base spline and the rotation around the stipe top. This parametrization will allow us to gracefully apply some noises to the surface later.

<details>
<summary><b>Cap generation code</b></summary>

<pre><code class="javascript">
cap_rSegments = 30; // radial resolution
cap_cSegments = 20; // angular resolution

cap_points = [];
cap_indices = [];

// cap surface as a function of polar coordinates
function cap_surface(a0, t0) {
  // 1. compute (a,t) from (a0,t0), e.g apply noise
  // 2. compute spline value in t
  // 3. rotate it by angle a around stipe end
  // 4. apply some other noises/transformations
  ...
  return surface_point;
}

// spawn surface vertices with resolution
// cap_rSegments * cap_cSegments
for (var i = 1; i <= cap_rSegments; i++) {
  var t0 = i / cap_rSegments;
  for (var j = 0; j < cap_cSegments; j++) {
    var a0 = Math.PI * 2 / cap_cSegments * j;
    var surface_point = cap_surface(a0, t0);
    cap_points.push(surface_point.x, surface_point.y, surface_point.z);
  }
}

// <- here you need to compute indices of faces
// and then create a BufferGeometry
var cap = new THREE.BufferGeometry();
cap.setAttribute('position', new THREE.BufferAttribute(new Float32Array(cap_points), 3));
cap.setIndex(cap_indices);
cap.computeVertexNormals();
</code></pre>
</details>

<br>

<p align="center">
<img src="/assets/imgs/cap.jpg" 
	 alt="Stages of cap generation: spline, vertices, faces" width="100%">
<i>Stages of cap generation: spline, vertices, faces</i>
</p>

### Cap noise

To be more realistic, the cap also needs some noise. I divided cap noise into 3 components: radial, angular and normal noises. Radial noise affects the relative position of the vertex on the base spline. Angular noise changes the angle of base spline rotation around the top of the stipe. And finally, normal noise changes the position of the vertex along the base surface normal at that point. While defining the cap surface in a polar coordinate system it’s useful to apply 2d [Perlin noise](https://en.wikipedia.org/wiki/Perlin_noise) for these distortions. I used [noisejs](https://github.com/josephg/noisejs) library for that.

<details>
<summary><b>Cap noise code</b></summary>

<pre><code class="javascript">
function radnoise(a, t) {
  return -Math.abs(NOISE.perlin2(t * Math.cos(a), t * Math.sin(a)) * 0.5);
}

function angnoise(a, t) {
  return NOISE.perlin2(t * Math.cos(a), t * Math.sin(a)) * 0.2;
}

function normnoise(a, t) {
  return NOISE.perlin2(t * Math.cos(a), t * Math.sin(a)) * t;
}

function cap_surface(a0, t0) {
  // t0 -> t by adding radial noise
  var t = t0 * (1 + radnoise(a, t0));

  // compute normal vector in t
  var shape_point = cap_shape.getPointAt(t);
  var tangent = cap_shape.getTangentAt(t);
  var norm = new THREE.Vector3(0,0,0);
  const z1 = new THREE.Vector3(0,0,1);
  norm.crossVectors(z1, tangent);

  // a0 -> a by adding angular noise
  var a = angnoise(a0, t);
  var surface_point = new THREE.Vector3(
    Math.cos(a) * shape_point.x,
    shape_point.y,
    Math.sin(a) * shape_point.x
  );

  // normal noise coefficient
  var surfnoise_val = normnoise(a, t);

  // finally a surface point
  surface_point.x += norm.x * Math.cos(a) * surfnoise_val;
  surface_point.y += norm.y * surfnoise_val;
  surface_point.z += norm.x * Math.sin(a) * surfnoise_val;

  return surface_point;
}
</code></pre>
</details>

<br>

<p align="center">
<img src="/assets/imgs/cap_noise.jpg" 
	 alt="Noise components from left to right: radial, angular, normal" width="100%">
<i>Noise components from left to right: radial, angular, normal</i>
</p>

### The rest of the shroom: scales, gills, ring

The geometries of the gills and ring are very similar to the geometry of the cap. An easy way to create scales is to spawn noisy sphere points around some random anchor points on the cap surface and then create [ConvexGeometry](https://threejs.org/docs/#examples/en/geometries/ConvexGeometry) based on them.

<details>
<summary><b>Scales generation code</b></summary>

<pre><code class="javascript">
bufgeoms = [];
scales_num = 20;
n_vertices = 10;
scale_radius = 2;

for (var i = 0; i < scales_num; i++) {
  var scale_points = [];

  // choose a random center of the scale on the cap
  var a = Math.random() * Math.PI * 2;
  var t = Math.random();
  var scale_center = cap_surface(a, t);

  // spawn a random point cloud around the scale_center
  for (var j = 0; j < n_vertices; j++) {
    scale_points.push(new THREE.Vector3(
      scale_center.x + (1 - Math.random() * 2) * scale_radius, 
      scale_center.y + (1 - Math.random() * 2) * scale_radius,
      scale_center.z + (1 - Math.random() * 2) * scale_radius
	);
  }

  // create convex geometry using these points
  var scale_geometry = new THREE.ConvexGeometry( scale_points );
  bufgeoms.push(scale_geometry);
}

// join all these geometries into one BufferGeometry
var scales = THREE.BufferGeometryUtils.mergeBufferGeometries(bufgeoms);
</code></pre>
</details>

<br>

<p align="center">
<img src="/assets/imgs/rest_of_mushroom.jpg" 
	 alt="Scales, gills, ring and the full geometry of a mushroom" width="100%">
<i>Scales, gills, ring and the full geometry of a mushroom</i>
</p>

### Collisions check

To prevent unreal intersections when spawning multiple mushrooms in the scene one needs to check collisions between them. [Here I found a code snippet](https://stackoverflow.com/questions/11473755/how-to-detect-collision-in-three-js) that checks collisions using raycasting from each mesh point. To reduce computation time I generate a low-poly twin of the mushroom along with the mushroom itself. This low-poly model then is used to check collisions with other shrooms.

<details>
<summary><b>Collisions check code</b></summary>

<pre><code class="javascript">
for (var vertexIndex = 0; vertexIndex < Player.geometry.attributes.position.array.length; vertexIndex++)
{       
    var localVertex = new THREE.Vector3().fromBufferAttribute(Player.geometry.attributes.position, vertexIndex).clone();
    var globalVertex = localVertex.applyMatrix4(Player.matrix);
    var directionVector = globalVertex.sub( Player.position );

    var ray = new THREE.Raycaster( Player.position, directionVector.clone().normalize() );
    var collisionResults = ray.intersectObjects( collidableMeshList );
    if ( collisionResults.length > 0 && collisionResults[0].distance < directionVector.length() ) 
    {
        // a collision occurred... do something...
    }
}
</code></pre>
</details>

<br>

<p align="center">
<img src="/assets/imgs/collisions.jpg" 
	 alt="Simplified models for faster collision check" width="100%">
<i>Simplified models for faster collision check</i>
</p>

### Rendering and stylization

Initially, I wanted to achieve an effect of 2d-drawing despite all the generation being made in 3d. The first thing that comes to mind in the context of stylization is the outline effect. I’m not a pro in shaders so I just took the outline effect [from this example](https://stemkoski.github.io/Three.js/Outline.html). Using it I got a nice pencil style of the shroom contour.

<p align="center">
<img src="/assets/imgs/outline.jpg" 
	 alt="Three js outline effect" width="100%">
<i>Three js outline effect</i>
</p>

The next thing on the way back to 2d is proper colorization. The texture should be a bit noisy and have some soft shadows. There is a lazy hack for those who, like me, don't want to deal with UV-maps. Instead of generating a real texture and wrapping it using UV one can define vertex colors of an object using [BufferGeometry](https://threejs.org/docs/#api/en/core/BufferGeometry) API. More than that, using this approach the color of a vertex can be also parameterized as a function of angle and position, so generation of a noisy procedural texture becomes slightly easier.

<p align="center">
<img src="/assets/imgs/color.jpg" 
	 alt="Adding some vertex color" width="100%">
<i>Adding some vertex color</i>
</p>

Finally, I added some global noise and film-like grain using [EffectComposer](https://threejs.org/docs/#examples/en/postprocessing/EffectComposer).


<details>
<summary><b>Effect chain code</b></summary>
<pre><code class="javascript">
var renderer = new THREE.WebGLRenderer({antialias: true});
outline = new THREE.OutlineEffect( renderer , {thickness: 0.01, alpha: 1, defaultColor: [0.1, 0.1, 0.1]});
var composer = new THREE.EffectComposer(outline);

// <- create scene and camera

var renderPass = new THREE.RenderPass( scene, camera );
composer.addPass( renderPass );

var filmPass = new THREE.FilmPass(
  0.20,   // noise intensity
  0.025,  // scanline intensity
  648,    // scanline count
  false,  // grayscale
);

composer.addPass(filmPass);
composer.render();
</code></pre>
</details>

<br>

<p align="center">
<img src="/assets/imgs/almost_ready.jpg" 
	 alt="Almost ready, colored and noisy shrooms" width="100%">
<i>Almost ready, colored and noisy shrooms</i>
</p>

### Name generation

For name generation I used a simple [Markov chain](https://en.wikipedia.org/wiki/Markov_chain) which was trained on 1k mushroom names [from here](https://www.prevalentfungi.org/fungi.cfm). To preprocess and tokenize those names I used the python library [YouTokenToMe](https://github.com/VKCOM/YouTokenToMe). With it, I split all names into 200 unique tokens and wrote their transition probabilities to a javascript dictionary. The JS side of the code only reads those probabilities and stacks tokens until it generates a couple of words. 

<details>
<summary><b>Here are some samples of mushroom names generated using this approach</b></summary>
Stricosphaete cinus, Fusarium sium confsisomyc, Etiformansum poonic, Hellatatum bataticola, Armillanata gossypina mortic, Chosporium anniiffact, Fla po sporthrina
</details>

<br>

## Part 3: Finalizing

<p align="center">
<img src="/assets/imgs/all_fungi.jpg" 
	 alt="The first 15 shrooms minted on fxhash" width="100%">
<i>The first 15 shrooms minted on fxhash</i>
</p>

### Preparing for the drop

To prepare a project for a release on fxhash one simply needs to change all random calls in the code to the fxrand() method as [described here](https://github.com/fxhash/fxhash-simple-boilerplate). The main idea is that your code must generate unique outputs for each hash but exactly the same output for the same hash. Then test the token in the sandbox and finally mint it when the minting will be opened. That’s it! 

### Drop!

This brings us to the Mushroom Atlas (what I named this collection). [You can check it out and see its variations here](https://www.fxhash.xyz/generative/9202). Although it was not sold out like some of my previous works, I think that this is the most advanced and challenging thing that I’ve made in generative art yet. Hope that those who minted this token also enjoyed their fungi in the non-fungible world!

