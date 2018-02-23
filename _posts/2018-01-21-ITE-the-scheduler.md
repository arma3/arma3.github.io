---
title: In the Engine - The Scheduler
description: Stuff
author: dedmen
layout: default
category: ITE
---

Now dedmen just needs to get some good writing skills.


When you [spawn](https://community.bistudio.com/wiki/spawn), [execVM](https://community.bistudio.com/wiki/execVM), [exec](https://community.bistudio.com/wiki/execVM) or [execFSM](https://community.bistudio.com/wiki/execFSM) a script it is added to an internal list of scheduled scripts.


There is one scheduler cycle per frame (`siScr` in Engine Profiler (New article to be linked here after it's written)) which happens right after the EachFrame event handlers. (Maybe future article about that)

At the beginning of a scheduler cycle a Timer is started, it will be used to keep track of the scheduler time limit.
The limit is 50ms in a loading screen and 3ms outside of a loading screen. We'll see how that's used a little further below.

Next up all scheduled scripts (SQF, SQS, FSM) are sorted by the time they were last executed. Meaning the script that was not processed for the longest time will land at the start of the new list.


Now we get to the actual execution of the scripts

The engine loops through our sorted list, executing the scripts in order.

Each Script type is handled a little differently:

- A SQS script will execute until it, is done, suspends, or after it executed for a total of 3ms. If the 3ms limit is reached, it will be forced to suspend.

- A FSM script will execute one update, no matter how long it takes in total.

- A SQF script will execute until it, is done, suspends, or after it the scheduler Timer reached its limit. If the limit is reached, it will be forced to suspend.

After one script was executed its “last executed time” will be updated so it can be used on the next scheduler cycle.

Before executing the next script in the list, the scheduler checks if the timer still has some time left to execute other scripts.

If the script that executed first already used up all the time we had, the scheduler will not try to execute the next script and directly stop.

After the time is up or all scripts were executed, the scheduler will end its cycle and won't execute scripts till the next cycle starts.

This is the reason why the timing in scheduled scripts is so unreliable.

Let's say you have 50 scripts which each use up the full 3ms each cycle meaning only one script is executed per cycle.
We have one cycle per frame. So at 50FPS this means any one of our 50 scripts will only be executed once per second.

Meaning if you use [sleep](https://community.bistudio.com/wiki/sleep) or [uiSleep](https://community.bistudio.com/wiki/uiSleep) and try to sleep for half a second, you will never get the expected result of half a second, because the engine simply only checks once a second if you are done with sleeping yet.
Even if you sleep for 1 second and you are so unlucky that the scheduler checks while your sleep timer is at 0.0001 seconds left, your script will be skipped and only executed after the scheduler executes your script again which will be about a second later.

This of course get's worse the more scheduled scripts you have and the lower your FPS are.