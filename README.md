My mostly unqualified observations of your SMB server
=====================================================

This is a part of a note I recently send to my doctor regarding his data management system. I was prompted to send him this when I found out that he did not know about RAID disk arrays.  As a general text it may be of interest to people who are using off-the-shelf computer equipment for critical business applications but who do not know about things like ECC memory and RAID.

1, Failures of primary storage.

Most hardware manufacturers sell what might loosely be termed business computers and commodity computers. There is often a big price difference between the two, even though they are substantially the same in many ways. Other than obvious differences like quality control or the amount of power/cooling, there is normally only one significant area where business computers (especially servers) differ in functionality that I can think of. This is in how the primary memory system is designed to deal with a number of failures (mainly things like cosmic rays penetrating memory chips) which can change the contents of primary storage (e.g. DRAM). Cheap computers usually have no way to even detect these problems, but as they are comparatively rare, most people can live with these problems. For example, if a small error were to be introduced into document being edited (like an ‘a’ turning into a ‘b’) once every year or two, I expect most people would accept this rather than buying another computer at higher cost. In contrast, the information held on business class servers is normally considered too important to allow errors of this kind to go unchecked. So they normally use what is called ECC memory. This kind of memory will not only detect memory errors, they will, in most cases, automatically correct them. (This is not 100% perfect because it is still possible to have extreme cases of corruption which can go undetected. This is very rare, so ECC is generally seen as a significant improvement over non-ECC memory.)

The problem with memory corruption is that it can go undetected for many years, and the consequences could be very bad. If, for example, in a medical application a code meaning a patent is allergic to Penicillin were changed to indicate the patient is allergic to dairy products, this might result in significant medical consequences (and this could easily occur with a single bit being changed if, say, the code for Penicillin was 1 and the code for dairy products was 65, or 513, or any other value that is 2^n + 1). Depending on how an application is written it is also quite possible for an entire record to become unreadable. Also any of these possibilities could occur for a record that was not being edited because such records are sometimes bound into groups for I/O operations, and while one record is being edited, a different one in the same group is corrupted, which is then written back to secondary storage when the changes are complete.

The use of ECC memory considerably reduces the dangers of all these kinds of errors. Hardware manufacturers are normally a little coy about telling people what consumer hardware will support ECC memory, presumably because they want businesses to purchase a “business server,” at many times the consumer price. However, a little research can reveal some attractive solutions.  For example, I built a server with 16GB of ECC memory and 18TB (6 x 3TB) of disk storage for about $2500 about a year ago. This is not to totally discredit the business option; the main reason people buy IBM equipment is that it comes with someone that has a big sign around his neck that says:  “It was my fault,” and will fix things very quickly when they go wrong.

For more on this subject please see:

https://en.wikipedia.org/wiki/Soft_error

and

https://en.wikipedia.org/wiki/ECC_memory

2, Failures of secondary storage.

Disks can exhibit similar errors despite the fact that even the cheapest hard drives are protected with an ECC at the sector level. This is a different kind of ECC than that used for primary storage, and it is not uncommon for users to encounter an uncorrectable ECC error from time to time. This is the reason people are told to backup their computers. Occasionally a disk may fail completely so that no part of the disk can be accessed. In some cases the disk can be repaired, but this is far from certain, can take a long time, and be quite expensive. All these problems can be solved by the use of a “Redundant Array of Inexpensive Disks” (or RAID). The simplest of these is called RAID 1, or mirroring, where every disk in the system is paired with another redundant disk that holds an “image copy” so that, if one were to fail, all the missing data could be found on the other. This kind of configuration will obviously double the cost of disk storage, and will also lengthen the time it takes to write to the secondary storage a little.  However, this extra time may not be at all significant, because the time it takes to read from this kind of configuration will (in many systems) be faster, making it possible that the server, as a whole, could actually be faster on average (this depends on the average read/write ratio). There are also many other sorts of RAID configurations with various tradeoffs of performance and capacity.

