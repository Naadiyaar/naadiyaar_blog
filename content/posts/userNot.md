---
author: "Naadiyaar"
title: "Not a user"
date: 2023-12-06
draft: false
ShowToc: true
---
## Intro
Telegram is not just a messenger or even a social media site; it is more like a massive bookmark manager where every group, 
channel or personal chat you have with fellows is just a bookmark on a special topic. Sometimes you're the audience for these 
bookmarks, and sometimes you're the one in charge. There is also another type of accounts/bookmarks 
on Telegram, which you can either create or use. Called telegram **Bots**.

A bot is basically just a service running as a backend and using Telegram as its frontend. Probably you've already encountered some of these special accounts; they're known for being useful and have the keyword *bot* at the end of their usernames (unless in rare cases). These bots are capable of doing lots of amazing stuff, such as searching for music, downloading videos from YouTube, being a crypto wallet, etc. However, these bots are pretty limited and can't perform most of the tasks; for example, they can't send a message to a group unless they have administrator privileges or can't start a conversation with other people. Fortunately, there's a solution for it...

We create **UserBots**! This way no one knows if they're talking to a human or an LLM. userbots are like normal bots, but instead of using a bot account, we place the bot behind a normal user account (the one you create by providing a phone number). userbots are capable of doing whatever you can, or even more ;) 

In this post, I go through creating a very basic userbot, so we understand how to have one in our pocket.

## Requirements 
### get API ID and hash
telegram likes to see these bots as different telegram clients, therefore you need to create an APP on their website and get our unique development ID:

