---
title: In the Engine - The Scheduler
description: Stuff
author: dedmen
layout: default
category: ITE
---

Now dedmen just needs to get some good writing skills.


When you [spawn](https://community.bistudio.com/wiki/spawn), [execVM](https://community.bistudio.com/wiki/execVM),[exec](https://community.bistudio.com/wiki/execVM) or [execFSM](https://community.bistudio.com/wiki/execFSM) a script it is added to a internal list of scheduled scripts.


At the beginning of a scheduler cycle (once per frame. `siScr` in Engine Profiler (New article to be linked here after it's written))
all scheduled scripts (SQF,SQS,FSM) are sorted by the time they were last executed. Meaning the script that was not processed for the longest time will land at the start of the new list.

Just before the sorting a Timer was started.
It will be used to keep track of the Time limit.
The limit is 50ms in a loading screen and 3ms outside of a loading screen. We'll see how that's used a little further below.

Now we get to the actual execution of the scripts

The engine loops through our list executing every script in the order it appears in the sorted list.

A SQS script will execute until it, is done, suspends, or after it executed for a total of 3ms.

A FSM script will execute one update, no matter how long it takes in total.

SQF scripts are a little more special. They execute until they, are done or suspend just like SQS.
But before the SQF script is executed the engine checks how much time is left on the Timer till the limit is reached.
The SQF script will execute at most till the limit of the Timer is reached. If the limit is reached it will be forced to suspend.


After one script was executed the time it was last executed at is being set so it can be used on the next scheduler cycle.

Before executing the next script in the list the scheduler checks the Timer and if it still has some time left to execute other scripts. If the script that executed first already used up all the time we had, the scheduler will not try to execute the next script.
The scheduler will end it's cycle and won't execute scripts till the next cycle starts.

This is the reason why the timeing in scheduled scripts is so unreliable.

Let's say you have 50 scripts which each use up the full 3ms each cycle meaning only one script is executed per cycle.
We have one cycle per frame. So at 50FPS this means any one of our 50 scripts will only be executed once per second.

Meaning if you use [sleep](https://community.bistudio.com/wiki/sleep) or [uiSleep](https://community.bistudio.com/wiki/uiSleep) and try to sleep for half a second, you will never get the expected result of half a second. 
Because the engine simply only checks once a second if you are done with sleeping yet. Even if you sleep for 1 second and you are so unlucky that the scheduler checks while your sleep timer is at 0.0001 seconds left, your script will be skipped and only executed after the scheduler executes your again which will be about a second later.

This ofcause get's worse the more scheduled scripts you have and the lower your FPS are.