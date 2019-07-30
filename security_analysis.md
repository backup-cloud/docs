The backup-cloud project is designed to provide a secure backup system for data both in cloud systems and in traditional physical systems connected with them. The systems in backup-cloud use cloud based systems for data transfer but back this up with isolated offline physical media.

This security anaysis document is meant to help system users understand potential risks with the system and mitigate against threats in their environment.  The document is far from complete and feedback in the form of bug repots to the backup-cloud tracker on GitHub will be important in making this system secure and useful.

Currently the backup-cloud system should be viewed as in beta-test for public use at most.  The system has not been adequately reviewed and audited and was built for and tested in a limited environment with experienced users able to resolve problems on their own.  

## Use and abuse of GPG

The current design of the system uses GPG and in particular the GPGME library for encryption and thus as a critical part of the security of the system.  GPG was chosen for a number of practical reasons:

* the users in the original system where GPG was introduced were familiar with it and able to set themselves up quickly
* the PGP ecosystem, which GPG is part of is widely publically known and has been well analysed meaning that it's security is reasonably well understood
* GPG includes a number of practical protections suitable for this system such as direct console input of keys, encrypted keyrings, and key stretching which aren't clearly included in other systems considered
* compared to generic systems like OpenSSL or LibreSSL, GPG has a smaller set of selections and a clearer set of defaults suitable for this system
* GPG is widely available and used by a many security experts.  We believe that even after a serious worldwide attack such as a war GPG will remain available for download and give our users a good chance of recovering their data even when others cannot.

We use GPG in a relatively conservative way.  Public key encryption is done on cloud systems which might risk exposure of session keys, however all private key manipulations are expected to take place on real physical systems and in a situation which will allow considerable care up to and including use of hardware encryption

Despite all of this there are a number of reasons for some concern with the use of GPG

* GPG procedures are somewhat complex and confusing - we mitigate this by insisting on regular verification
* There have been a number of criticisms of the GPG projectes handing of security flaws
* we currently assume that a fixed length header at the beginning of encrypted data will contain the session key and won't contain any secret information - in our experiments this has always been true but theoretically a future GPG implementation could change this in problematic ways

Organisations subject to high risk of advanced attacks or with high assurance requirements are recommended to hire external security experts audit the system and provide improvement recommendations.  Those organisations are then recommended to feed some or all those improvement recommendations back into the project to improve the security of the system overall.  

## Risks

Risk analysis should be done starting with what valuable "assets" a given organisation has and what bad things could happen related to those assets.  In the case of the backup systems risks like the following should be evaluated:

* the data could fail to restore - meaning you wouldn't be able to do whatever it is you do with the data

* the data from the system could leak to an attacker or other organisation - meaing you could be liable for leaking someone else's data; could be blackmailed or could have the data used against you

* it could take too long to restore the data and so you may not be able to use the data when you need it.

* the data could be corrupted or manipulated meaning that you use incorrect data for a decision or give incorrect data to a partner.

For each risk you should decide if the risk is acceptable as it stands with the current system and if not identify appropriate mitigations.


## Data retention and expiry

Backups represent an image of a datastore at a particular time.  By their very nature, changes since that time are not replicated to the backup.  Certain data, such as private data may have been deleted during that time due to company policies.  After restoration this may represent a problem.  

Currently the backup-cloud system uses S3 buckets for storage of the main live data-archive which provides the data to the offline system.  

## Vulnerabilities

This is a register of known vulnerabilities in the system.  None of these are considered to be a serious threat to the system's viability.  They are registered in order to

Software vulnerabilities

As with all normal modern computer systems this system is subject to potential software vulnerabilities.  We know these exist we just don't know, for the most part, what they are yet.  As these vulnerabilities are fixed

Software vulnerabilities in our code

The code for the system has gone through static code checkers to reduce the risk of code vulnerabilities.  Turning on more tests and improving the code to match can continue to reduce this chance.  

The developers have run a system of peer reviews which gives a second chance to pick up incorrect code.  

### Destruction of the backups

The most serious

Future discovered vulnerabilities

in future

## Countermeasures

## Design principles

### Simplicity and minimal new components

> simplicity is good but not always simple to achieve - we try hard

Unfortunately a backup system which backs up data from many different systems each with their own bulk data storage format must inherently somewhat complex.  This makes the need for simplicity higher.  

1. simplicity of operations of the system
2. simplicity of use of the system APIs
3. simplicity of development of the system
4. simplicity of the system components

All of these elements are important but sometimes they are in opposition to each other. Although these are ordered in an approximate order of priority various selections of taste have had to be made.  

* the system uses cloud components like S3 which are reasonably simple to use and understand but which are complex to fully manage and which have complex underlying security mechanisms - the decision was made that the target system users are familiar with these systems
* there are multiple interfaces to the underlying base system which sets up encryption which should make backup scripts simpler to write and understand at the cost of some extra complexity in the underlying python modules.
* the system is broken into a number of smaller modules spread across different git repositories.  Whilst this inherently increases development and operational complexity somewhat this is counteracted by much greater clarity about which parts of the system are in use and by avoiding the additional operational complexity of having all of the system deployed when it is not needed.  

### Avoid increasing risk over an AWS based cloud system

> attempting to match the security of AWS will enable our target audience - cloud based systems to deploy with confidence.  At the same time avoiding

* data that is encrypted on AWS should be encrypted in the system
* it should not be noticeably easier to recover data from the system than it would be to access the live AWS system

### Use public key cryptography and assume that backup data may be compromised to support widespread storage

> public key cryptography allows us to keep the unencrypted data and the ability to encrypt it fully separated from the ability to decrypt the same data

Our  backup system is designed to support disaster recovery which means that storing data offsite is a definite advantage.  Such storage and transport to the storage, whilst possible to secure to some level, inevitably provides vulnerabilities to loss and theft.  

### Use standard encryption tools designed for file encryption.

* standard tools (such as gpg) provide safer interfaces which include protection such as
  * non-echoing passphrase entry
  * protection against secret data being written to non-volatile storage
* the algorithms in GPG are well known and widely examined
* if used in the standard way the security of the tools is better known that something we could create ourselves

### Pure serverless design where possible.  Ephemeral servers otherwise.

> by avoiding having any long running servers we avoid some of the problems of managing them

In order to avoid patch management and reduce vulnerability windows a pure serverless design is used wherever that is reasonable.  This means either FaaS systems such as AWS lambda where there is no operating system management or Container systems such as docker where we use fully automatic image building to ensure that software us updated combined with short container lifespans to ensure that exploitation windows are limited and strong container isolation to ensure that vulnerabilities are difficult to exploit.

In some specific cases such as large database backup, raw virtual servers need to be used for performance reasons.  In this situation the system is designed to start and run the servers only for the time that the actual backup is running.  Besides reducing costs this also ensures that the window for exploitation of a vulnerability is severely limited.  The instances used should be the latest at the time of launch with all configuration and software installation done via user scripts ensuring that there is no need for ongoing patch management.  

##
