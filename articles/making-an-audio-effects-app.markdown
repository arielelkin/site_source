---
layout: page
title: "making_an_audio_effects_app"
date: 2014-03-20 00:37
comments: true
sharing: true
footer: true
---


##Intro

In this tutorial we'll explore a technique for making iOS apps that manipulate sound on-the-fly.

 by combining two fantastic open-source libraries: The Amazing Audio Engine and the Synthesis Toolkit in C++. 

A good musical instrument needs to offer the user a substantial amount of control over its sound. Some apps play readymade audio files, other apps generate the audio from scratch. We'll use this second approach: our app will generate and control its own sounds.

We'll need a *synthesizer* to generate the samples of audio according to user input, and an *audio engine* that will deliver the audio to the user. We'll use The Amazing Audio Engine as our audio engine and a class from the Synthesis Toolkit in C++ as our synthesizer. 

Our final result will be a handheld audio augmenter.

##First steps

1. Open up Xcode and create a new project.
1. In "Choose a template for your new project":
    1. Under the heading "iOS", pick **Application**
    1. Choose **Single View Application**
    1. Click **Next**
1. In "Choose options for your new project":
    1. For "Product Name", enter **Mandolin**
    1. Under "Devices", select **iPhone**.
1. Click **Next** and save the project somewhere dear to you.

##Add the Amazing Audio Engine
1. Add The Amazing Audio Engine to your project. 
    1. [Visit this page](http://theamazingaudioengine.com/doc/_getting-_started.html)
    1. Follow steps 1-6. Follow them to the letter. 
1. Open **AppDelegate.m**, under `#import "AppDelegate.h"` write

```objective-c
#import "AEAudioController.h"
```

Let’s see if you’ve correctly added TAAE. Build (⌘+B), make sure the build succeeds. If the build fails, you haven’t followed the above steps to the letter, go back and make sure you do. If the build succeeds, you get to move on.


##Our audio plumbing

We'll be generating our Mandolin sound within a *channel*, and we'll connect this channel directly to the *audio engine*. The audio engine gathers the sound it receives from all the channels connected to it, and sends it to the headphones or speaker. 

Let's now set up our app's *audio engine*. Our audio engine will be the thing responsible for managing audio sources and outputs, and communicating with the OS. In TAAE, `AEAudioController` is the class that wraps the audio engine. We will place it inside our app delegate, for a couple of reasons:

  * The app delegate should handle launch-time initialization of our core app components, and those include our audio engine. 
  * Given that the app delegate manages transitions to and from the background, it should deal with the audio engine during those transitions. For example, it should suspend audio rendering when our app is sent to the background. 