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
- -type f lets it know we are searching for a certain file type, specifically only regular files. It works without it too, but this makes the search more exact.
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

# Level 7
**Objective: Filter file contents**

We have a single file presented to us; data.txt. It contains the password, the catch is that the file is HUGE and will take a very long time to read through. We do have a hint though:
- The password is next to the world *millionth* in data.txt.

This is exactly what the command *grep* is made for! We simply specify what information we're looking for, and what file we're searching, like so:
``grep "millionth" data.txt``

The reason millionth are in quotation marks is because we tell linux this is a *string*, not a command. We ran into this in Level 2 as well. Running the command easily gets us the password!

# Level 8
**Objective: Filter file contents (again)**

Similar problem, but it won't be so easy this time. We don't know what line we're looking for in data.txt this time, but we that the password is on the **only** line that only occurs once in the file. So we're looking for a unique line, perfect for the *uniq* command! As you might've guessed, uniq looks for unique lines, but there's a catch: It doesn't work well when the lines are shuffled all over the place. So what do we do?

## Introducing piping
Linux is very handy in how it allows multiple commands to be run as one through *piping*. We send the output from one command into the next command and so can create a chain of events instead of doing it one at a time. 
Before we run the uniq command, we need to sort the file. This might sound like it's time-consuming or will need editing of the file but linux got us covered there with the *sort* command.

``sort -h data.txt | uniq -u`` 
- the -h argument on sort means human-numeric-sort. Instead of a pure numeric sorting (the -n argument) we sort in a way that makes sense to humans. For this exercise, either will work fine.
- | is the mark of the pipe. Everything to the left of it will be sent to the right side of it. If you do more than one pipe it will create blocks and will only send each block over to the next.
- -u on uniq simply stands for unique. We want that unique line!

# Level 9
**Objective: Filter file contents (once again)**

Another data.txt, another password. This time the file is filled with non-human-readable data, just a bunch of gibberish to a non-computer. What we know is that the password is in one of the human-readable lines in the file, preceded by several "=" characters. That's plenty of information to go on.
By default, grep has trouble reading files that contains binary data, but there's a very handy argument to force it to process it as if it was just text; *-a* or *--text*.

``grep -a "===" data.txt`` 
For clarity, the other readable lines in the file contains:
========== the
========== password
========== is
followed by the password. Classy! 

# Level 10
**Objective: Decrypt file contents

Forget the binary data, now we got base64 encoded data! Base64 is named from its 64 characters it uses to each represent a 6-bit segment of a sequence of bytes. This allows binary data to be transmitted through channels that only supports text. Of course, it can't be read while encoded so we need to decode it first. 
The room hints that we may want to look into the *base64* command. Wouldn't you know it, there's a very simple argument that decodes data, just as the same command can encrypt it.

``base64 -d data.txt``

# Level 11
**Objective Decrypt file contents (again)**

This time, the encoding is different. The room tells us that all lowercase (a-z) and uppercase (A-Z) letters of data.txt have been rotated by 13 positions. 
This is what's called a *caesar cipher*. It's been around for a very long time and as such, it's rather simple (but efficient). You rotate each character of the data x amounts of steps to the left or right of the alphabet. The "alphabet" in this case could include however many letters, numbers or characters you decide on beforehand but the principle is the same. We simply shift the characters back to their original position to get the cleartext. 

In this case we know exactly what direction to shift it and how many steps (This variant of caesar cipher is called **ROT13** and was used by Julius Caesar himself! The ciphers unsurprisingly was developed in ancient Rome). These days there's a wide array of online tools to decrypt caesar ciphers easily even if you have no idea how many steps it's shifted, but we're gonna do it with only linux commands. It'll end up like this:

``cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'``
- First we read out the contents and pipe it to our next command
- tr stands for translate, which takes two strings and replaces characters in the first string with corresponding characters in the second string. This is where it gets a bit complicated.
- 'A-Za-z' targets all uppercase and lowercase letters. Remember our room description? Since this is a ROT13 cipher, ONLY letters are rotated so we wanna make sure we target only them and not any numbers or other characters. 
- 'N-ZA-Mn-za-m' is our cipher description. A-Z from our previous string maps to N-ZA-M. We're telling tr "For each letter in A-Z, N should become Z and A should become M". This works great because ROT13 shifts letters 13 spaces, and the alphabet is 26 letters long so no matter what direction you shift it you'll end up with the original message. Obviously A-M is the first 13 letters in the alphabet and N-Z is the last 13, so we've told it to shift it 13 steps! Then we do the same with the lowercase letters. 

## Personal notes
This was the first level that took a bit of brainpower for me. Ceasar ciphers are simple and easy to understand, but the tr syntax took a while for my head to grasp. Maybe it was the lack of coffee. 
I also learned the cat is unnecessary since tr can handle input on it's own by running: 
``tr 'A-Za-z' 'N_ZA-Mn-za-m' < data.txt`` 
But I thought it was nice to write it out in a way that's easier to understand. 