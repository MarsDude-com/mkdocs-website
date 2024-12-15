---
published: 2024-12-10
permalink: microos/planning-automatic-updates
aliases:
  - 1.+Projects/Blog/OpenSUSE+MicroOS/Planning+your+automatic+transactional-update
image: https://publish-01.obsidian.md/access/cb21bcbdb7ed27e8f2fc67c075437e18/System/Attachments/20241210_001_TransActionalUpdateTimer.png
tags:
  - MicroOS
---


OpenSUSE MicroOS has the feature that it automatically updates, and reboot to use the new snapshot. And it does that every day.

To see when the next attempt to update is, you can do this:

```
sudo systemctl status transactional-update.timeer
```

You'll see something like this.

![[20241210_001_TransActionalUpdateTimer.png]]

Look at the 'Trigger' line. That's during the night, when my laptop is turned off. And on my server, I don´t want it to reboot every night.
On the laptop, it will be triggered again when it's on, but that will be explained later (spoiler: it's the `Persistent`setting).

# Setting your update timer

The timer is set in the file`/usr/lib/systemd/system/transactional-update.timer`. Let's look at the contents:

```
[Unit]  
Description=Daily update of the system  
Documentation=man:transactional-update(8)  
After=network.target local-fs.target  
  
[Timer]  
OnCalendar=daily  
AccuracySec=1m  
RandomizedDelaySec=2h  
Persistent=true  
  
[Install]  
WantedBy=timers.target
```

The important part here is of course the Timer section.

* **OnCalendar=daily** : This is self-explanatory I think -> do this every day.
* **AccuracySec=1m** : Specifies the accuracy the timer shall elapse with. Defaults to 1min. So in this case. Every day within the first minute... but, we have another setting...
* **RandomizedDelaySec=2h** : Delay the timer by a randomly selected amount of time between 0 and the specified time value. So it will trigger somewhere between 00:00 and 02:00
* **Persistent=true** : When your timer is inactive (for example, when your computer is turned off) it will activate when our computer is turned on again. As we have a delay in the setting above, your update will take place within the first two hours after booting up. Which is fine. 

You can change these values. Just copy the file from the immutable part to the mutable part :-)

```
sudo cp /usr/lib/systemd/system/transactional-update.timer /etc/systemd/system
```

And edit with your favourite editor (I won't judge you, whatever it is).

So... You don´t want to update every day?

* set the `OnCalendar` to `Sun`for example. And if you do that, make sure you also update `Description` in the `[Unit]` section. It doesn´t make a difference, but we all know it' the right thing to do.

Or maybe not between 00:00 and 02:00 ?

* set the Oncalendar to `Mon..Sun 20:00` to set it to occur every day at 20:00. (remember, if the delay is set, it will occur between 20:00 and 22:00)

You don´t want to have it done automatically at all? That's also an option. Just disable the timer:

```
sudo systemctl now disable transactional-update.timer
```

The way you can update manually whenever you feel like it:

```
sudo transactional-update dup
```

# Configuring the reboot

Want to know when the automatic reboot is planned?

```
rebootmgrctl get-window
```

By default it will start at 03:30:00 with a timewindow of 1h30

For the strategy:

```
rebootmgrctl get-strategy
```

This one defaults to 'best-effort', which basically means: If there is a time window, use that. Otherwise, reboot immediately. Just leave it at that.

If you want to set it at 06:00...

```
sudo rebootmgrctl set-window 06:00:00 01h30m
```

The update will request the reboot, so if transactional-update is done on sunday 22:00, it will request a reboot the next 06:00 with a window of 1h30m.

You can also just disable it. Then you can reboot whenever you want. On Kalpa (MicroOS Plasma) you get a notification that your system has been updated and that you can reboot to use the new snapshot. Or you get to use the latest snapshot when you boot up your laptop the next day.

To disable the reboot:

```
sudo systemctl --now disable rebootmgr
```

I hope this has been useful to you. Let me know if it was, or if you have any comments / additions. You can find out where to contact me at the [[Contact]] page.