1. [Login to your Telegram account](https://my.telegram.org/)
2. click on API development tools
3. fill your new application information (first two fields are enough)
4. Click on *Create application* at the end

> Warning!
> 
> your API ID and API hash are **secret** and telegram won't let you revoke them.

> Note
> 
> you can use this API with any phone number.

### set-up development environment 

It's possible to create such an application using [the telegram core API](https://core.telegram.org/api), but here we're using a Python package called Telethon which does a lot of magic and makes life much easier. It's a very straight forward library and lets you focus on developing ideas. 

We begin by installing the latest version of Telethon, firstly let's create a python [virtual environment](https://docs.python.org/3/library/venv.html)
```python
python -m venv /path/to/virtual/environment
```
Then we enter the VE by sourcing the `bin/activate` script, it is available for different shells. `activate` for Bash and Zshell, `activate.fish` for fish:
```zsh
source ~/Desktop/python/env/userNot/bin/activate
```
now install telethon using pip and check if it's installed:
```python
pip install --upgrade telethon
python -c "import telethon; print(telethon.__version__)"
```
## Dive into coding
### initialization
So what's the final goal? In the end of the post we have created a bot which goes onto a complex chat, like a super group with topics, and sends specified messages at specified times in the desired topic.

Now let's `import` all modules we're going to use:
```python
from telethon import TelegramClient
from asyncio import sleep, create_task, gather
from time import sleep as tsleep
```
So let's see, what is the very first thing we do before using telegram? We log in using information like phone number and 2FA password (if enabled). Use `TelegramClient` class to create the instance of a client. Then use the `start` method to log in to your account:
```python
# define necessary variables
# fill with your own info
api_id = 1234
api_hash = "1234"
phoneNumber = "+1234"
password2FA = "1234"
session = "/full/path/to/.session/file"

# connect to telegram
client = TelegramClient(session, api_id, api_hash) # the first parameter is name of the .session file, it can be absolute path
client.start(phone=phoneNumber, password=password2FA)
```
> if it's first login or script doesn't fine the *aghaz.session* file it will ask for authentication code sent to you.
### finding topic

It's time to choose a topic to send message, but there is a point, *topics* don't really exits. They're just visual candies in Telegram official client to organize chats. There is no API to interact with topics directly and if you send a message it will appear in a random topic, which isn't what we want. One solution is to reply to a message from that topic, in this scenario our message will be sent in topic of the message we're replying to:
```python
async def get_initial_message():
    async for theMessage in client.iter_messages(entity=chatEntity):
        if theMessage.from_id.user_id == userID:
            return theMessage
```
First, we create the coroutine, which suppose to find and return the [message object](https://docs.telethon.dev/en/stable/quick-references/objects-reference.html#message) of the message we know is sent in the topic. You can use any pattern to get the correct message, here we're looking for message of a user we know their last message is in our desired topic, or we can simply send a test message in that topic, either way the new message that bot sends will reply this last message and being sent in wanted topic.

### set timing
Now we should create a scheduler for our bot:
```python
# determine how many seconds later the message should be sent
def time_to_sleep(**kwargs) -> int:
    current_hour, current_minute = time.gmtime().tm_hour, time.gmtime().tm_min
    time_difference = (kwargs["postAtHour"] - current_hour) * 60 + (kwargs["postAtMinute"] - current_minute)
    if time_difference < 0:  # check if message is scheduled for today or tomorrow 
        time_difference += 1440
    return time_difference * 60
```
this function calculates the amount of time we have to wait for the message to be sent in seconds. note that time must be given in UTC and 24H and decimal format (.e.x 22:8).

### send a message
Now we have everything needed to pass to the final coroutine, including message we need to reply plus message(s) and time(s) they should be sent:
```python
# send the planned message on time
async def sending_message(to_reply, **kwargs) -> None:
    await sleep(time_to_sleep(**kwargs))
    return await to_reply.reply(kwargs["message"])
```
This function takes the message (object) to be replayed on it and one element of the dictionary containing our information.

### run it
And finally establish the main coroutine:
```python
async def main():
	 # get the message to reply on
    initial_message = await get_initial_message()
	 # main loop which keep the bot running
    while True:
        # create a list of coroutines to execute
        tasks = []
        for i in range(len(SCHEDULED_MESSAGE)):
            tasks.append(sending_message(initial_message, **SCHEDULED_MESSAGE[i]))

        new_message_to_reply = await gather(*tasks, return_exceptions=False)
       
        initial_message = new_message_to_reply[-1]  # update initial message
		tsleep(60)
```

And that's it, it's all we need to send a scheduled message in telegram. Now we should create an event loop and execute the `main()` coroutine:
```python
if __name__ == "__main__":
    client.loop.run_until_complete(main())
```
So the code will be running unless an exception occurs.

## Deploy
### set up the script
in the following, I show you a way to run your script on a Linux server. This is not considered as best practice, but does what we need in the scope of this article.
Firstly, let's copy the script on the server. All the code explained here is available on my Codeberg, so you can simply clone it on server 
using `git clone https://codeberg.org/Naadiyaar/Aghaz.git`.

Now change the value of initial variables with your own information and create a virtual environment as explained before, then install required dependencies. As an additional step, you should specify which interpreter must be used to run the script using shebang(#!). All you need is to address the python interpreter in your virtual environment using this syntax:
```bash
#!/full/path/to/python
```
for me, it looks like this:
```sh
#!/home/naadi/py_env/userNot/bin/python3.10
```
### automate it
Right now you can simply execute the program, and it will work, but if anything interrupts the execution process, like a server reboot bot, it will stop working. One way to solve this is to run the script as a Daemon. A daemon is just a service running in the background and managed by a *system and service manager*, **Systemd** in the majority of modern Linux systems. In order for Systemd to manage our bot and keep it running, we need to create a system unit for our service. A system unit is a set of instructions which let Systemd know how to run the bot. Create an unit file by running this command, which creates `/etc/systemd/system/aghaz.service` and writes down necessary text to it.
```bash
printf "[Unit]\nDescription=Aghaz the automatic telegram message sender\nAfter=network.target\n\n[Service]\nExecStart=/root/lyam/aghaz.py\nType=simple\nRestart=always\nRestartSec=1\n\n[Install]\nWantedBy=default.target\nRequiredBy=network.target" > /etc/systemd/system/aghaz.service
```

> YOU MUST HAVE ROOT ACCESS TO RUN THE COMMAND ABOVE

If you open the file, must see something like this:
```
[Unit]
Description=Aghaz the automatic telegram message sender
After=network.target

[Service]
ExecStart=/full/path/to/script.py
Type=simple
Restart=always
RestartSec=1

[Install]
WantedBy=default.target
RequiredBy=network.target
```
I'm not going to explain every line of this unit file here ([man page](https://man7.org/linux/man-pages/man5/systemd.service.5.html)) but it requires you to **change the value** of `ExecStart=` and set full path to where your main python file is located.

To tell Systemd to read our unit file, we need to issue the following command:
```bash
sudo systemctl daemon-reload
```
we can enable and start our system service using following syntax:
```bash
sudo systemctl enable SERVICE-NAME.service
sudo systemctl start SERVICE-NAME.service
```
in our case, the service name is `aghaz.service`, so I will run the following commands:
```bash
sudo systemctl enable aghaz.service
sudo systemctl start aghaz.service
```
Let's check the status of service we just created, since my service is `aghaz.service`, I will change it accordingly. Bellow is the output of checking if the service is enabled and running:
```bash
$ systemctl is-enabled aghaz.service
enabled

$ systemctl is-active aghaz.service
active
```
As the output suggest, our service is running and will start automatically in a reboot.

## Final thoughts
By using Telethon you can develop ideas on Telegram easily up to far limitations. I tried to explain the process of creating a very basic userbot and explaining the what and why of steps I took in a correct manner and avoid making any shape of complexity. Now you should be able to read the [Telethon official reference](https://docs.telethon.dev/en/stable/index.html) and learn deeper into what Telegram have to offer.

**Thanks for reading; please let me know what do you think; good luck.**
