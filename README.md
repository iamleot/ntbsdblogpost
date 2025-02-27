# NetBSD Blog post
This summer I worked on NetBSD's ATF for coverage of root file system and root device selection. This area of subsystem is not very well documented and process flow has to be determined by looking through code. this is not very feasible for quick debugging.
but it does its work and has been working for 30+ years,but Christoph Badura experienced that RAIDframe gives problem with higher capacity drives when finding for root but in any case fallback option is asking user for device and partition but manually doing it with hundreds of server is tiresome and hence the need for testing coverage which will eventually turn into rewriting of whole process code. 

I would also like to tell about my early interactions with project ,let me start from project finding 

## Why NetBSD ?
2024 gsoc is not my first time application my first time was in 2022 in my first year of College, i was fascinated with newer Windows versions
since my core duo intel i3 days as every new Windows versions used tobring new changes and features, from windows XP to Vista to 8.1 and then to 10 (cosmetic only) but was very heavy on 
my PC idle memmory uasge was upwards of 60% leaving very little room for exotic applications and games But a little later when we had access to (usable) internet connection, I got to know that there was a world beyond Windows,an entire Universe of UNIX like OS, which didn't require a top level PC hardware my 2gb intel i3 would be enough for it. the above whole experience led me to decide that I wanted my project to be in operating system space.I ventured all OS projects on GSOC website first was Linux gentoo , debian but that didn't turnout well therefore I decided to go with BSDs, community seemed lot more friendlier, responsive and active to help beginners. I started mailing all the mentors about my interest ( I realised should've done a lot more work before mailing them ,it shows more dedication towards project)

Chritoph replied to my mail with positively and gave me a headstart explaining project details, I had problem with x86 qemu vm to start with NetBSD and he helped me graciously, investing his valuable time to help me boot with VM on debian machine. After exchanging some more mails we developed timeline and objective and submitted my propsal and fortunately it was accepted.

## Project Details 
root device and file system selection is made during later stages of boot process by kernel. config file defines candidates for device and kernel selects it given, some conditions meet
criterion, you get head over to my docs for more details (here)[] . This functionality is handled primarily by setroot function in file kern_subr.c it also has specialised associated 
functions for different cases. Our task was to add tests in ATF for this function and some other functions that assists setroot. This part of kernel works, and it works for over 30 year
but code is rather complex to read and there is no documentaion, only way to debug is to read code. We will try to rewrite this code. 
at any given stage when any condition fails the fallback option is to ask user for device. Thus this part of kernel rarely needs attention but Chris was having problems with RAIDframe 
setup with higher capacity drives, this time too fallback option worked but in setup of hundreds of server this will be a problem. 

there are some global function used in conditions inside setroot rootspec ,bootspec etc. these functions are either set through config file or through other machine dependent kernel 
function findroot etc. we need to modify them in our testcases and also need to stub kernel functions used by setroot and other function. these testecases will eventually be ported to 
use vnd. We check for global variables and if a local variable is to be checked we intialize it. 

since root_device can also be a network device ,this has to be defined in config file, we alto test tftproot_dhcpboot function. this function can also set root_device.  

