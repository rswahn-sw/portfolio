# Introduction
This is a walkthrough on the Linux learning game Bandit from Wargames on overthewire.org. I will make notes explaining the commands used, how they work and personal thoughts.

The game comes in 35 different levels. Each level is a machine connected to with ssh which requires a password. The first level has its password provided on the website, after that the password has to be found to gain access to the next level. The levels are designed to teach the users about useful linux commands. 
**Please note** as per the creator's request that this document will not contain any of the passwords needed to progress. I will however write out detailed descriptions about how exactly you reach them, so if you want to play the games for yourself you should *NOT* read this guide. 

Without further ado, let's get started!

# Level 0
**Objective: Perform SSH connection
SSH password: bandit0**

This is simple. We grab our terminal of choice (I will use the standard Windows Command Prompt (cmd.exe is NOT my favorite for the record, but seeing how we'll only use it to ssh into linux environments and go from there the terminal chosen doesn't really matter)) and SSH into the first machine by typing:
``ssh bandit0@bandit.labs.overthewire.org -p 2220
``
Let's break this command down:
- **ssh** - We use the ssh protocol that defaults to port 22. This allows us to start a remote connection to a machine and access it through our terminal as if we were there in person.
- bandit0@bandit.labs.overthewire.org - This shows ssh where we want to go using a user:destination structure. Our username we log in with is bandit0, the @ shows we want to reach whatever follows, which is the website url. This might as well be an IP address which is more common in a local network. 
- **-p 2220** - This is an added argument. Since this challenge is done over the internet and anyone can accesses, the creator (understandably) doesn't want to use the default ssh port 22 for security reasons. Since ssh defaults to port 22, we tell it "hey, use port 2220 instead".

We successfully ssh into the machine! The password is readily available in a text file right there in the home folder. We use this command to look at its contents:
``cat readme`` 

# Level 1
**Objective: Open unconventional filenames**

The file containing the next password is named "-". This makes it hard to open since the Linux terminal uses - to register commands. `` cat -`` won't work here on its own, but if we define that we want to look at this file INSIDE THIS FOLDER, the terminal will understand what we want to do. We do this by writing:
`` cat ./-`` 

# Level 2
**Objective: More unconventional filenames**

The next file is named "--spaces in this filename--". The way the linux terminal use spaces is to separate arguments from each other. 
`` cat file
[cat] - I want to use cat
[file] Use cat on this`` 

This is why we avoid spaces in filenames on linux, since it can't be read by the terminal by default. So how do we fix this? 
We make the filename into a string variable! A string can contain any number of characters including spaces; but it still count as one argument in the terminal. We define our string with "", like so:
`` cat ./"--spaces in this filename--"

# Level 3
**Objective: Find hidden file**

We're directed to a directory that contains the file we need, but it's a hidden file. The linux command for showing directory contents is ``ls`` but in this case it comes up empty. We simply needs to add an argument to it that includes showing hidden files:
``ls -a`` 

We find the file "...Hiding-From-You" which contains the next password.

# Level 4
**Objective: Find the only human-readable file**

This directory contains 10 files, "-file0x" where x is 0-9. Trying to read a file at random will only show gibberish encoded data. 

## The easy way

It's only 10 files, so we could simply brute-force our way to the password. Either we use ``cat`` on each file until we find readable text, or find out which file's filetype is ASCII text with ``file ./-file0x``. It won't take us long to find the password. 

## The best way

This exercise only has 10 files, but it would be awful if there were 100 files or more. Since there's a good naming convention here, we could write a script that checks each file until we find readable ASCII text. I'm gonna write a simple bash script for this.

```
#!/bin/bash
filename="./-file0"
number=0
while [ $number -le 10 ]; do
   file $filename$number
   ((number++))
done
```

I had to recreate this challenge in my own linux machine to try this as this environment doesn't allow scripts to be run or files to be created (which is understandable). So for this specific challenge, brute-force is the way to go. 

# Level 5
**Objective: Find the file with specific properties**

For this challenge we need to find a file inside a long list of directories. We do know some specific things about this file though:
- It's human-readable
- It's 1033 bytes in size
- It's not executable
We can use any of these as filters for our search. I'm gonna go with option 2 since there should be few to none files to have the exact same file size. 

``find . -type f -size 1033c`` 
Let's break this down. 
- find is the command we use that finds files.
- the dot (".") lets it know we want to search this directory and all subdirectories within it.
- -type f lets it know we are searching only for regular files. It works without it too, but this makes the search more exact.
- -size 1033c defines the search criteria as look for file size. 1033 is the size we're looking for, and the c defines the size as bytes specifically. 

We find our file quickly and can use the ``file`` command to confirm it matches the other criteria as well. 

# Level 6
**Objective: Find the file with specific properties (again)**

Similar to last level, we're looking for a file with the following properties:
- It's owned by user bandit7
- It's owned by the group bandit6
- It's 33 bytes in size
This time the file is somewhere in the entire machine, not just in a subdirectory from home, so we gotta change our previous command with some new arguments. Let's compare the two:
**Level 5:** ``find . -type f -size 1033c`` 
**Level 6:**``find / -type f -size 33c -user bandit7 2>/dev/null`` 

- We change . to / to look at the root directory and everything below instead of our current directory.
- The size is changed to match our new file.
- We add the -user command which we know in bandit7
- Finally, searching all directories is bound to flood us with errors because we don't have permission to search the entire machine. So we add `2>/dev/null` to only show successful searches and not the ones that return errors.

We only find two files now! We only have permission to read one, which contains our next password.

