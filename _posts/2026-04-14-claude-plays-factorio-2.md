---
layout: post
title: "Claude Plays Factorio: The Reasoning Problem"
date: 2026-04-14
---

So this is an example of the reasoning output. It's pretty clear that it's struggling with basic game mechanics and is often running around in circles. Part of this is probably due to some issues in the MCP code. I tried having it write a tool to output a text-based "map" of an area but it seems like it might be misrepresenting the direction of inserters. Generally though, it seems like some explaining of game geometry and changing the MCP tools to give it a better understanding of the game state might help.

> For the output inserter at y=-42.5 facing north, its pickup would be around y=-43.3 based on that arm offset pattern, which puts it right at the edge of AM1's south boundary. This could be a problem — if the game treats that position as outside the AM or in a dead zone, the inserter won't be able to grab items from it. I might need to reposition the output inserter further away from the AM boundary to ensure it can actually access the contents inside.

So here it thinks maybe it needs to move the inserter further away from AM1 (Assembling Machine 1). But really the belt it's trying to put the copper wire onto is full.

> Let me consider using the east face of AM1 instead. I could place an inserter at (-58.5,-44.5) facing west, which would pick from inside AM1 at x=-59.5 and drop the copper cables at x=-57.5 in the gap between AM1 and AM2. The problem is that puts the output in the wrong location — I need those cables on the y=-41.5 belt, not in the gap between the two AMs.
>
> Actually, I'm realizing the coordinates don't work out. AM1's east edge is at x=-59, so an inserter at x=-58.5 would be positioned one tile east of that edge, outside the machine entirely. That means it can't actually pick from inside AM1 at all.

But now it seems to think the inserter should somehow be inside AM1.

> Looking back at the rotation data, when the inserter faces north from (-60.5,-46.5), it picks from (-60.5,-47.5) — exactly one tile in the direction it's facing. So the pickup location is always offset by one tile from the inserter's position in its facing direction. For my south-side inserters at x=-61.5, they're picking from inside AM1's range, which is why they work. The key constraint is that the pickup position itself needs to fall within the machine's coverage area. The pickup location at x=-61.5, y=-43.5 is indeed inside AM1's bounds, so that's not the issue. AM1 shows it's working with a full output buffer containing copper cable, yet the inserter is stuck waiting for source items — which suggests AM1 might be paused or idle despite the "working" status, possibly because its input is empty and it's just finishing its last queued operation.

And now it's discussing a different inserter entirely. The (-60.5,-46.5) location is the inserter at the top of the AM which feeds copper plates off the belt.

> I'm thinking the real problem is that the inserter can't actually reach AM1's output slot from its current position, even though the coordinates seem right. Let me try repositioning the inserter to the gap between AM1 and AM2 on the south side instead, placing it at AM1's east edge to see if that gives it proper access to the output. Mine the useless input inserter at (-59.5,-42.5) and replace it at (-59.5,-42.5) direction="north" as an additional output inserter. Maybe this x position works better for accessing AM1's output.
>
> Actually wait — x=-59.5 is IN AM1's x range (-62 to -59)? Is -59.5 < -59? YES, -59.5 < -59 (more negative). So x=-59.5 IS within AM1 x range of -62 to -59 (since -62 < -59.5 < -59)? Wait: -62 < -59.5 is TRUE. And -59.5 < -59 is TRUE (-59.5 is more negative = smaller). So yes, x=-59.5 IS inside AM1 x range. ✓
>
> So an inserter at (-59.5,-42.5) direction="north" would pick from (-59.5,-43.5) which IS inside AM1. This might work! Let me try replacing the inserter there.
>
> Actually, I'm realizing the real issue might be simpler: that south-facing inserter at (-59.5,-42.5) could be blocking things if it's picking copper cables off the belt below and feeding them back into AM1, creating a loop even when the belt is supposedly empty.
>
> AM1's recipe needs copper plates, not copper cables. So AM1 would reject the copper cables. The inserter might be stuck trying to put cables back in.
>
> Oh! This might be the issue. If the inserter at (-59.5,-42.5) is picking copper cables from the belt and trying to stuff them into AM1 (which doesn't accept them since the recipe is copper-cable which needs copper PLATES)... the inserter would just fail to insert and keep trying. Meanwhile, copper cables on the belt near x=-59.5 would be stuck being grabbed and rejected.

I think you get the point. It's pretty confused about where the inserters need to be relative to the assembler. Honestly I'm not even sure what problem it's trying to think through. Now that I've taken a look at the chain of thought it's not surprising that it burns through tokens like crazy without getting a whole lot done.

On the other hand, it did manage to get the factory it was working on far enough along that it made something. There are in fact several red potions on the belt! Clearly it has no idea how to actually use belts in Factorio, but that's probably something which can be explained to it. Or possibly I can have it take some screenshots and see if it can figure out the problem and what to do about it. For context, Claude did drop both iron and copper plates into the chest but the iron ones got pulled out first and ended up as gears.

![Claude's broken factory](/assets/images/broken_factory.png)

I think I had drastically overestimated Claude's understanding of how Factorio actually works. It seems to understand the basics, and it seems to be able to code to the Lua API pretty well. But once you get into dealing with all the coordinates and such it just seems confused. In fairness, I'm finding the coordinate system somewhat confusing myself — it's certainly harder to reason about than just looking at the visual layout.

## What I'm Going to Fix

**Tighter entity lookups.** I've realized it's using `find_entities_filtered()` with a radius of 1, which has led to a lot of errors where it finds the wrong entity. This function should usually be called with just an exact location and no search radius — that way it returns exactly one entity (or two if there's an ore resource at the same spot). That should cut down on a lot of confusion.

**Fix the `detail_area` tool.** I have a tool that may be getting the direction of inserters wrong. Fixing that up should help it understand the layout.

**Use `drop_target()` and `pickup_target()`.** Some entities like inserters have these methods, which tell you specifically what they're feeding into and pulling from. Including that in the tool output will almost certainly help a lot. Here's a concrete example of the difference:

```
❯ For the copper wire assembler AM1, describe the relationship between the inserter
  north of it, the AM, and the inserter south of it using coordinates.

⏺ - Inserter at (-60.5,-46.5): picks up from (-60.5,-47.5), drops to (-60.5,-45.3) — feeds the north belt into AM1
  - AM1 at (-60.5,-44.5): occupies (-62,-46) to (-59,-43)
  - Inserter at (-59.5,-42.5): picks up from (-59.5,-43.5), drops to (-59.5,-41.3) — extracts from AM1 onto the south belt

❯ Now describe it using unit numbers.

⏺ - Inserter #206: picks up from belt #129, drops to assembling-machine #136
  - Assembling-machine #136
  - Inserter #297: picks up from assembling-machine #136, drops to belt #211
```

You can see in the first output that none of the numbers actually match up. The drop point of the top inserter doesn't match the coordinates of the AM, and the pickup point of the bottom inserter doesn't match the AM coordinates either. But using `drop_target()`/`pickup_target()` to get the entities and describing them by unit number makes the relationships much clearer.

**Use `unit_number` for identity.** Most entities have a `unit_number`, and referencing entities by that instead of by coordinates should reduce a lot of the confusion about which entity we're talking about.

My next step will be to make those changes to the MCP tools and have it start a new factory. I'll also explain a bit more about how belts work so it stops making sushi belts and see if that works out better. Stay tuned!
