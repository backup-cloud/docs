The following instructions are for nominated key holders.  This is designed to be a standalone document which you can distribute to them without them having to learn much about the rest of the document.  Probably you will want to customise this document for the details of your organisation.

*This document explains your duties as a key holder. For us to be able to recover the system, you must read and follow the instructions here now, in advance of the need for recovery.   *

# Introduction

As a key holder, your most important job is to keep safe a key which can be used to recover an offsite backup.

You are expected to generate the key, protect it and store it correctly.  You also need to have access to the password which protects the said key.

The backup keys are:

a) The primary protection against the leak of data if the backup were to be stolen or misplaced.

b) The only way of recovering the backup if everything else in the company is destroyed.

So you want to keep them safe and keep them only for yourself.

## Install GPG

Find below the steps to install GPG on your system

* If you use homebrew, install `gpg2 -  *N.B.*`*see notes below about troubleshooting if you have problems.*

**OR**

* install [GPG Suite](https://gpgtools.org/) from [https://gpgtools.org/](https://gpgtools.org/)

# Generating and Storing Backup Keys

1. **IMPORTANT - **Ensure that the computer you are working on is properly encrypted and secured before doing anything.
2. Create a key protection password - this should typically combine a written part with a secret part known only to yourself but reliably memorable.
3. Generate a key following the these instructions →  [Generating a new GPG key - User Documentation](https://help.github.com/articles/generating-a-new-gpg-key/). Ensure that you make a safe passphrase (see below)
4. Copy the key twice to some **safe media** like a USB pen drive and store them in two separate safe places away from the office like a safe at home and another in your safe deposit box at the bank.  (see Backing up your GPG data below for details of how to copy the keys)

## Backing up your GPG data

To be able to recover your private key in the event your laptop is hacked, you should copy it to at least two pen drives that you will store in a secure location.

The simplest way to do this is to copy the directory, `\~/.gnupg`, onto the pen drive.  Since the data in that is password protected, this will also keep it reasonably secure.

## Recovering a backup directory from a tarball

Your backup operator will send you a file.  Cut and past the following locally and run it, putting the downloaded filename in to replace the FILE variable with the name of your file.

```
FILE=<filename here>
tar xzvf $FILE
DIR=`echo $FILE | sed 's/.tgz//'`
KEY_DIR=`echo $DIR | sed 's/header/key/'`
mkdir $KEY_DIR;
for i in `find $DIR -type f`
do
   echo $i; gpg --show-session-key -o /dev/null $i 2>&1 |
        grep session | sed -e "s/[^']*'//" -e  "s/'[^']*//" > `echo $i |
        sed 's/header/key/'` ;
done
tar -czvf $KEY_DIR.tgz $KEY_DIR
```

## Verifying Backup Recovery with your Key (normal method)

To check that your key is being used correctly the easiest way is for the backup operators to restore some data.  Follow the procedure below for restoring a backup above.

## Verifying Backup Recovery with your Key (local method - with support from backup administrator)

You will be given a partial recovery file;  Try running

* `gpg  --decrypt  backupfile.gpg | gzip -cd > output.txt`

if you look at output.txt with a file viewer it should now contain visible but incomplete application data.  This shows that your public key is correctly installed in the backup system and that your private key is able to decrypt the backup data.  

### Troubleshooting Problems with GPG

If you get an error from GPG that no key is present  and "inappropriate IOCTL for device"

`export GPG_TTY=$(tty)`

before running GPG may fix the problem
