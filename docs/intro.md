---
sidebar_position: 1
---
# Intro
:::warning
**This documentation is provided AS IS and without any warranty; anything that you do to your car is FULLY YOUR RESPONSIBILITY, please DO NOT blame me for any (highly unlikely) damage you cause to your car**

This documentation was fully researched, and written, by a single person, that is me. This documentation may have some slight inaccuracies, and this documentation may not be fully complete as I didn't have too much time to try out sending every message to my car, and had to guess some bits to write for the documentation.

Everybody ***can***, and is ***encouraged*** to clone this documentation's repository, modify any inaccurate bits, or add missing information, and create a pull request which will be reviewed and likely merged.

This documentation's repository can be found on the navigation bar, or [here](https://github.com/ArchGryphon9362/teslabtapi)
:::

:::note
Using most of the following information requires knowledge in the following areas:
- How cryptography works
- End to end encryption
- How to store private keys securely
- How to use BluetoothLE
- How to use Google's protocol buffers
:::

## Get started

First of all, you'll need the proto file which you can grab from [Github](https://gist.github.com/ArchGryphon9362/fc55736f5bb5f2b19c12f68abe0ed3f7).
Next generate the source/header files for your respective language which you can find out how to do [here](https://developers.google.com/protocol-buffers/docs/overview#generating).

Now, to begin, go to the [getting started](start) section.

If you want to contribute just message me on Discord (ArchGryphon9362#6132), or submit a PR/issue.

## Some backstory

I began this project many months ago as a way to be able to open my car's frunk very quickly, and without using the Tesla app. My initial idea was to unlock it using the Tesla web API, but that wasn't very consistent, slow, and you had to manage tokens all the time. Then I came up with this idea, to unlock it using bluetooth.

I was very clueless on where to start back then, but I ended up snooping the bluetooth messages coming to and from the app. That didn't get me very far, after which I just forgot about this project. But very recently (somewhere around beginning of Jun/2021), I needed to do something, and I was researching reverse engineering for it.

Then I was like... wait, do java decompilers exist since it's such a high-level language (I later on realised that Java basically has its own bytecode, so it was essentially machine code of its own, but it's still more high level and keeps some/many class names which proved to be very useful)? And next thing you know, I came across JadX which is an android decompiler.

After 3 weeks of deobfuscation work and starting this attempt over when I realised that what I was reverse engineering was an autogenerated protobuf, I learned how to extract a protobuf file from any app, and here I had, what can generate most of the code to control the car.

All I had to do was figure out the other puzzle pieces, until today, 25th July 2021, when I unlocked the car with my own key for the first time! So... here is... the documentation!
