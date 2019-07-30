Randsomware is a specific example of a hostile data destruction attack.  In the case of randomware, the data still exists but we do not want to pay the randsom, so for the most part we should assume that we will be unable to access it.

## persistent attacks

A key aspect of a randsomware attack is that the attacker is determined to deny you the use of your data.  If you have backup the randsomware author knows that you will not pay the randsom.  For this reason they will also attempt to destroy your backups and will likely persist in your system, waiting to carry out their final attack and trying to destroy your backup system until they believe that your backups are no longer valuable.

There are a number of countermeasures to this long term persistency:

* ensure that you have some form of active monitoring and/or malware detection
* use an isolated system for backup verification
* uss write protection to ensure attackers cannot destroy old backups

### restore to an isolated system regularly

If you have an isolated restore system that cannot be controlled directly along with your normal system and on which you can verify the integrity of your data, the chance of the restoration process later being unable to restore your data is consiterably lowered.  Even if you can demonstrate the ability to access and use data in an isolated system, even if the attacker manages to partially compromise that system then it should be possible to extract data from the system.

1. create a separated / isolate system, e.g. on a laptop with networking physically disabled
2. randomly select an item from your backup library
3. extract some data from that system onto a new pendrive
4. verify the data on the pendrive on your live system

### use disk write protection when verifying backups.

The need to verify backups directly interacts with our need to keep the backups as completely isolated from our production system as possible.  If we connect a disk in read-write mode to a system which has been compromised during an attack then the attacker may use our restore verification as the moment to destroy the data on the backup disk.  By using disks through hardware based read-protected interfaces we can ensure that the data cannot be destroyed.

It is important to note, at this stage, that there are different levels of "hardware" read protection.  The best of which are literally hardware - the write signal is fully disabled - and some of which use a hardware switch to change a setting in a software driver.  The hardware protections will work.  [Forensic disc controllers](https://en.wikipedia.org/wiki/Forensic_disk_controller) should be reliable.  Software based write protection can be bypassed and should be avoided. 

## randsomware related procedures

You should drill and prepare for randsomware attacks.

* run a drill in which your data is all recoverable
* run a drill in which the randsomware has destroyed some of your backup
