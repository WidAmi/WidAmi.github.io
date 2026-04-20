---
layout: post
title: "Claude Plays Factorio: Update on FactoMCP and Claude Code"
date: 2026-04-19
---

I've been experimenting with having Claude Code spin up subagents using Haiku to take care of things like collecting resources, fueling things, and construction. This seems to be working OK. I don't have any hard measurements, but it does seem like this gets more done before exhausting the 5-hour usage limit I run into. The weekly limit is a somewhat bigger deal. If I use up a five-hour limit I can wait, but using up the weekly limit means I can't get other things done with Claude for days at a time.

Placement and geometry is still an issue. I spent about an hour fighting to get Claude to put down a standard two-drill setup on the coal patch. This is a standard early-game design to keep coal drills running longer by having them feed into each other. For some reason it was convinced there needed to be a two-tile gap between the drills, and I got pretty frustrated trying to get it to perform what is really a simple action. I'm pretty sure the root cause was placing a drill and adding fuel before putting the other drill into place — if you do this, a piece of coal drops on the ground preventing placement of the second drill. It's the kind of thing humans figure out pretty fast to avoid, but I've had to explain it to Claude multiple times.

Which brings me to one observation about its behavior while playing: it often jumps to conclusions that are wrong. I wonder if this is because the game state progresses but the model can't tell unless it checks again. It can look at an area, see that it's empty, and start trying to place the drills — but when coal drops on the ground, there's no update forced back to the model. It won't see that unless it looks, and it usually doesn't look; it just jumps to erroneous conclusions.

Claude also has a tendency to do manual workarounds instead of fixing problems — hand-feeding coal into furnaces instead of figuring out why they're not getting coal, or hand-crafting items instead of fixing the assemblers that are supposed to make them.

## Some Progress

All of that said, I was able to make slightly more progress. It managed to build a simple power plant without manual intervention, including placing the water pump. It even managed to run power poles through a forest, which is the kind of thing it often gets stuck on.

![Power plant](/assets/images/power_plant.png)

I told it that it was going to need a lot of belts and should automate that using crates for input, and it managed to get that built OK.

![Belt automation](/assets/images/belt_automation.png)

It even almost came up with a working iron plate setup. Almost. I'm not sure what the extra belt is supposed to accomplish here, and it didn't quite manage to get coal into the furnaces — but it was close. There's just one belt needed on the west side to get that working. It also used a burner drill to get the coal instead of an electric one, which means it would still need to fuel that drill manually. But it's closer!

![Furnace setup](/assets/images/furnace_setup.png)

## MCP Updates

There are also some updates to the MCP code worth noting. I now include information about the pickup and drop target of entities when appropriate. This makes it easier for the model to evaluate whether things are lined up correctly.

## Next Steps

I'm thinking about trying a different harness rather than Claude Code. It's so hard to make progress on the Claude Pro plan without hitting limits. The [Hermes](https://hermes-agent.nousresearch.com) agent looks promising, and OpenRouter has some free tiers. I'll hit limits there too, but I'm not sure how fast. Having better tracking of actual usage will give me a clearer picture of what I really need — the Claude Pro plan is pretty opaque about actual usage and what that would translate to in API costs. So that will be the next step. Stay tuned!
