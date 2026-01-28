---
layout: post
title: "ROS Topics: The Wrong Tool for the Job (A Love Letter to Proper Abstractions)"
author: "Rango"
catalog: true
header-img: "img/dataflow.png"
header-mask: 0.4
tags:
  - Robotics
  - ROS2
  - Topics
  - Pub Sub Architecture
  - Clean Coding
  - Software
---

# ROS Topics: The Wrong Tool for the Job (A Love Letter to Proper Abstractions)

## Introduction
There are few things in life as certain as death, taxes, and engineers misusing ROS topics for things they shouldn't. If you've ever seen a team control a robot by spamming topics instead of using actions or services, then congratulations! You've witnessed one of the most common (and painful) anti-patterns in ROS development.

Today, we’re going to take a humorous but educational dive into why using topics for command execution is a terrible idea, why your team might be doing it, and how to fix it before your robot files a formal HR complaint against you.

---

## The Problem: Topics Are Not Commands (Stop Spamming!)
A topic in ROS (whether in ROS 1 or ROS 2) is best suited for *continuous streams* of data. Think sensor readings, real-time telemetry, or anything that’s being published at a high frequency with no need for acknowledgment.

However, some teams (maybe yours!) decide to use topics for sending commands, like telling a robot to move from A to B. And how do they make sure the message is received? By **spamming the topic** over and over again. Because, hey, if one message is good, then a hundred must be better, right?

### Why This is Dumb:
1. **No feedback:** You have no idea if the robot actually received the command unless you add another topic to confirm it (which is just doubling down on bad design).
2. **No guarantee of execution:** Topics are fire-and-forget. If the message gets lost, too bad, try again (or spam harder!).
3. **No tracking of execution state:** Is the robot still moving? Did it reach its goal? Who knows! Maybe it’s just doing donuts in the parking lot.

---

## The Alternative: Use Actions (Like a Sane Person)
If you’re sending a command that requires execution over time (like moving to a goal), **actions** exist *specifically* for this purpose. 

### Why Actions Are Better:
- **Feedback:** Actions provide real-time updates on progress.
- **Preemptibility:** You can cancel or modify a command mid-execution.
- **Guaranteed delivery:** The request-response mechanism ensures that your command isn’t lost in the void.
- **State management:** The action server keeps track of what’s happening, so your robot doesn’t enter an existential crisis.

How hard is it to switch? Not very. ROS 2 has a built-in `rclcpp_action` API (and `actionlib` for ROS 1). Instead of spamming messages into the abyss, you can send a well-structured goal request and get proper feedback.

---

## But Why Do Teams Keep Doing This?
Despite all the arguments against using topics for commands, teams still do it. Why?

### 1. "We’ve always done it this way."
Yes, and people also used to think the Earth was flat. Traditions are great for holidays, not software design.

### 2. "It's easier to implement."
Easier in the *same* way duct-taping a car door shut is easier than fixing the lock. Sure, it works—until it doesn’t.

### 3. "Actions are too complicated."
They really aren’t. If you can write a topic publisher, you can write an action client. It's just one step further in structure.

### 4. "We don't want to refactor."
I get it. Refactoring is scary. But so is debugging an architecture held together with `ros::Rate loop.sleep()`. Future you will thank you for fixing this *now*.

---

## The Path to Redemption
If your team is currently stuck in topic-spamming purgatory, here’s how to get out:
1. **Educate the team** – Explain why actions exist and how they solve actual problems.
2. **Start small** – Refactor just one part of the system (e.g., move-to-goal commands) to use actions.
3. **Ban unnecessary topic spam** – If a topic is being published at 10 Hz just to send commands, stop it.
4. **Use services where appropriate** – If you just need a simple request-response mechanism, a ROS service may be the right tool.

---

## Conclusion
Using topics for commands is a bit like texting “Are you there?” 100 times instead of just waiting for a response. It's inefficient, unreliable, and makes your system look like it was built by a caffeinated squirrel.

If your team is guilty of this, don’t worry—you can fix it. And when you do, your robot (and your sanity) will thank you.

Go forth and use actions. Or at the very least, stop the spam.

Happy coding!
