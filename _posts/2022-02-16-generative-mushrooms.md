---
title: "Drawing generative NFT mushrooms with Three.js 🍄"
date: 2022-02-16
layout: post
---
<p align="center">
<img src="./assets/imgs/shrooms_main.jpg" alt="Otinium caseubbacula - one of the generative mushroom specimens" width="50%">
</p>

### Part 1: Intro

In this post, I’ll try to give a brief but complete overview of what generative art is, how it is connected to NFTs, and how one can start making generative things on a blockchain. I’ll try to answer all these questions based on my personal experience of making and releasing a NFT collection of generative mushrooms written in javascript.

## Background

I love to code unusual things just for fun. During the New Year holidays, I was spammed so hard by news about NFTs that I finally decided to try to make something creative in this paradigm. I was never excited by the idea of uploading JPEGs onto a blockchain, but the possibility of onchain generative art grabbed my attention. 

Briefly, the idea behind it is to make some token generator that gives you a unique art object each time you “mint” it (in actuality, call a method in the blockchain which spends some of your money on its execution and also gives some money to the artist). Definitely, there is some magic in the feeling that your transaction generates a unique object which will be stored into the blockchain forever, isn’t it? 
There are some art platforms that exploit this idea, the most famous of them is artblocks.io. But as it is has a lot of bureaucracy to enter and also it is built on the Ethereum blockchain, which still uses proof-of-work and has a very high gas price, I decided to try myself on a more democratic, cheap, and eco-friendly platform - fxhash.xyz

## What is generative NFT artwork?

All the generative NFTs are basically webpages that draw something on the canvas using either vanilla javascript or some third-party libraries. Taking a stab at classification, from my perspective I’d broadly divide all generative NFTs into 3 categories: abstract math artworks, concrete procedural artworks, and variative hand-drawn artworks. The first class utilizes some mathematical concepts to generate an abstract image: there may be some fractals, attractors, cellular automatons, etc. Procedural arts are trying to describe some concrete things using parametrizations. And the third class is usually simple randomization of some pre-drawn parts of the image.

Also, there are some experimental and interactive works, even modular synthesizers and games, but these are much rarer.

Left to right examples of math, procedural and variative artworks. Credits: ciphrd, zancan, littlesilver
So what we will do during this article is to describe a procedural model of a mushroom and randomize it using the transaction hash. Combined with an artistic vision, composition, and stylization this gives us what’s called a generative NFT artwork.

### Part 2: Drawing a mushroom 🍄 

