Automated restoration scripts have definite value and should certainly
form part of your process for verifying that your backups work
correctly.  However, following a proper disaster you may find that
resources that they rely on are not available or that your recovery
resources are sufficiently different from the original resources

In a traditional environemnt this kind of problem is properly
mitigated with fail over data centres.  The assumption was that two
systems separated by several hundred miles would not fail together.

In the modern threat environment this assumption is no longer valid.
If the data centres are duplicates then they run identical or at least
similar software.  Class breaks used by randsomware can be assumed to
destroy both your primary and backup data centre.

For this reason the backup-cloud project strongly recommends primarily
drilling manual / semi-automated recovery where it will be easier to
respond to

## Recovering a backup

1. Select the backup file or directory you want to recover - move it so it can be accessed on a secure EC2 instance with sufficient storage (normally you will have to create an encrypted EBS to store the data on)  
2. Decrypt the backup (see below)
3. Run the recovery procedure for the system the backup applies to

### Backup Directory Decryption

> *This depends on gpg-agent to allow repeated fast decryption of multiple files.  If during testing you find that you have to put your password in more than once you may need to investigate if your gpg-agent is working.  If the problem occurs during restore you will just have to paste your password repeatedly, so please test first. *

one of the key backup people must be contacted and asked to get the session key to do the decryption

* send them the header of the backup file you can create it with this
  * `mkdir header-<restore-directory>\
    find `<restore-directory>` -type f | xargs -I '{}' -P4 dd if='{}' of='header-{}' count=32`
* ask them to recover the session key and send it to you (see the section *"Recovering a backup with your key" *in [Instructuctions for Key Holders](instructions_for_key_holders.mdinstructions_for_key_holders.md).  \
  then they should securely send you the session keys in a tarball;
* unpack the tarball and you should run
*   `find `<restore-directory>` -type f | xargs -I '{}' -P4 dd if='{}' of='header-{}' count=32`

  N.B. old versions of gpg don't support this, gpg 2.2.10 or later definitely do.

### use the session key to decrypt the backup:

make a script `decryptfile.sh `with the following

```
#!/bin/bash
FILE=$1
set -vx
gpg --decrypt --override-session-key `cat key-$FILE'` "$FILE" -o "decrypt-$FILE'
```

then run

```
mkdir  decrypt-<restore-directory>
find <restore-directory> -type f | xargs -n1 -P4 ./decryptfile.sh
```

## Using the backup file

Once the files are decrypted  restore instructions for each system should be run using the data output.  

running the output file will restore the database contents.  

send the header file to the key holder and ask them to follow the instructions in the document

* Preparation and Recovery Instructions for Key Holders

## Verifying the restore process

> *Verification of the manual restoration is a delicate and potentially dangerous operation.  You will be building a system equivalent to your production system in a short time nd your real data with real confidential information will be used in this stage so security measures must be at least equivalent to those used in production.  Set up a system with security measures at least equivalent to your production system for this and consider using private networks in all cases where possible*

The backup should be verified regularly (at least once a month for each backup category, it is recommended that each week at least one backup category is checked)

* Restore the backup to equivalent storage to the production system

For a full check (needs to be implemented together with each team)

* restore all of the data from each system to a separate (non-production) account
* run end to end tests on the system as if it was production