The big advantage of RAID is that in a very real sense you are backing up your whole system automatically and in real-time, every time something is updated. This is not to say that it should be seen as a replacement for normal backups, but it provides a good level of security for data that has been changed, but not yet backed up. If, for example, backups are done on a daily basis, this could prevent the loss of an entire day’s work, often for a very small capital cost, and no significant change in performance.

For more on this subject please see:

https://en.wikipedia.org/wiki/RAID

3, ZFS

ZFS is a RAID system I use a lot. It can be found as part of a very solid and well respected server operating system called Solaris. The only big drawback with Solaris is that it requires considerable systems administration training to use, so as I do not have this training (meaning, incidentally, that you should be very cautious of any of my advice!!!) I use ZFS in a much easier system called NAS4Free, which has a simple web browser interface. The main disadvantage of NAS4Free is that it is never going to be as well tested as Solaris, meaning it must be a less-secure system with which to trust your data. I suspect this may not amount to big difference if you do not stray too far from the standard configurations, and I have never seen any critical problems personally. Many people like me use NAS4Free “for free,” but professional support is also an option as I recall.

ZFS is nice for many reasons. It is does not have a “traditional” RAID-5 algorithm (with RAID-5 you lose less of your total disk capacity to redundancy). Instead, it uses several very clever ideas that directly address a significant performance issue with writes to RAID-5 arrays. This they call this RAID-Z. In my system I use a variation of this called RAID-Z2, which means (in my case) that two of the six disks are used for redundancy meaning that 2/3rds of my original capacity is useable for user data. The advantage of this configuration is that any two of the six disks can completely and permanently fail, and all the data will still be intact. The disadvantage of RAID-Z2 is that writing a little is slower. Another significant difference with ZFS is that unlike most RAID solutions, ZFS requires no special hardware. Commodity disks and disk controllers work very well, and do not compromise the integrity of the system. This comes with a small performance cost because various RAID functions that are often performed in specialized disk controllers must now be done using the CPU in the server. But as modern CPUs are now so fast, the computational time is small compared to the still very long mechanical latency inherent to all disk operations, so the additional time will often be completely insignificant (this is not true for newer "SSD" type disks, but I digress). What is significant is that it is possible to get ZFS to automatically perform a data integrity test on a regular basis that virtually guarantees that all the data is complete and correct at the systems level (so if an application messes things up---a very different kind or problem---this would not be detected). This is an operation called “scrubbing,” where all the data is read from the disk and checked to be correct using an independent and extra level of error checking. This will prove (to some large sigma) that the data is still as it should be. All the user data checked along with the redundant copies, and if anyting is wrong the data is rewritten (and hence scrubbed). This operation checks that various system componants worked correctly when the data was written. These includes the motherboard, the disk controller, the disks, and to some extent the memory (the undetected EEC error I previously had mentioned). Scrubbing can take a long time (currently, 9 hours on my system) and while this is being done the server’s performance is reduced (so I run it at night). If a failure is detected when scrubbing, then an email is send to the operator (i.e. me) to prompt for some action like loading data from a backup or whatever is needed (I set things up so I that I get the server to send a  test email once a day, so that I will notice if this stops working.)

Lastly, ZFS can be configured to make what are called “snapshots,” which are frozen versions of the file system taken at various times. I have my system configured to make one snapshot every hour (each of which are deleted after 24 hours), and one per day (each of which are deleted after a month). This pretty much guarantees I cannot ever do something really dumb like accidentally deleting all the files only to find I had lost everything (the worse would be that I would lose all the changes made over the previous hour). It is also very good protection for the very nasty “cryptolocker” type of virus. One can tune these parameters to suit your own needs.

ZFS is a remarkably well designed system that I have now used on two different ECC-memory based consumer machines for around ten years. During this time I have never seen a single error or loss of data (that I know of).  It should, however, be said that the fact that these systems always run 24/7 (i.e. are not routinely powered down) may be significant in this claim because many computer failures occur when they are powered on or off.

