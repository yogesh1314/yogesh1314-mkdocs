# How to Automate Screen Timeout in iPhone

Ever felt the need to have your iPhone's screen timeout to 'Never' just when you are cooking, reading, etc. and then switch back to normal '30s' or '1min' afterwards

We iPhone doesn't have a straight away solution for that because you can't see it in 'Shortcuts' app when trying to automate it

Then how do we do it ?

There's a workaround.

First decide how you want to trigger the screen timeout, for example : We want to set the screen timeout to be 'Never' during 10am to 8pm and then it switch back to normal 
Or, you can have your own trigger : **"Hello Siri, start cooking"**

Now comes the real trick, we can't automate screen timeout directly but we can automate battery saver mode and we will use same for our automation

So, default behaviour of Battery Saver Mode is it switches the screen timeout to 30 seconds when it is ON, and it once it's off it switches screen timeout back to what it was before.

First

1. Go to Settings 
2. Go to 'Display and Brightness'
3. Go to 'Auto-Lock'
4. Set it as 'Never'

Now, Let's setup our Shortcut which will set screen timeout to Never during our activity

1. Open 'Shortcuts' App
2. Go to 'Automation' Tab
3. Click on top right at '+' sign to create new automation
4. Select your trigger: 'Time of Day' 
5. Select time and when to repeat it - 'Daily', 'Weekly' or 'Monthly'
6. Select if you want to confirm before Automation kicks in, or let it start in background - Select 'Run After Confirmation' or 'Run Immediately' 
7. Go to Next at top right 
8. Search for Low Power Mode
9. Select it
10. Set it to `Turn` **Lown Power Mode** `Off`
11. Done

Now screen timeout will be 'Never' and you can Cook, Read do whatever you want without touching your screen to stop it from dozing off

Now once you are done, let's setup an Automation to go back to Normal mode or Turning Low Power Mode which will set the screen timeout back to 30 seconds 

1.  Open 'Shortcuts' App
2.  Go to Automation Tab
3.  Click on top right at '+' sign to create new automation
4.  Select your trigger: 'Time of Day' 
5.  Select time and when to repeat it - 'Daily', 'Weekly' or 'Monthly'
6.  Select if you want to confirm before Automation kicks in, or let it start in background - Select 'Run After Confirmation' or 'Run Immediately' 
7.  Go to Next at top right 
8.  Search for Low Power Mode
9.  Select it
10. Set it to `Turn` **Lown Power Mode** `On`
11. Done

That's it !! Enjoy :) 
![error](static/error.gif)