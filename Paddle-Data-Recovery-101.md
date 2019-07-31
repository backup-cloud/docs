# Paddle Data Recovery 101

This document was created in order to outline the actions required from all parties during the Data Recovery Process (DRP).


## `CONTENTS`

* [HOW DO WE BACKUP OUR CORE DB](#BRIEF-ON-HOW-BACKUP-IS-MANAGED)
* [GUIDE FOR TRUSTED RECOVERY RESPONSIBLE PEOPLE (TRRP)](#TRRP)
    * [HOW TO INSTALL GPG ON YOUR MACHINE](#)
	* [HOW TO GENERATE YOUR GPG SECRET KEY](#)
	* [HOW TO CREATE KEYS TARBALL USING HEADER TARBALL](#)
	* [HOW TO SEND KEYS TARBALL TO ENGINEER](#SENDING-KEYS-TARBALL-TO-ENGINEER)
* [GUIDE FOR ENGINEER](#ENGINEER)
    * [HOW TO PREPARE BACKUP FILES FOR RECOVERY](#RETRIEVING-BACKUP-FILES-TO-RECOVER)
	* [HOW TO CREATE HEADER TARBALL FOR BACKUP FILES](#CREATING-HEADER-TARBALL-FOR-BACKUP-FILES)
	* [HOW TO SEND HEADER TARBALL TO TRRP](#SENDING-HEADER-TARBALL-TO-TRRP)
	* [HOW TO DECRYPT BACKUP FILES USING KEYS TARBALL](#USING-KEYS-TARBALL-TO-DECRYPT-BACKUP-FILES)
	* [HOW TO RESTORE THE DECRYPTED FILES](#RESTORING-DECRYPTED-BACKUP-FILES-TO-DATABASE)
* `**INFRASTRUCTURE 101**`
    * [HOW TO CREATE AND CONFIGURE RECOVERY DATABASE](#BUILDING-RECOVERY-DATABASE-101)
	* [HOW TO CREATE RECOVERY INSTANCE](#BUILDING-RECOVERY-INSTANCE-101)

## `BRIEF ON HOW BACKUP IS MANAGED`

There are two stages to our backup service, `incloud` and `offsite` - our `offsite` stage is dependant on it's former. We have a service to initiate the backup procedure of our RDS instance. This service replicates our core RDS database, creates a temporary
EC2 instance with the necessary packages, AWS Security Groups, EBS Storage, VPC, and writes all data `encrypted` data to a preconfigured S3 bucket. Once the back up is complete to the S3 bucket, a member of the SOC Team
connects to the S3 bucket and makes a `offsite` copy of the backup files. 

`NOTES`
* `Incloud` data older than 1 month will be deleted from S3 Bucket
* `Offsite` data older than 3 months  will be securely removed from drives
* The encryption of data is done prior to uploading data to bucket, GPG keys are used.

# `TRRP`
You are the key holder

## `HOW TO INSTALL GPG ON YOUR MACHINE`




# `ENGINEER`
You are responsible for getting our data back to a usable state

## `RETRIEVING BACKUP FILES TO RECOVER`

In order to run a data recovery we must first have the data that will be recovered. As mentioned in the [backup brief](#BRIEF-ON-HOW-BACKUP-IS-MANAGED) a backup can be classified as either `incloud` or `offsite`, below we have the instructions on how to retrive both.

### INCLOUD FOLDER
Having the data `incloud` means we have reduced the overhead recovery time significantly. from here we need to:

1. Prepare an EC2 Instance, [more information here](#BUILDING-RECOVERY-INSTANCE-101)
2. Ensure the EC2 Instance has permissions to Read from the S3 Bucket
3. SSH to the instance
3. Using the `awscli` run the following command in the home directory of the SSH user (`aws s3 cp s3://<NAME-OF-S3-BUCKET>/export-<BACKUP-DATE> ./<NAME-OF-BACKUP-FOLDER> --recursive --region <BUCKET-REGION>`)

### OFFSITE FOLDER
Having the data `offsite` means that there are a number off additional steps and factors we need to consider, i.e. internet connection speed, external drive -> PC connection type etc.

1. Find a safe location with decent internet speed
2. Assuming the new AWS account is set up, create an `Encrypted instance` of a S3 bucket (KMS AES-246)
3. Create an IAM User with `Administrator` permission and assign the user an Access Key. Download the `Access Key credentials` to your local machine
4. Ensure your machine has `python3.6` and `pip3` installed. Download the `awscli` using pip (`pip install --user awscli`)
5. Configure the `awscli` using the `Access Key credentials` files from `step 3` using the command `aws configure`.
6. Connect the `offsite` back up drive.
7. On the PC navigate to the newly inserted drive using the `terminal`.
8. Using the S3 bucket's S3 Unique Identifier (S3://) name, use the `awscli` tool to copy the backup files onto the cloud using the command `aws s3 cp ./<NAME-OF-BACKUP-FOLDER> s3://<NAME-OF-BUCKET-CREATED-IN-STEP-2>/export-<BACKUP-DATE>  --recursive --region <BUCKET-REGION>`
9. Once completed, follow the steps outlined in our [incloud backup files recovery section](#INCLOUD-FOLDER)	

## `CREATING HEADER TARBALL FOR BACKUP FILES`

As explained in [the summary of our backup procedure](#BRIEF-ON-HOW-BACKUP-IS-MANAGED) we encrypt our backup files. In order for us to restore the files, we must first decrypt them. The Engineer overwatching the recovery must contact the `Trusted Recovery Responsible Person` with the neccessary `header tarball` so they can provide a pairing `keys tarball` in return. In this step we will show you the `engineer` how to prepare the header tarball.

1. Assuming you are in the same directory as the backup files folder, create a directory to hold headers using `mkdir header-<restore-directory>`
2. Remove any files from the `export-<BACKUP-DATE>` in the home directory that you do not wish to recover/ get the key for.
3. Again, make sure you are at the home directory (you can run `pwd` or `ls`)
4. Create a script called `create_h_tarball.sh` using your favourite terminal editor i.e. `nano` and paste the codeblock below into it. This block will to get the header of all the files in the backup file folder, save them to the header folder created in `step 1`, and tar the header directory

```bash
#!/bin/bash

## RESTORE DIR
RESTORE_DIR=$1
HEADER_DIR="header-$RESTORE_DIR"

## FIND ALL THE FILES IN RESTORE DIR AND SAVE HEADER TO HEADER DIR
find $RESTORE_DIR -type f | xargs -I '{}' -P4 dd if='{}' of='header-{}' count=32

## TARBALL HEADER DIR
tar -czvf $HEADER_DIR.tgz $HEADER_DIR
```
5. Make the script executable using `chmod +x create_h_tarball.sh`, and run it passing the backup files directory as an argument - `./create_h_tarball.sh export-<BACKUP-DATE>`
6. Confirm the new header tarball is created and not empty by running the command `du -s header-export-<BACKUP-DATE>.tgz`
7. Open a local terminal on your PC, and copy the `header tarball` locally using the command `scp -r -i ~/<PATH-TO-PEM-FILE> ubuntu@<EC2-INSTANCE-PUB-IP>:~/<HEADER-TARBALL> ./`

## `SENDING HEADER TARBALL TO TRRP`
Now you have the tarball of headers, contact the available `TRRP` and let them know you will send them the tarball. Whatever method you use, please ensure it is safe.

1. Find a safe, secure method of transferring the tarball to the `TRRP`. This can be done via. creating a `1Password` vault or something equally as secure.
	

## `USING KEYS TARBALL TO DECRYPT BACKUP FILES`
So you've received the `keys tarball` from the `TRRP`. Using this file you will be able to decrypt the backup files in the `export-<BACKUP-DATE>` directory, see how below.

1. Navigate to wherever you've saved the `keys tarball` on your local PC, and copy it to the EC2 instance using the command `scp -r -i ~/<PATH-TO-PEM-FILE> ./<KEYS-TARBALL> ubuntu@<EC2-INSTANCE-PUB-IP>:~/`
2. Make sure you are at the home directory of the EC2 instance (you can run `pwd` or `ls`)
3. Extract the `key tarball` using `tar -xvf <KEYS-TARBALL>`
4. Create a script called `decryptfile.sh` using your favourite terminal editor and paste the code block bellow into it. This block will use the passed argument to attempt to generate the name of the key file, and then run the key against its corresponding encrypted file, outputing the unencryted file
to the decrypted folder (`decrypt-export-<BACKUP-DATE>`) you will create in the next step.
```bash
#!/bin/bash
FILE=$1
KEY_FILE=$( cat "key-$FILE" )
set -vx
gpg --override-session-key "$KEY_FILE"  -o "decrypt-$FILE" --decrypt "$FILE"
```
5. Create the decrypt folder using `mkdir  decrypt-export-<BACKUP-DATE>`
6. Run the following find command against the folder (`export-<BACKUP-DATE>`) with the encrypted backup files. **NOTE** - It may take more than 5 minutes dependant on the number of files you are decrypting.
```bash
find export-<BACKUP-DATE> -type f | xargs -n1 -P4 ./decryptfile.sh 
```
7. Once completed, the decrypt folder should have decrypted files inside of it. You can cofirm by running `ls -l <FOLDER-NAME> | wc -l` against both the `export-<BACKUP-DATE> ` and `decrypt-export-<BACKUP-DATE>` directories. 
The number should match.	

## `RESTORING DECRYPTED BACKUP FILES TO DATABASE`
The neccessary files have been decrypted and we're assuming that the [recovery RDS instance](#BUILDING-RECOVERY-DATABASE-101) has been created and configured, we now have all the pieces need to restore the database. Let's begin.

1. Make sure you are at the home directory of the EC2 instance (you can run `pwd` or `ls`)
2. Open a screen with the name recovery, this can be done with the command `screen -S recovery`. You will be automatically attached.
3. Get and note the following database credentials for the [restoration database](#BUILDING-RECOVERY-INSTANCE-101) as we will need them for the following mydumper command.
	* Hostname - `<rds-host-endpoint>`
	* Password - `<db-password>`
	* Username - `<db-user>`
	* Number of vCPU - <number-of-threads>
4. Run the block code below in the screen, replace the variables with corresponding values. The code block echos the current date and time then proceeds to start the recovery with `myloader`, on completion it echos the date again.
```bash
echo $( date );  myloader --directory decrypt-export-<BACKUP-DATE>  -h <rds-host-endpoint> -P 3306 -u <db-user> -p <db-password> -B <target-database> -t <number-of-threads>; echo $( date )
```
5. After starting the restoration in the screen you can deattach (`Ctrl + A D`) and reattach (`screen -r`) at any time
6. You can monitor the progress of the restoration by using the `mysql` tool
	* Connect to the database - `mysql -h <rds-host-endpoint> -P 3306 -u <db-user> -p <target-database>`, you will be prompted for the password (`<db-password>`).
	* Use the command `show databases;` to see existing DBs, including your `<target-database>`
	* Run the SQL block below which will display the size of your `<target-database>`, remember to replace with your actual value.
	```sql
	SELECT now();SELECT table_schema AS "Database", SUM(data_length + index_length) / 1024 / 1024 / 1024 AS "Size (GB)" 
	FROM information_schema.TABLES WHERE table_schema='<target-database>';
	```


## `BUILDING RECOVERY DATABASE 101`

It is important that we correctly configure the `restoration database` to minimise restoration time



## `BUILDING RECOVERY INSTANCE 101`

When building a recovery instance, please follow the guide below as a point of reference

### Step 1: Choose AMI

* Choose an AMI based on Ubuntu (`Canonical, Ubuntu, 18.04 LTS, amd64 bionic image build on 2019-05-14`) i.e. `ami-024a64a6685d05041`

### Step 2: Choose Instance Type

* Choose an instance type with at least `eight` vCPUs, we used \`c5.2xlarge (34 ECUs, 8 vCPUs, 3 GHz, Intel Xeon Platinum 8124M, 16 GiB memory, EBS only)
* For ultimate recovery speeds use, `c5.12xlarge`which comes with:
  * `48 vCPU`- CPU
  * `96 GiB` - Memory
  * `12 Gbps`  - Bandwidth

### Step 3: Configure Instance

* Choose the same VPC in `Network` as that of the RDS instance
* For best transfer speed, please ensure the `Subnet` is in the same region group, i.e. `us-east-1b`.
* Enable `Auto-assign Public IP` so we can reach the instance
* Expand the Additional Settings, and update the `User Data` with:

    ```bash
    #!/bin/bash

    apt-get update && apt-get -y upgrade

    apt-get install -y mysql-server mydumper gpg python3 python3-pip

    echo -e "\n#Alias\nalias pip=\"pip3\"\nalias python=\"python3.6\"" >> /home/ubuntu/.bashrc

    chown ubuntu:ubuntu /home/ubuntu/.bashrc
    su -u ubuntu bash -c 'pip install --upgrade pip --user && pip install awscli --user'
    ```

### Step 4: Add storage

* Ensure the Root `Volume Type` has at least 350 GB `Size (GiB)`
*  Make sure the volume is `Encrypted`

### Step 5: Add Tags

* Add an `Key` for `Name` and give it the `Value` of `RDSBackup` or similar

### Step 6: Configure Security Group

* `Create a new security group` that allows access from the office and OpenVPN.

### Step 7: Launch Instance

* When launching instance create a new `Key Pair` and distribute to the relevant parties using `1Password` or equivalent


#### `TIPS`

* Ensure the newly created security group is sourced and added to the RDS's security group as an `Inbound` rule, on port `3306`.