4, Disaster recovery rehearsals.

It is not uncommon for people to setup their backup system and not test them to see if they work. The most spectacular example of this I came across was a company that religiously created tape backups of their mainframe computer and stored them offsite (like you should) only to discover (when it was too late) that on the other side of one wall of the tape store room was a large electricity substation, the magnetic flux of which proved more than sufficient to corrupt every tape in the room.

So it is obvious to test that data can be restored correctly every so often. Big companies often will do this by forwarding a part of each tape store to a different facility, testing it on a weekly schedule. (This adds further geographical distribution, and tests that the backups can be read on a system that was not identical to those on which they were created.)  For a SMB it is probably sufficient to return a backup once every three months or so to see that it can still be read.

5, Other equipment failures.

Even when all of the above is done, and the data is safe, it may be important to consider what the effect of each component failing would be. As computer hardware is getting ever cheaper I’m assuming that the cost of replacement is not an issue (get insurance if it is), but the cost is in the downtime when, say, the power supply in the main server fails and the system is thus unusable while it is being repaired. In some cases adding extra levels of hardware redundancy may be worth considering.  Although I’ve never done it, I understand that it is not hard to have one ZFS based server configured as a “hot standby”, so that it can take over quickly if the main server fails. (Also all this can also be done “In the cloud,” though we are now going into complex areas where I have no experience.) But hardware redundancy should be considered at all levels, i.e. routers and switches are cheap so having an extra router might be a good idea etc. Other things to consider would be battery based temporary power supplies (a UPS), and an alternative internet connation (which might be as simple as using a mobile phone).

6, Security

You may wish to review:

1.	That no one can get into your system from the internet.
2.	Your virus/malware detection (Google “cryptolocker” if you want to see how nasty things are getting these days)
3.	Should your data be encrypted
4.	Do you hold credit card information that could be placed on a non-networked computer
5.	How sure are you that the medical software you have is not secretly stealing confidential information (unlikely, but exactly the sort of thing a sufficiently disgruntled employee might try)
6.	If the building were to be totally destroyed by fire, earthquake, or something like a major aviation accident, you could setup a replacement data system in a short enough time so as not to inconvenience anyone.
7.	All data is maintained in accordance to any necessarily regulations.
8.	Are your backups encrypted. This is especially important if you use "The Cloud" as your offsite storage. If you do this then a) you must use very strong encryption technology that you control and that they have no knowledge, b) never use any encryption technology you are offered by anyone (including me, your uncle Fred, or your old friend Sam from school) c) that you use at least two and probably better three, different suppliers (and you can prove these are not virtual fronts for one physical provider). so you know that if one were to fold, you could access the same data from the other. Personally, I might go for one copy at home and another in the cloud, or better still find a "data partner" where you keep each other's encrypted backups. In all cases I would seek out the best specialist advice because this is an area where some awful security mistakes are possible.

Note also that many people employ computer security specialists to try to hack in to their systems in order to expose weaknesses. This may be worth considering, but you will have to satisfy yourself that they are completely trustworthy.

7, My configuration:

* 1 x Supermicro X9SCM-F Server Motherboard - Intel C204 Chipset - Socket H2 LGA-1155

* 1 x Intel Pentium CPU G2120 @ 3.10GHz

* 6 x Seagate Barracuda 7200.14 ST3000DM001

* 2 x Kingston 8GB 240-Pin DDR3 SDRAM DDR3 1600 ECC Unbuffered Server Memory w/TS Model KVR16E11/8

* 1 x Fractal Design Define R4 Black Pearl w/ USB 3.0 ATX Mid Tower Silent PC Computer Case

* 1 x SeaSonic SSR-360GP 360W ATX12V v2.31 80 PLUS GOLD Certified Active PFC Power Supply


(One nice feature of the motherboard is that it can be "headless," i.e. without a keyboard, mouse, or display.)

