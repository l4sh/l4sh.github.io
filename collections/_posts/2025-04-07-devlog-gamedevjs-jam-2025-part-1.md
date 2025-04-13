---
title: "DevLog: GameDevJS Jam 2025 - Part 1"
date: 2025-04-13
layout: post
current: post
navigation: True
author: l4sh
category: development
tags:
  - web development
  - gamedev
  - game development
  - phaser
  - react
  - jam
comments: true
---

I've been putting off for some time doing a game jam. Seems like a fun exerice to get into game development, but always end up either way too busy or distracted by other projects.

So I've decided to finally just go for it and joined the Gamedev.js Jam 2025.

- Jam details on itch.io: [https://itch.io/jam/gamedevjs-2025](https://itch.io/jam/gamedevjs-2025)
- Jam details on Gamedev.js: [https://gamedevjs.com/jam/2025/](https://gamedevjs.com/jam/2025/)


## Jam Details

- Timeframe: 13 days.
- Start date: April 13, 2025
- End date: April 26, 2025
- Theme: "Balance" (Revealed when the jam started)
- Game engine: open
- Game type: open
- Game genre: open

### Hard requirements

- Game must be an HTML5 game playable in a web browser.
- Game must be submitted by the deadline.
- Game must be new and made during the jam.

### Evaluation criteria

Games will be voted based on: Innovation, Theme, Gameplay, Graphics, and Audio.

### Challenges

There are a few challenges set by some sponsors that one can choose to complete. To keep this log simple I'll only list the ones I'll be attempting to complete:

- Use Phaser game framework (Phaser Studio)
- Provide the source code of the game (GitHub)

## Pre-Jam preparations

Before the game jam I spent some time brainstorming ideas, outlining a rough schedule and doing some testing with Phaser.

I had inititally planned to use Phaser with Svelte to save time on the UI by keeping a clear separation of concerns by using Svelte for the UI and Phaser for the game logic. However when testing the Phaser+Svelte starter template ([https://github.com/phaserjs/template-svelte/](https://github.com/phaserjs/template-svelte/)), and uploading a test project to itch.io noticed that the game would not load due to sveltekit not being able to use relative paths for the build.

To be honest I didn't spend too much time on this. So while there's probably a solution I did not find it.

In the end I decided to just use Phaser with React using the Phaser+React starter template ([https://github.com/phaserjs/template-react/](https://github.com/phaserjs/template-react/)). Uploaded a test project to itch.io and it seemed to load properly.


If you're interested in the ideas check out the following Gist:

- [https://gist.github.com/l4sh/cf66906ea5aa14eae79d961ed33c6093](https://gist.github.com/l4sh/cf66906ea5aa14eae79d961ed33c6093)


## Day 1 - 2025-04-13

First day starts.

Tasks for today are

- [ ] Brainstorm ideas
- [ ] Define a game concept
  - [ ] Theme
  - [ ] Game mechanics
- [ ] Define schedule
- [ ] Create proof of concept

### Game concept

Game where the player controls a strongman that lifts a log over his head and balances the log.

The game is controlled by the player using the arrow keys to balance the log left and right and walk forward/backwards.

The player must reach a goal while keeping the log balanced.

The idea is to have a set of levels with increasing difficulty and a procedurally generated infinite mode that becomes increasingly difficult. That way we can go introducing the game mechanics and then have a mode where the player can just play for fun.

To list some of the things that can be done with this idea:
- Balance the log
- Balance the log and walk towards the goal
- ...with extra weight.
- ...with obstacles
- ...with things stacked on the log. If things fall off the log the player loses points.
- ...with gusts of wind that make the player lose balance.
- ...collecting things found in the ground while walking to stack and earn extra points.
- etc.

We can add a timer so that the best time is recorded and the player can try to beat their own time.

Asked ChatGPT to generate an image for this concept and it came up with the following:

![Strongman log game concept](assets/img/posts/gamedevjs-jam-2025-gpt-idea-concept.jpg)

Not exactly what I had in mind but I think it serves to illustrate the idea.

### Schedule

Onto the schedule...

So, we have 13 days to deliver, meaning that we actually have 12 days to work on the game. For simplicity I'll use 12 days for the planning.

Since we need to test, polish, add levels, etc. I'd say we actually have 6 days to work on the core game and the rest of the time to polish it.

Current schedule is as follows. At the end of the project I'll compare between the real schedule and the planned schedule to see how well we did. (Using my see the future spell: it'll be off 😂)

- [ ] Day 1 - 2025-04-13 - Sunday
  - [ ] Brainstorm ideas
  - [ ] Define a game concept
    - [ ] Theme
    - [ ] Game mechanics
  - [ ] Define schedule
  - [ ] Create proof of concept
     - [ ] Placeholder sprites
        - [ ] Background
        - [ ] Strongman
        - [ ] Log
     - [ ] Add simple balance log mechanic
     - [ ] Add simple walk forward/backwards mechanic
     - [ ] Add simple move sideways mechanic
     - [ ] Add simple tipping mechanic
- [ ] Day 2 - 2025-04-14 - Monday
  - [ ] Implement core game mechanics
    - [ ] Balance the log sideways
    - [ ] Walk forward/backwards
  - [ ] Think on level progression
- [ ] Day 3 - 2025-04-15 - Tuesday
  - [ ] Implement core game mechanics
    - [ ] Move sideways when the log starts tipping
    - [ ] Add falls
  - [ ] Think on level progression
- [ ] Day 3 - 2025-04-16 - Wednesday
   - [ ] Implement core game mechanics
    - [ ] Add stacked objects that can fall off the log when it tips
    - [ ] Add scoring system for objects that fall off the log
    - [ ] Collect objects on the ground to stack on the log ??
  - [ ] Think on level progression
- [ ] Day 4 - 2025-04-17 - Thursday
  - [ ] Implement UI
    - [ ] Add start screen
    - [ ] Add pause screen
    - [ ] Add game over screen
    - [ ] Add level selection screen
  - [ ] Add sound effects
  - [ ] Add music
- [ ] Day 5 - 2025-04-18 - Friday
  - [ ] Define levels
  - [ ] Start implementing levels
- [ ] Day 6 - 2025-04-19 - Saturday
  - [ ] Add infinite mode
  - [ ] Continue implementing levels
- [ ] Day 7 - 2025-04-20 - Sunday
  - [ ] Continue implementing levels
  - [ ] Continue implementing infinite mode
  - [ ] Testing & polishing
- [ ] Day 8 - 2025-04-21 - Monday
  - [ ] Continue implementing levels
  - [ ] Continue implementing infinite mode
  - [ ] Testing & polishing
- [ ] Day 9 - 2025-04-22 - Tuesday
  - [ ] Testing & polishing
- [ ] Day 10 - 2025-04-23 - Wednesday
  - [ ] Testing & polishing
- [ ] Day 11 - 2025-04-24 - Thursday
  - [ ] Testing & polishing
- [ ] Day 12 - 2025-04-25 - Friday
  - [ ] Testing & polishing

### Proof of concept

I started working on the proof of concept. The idea is to create a simple game where the player can move the strongman and the log.

After a bit of research looks like the right physics system to use is Matter.js. Since I want to be able to tilt the log, stack trhings on top and have them fall, etc.

The initial mechanic for balancing the log sideways was fairly straightforward. Just update the angular velocity of the log based on the input from the player and when the log reaches a certain angle just make it drop.

The problem came when I tried to add the ability to stack items on top of the log. The log would come down due to the weight of the items. In the end I resolved this by adding a second "ghost" log that would be set to static and make the boxes collide only with this ghost log that is synced with the real log position and angle.

Finally added some forward and backwards movement. This is done by moving the strongmand forward and backwards a bit and having the environment move to create the illusion of the strongman moving forward and backwards.

This is really rough and clunky but serves as a proof of concept for what I intend to build.


The proof of concept is available on:
- [https://l4sh.github.io/game-strongman-log-poc/](https://l4sh.github.io/game-strongman-log-poc/)

The code is available on GitHub:
- [https://github.com/l4sh/game-strongman-log-poc](https://github.com/l4sh/game-strongman-log-poc)


### End of day progress

According to our schedule we had planned the following

- [x] Day 1 - 2025-04-13 - Sunday
  - [x] Brainstorm ideas
  - [x] Define a game concept
    - [x] Theme
    - [x] Game mechanics
  - [x] Define schedule
  - [x] Create proof of concept
     - [x] Placeholder sprites
        - [x] Background
        - [x] Strongman
        - [x] Log
     - [x] Add simple balance log mechanic
     - [x] Add simple walk forward/backwards mechanic
     - [x] Add simple move sideways mechanic
     - [x] Add simple tipping mechanic


So far we're right on track.

I'll likely be updating the schedule along the way to reflect new findings.
