---
author: "Naadiyaar"
title: "PTM done right"
date: 2024-03-17
draft: false
ShowToc: true
---
I always believed one tool can't work best for everyone and people should choose their instruments themselves.
There is not a single tool suitable for everything and or everyone, and it's always part of the work to find the right tool.  
And for the first time ever, finally I know how to get things done and see my tasks list is actually shrinking.
Here I'm going to write about my approach which might work, or at least inspire you too.

In this blog you will read about a command line task management program called [Taskwarrior](https://taskwarrior.org/) and a little bit about how I use it by help of [GTD](https://gettingthingsdone.com/) method.  
I've started using these two almost 2 months ago and seen very good results so far.
But never liked to spend so much time digging deep into TW or GTD, so everything might not be best practices but this is how I do things and will love to hear your opinion either.

## On Taskwarrior
Taskwarrior lets you create, organize, manage and get visual reports for your tasks and ToDos all within terminal.  
If you're wondering [why you want to use a terminal based task management app]({{< ref "/content/posts/the-suprior-interface.md" >}}), I already wrote a post discussing that.
But if that's not the case (hopefully) please continue reading.

An important characteristic about TM is, it's not trying to be a minimalist and small CLI tool, on the other hand it's a very feature rich and flexible tool that let you organize tasks however you wish.
It let users get prepared for any activity and put them into their tasks list with no distraction.

Taskwarrior has a comprehensive documentation which guides you through using its features and users can realize how to add a specific task to their list by taking a glance it docs.
Though you should **be careful** not to dive into documentation to learn everything and forget actual tasks and things you must do.
Rather it's best to start by reading the [start guide](https://taskwarrior.org/docs/start/) and refer to documentations only when you needed to add an option to a task.  
Also if you needed any help it's always possible to ask questions on their active IRC channel.

Another very important TM ability is synchronization among all devices using *Taskserver*.
By setting up a Taskserver you will be able to keep your tasks up-to date among multiple clients automatically, while having the full control over your data.  
It's also possible for a team to use a single Taskserver for a more robust task management.

## Getting Things Done
GTD is critical to me because it reduced a lots of pressure that my brain supposed to carry alone.
GTD matters from this perspective that eliminates the need to keep conditions and activities in our short term memory.
But GTD's core philosophy rests on writing everything and anything in your task list.
So this humane brain can finally enjoy some free space and focus on more necessary choices and functions.

First and (for me) most influential part of not carrying every little detail in memory, is being rescued from the anxiety and confusion of remembering too many things.
Specially if you're involved in many activities, like School, Work, family matters etc... 
Taking care of everything can be overwhelming and push people into burden and prevent us from recognizing the most essential tasks that must be done.
But when everything is on a piece of paper or on screen determining urgency is much easier.

In general GTD is comprised of five simple steps which are:
- CAPTURE  
*Collect what has your attention*
- CLARIFY  
*process what it means*
- ORGANIZE  
*Put it where it belongs*
- REFLECT  
*Review frequently*
- ENGAGE  
*Simply do*

For more information about **Getting Things Done** please refer to their [original website](https://gettingthingsdone.com/what-is-gtd/);
And again it's important not to get lost into mastering this method.

## Let's start
Let's start with the most critical part of GTD, which taking note of everything.
It's necessary to write down everything that needs an action.
Nothing is too small or unimportant to be ignored.
Making decisions about if something actually worth your attention or not is done at another time.
And for this matter you can use a simple note taking app on your phone and since cross device synchronization doesn't really matter any app suits well.
I personally use *Nextcloud* note-taking client on android and *Iotas* on Fedora.

At the end of every night, is time to review notes and decide what to do with each one, either delete them or enter them into Taskwarrior.
Taskwarrior supports tagging tasks so let's make best use of it.
Separate tasks that take very little time to finish (e.g, ~5 minutes) and tag them with something representing this point.
I like to use the tag (**s**)mall for these, alongside other tags like (**m**)edium (less than 2 hours to complete), (**l**)arge (less than a day to complete) and (**xl**)arge.  
Adding a tag to a task is done by `+` operator:
```
task add A small task +s
```
Then you can filter output to see only *small* tasks by:
```
task +s
```
The purpose of distinguishing small tasks is because those must be done first, even if their priority is lower than some other tasks, brushing teeth or feeding your fish won't prevent you from finishing your book.

Another important functionality is to create receptive tasks, which automatically recur after a specific period of time.
Such as when you want to run for 15 minutes every day, make the bed or call your mother at least once a week.  
You can achieve this by creating a `recurring` task:
```
task add Call mom recur:Sunday duo:eow priority:H
```
What we did here is creating a task which recurs every Sunday with a duo until *End Of Week*, since it doesn't make sense to do it twice a day because you forgot about it for a whole week. (what's wrong with you?)  
Also we added a (**H**)igh priority for this task to help us organize it better.

## Integration and configuration
Another important part of GTD is *reflect* which requires you to have an eye on your tasks list.
And what can be better than seeing your most urgent tasks at terminal start-up?
As for this I like to add following little *if* statement to the *zshrc* (also works for Bash):
```zsh
if [ ! -f /tmp/task_executed ]; then
    task list priority:H or +s
    touch /tmp/task_executed
fi
```
This statement checks if the file *task_executed* exist in *tmp/* directory and if it doesn't, *task* will run.
It prints tasks with **H** priority and tasks containing **s** tag.
And then creates the *task_executed* file in *tmp/* to prevent running this command on each shell start.
Therefor you will only see tasks list at machine startup and first shell start.

Also creating some aliases can boost productivity and remove some boilerplate, my desired aliases are:
```zsh
alias t="task"
alias ts="task list +s"
alias th="task list priority:H"
alias tm="task list priority:M"
alias tl="task list priority:L"
```
---
Like any other CLI tool you can expect Taskwarrior high levels of customizability done in *.taskrc* file.
For instance you can modify the amount of **urgency** a label, project or any other characteristic of a task adds.
Take this configuration as example:
```
urgency.user.project.school.coefficient=1
urgency.user.tag.d.coefficient=100
```
Here we're telling TW if a task was part of project *school* add 1 score to its urgency.
And when a task had the label (**d**)eadly add 100 score so it always stays at top and represents it needs immediate action.  
For full guide refer to [official documentation](https://taskwarrior.org/docs/configuration/).

## Final words
In this blog I wrote about my favorite way of managing tasks and duties without loosing track of projects and it's been working so far.  
Of course it's not suppose to work for everyone and some maybe even find this approach weird. 
But both *Taskwarrior* program and *Getting Things Done* method are well known tools that work in harmony.

**Thanks for reading; please let me know what do you think.  
Good luck.**
