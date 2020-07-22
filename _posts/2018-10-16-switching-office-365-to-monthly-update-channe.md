---
layout: post
title:  "Switching Office 365 to Monthly Update Channel"
date:   2018-10-16 12:00:00 +0700
categories: office365
---
# Switching Office 365 to Monthly Update Channel

Office 365 recently updated on my Surface Book 2 and I noticed it has a refreshed look and feel to it and I liked that a lot! I especially like what has happened to the Ribbon:

![image](images/image_636753160613007211.png)

Figure 1: Collapsed

![image](images/image_636753160630366388.png)

Figure 2: Expanded

After using this for a while I switched to my desktop and **HORROR** it was the regular look and feel. A quick check of the versions and I saw that my desktop was on the "Semi-annual Channel" and my laptop was on the "Monthly Channel (Targeted)" – what to do? After some digging, I found the steps necessary to switch channel:

1. Open the command prompt as an admin

2. In order to switch the channel, enter the following:
    
        OfficeC2RClient.exe /changesetting Channel=Insiders
        

3. In order to start the channel update, enter the following
    
        OfficeC2RClient.exe /update user
        

The office update process should launch. Once this has completed you should be able to relaunch Outlook and see that the version has changed.

![image](images/image_636753160635873434.png)
