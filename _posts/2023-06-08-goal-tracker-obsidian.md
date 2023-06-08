---
layout: post
date: 2023-06-08
title: Track your goals with Obsidian
giscus_comments: true
related_posts: false
tags: productivity obsidian plugins
---

This blog post presents how I got started setting up my goal tracking process using Obsidian and a few amazing plugins. At a later post I may come back to assess how it worked. 

# Motivation

As someone who struggles to stick to one thing, I've been looking for a tool to help me set and track goals. I've been using Obsidian for a month to keep notes on various interests of mine. It has been a very helpful so far, so I started looking for ways to organize my checklists and tasks with it as well. I am already running a checklist for the week and a general check list for things I am curious about. However I found that the weekly checklist was inadequate for my needs. I needed something to actually organize my time at the micro level (hour or day) and to connect it to macro goals (monthly or even yearly). Simply having a weekly priority list didn't cut it. 

# Daily and weekly notes

I was inspired by  [Nicole van der Hoeven](https://nicolevanderhoeven.com/) to use obsidian for goal tracking. Her youtube channel and blog contains many useful processes for increasing your productivity in general. After browsing the materials on her website I  decided  to try set up a system, heavily based on her suggestions. 

First I installed the [Calendar plugin](https://github.com/liamcain/obsidian-calendar-plugin) which is perfect for daily logs. But if you pair it with another plugin called [Periodic notes](https://github.com/liamcain/obsidian-periodic-notes) you can go beyond daily to weekly/monthly etc. That's what I am interested in, so that I can connect my daily work to my larger monthly goals all the way to the yearly if needed. 

Here are the basic steps : 

1) create a daily notes template(more on this in a minute) to help me organize my work at the lowest level possible, the day. This is something I didn't have before so I hope to provide me with the structure I need.
2)  create a weekly template to replace my current weekly checklist notes. 
3) put all the templates  in my templates folder.
4) create a reviews folder with daily and weekly subfolders to save all of the notes I intend to create.
5) install Periodic notes and set the appropriate paths in the settings of the plugin
6) Set the appropriate template-folder pairs in the settings of the [Templater plugin](https://github.com/SilentVoid13/Templater) (if you don't use it I recommend checking it out). 


# Effective templates 
To create effective templates I start from one recommended by others and try to understand what it does by breaking it down. This way I can keep only the bits that work for me.  

As I worked with the daily note template I realized that there are basically three things I needed. 

- a daily log, a place to brain dump with no regrets :)
- a list of tasks I need to finish to keep me on track
- a way to connect these tasks to the long term goals

Here is the daily template I landed on (after 2-3 days of experimenting):
```
## Direction
This is what you are working towards!

![[reviews/weekly/Weekly Review <% tp.date.now("YYYY-[W]ww") %>#This week]]

## Task due today

# Log

### Tomorrow, I need to ...

```

- Direction: this pulls in the goals I've set for the week (more on this below)
- Task due today: is a checklist that I create 
- Tomorrow I need to: prepare checklist for tomorrow's note
- Log: brain dump area

The following part uses the [Dataview plugin](https://github.com/blacksmithgu/obsidian-dataview) which is an extremely powerful too. Here I am doing a simple query from the weekly note to pull in the goals of the week and present them in my daily note. 
```
![[reviews/weekly/Weekly Review <% tp.date.now("YYYY-[W]ww") %>#This week]]
```

For the weekly note I needed three things: 

- goals for the week
- and end of week checklist for things that need to happen every week
- a space to reflect on the week and jot down lessons for the next week
- A way to connect the weekly goals to the monthly level (more on this later)

Here is the weekly template I went with (after some tinkering again): 
```
> [!warning] No more than 3 !
> If you could only get three specific goals done this week, what would they be? 
## This week
- [ ] 

## Monthly initiatives

Keep these things in mind as you do this week's review:

## Monthly review <% tp.date.now("YYYY-[M]MM") %> 

[[reviews/monthly/Monthly Review <% tp.date.now("YYYY") %>-M<% tp.date.now("MM") %>#This month]]


## Reflections for the week 
> [!question]
> Did you finish all of last weeks items?


## End-of-week-checklist

- [ ] review events for the week
- [ ] backup computer
```


You notice the pattern. A place to keep organized on the daily level but connect it to the level above too. Then at the weekly level a place to keep organized and connect it to the level above. And so on. 

For now to connect the levels all I do is to list the goals of the levels above within the note for the tasks below. I will come back to this and reassess if this is sufficient or not, but in these few days I've been using it's working fine. 

One final note on template. I found that using the [Callouts functions](https://help.obsidian.md/Editing+and+formatting/Callouts) to create visual warnings help me write better goals. 
```
> [!warning] No more than 3 !
> If you could only get three specific goals done this week, what would they be? 
```


# OKRs for the quarter 

Most goals or tasks take a little bit of time, say a couple of hours. But some of my learning goals, such as learning a programming language or following a 12 week course, span months. I am writing down quarterly goals and then creating monthly initiatives to help me achieve the goals. The initiatives are written to be as factual as possible, the technical term being OKRs. It seems a little formal to track goals this way. But I’ve never found an effective way to track such long term goals, so I am willing to give the OKR method a try.  For a more detailed description you can see this [post](https://marcus.se.net/how-i-set-and-track-goals-in-obsidian/) by Marcus Ollson.

Again following the same idea as above, I embed the monthly initiatives in the weekly notes, and the quarterly goals in the monthly review notes. I did not see the need to plan the year yet, so I stopped at the quarter. Nicole goes into more details in her [video](https://www.youtube.com/watch?v=T2Aeaq4sk7M) which is very helpful. 

For now I will stick with the simple methods I described above but I will revisit this post to assess it. 


