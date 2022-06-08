# Audio Editor Frontend

This is the frontend part of my audio editor.

## What does it do

The audio editor is an app to allow collaborative editing of audio clips, allowing users to upload their own audio files, rearrange the order of play of those files and allow to play/pause the track. The record button is still a WIP, but the idea is to be able to work on a podcast in real time with some friends, or play music together. For that, we would also need to work on a multichannel setup, since right now we only have one channel (visualize as one line, being able to have multiple line is also WIP).

## Start

To access the app, you can go directly to https://sound.xbuss.ca/
There, you can start by clicking GET AUDIO, this will make a request to a Cloudflare Worker who will fetch data from a R2 storage, where I already put some data in. You can also upload your own mp3, you can find some [here](https://en.wikipedia.org/wiki/Wikipedia:List_of_sound_files) or [here](https://freesound.org/).
Then, you can rearrange the track in any order you like. When the tracks are in the order you like, you can press Play to enjoy the fruit of your labor! Enjoy!

## Technologies used
- Vue.js
- JavaScript
- Tailwind CSS
- Cloudflare Workers
- Cloudflare Pages
- Cloudflare R2


## Challenges
This was my first fullstack project, and also my first in frontend. I'm getting quite good in backend, but I had a lot to learn in the frontend space. This took a huge part of the project, but I'm happy I learned it, now I can better understand what kind of data is expected. I also wanted to try Cloudflare's new products, and this project was very helpful.

