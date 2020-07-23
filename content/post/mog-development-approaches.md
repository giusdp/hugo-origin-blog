---
author: "Giuseppe De Palma"
title: "MOG: Development Approaches"
description: "A short introduction to multiplayer online games (MOG) development."
categories: ["Game Dev"]
date: 2020-07-23T17:24:57+02:00
draft: false
tags: ["game dev", "multiplayer"]
series: ["MOG Development"]
imagelink: "/MOG/DedicatedServer.png"
---

Lately I've been reading a lot about what's behind the "multiplayer" in multiplayer online games, commonly called netcode in gamers lingo.
How does networking work and how could we develop it from scratch?

The literature is very sparse, books are few and tutorials and guides are almost always focused on a single ready-to-use solution. I'll try to summarize what I found.

# Multiplayer Online Games (MOG)

There are three well defined architecture for a MOG:

### 1. Dedicated server

The dedicated server implies a standard _server/client_ architecture, where a **client** is the game itself, and the **server** is a separate project that you have to build in parallel with the game.

Many clients connect to the server and ideally they communicate every single input performed by the player.
The server updates the state of the game and sends the results back to each client, so they are all synchronized between them.

<p align="center">
<img src="/MOG/DedicatedServer.png"
     alt="Dedicated Server Layout Image" />
</p>

This is also called **authoritative** server, because it's the server that governs the game, it updates the state and the clients show to the player only what the server says. This is important to fight **cheaters**, if a client had power over the server, a cheater could keep telling the server that his character is at full health all the time.

In reality it's not possible to have a completely dumb client that only sends inputs over the internet, waits for the next frame from the server and shows it. The latency would be too big and you would be playing a slide show. Several techniques are used to mitigate this problem while also keeping the server authoritative (client prediction, client interpolation, etc...). We will see them another time when we dive into authoritative servers.

### 2. Player-hosted server

This might be the most used architecture in my opinion, at least for the majority of online games I play (lots of indies and I'm sure Super Smash Bros. Ultimate uses this one too). It's basically a dedicated server inside the game, hosted on one of the player computers.

Games where people play in matches of few players usually go this way,
a player is selected as host and the other players connect to them.

<p align="center">
<img src="/MOG/PlayerHost.png"
     alt="Player Hosted Server Image" />
</p>

The main advantage is saving costs, the developers don't have to rent machines to host a dedicated server. I don't think I ever saw an indie game with multiplayer aspects that don't do multiplayer this way.

Unfortunately this is not as secure against cheaters (the host might mess with the server), and cheating is the biggest problem when making an online game. Wherever I read about MOGs, everyone
take this as a rule:

<blockquote class="border-l-4 border-teal-500 font-bold italic my-8 pl-8 md:pl-12">
    You have cheaters in your multiplayer online game. Even if you think 
    you don't, or there is no reason to cheat.
</blockquote>

There are also two more problems: latency and host disconnection. Having the server on a normal residential connection won't always be a good experience for everyone.

Latency will always be worse for everyone but the host, compared to a dedicated server positioned in near-to-players datacenters. This also means that the player host will always have an advantage against the connected players.

The second problem happens when the host disconnects. No more host = match gets shut down, but this can be mitigated with some host migration technique to change host on the fly.

### 3. Peer to Peer

Peer to peer online games sounds nice, there is no server at all and every client performs its own computation and shares them with the other players. It looks like it's scalable out of the box, but
it's the most susceptible to cheating.

Without an all-knowing system that checks everything is in place,
a peer could simply send the others wrong data to gain an advantage. For what I'm seeing, everyone nowadays uses the first two approaches, meaning that authoritative servers became the standard.

## Development Workflows

Let's say we decided to make a MOG with a dedicated server, how would we go on to develop it?
One thing to keep in mind is that if we start working on our game leaving the networking part for later,
it will be really hard (read impossible) to make it multiplayer.

The server part has to be developed at the same time and it must be a priority during the entire development. In general there are two workflows for these types of games:

- Server-Driven Workflow
- Client-Driven Workflow

### Server-Driven Workflow

The former is entirely focused on the server, _the game is defined by its rules_ (think classic card games) that are executed on the server. In this case the client is really just an interface.

The majority of the work goes directly on the server side since that is the actual game, there is no "game world" or advanced graphics that play a role, no particular computation as a 3D game would require.

If you want to create software such as stock exchanges, casino-like games, social games, digital board games, then approach them with a server-driven mindset. The game is the server itself, and you should have a good starting point to develop them.

### Client-Driven Workflow

The latter is the typical workflow used for most games. This workflow comes into play where _the game is defined by its content_. The development is more focused around the client,
there is level design to do to create the world, the assets to be made, maybe it's even 3D etc. etc.

With this workflow the designers have to keep in mind the game is not single player, so when designing content they have to take into account the distributed nature. The server-side has to be developed together with the client and tested at each step of the process.

As I said above, making a single-player game and trying to transform it into multiplayer later it's a big no-no.

This means that we have to do a "continuous conversion". Develop the game as if it's a single-player for the most part and at the same time convert it to a multiplayer one, never skipping the conversion step. It helps to integrate the server into the developers' toolchain. Everytime the game is tested, it is tested as a multiplayer game.

### Conclusion

Now you can kinda understand how to approach online multiplayer, depending on the game. I will go on to the dedicated server option for a Client-Driven workflow and try to understand better it myself and explain what it consists of.
