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

# Level 12
**Objective: Uncompressing files**

We have a data.txt file which has compressed hexdata in it. It looks like this:
```
00000000: 1f8b 0808 10da cf69 0203 6461 7461 322e  .......i..data2.
00000010: 6269 6e00 0140 02bf fd42 5a68 3931 4159  bin..@...BZh91AY
00000020: 2653 59e1 71be e800 0018 7fff dec6 ff7c  &SY.q..........|
00000030: bd9f 4fbf ff77 ffff bfed af5d bffb dffd  ..O..w.....]....
00000040: a8fa cfdf fbfb ffbb dd7f f5fb b001 3b18  ..............;.
00000050: 1006 83d4 0340 d000 1934 0034 0006 81a0  .....@...4.4....
00000060: 00d0 000d 0034 0d0c 8000 0d1a 3406 8068  .....4......4..h
00000070: 69a6 4d1a 0d1b 48da 40da 3510 0003 4006  i.M...H.@.5...@.
00000080: 8000 001e a00d 001e a680 3400 01a7 a800  ..........4.....
00000090: 0680 c4d0 000d 1a3d 11ea 1a00 d343 f541  .......=.....C.A
000000a0: a006 269a 03d4 0e9a 1a68 3434 340d 0d06  ..&......h444...
000000b0: 8193 400c 8320 0340 3434 68d1 a000 68c4  ..@.. .@44h...h.
000000c0: 6026 2000 1a06 2000 064d 000d 0000 6432  `& ... ..M....d2
000000d0: 3c08 0200 4056 d394 6653 6796 5b22 e9b8  <...@V..fSg.["..
000000e0: da82 c52c 0888 c1d0 6cee 6a43 f164 4a14  ...,....l.jC.dJ.
000000f0: 6b4a 1d69 111a 91c1 93db ee12 8667 ca43  kJ.i.........g.C
00000100: d036 43f6 3d4f 4999 6065 4091 9a2f bc4d  .6C.=OI.`e@../.M
00000110: 6516 68e6 34ef a4ce 1091 b9ea 52a7 cf48  e.h.4.......R..H
00000120: 3e4f 84c1 a2c5 2383 200a c41e 28ed 8e9b  >O....#. ...(...
00000130: 7868 a526 970b 4041 054d 3b25 c0bb 6bdf  xh.&..@A.M;%..k.
00000140: 1afe 9771 045e 3213 58a5 d129 9cd8 3dd8  ...q.^2.X..)..=.
00000150: 9ca1 2561 c91b 1527 afc0 5643 0425 45ea  ..%a...'..VC.%E.
00000160: dc87 cf98 2104 c30f 01ad 19fb 7e34 c0ba  ....!.......~4..
00000170: 30e1 135a 743d f3d4 6467 cb43 9f4e 0cc1  0..Zt=..dg.C.N..
00000180: 052a 12c1 55f3 2344 2254 b108 6571 016d  .*..U.#D"T..eq.m
00000190: caab c4f6 8c3c e383 2e61 1088 490f 588b  .....<...a..I.X.
000001a0: e6a4 e14a 8cc5 c226 9950 c091 3c2c 6ec5  ...J...&.P..<,n.
000001b0: 7150 851a ac29 1272 422b 3c62 0da4 1bd7  qP...).rB+<b....
000001c0: 605d 7981 aa02 332b bb27 9358 bac9 6ddc  `]y...3+.'.X..m.
000001d0: 1aae 9848 0ff1 46cb c3a0 1f43 9871 0ef8  ...H..F....C.q..
000001e0: 4429 ca3b 9fab 2e74 2b96 6f24 ad53 e4ad  D).;...t+.o$.S..
000001f0: e247 28c8 86d4 0ec0 10ad 412a 0fec 11bc  .G(.......A*....
00000200: 6cd6 3c01 ff5f 8f88 9247 582a 4d44 4942  l.<.._...GX*MDIB
00000210: 92d2 5f6b 61d4 2d2b 5723 179d 98cc a44c  .._ka.-+W#.....L
00000220: 951d c6c6 f143 2af1 5219 1fdd 3e81 8dc4  .....C*.R...>...
00000230: c586 98f0 98e4 d5bd 910c f59a 0142 864b  .............B.K
00000240: b8f2 08f3 65d4 9d5d 5e29 0130 fe7f c5dc  ....e..]^).0....
00000250: 914e 1424 385c 6fba 0081 589d 8f40 0200  .N.$8\o...X..@..
00000260: 00
```

Overall unreadable, but there's a clue in there! The level description here reveals there's *multiple* compressions on this file. Notice the "data2.bin" at the top? This is an indicator of this and will be relevant later! 
I had to explore the basic compressing tools included in linux for this assignment, but with some trial and error I took these steps. 

## Step 1: xxd

The xxd command creates hexdumps from files. Not unsurprisingly it also has an option to revert compressions, so we do just that. It's a simple manner of adding the revert argument and writing it to a new file:
``xxd -r data.txt > unhexed.txt
``
The file is certainly uncompressed now, but is more unreadable than ever. This is because it's been converted back to binary and isn't human-readable. So what now?
It's always a good idea to explore what you're dealing with when it comes to files. Linux is very accessible when it comes to this by running the `file` command. By running file on unhexed.txt we get an interesting output: 
``unhexed.txt: gzip compressed data, was "data2.bin", last modified: Fri Apr  3 15:17:36 2026, max compression, from Unix, original size modulo 2^32 576
``
It's no longer a txt but a gzip file! That's the next compression form we need to focus on.

## Step 2: gzip

Let's start by making sure gzip recognizes the file for what it is. We rename the file:
``mv unhexed.txt unhexed.gz`` 

mv is also used to move files, and it the process you can rename it. Since we don't specify a path, we skip that step and just rename it. Now we can run a gzip decompression:
``gzip unhexed.gz -d``

What we get now is an unpacked file without the gzip extension, in fact it doesn't have any at all. Let's run file on it again and see what we get: 
``unhexed: bzip2 compressed data, block size = 900k``

## Step 3: bzip2

You know the drill! We need to rename the file so bzip2 will recognize it, then start yet another decompression:
``bzip2 -d unhexed.bz2`` 

What do we end up with this time?
``unhexed: gzip compressed data, was "data4.bin", last modified: Fri Apr  3 15:17:36 2026, max compression, from Unix, original size modulo 2^32 20480``

## Step 4: gzip (again)

We just repeat step 2 here, nothing new. It does yield a new result when looked at with file though:
``unhexed: POSIX tar archive (GNU)``

## Step 5: tar

You know the process by now. Rename the file with the .tar extension and unpack it with:
``tar -xf unhexed.tar``

NOW things are getting interesting! We've used the name "unhexed" for these files we've uncompressed to. But now a new file has appeared with a name we haven't set ourselves: data5.bin. Remember how the file looked when we started? It had data2.bin in cleartext in there. If we look at it with file though, we'll find it's a another tar archive file. Let's rename it and run the same tar command on it before moving on.

## Step 6: Repeating steps

The remaining parts are repeated methods from step 2-5. We eventually get a file that's ASCII text and wouldn't you know it, we finally have our next password!

# Level 13
**Objective: Use a SSH key**

So far we've used ssh connections using a username and a password. This time, we need to figure out how to use an ssh key to connect to the next level. 
The level provides us with a private SSH key. Our first step is to copy it to our computer. The manual page for scp reads:
```
The source and target may be specified as a local pathname, a remote host with optional path in the form [user@]host:[path], or a URI in the form scp://[user@]host[:port][/path]. Local file names can be made explicit using absolute or relative pathnames to avoid scp treating file names containing ‘:’ as host specifiers.
```

Since we're SSH'd into a remote machine that we know have ports open, we quit the connection and use our local terminal to start a remote connection. There's some important things to note here:
- The SCP protocol uses SSH as its default setting. Bandit uses a non-standard port for SSH, so we need to define that in our command.
- Copying to certain folders might be denied due to lack of privileges. I run Windows 11 for this room and can't copy to C:\ so I will use my user folder instead (real name censored).

The command we'll run looks like this:
```
scp scp://bandit13@bandit.labs.overthewire.org:2220/sshkey.private C:\Users\(MyUserName)
```

Since we're using an SSH connection, it'll prompt us for the same password we used to SSH into the machine. After that, the download will start and we'll have copied a file from a remote machine to our local machine!
With SSH key in hand, we can connect to the next room by moving the the same directory that holds the downloaded key and running:
```
ssh -i .\sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```

Presto! We entered the same remote machine by using an ssh key for identification instead of a password!

# Level 14
**Objective: Connect to localhost port**

Now that we've logged into the same remote machine as another user, we have permission to access a folder holding a password. We need to submit this password to port 30000 on localhost to receive the next password. 

**OPTIONAL: What service is running?**
So we know the port, but what's running there? This machine provides us with nmap, a port scanning tool with a variety of options so we can find out!

## IMPORTANT NOTE
```
nmap is a troubleshooting tool that is also commonly used in ethical (and non-ethical) hacking. Running port-scanning tools without authorization on machines is a legal grey zone and could potentially lead to legal repercussions. Most enterprise networks have detection systems that could trigger alerts if their ports are scanned. In this case we only scan our own ports in a controlled lab environment.
```

We find out port 30000 runs "ndmps", the Network Data Management Protocol. 

### Connecting

We got several options here. The old-school way is to use telnet which certainly works, but comes with security concerns as the traffic is completely unencrypted which we want to avoid in at least 99% of cases. We will use Netcat which isn't completely security safe either, but at least it supports of encryption. Netcat can create listen ports, serve things like webpages or scripts on ports, or in our case connect to a port's service. The syntax for connecting to a port is very simple:

```
nc localhost 30000
```

We provide the password we found in our file and are rewarded with the next password in response!

# Level 15
**Objective: Connect to localhost port (with SSL/TLS)**

The premise for this room is the same but kicked up a notch. While looking up how to do this I found that SSL is notoriously complicated to use, made even harder from how it's been made more secure due to big cyberattack incidents. I will admit I have a hard time grasping the specifics, but in theory it's no different from Netcat. We enter a host we want to connect to and what port. The full command I used looks like this: 

```
openssl s_client 127.0.0.1:30001
```

**NOTE:** The guide I used is from the third edition of OpenSSL Cookbook by Ivan Ristić. I stripped down unnecessary complexity from his example which might be needed when connecting to an actual remote server. Since we do this from localhost, it wasn't that complicated. 

We get A LOT of diagnostics data output from using this command showing the certificate handshakes. When it's finished we are prompted for the password (the same one used to connect to this server) and receive the next one in response!

# Level 16
**Objective: Find the right SSL/TLS localhost port**

This time, we need to repeat the last objective, only we only know the span of ports that has what we seek (port 31000 to 32000) and not the exact port. I actually did this with nmap in Level 14, not knowing it was gonna be an actual objective later. In that level, we scanned all ports but now that we have a specific span we can fine-tune our scan to only focus on those. 

```
nmap 127.0.0.1 -p 31000-32000 
```

We end up with 5 open ports all running unknown services. One of these will return our new password while the others will just return the one we entered. With such a low number of options, we could manually connect to each and see if we get a response, but let's see if we can reduce the number of likely ports.

Open ports:
- 31046
- 31518
- 31691
- 31790
- 31960

I realized after running this that we can also use nmap to probe each open port to figure out what services are actually running there. 

```
nmap 127.0.0.1 -p 31000-32000 -sV
```

We recieve the following output:
```
PORT      STATE SERVICE     VERSION
31046/tcp open  echo
31518/tcp open  ssl/echo
31691/tcp open  echo
31790/tcp open  ssl/unknown
31960/tcp open  echo
```

There we have it, 2 SSL ports. The one running ssl/unknown is definitely the most interesting and likely the one we're looking for since the others all have echo. We'll connect to it the same way we did last level:

```
openssl s_client 127.0.0.1:31790
```

Now we get prompted for the password, but when entering it, the only returned value is "KEYUPDATE". Why is this? 

### Openssl-s_client specifics

If we go into the manual page for openssl-s_client we find the answer. This type of SSL connection has a couple of different modes it can connect with and unless specified it will default to the "basic mode". In this mode inputs are interactive; meaning that certain letters are bound to commands with different functions. When one of these letters are put in, everything after it is ignored and the command is run. Writing k runs the command "Send a key update message to the server" and wouldn't you know it, our password happened to start with a lower case k. 

The manual page also describes the other modes. You can start in "advanced mode" with has even more options, but what we're looking for is to disable the interactive mode by using the -quiet command. The command for starting the connection will look like this: 
```
openssl s_client -quiet 127.0.0.1:31790
```

A lot less information gets dumped on us this time, including the prompt for a password. But as we know that's what it wants we can enter it anyway. This time the server will return the next password!

# Level 17
**Objective: Compare two files to find the changed line**

We have two files in our home directory; passwords.new and passwords.old. One of the passwords has been changed from the old file to the new one. This is a simple case of using the diff command:
```
diff passwords.old passwords.new
```

# Level 18
**Objective: Bypass automatic SSH logout**

This level has the next password in a file named readme in the home directory. However, .bashrc has been modified to log us out as soon as we've logged in with SSH. How to solve this?

We actually did solve this in an earlier level, we just have to think outside the box. If we can't manually access the file after connecting, we just have to make something fetch it for us while using the connection. Since we know the name of the file and exactly where it is, this is a perfect use case for scp! We use it the same way as in Level 13:
```
scp scp://bandit18@bandit.labs.overthewire.org:p:2220/readme C:\Users\(MyUserName)
```
After copying the file, we can easily access it on our own machine!

# Level 19
**Objective: Get file contents by raising permissions through setuid**

Our home directory has an executable file named bandit20-do. While we can't read the script, it uses setuid to run whatever command is put into it as another user. The example provided is trying to run *whoami*, which displays what user you're currently logged in as. Running it as is returns **bandit19**. But if we execute the script with whoami as an argument, it returns **bandit20**.

So what can we do with this? Different users have different home directories, but more importantly they have different *permissions*. Now that we're able to run commands with another users permissions, we could for instance look inside their home directory, something we can't usually do. 

The OverTheWire/Bandit games have a folder that contains the passwords for all levels, but you only have permissions to read the levels you've already completed. Since we run commands as the user from the next user, we can just fetch the password for the next level right now:
```
./bandit20-do cat /etc/bandit_pass/bandit20
```

We're presented with the content of the file, something we won't be able to read without running it through the script.


# Level 20
**Objective: Use setuid script to connect to localhost port**

I had some misconceptions about how this room was structured and had to get some help from an online guide. We have another script called *suconnect*. I was under the impression that this scripted worked by providing it a port number for a connection and then enter a password, but this is incorrect. It's hard to know exactly what the script does since we can't look at the code, but here's the description of the level:
```
There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

**NOTE:** Try connecting to your own network daemon to see if it works as you think
```

So the password needs to be fed into the script but not as an argument while running it, but given through a port connection. What we do is start netcat listening on a port, like we've done in earlier levels. I used tmux to split the terminal view into separate instances, so I could start the nc listen and look at the feed while using another terminal instance to connect to it. We then add an echo of our password to it, like so;
```
echo -n '***PASSWORD***' | nc -l -p 1337
```
Now we just need to run the script towards that port:
```
./suconnect 1337
```
We get a confirmation of the string that has been read by the script, and that it does indeed match the password! The password for the next level shows up in our nc listen feed!


# Level 21
**Objective: Investigate automated process.**

The level description tells us of a program that runs at regular intervals from cron, the time-based job scheduler and that we should investigate /etc/cron.d for what command is being executed.

