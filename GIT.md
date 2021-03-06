## Overview

This page explains one way to set up a Git server to be used in courses.
The process followed by the teacher is as follows:

- Students are given instructions to create ssh keys.  
- Students email their public key files to the instructor.
- The instructor renames each public key file to a name they can use to recognize the student.
- The instructor creates a repository by the public key name and gives read/write access to the student and the instructor.
- The instructor emails the url to the student to be used to clone the repository.

This page assumes that you have set up a server using the instructions in 
[BASE.md](https://github.com/csusbdt/centos/blob/master/BASE.md).

After you follow these instructions to set up your system,
you can provide students a link to
[the student instructions](https://github.com/csusbdt/centos/blob/master/GIT-SUBMISSION.md).

## Server Setup

Install git.

    yum install git-core

Simplify git push command.

    git config --global push.default current

Get the instructor's public key; assume it is _zed.pub_.

Create skeleton files to be copied to home directories of new accounts.
As root, do the following.

    cd
    mkdir skel
    cd skel
    mkdir .ssh
    chmod 700 .ssh
    cp zed.pub /root/skel/.ssh/authorized_keys
    mkdir repo.git
    cd repo.git
    git --bare init
    cd

If the instructor has a teaching assistant named Yed,
then append Yed's public key to the list of authorized keys.

    cat yed.pub >> /root/skel/.ssh/authorized_keys

Create a README file to convey information and so that student does not get a warning when cloning repo.
As an example, create _README.md_ with the following contents.

````
Run the following commands once to set your name and email.
(Use your real name so I can recognize you more easily.)

    git config --global user.name "Your Name"
    git config --global user.email you@example.com
````

Add the readme file to the repository.

````
cd
git clone skel/repo.git
cd repo
mv ../README.md .
git add README.md
git commit -m "Added readme file."
git push 
````

## Per Student Setup

Suppose there is a student named Alice. 

The instructor asks Alice to run the following command to generate 
a public/private key pair for use with ssh.

    ssh-keygen -t rsa

Alice emails the following file to the instructor.

    ~/.ssh/id_rsa.pub

The instructor renames the file to _alice.pub_.

The instructor uses the following commands to create a linux account with id _alice_.

    useradd --skel /root/skel --shell /usr/bin/git-shell alice
    cat alice.pub >> /home/alice/.ssh/authorized_keys

The first command creates account alice with a home directory populated with the
skeleton that contains an empty repository and pre-configured
with the public keys of the instructors.
It also specifies git-shell as the login shell,
so that students are restricted to git operations.
The second command adds the student's public key.

Assume the host name of the server is _server_.
The instructor sends the student the following command to clone the repository.

    git clone alice@server:repo.git

The instructor uses the exactly the same command to clone Alice's repository
(with the name _alice_ and NOT with the name _zed_).

## Scripts

Here is a script that creates a student account;
it takes the desired account id as an argument.

````
# create.sh
# If account name not specified on command line, print error message and terminate.
if [ -z "$1" ]
then
  echo "ERROR: Account id not specified."
  exit 1
fi

# If account already exists, print error message and terminate.
if [ -d /home/$1 ]
then
  echo "ERROR: Account $1 already exists."
  exit 1
fi

# If public key file doesn't exist, print error message and terminate.
if [ ! -f $1.pub ];
then
   echo "ERROR: File $1.pub does not exist."
  exit 1
fi

# Create account.
useradd --create-home --skel /root/skel --shell /usr/bin/git-shell $1

# Append public key to ssh authorized keys.
cat $1.pub >> /home/$1/.ssh/authorized_keys
````

I have not found the remaining script to be useful.

Here is a script that deletes student accounts;
it takes the account id as an argument.

````
# delete.sh
# If account name not specified on command line, print error message and terminate.
if [ -z "$1" ]
then
  echo "ERROR: Account id not specified."
  exit 1
fi

# If account already exists, print error message and terminate.
if [ ! -d /home/$1 ]
then
  echo "ERROR: Account $1 doesn't exist."
  exit 1
fi

# Delete account.
userdel --remove $1
````

Here is a script that adds an additional public key for an account;
it takes the account id and name of new public key (minus the .pub extension)
for arguments.

````
# append.sh
# Check for account id argument.
if [ -z "$1" ]
then
  echo "ERROR: Account id not specified."
  exit 1
fi

# Check for account id argument.
if [ -z "$2" ]
then
  echo "ERROR: Public key not specified."
  exit 1
fi

# If account doesn't exist, print error message and terminate.
if [ ! -d /home/$1 ]
then
  echo "ERROR: Account $1 doesn't exist."
  exit 1
fi

# If public key file doesn't exist, print error message and terminate.
if [ ! -f $2.pub ];
then
   echo "ERROR: File $2.pub does not exist."
  exit 1
fi

# Append public key to ssh authorized keys.
cat $2.pub >> /home/$1/.ssh/authorized_keys
````

