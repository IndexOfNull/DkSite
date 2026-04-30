---
title: "ValvanoWare"
date: 2024-04-30T12:00:00-05:00
draft: true
tags: []
icon: "/images/projects/valvanoware/icon.gif"
---

# Introduction

ValvanoWare was my entry for the UT ECE 319K final lab, which was also a game competition. This game ultimately won the competition superfinals, so I have the pleasure of saying that I (along with my lab partner, Randy Aguilar) created the fan-favorite 319K game for the semester! Essentially, the task was to use an MSPM0G3507 LaunchPad development kit to create a game with digital and analog inputs, graphics, and sound. Essentially, we had to make our own handheld video game!

In case you're curious about the name: "ValvanoWare" is a double entendre. The first meaning refers to the name of the starter code package we were given: "ValvanoWare". The second meaning is a reference to the WarioWare game series, which was an inspiration for the game. 

# The Game

The premise of the game is simple: the evil Dr. Valvano (the head professor of the course, whom I do not actually believe is evil) has trapped you and will only let you go if you complete his evil 319K assignments. Each "assignment" is a mini-game that you play in about 10 seconds. If you win the mini-game, you win points and make the evil Dr. Valvano happy. If you lose, he gets angry with you and you lose a life.

As for the minigames themselves, most of them were humorous in some regard and frequently made inside jokes. For instance, the shaving minigame involves shaving my professor (Dr. Holt's) mustache: something that actually happened after he lost a bet that his class would not make the best class average on one of the exams (naturally, I loved his class).

Aside from the shaving minigame, there are three minigames: update CCS, find the pin, and kill the MSPM0. Update CCS (CCS was the IDE we used for the class) is essentially a button masher, where you have to fill up a progress bar in time. Find the pin has you using a slide potentiometer and buttons to find a certain pin on the LaunchPad dev kit. Finally, kill the MSPM0 has you playing a pong-like game where you destroy the dev kit as you bounce the ball off of it.

# The Engine

Being an embedded systems class, the entire game was written with C. In order to make building the game easier, I decided to make a basic engine that abstracted away certain functionality like inputs, sounds, graphics, and localization.

Essentially, the engine always has a currently active "scene". The engine will send events (e.g., a button press event) to a callback function inside the scene such that the scene can react accordingly. Additionally, the engine provides a mechanism for scenes to request a scene-change, allowing for game progression. Notably, scenes are effectively written like smaller subprograms with the engine's facilities at its disposal. This means that each scene can implement complex logic while not needing to directly interface with input devices, for example. Using this kind of architecture also made collaboration easier; my lab partner was able to quickly pick up and use the tools provided by the engine to make minigames on his own.

More broadly, the game engine runs three tasks simultaneously. First, the main game task is primarily responsible for performing draw calls, SD card IO (for music), and some periodic tasks, since performing such calls in an interrupt service routine (ISR) would take far too long. Second, an ISR called 30 times per second handles inputs and generating events for the scene event handlers. This ISR also serves as timer for the main game task. Third, an ISR running at 22.05 kHz outputs music and sound effects to an audio jack.

For now, that should give you a broad overview of how this whole thing is put together. I'll get more specific, so keep reading!

# Graphics

One big challenge with this project was the limited ROM and RAM at our disposal: 128k of ROM and 32k of RAM. This made storing good-quality graphics a challenge. The easiest way of storing graphics is simply to store the colors for each pixel in an array, but that would occupy tons of ROM. For instance, a fullscreen image on the game would occupy approximately 61K of ROM—nearly half of our budget! Naturally, I had to be smarter about how we approached encoding pixel data. I settled on two solutions.

First, I computed a color table for each bitmap. Then, instead of storing the color for each pixel on every bitmap, I only needed to store the index of the corresponding color on the color table. By limiting the number of colors in each bitmap to be 256 or less, I could store each pixel color as one byte, roughly halving the ROM needed for each bitmap (colors were originally stored as packed 2-byte values). If you're into retro consoles, this is essentially the same approach taken by [consoles that use color palettes](https://en.wikipedia.org/wiki/List_of_video_game_console_palettes).

Second, and slightly more cleverly, I used a simple compression technique called run-length encoding. Perhaps the easiest way to describe run-length encoding is with an example. If I have ten pixels that are the same color, I could say that the first pixel is red, then the second pixel is red, and so on until I've painstakingly described the color for each individual pixel. However, it would also suffice to just say that there's ten red pixels in a row—much easier right? That is essentially the principle of run-length encoding; it is efficient to describe large spans of identical data, so instead of individually describing each data point, you describe a value and how "wide" it spans (even if the width is one). If you're still curious, [Wikipedia is a good start](https://en.wikipedia.org/wiki/Run-length_encoding), as always. One notable caveat to this encoding method is that it depended on our bitmaps having large spans of similar pixels, so images with lots of different colors in close proximity had to be avoided. Luckily, since I had control over the game's graphics, it was not difficult to design around this caveat.

Between these two methods, we managed fullscreen backgrounds and a large variety of graphics—something most other games did not achieve. We even still had a decent amount of ROM to spare when the game was finished.

# Sound

There are two main tasks that the audio system in ValvanoWare.

The first responsibility of the system is to play sound effects with polyphony. Sound effects, due to their short length, are able to be stored in ROM. Each sound effect encodes samples using one byte, with 11-thousand samples per second. Without explaining the [Nyquist Theorem](https://en.wikipedia.org/wiki/Nyquist%E2%80%93Shannon_sampling_theorem) in depth, it is important to know that the sample rate directly impacts the maximum audio frequencies that can be played back. Practically, the lower the sample rate, the more muffled the audio will sound. In accordance with the audio sample rate, the engine mixes up to two audio samples from two streams at about 11 kHz. Since the ISR responsible for outputting audio runs at 22.05 kHz, sound effect samples are only processed every other ISR execution. Since the engine handles playing audio, scenes only need to request that the engine play a sound.

The second responsibility is to play music—a much more involved task. First, music demands a high sample rate, 11 kHz sounds unacceptably muffled. If you've been wondering why I selected 22 kHz as the frequency for the audio ISR, the reason is that 22 kHz is the sample rate of the music and the highest sample rate across the whole program. Since music is longer than just a few seconds, using ROM was also not an option for any practical sample rate. Luckily, we had an SD card slot integrated onto our display. Using SPI, the game interfaces with a FAT16 filesystem, streaming music as the game is played. Essentially, the game will periodically take some extra time to read 4 kilobytes of audio data from the SD card: just enough not to stutter while keeping the time between reads large. The audio data is placed into a holding buffer, while another buffer is used for audio playback. When the audio playback buffer is exhausted, the two buffers switch places, and the audio playback buffer is then filled with new audio data. Savvy folks will recognize this configuration as [multiple buffering](https://en.wikipedia.org/wiki/Multiple_buffering).

For outputting audio, I opted to use the built-in 12-bit DAC on the microcontroller out of convenience. While an 8-bit DAC would have technically sufficed, given the size of each audio sample (8 bits), the fact that multiple audio samples are mixed necessitated at least a 9-bit DAC to avoid clipping. It would not have been difficult to build a DAC from resistors, but doing so would have required a lot more wiring with little tangible benefit over the built-in DAC. One caveat to the built-in DAC is that it can only drive lower impedance speakers without an amplifier. Headphones seem to work well, though.

# Hardware Setup

[INSERT IMAGE OF BREADBOARD]

The hardware setup for the game was fairly standard. There are three inputs: two buttons and one potentiometer. The buttons connect to digital inputs, and the potentiometer is sampled with an ADC. As mentioned before, the microcontroller reads data from an SD card and outputs audio through the inbuilt 12-bit DAC. Finally, the microcontroller outputs display data to a screen over SPI.

By the time the competition deadline rolled around, I had only laid out the hardware on a breadboard. It was not a requirement to do anything more ellaborate, but I do slightly regret not having it a little more polished by that time. Over the summer, I decided to more properly finish the hardware. I created a module that could attach to the dev kit similar to an Arduino shield or Raspberry Pi HAT. The module broke out all of the necessary connections to a IDC-16 connector. I created a handheld module that had all of the inputs and screen on it; an IDC cable connects the two modules. 

[INSERT IMAGE OF PROTOBOARDS]

# Conclusion

Prior to building this game, I had little practical experience using C. As someone who largely used interpreted languages, programming in C turned out to be a relatively pleasant experience. I also learned a lot about the deeper parts of microcontrollers, including hardware SPI, ADC sampling, DAC initialization, interrupts, and more.

Naturally, I had fun designing the audio/visual aspects of the game. Any project where I can combine programming and design is a project that I welcome. Naturally, I am happy that my fellow classmates seemed to feel the same way about how the game looks and feels as I do. They did vote it as the best game, after all.

As always, thanks for reading!