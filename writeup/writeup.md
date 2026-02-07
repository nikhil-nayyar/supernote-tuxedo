---
Title: My Document
Summary: A brief description.
Authors: John Smith
         Jane Doe
Date: October 2, 2007
---

## A (hopefully) brief introduction

I like many others believe that I only need to buy one more thing to finally fulfill the desire to become a minimalist.

For me, the one tool (of many that have tried and failed) that comes close to achieving this is my Supernote. It's an e-ink writing tablet. It's more popular competitor is the ReMarkable, but I prefer the Supernote for it's repairability, it's more open ecosystem (it runs a modified form of Android which allows sideloading applications), and it's (supposedly) more permanent ceramic nib, which doesn't need constant replacement like a rubber nib. Though, admittedly, I've only used it for a year straight (not sure when you really need to replace rubber nibs).

[picture of supernote]

Either way, I already do some light personalization on my Supernote to make my life easier. They offer some cloud sync services, their own cloud, which I'm pretty sure is just a wrapper to AWS, Dropbox, etc. As a novice schizophrenic who dabbles in conspiracy theories, I prefer to host my own storage. They did recently unveil the ability to run your own instance of their cloud software, but I sideloaded syncthing which connections to my home server to automatically backup my files. More can be read on that here: TBD.

They also provide a [desktop application](https://support.supernote.com/en_US/Tools-Features/supernote-partner-app-for-desktop) to allow quick cataloguing, reorganizing of your notebooks. However, this app is developed by a partner (not entirely sure what that means but I assume another 3rd party developer with official recognition by Supernote themselves). 

But, here's the key point. Remember that I am a 1) minimalist 2) who aspires to minimalism by buying, or in this case developing yet another item. It should be no surprise then that I'm a Linux user. And I don't want to setup Wine to rune this application...

So, the logical conclusion is I'm going to spend my free time to design an application that allows for the quick reorganization of my notes.

## Inspiration

The work for this project was inspired by the [python supernote-tool](https://github.com/jya-dev/supernote-tool). This is another 3rd party piece of software that converts the properitary `.supernote` files to PNGs from the laptop. I'm going to be doing all of my own reverse engineering, but it's existence provided me some signal (and motivation) to try and figure it out on my own.

The other major inspiration is my own work as an embedded engineer. Recently at work, I developed a bootloader for our PCB to accept new images over USB, specifically as a DFU file. As such, I designing our DFU prefix/suffix schema around the existing raw binary and created some python scripts to automatically generate our DFU files. The main takeaway is that any file is really just binary, and if you know the special recipe to interpret the binary, you have the data.

I'll note the classic engineering cliche that everything is scary until you break it down to its component parts and then it's not scary anymore. I'll also note that we're used to working with binaries that are obfuscated by the black magic that is information theory (jpeg, mp4, etc) or somewhat intense propietary standards (.docx, .pdfs). But, again, the existence of a python tool gave me the confidence to dive into tihs.

[picture of binary schema]

## Reconstructing the File Format

There are plans to eventually release the file format when it's in a more stable state which is understandable. I think their point is that the file format is subject to change so better not to release something as a promise and then have people get mad when things break. Maybe, I'm unleashing a can of worms by trying to build this ( ¯\(ツ)/¯), but I'm also impatient and could really benefit from a tool like this now. 

The first thing I did was open up a supernote file in a hex editor. I'm starting with the most simple file to get a sense of the basic file format: a blank file with one page and one layer. Then, if I get more useful information, I can work my way up from there. 

There are a few important things to note:
- There's important file metadata encoded as plain ASCII. 
- There's a blob of such data at the front of the file. Presumably this is standard for each file.
- There's a blob of such data at the end of the file. I see references to layer and page data so this is presumably not for the EOF. Rather, we'll see multiple blobs as we see more layers and pages respectively.
- The random data between the blobs is the actual raster data for the layer, page, file, etc.

All this to say, creating more "edge case" files will allow use to understand 
    1. file organization -> we can then learn to reorganize layers and pages within a given file or between files
    2. raster data -> we can potentially add the same algorithms to allow editing of files from the desktop.

Ambitious hypotheticals so lets dive in.

The main goal of this project is file reorg, so I'm going to start with that. Here are the files I'm going to create:
- 1 Page / 1 Layer / No Data
- 1 Page / 2 Layers / No Data
- 2 page / 1 layer / No data
- 2 page / 2 layers / no data

- 1 Page / 1 Layer / Dot
- 1 Page / 1 Layer / Line
- 1 Page / 2 Layers / Dot on each layer
- 2 page / 1 layer / dot on each page
- 2 page / 2 layers / don on each layer
