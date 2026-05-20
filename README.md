# PORTFOLIO
## Rikard Swahn
### Cybersecurity Engineer

# Introduction
Welcome to my portfolio! I am currently studying at a two-year education to become a cybersecurity engineer. My primary interests lies in networks and forensics, but I have put a lot of effort into learning red-team and pentesting skills to better understand security. 
This portfolio include both projects I've done from education classes and other personal projects. The education projects tend to be production-driven while the personal projects are mostly case studies or walkthroughs. Note that some of them have been collaborative efforts. 

# Education Projects

## Automation and Virtualization

### SIEM-in-a-box
This project was a collaborative project in groups of 2. It was done over 8 weeks, but included introductory lectures, so the actual production time was just under 5 weeks. 
Our task was to create Code-As-Infrastructure in virtual machines using Vagrant and Ansible. We used Virtualbox for our virtual machines. If you want to clone the repository and run it for yourself, you will need Virtualbox, Vagrant and Git installed. 
You can find the repository here: https://github.com/rswahn-sw/project-siem

We were presented with several examples of projects we could do for this class, but we decided to go for our own idea. We wanted to make our own SIEM setup that is able to monitor a webserver, send logs that is formatted and visualized. We chose this for several reasons;
- Several of the projects presented were simple in nature. We wanted something that would be a bit challenging so we wouldn't be done way ahead of the deadline.
- The other presented projects usually had a very specific goal that we didn't see a way to improve once it was achieved. A SIEM setup had potentially endless possibility of customization and expansion.
- At this point in the education we had heard of Security Operations Centers and the basics of a SIEM, but seen very little of it. We wanted to set up a simple environment to see how it was used in practice and if it was a potential career path for us.

Please note that while I personally had some self-learned programming experience with JavaScript, neither me or my partner were programmers at the time of the project. There is bound to be some weird things and potentially questionable decisions in our code that doesn't follow best practice. Regardless, I'm proud of the product we were able to deliver.
AN IMPORTANT NOTE: You'll find that the security of the project is highly insecure. This is because we put all focus into making a working proof-of-concept model that would be improved on in the upcoming class: Security Hardening. 
