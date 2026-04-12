---
layout: post
title: "Claude Plays Factorio (Sort Of)"
date: 2026-04-12
---

This is my attempt to have Claude Code play Factorio via an MCP server. The short version: it works, kind of, and it cheats a lot. The MCP server code is at [FactoMCP](https://github.com/WidAmi/FactoMCP) on GitHub.

## The Setup

The Factorio Lua API is available via RCON, which is how my MCP server talks to the game. The catch is that the API isn't really meant for this use case — it's designed for writing game extensions, not for controlling a player entity. There are functions to have a player walk or mine at a certain spot, but it's very easy to write MCP tools that just teleport, or magically place and remove items with no regard for normal game rules.

## First Contact

The first tool I had Claude write was `run_lua()` — a general-purpose escape hatch to execute arbitrary Lua in the game. My expectation was that it would issue pretty simple commands, like single function calls to get state or perform actions. Surprisingly, it would write fairly involved Lua scripts right out of the gate, often with loops and conditionals.

And with no guidance about playing by the rules, it was happily cheating. No need to walk when you can teleport. If mining isn't working the way it expects, it just spawns items directly into its inventory. That was a big motivation for writing more specific tools that at least try to follow the rules.

## Token Burn

It burns tokens like crazy. Most of this is probably internal reasoning, but in recent versions of Claude Code the model's thinking isn't shown by default, so it's hard to know what would help it reason more efficiently or where it's going wrong.

## The Coordinate Problem

The coordinate system is trickier than I expected. As a player, everything looks like a simple grid. But where items actually get placed depends on their size. A 1x1 item like a power pole is centered in its grid cell — so it ends up at `(5.5, 5.5)`, not `(5.0, 5.0)`. Claude consistently wants to use even offsets, which means the game snaps placements to unexpected locations. This is something I need to address with better instructions.

## Seeing the World

It's really hard to get a useful picture of game state via the API compared to what a human player takes in just by looking at the screen. I added the ability for the MCP to take screenshots, and Claude can interpret those — it seems to help a little. I haven't leaned into it heavily though, since image interpretation likely burns a lot of tokens.

## What It's Actually Built

Despite all this, it does occasionally get something done.

Here's a boiler setup Claude built. Not how I'd do it, but the parts are wired up correctly. Fluid networks and pipe placement were a struggle — a dedicated fluid tool helped somewhat, but I still haven't gotten it to place a water pump successfully. The weird offset required for a 2x1 item, the dependency on rotation, and the restriction on which water-adjacent edges it can use means it can spiral indefinitely on that one task. Since pump placement is rare in the grand scheme of things, I haven't focused on it.

![Claude's boiler setup](/assets/images/claude_boilers.png)

Here's an early-game coal mining setup. I seem to need to remind Claude every time how to arrange drills so they feed a belt in a cycle. Left to its own devices it wants to just drop coal on the ground, or at best put a chest at the output of each drill — neither of which is a real strategy.

![Claude's coal mining setup](/assets/images/claude_coal.png)

And here's where it's started building a factory. I'm about 99% sure this layout isn't going to work, but I'm curious what it thinks it's building. The problem is it'll probably burn through a week's worth of tokens trying to get there, so I may not let it run.

![Claude's factory start](/assets/images/claude_factory.png)

## Where to Go from Here

**See the reasoning.** Anthropic seems to be making internal reasoning harder and harder to inspect. But whatever Claude is thinking, it's not efficient — tons of tokens for modest results. Getting visibility into the reasoning chain would help a lot.

**Better MCP tools.** The game state tools need work. Right now it's hard to answer basic questions like: is power out? Is fuel running low? Is a drill actually outputting onto a belt, or just dropping on the ground? Is placement failing because of a coordinate offset error? Better tools that surface these answers directly would reduce the trial-and-error loop considerably.

**Sub-agents.** One idea I want to try: use a more capable model with extended thinking to produce a plan, then hand that plan off to the cheapest possible model with no thinking to actually execute the placements. Separation of planning and execution might be the key.

**Explicit planning steps.** I'm also working on a Factorio calculator MCP (stay tuned). The idea is to give Claude an explicit goal — "automate 5 red and green science per minute" — and require it to: (1) use the calculator to determine what's needed, (2) generate a proposed layout, and only then (3) execute by walking around and placing items. That kind of structured breakdown might get it to produce a reasonable plan instead of improvising tile by tile.

There's a lot of room to improve here, but it's been a fun experiment so far.
