Separation of responsibilities in the system is largely enforced through IAM roles

## The backup administrator

Administration of the backup system could be split between different people and details of this role are most complex, however key things have to be possible

* setting up IAM roles and configuring rights for other users and backup processes
* creating, verifying (read), updating and removing public keys

This tends to mean that the backup administrator in the account where backup data is being gathered.  If data is coming from other accounts, however, then the backup administrator needs, at most read access to that data and in fact that read access could be granted only to the backup process.  

## The backup operators

The backup operators should be able to either read data from the S3 bucket or from a local server.  The backup operators also need access to be able to recreate whichever parts of the system they restore during backup verificaion but normally this could be provided to them separately from the system being backed up.  

## Backup processes

Backup processes should be able to

1. read the data used by their application (e.g. RDS access if backing up an RDS database)
2. write to the S3 bucket to the folder where the data will be stored
3. have read access to the public key folder in S3 to read keys for encrypting data

## The Key Holders

Normally key holders should be given no permissions, however they could have read/write permissions to their own public keys in the system to permit them to update those keys if needed.
