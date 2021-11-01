# Data management and transfer

This section covers best practice and tools for data management on
Tursa.

!!! information
    If you have any questions on data management and transfer please do not
    hesitate to contact the DiRAC service desk at <dirac-support@epcc.ed.ac.uk>.

## Useful resources and links

   - Harry Mangalam's guide on [How to transfer large amounts of data
     via
     network](https://hjmangalam.wordpress.com/2009/09/14/how-to-transfer-large-amounts-of-data-via-network/).
     This provides lots of useful advice on transferring data.

## Data management

We strongly recommend that you give some thought to how you use the
various data storage facilities that are part of the Tursa service.
This will not only allow you to use the machine more effectively but
also to ensure that your valuable data is protected.

## Tursa storage

The Tursa storage is provided by a parallel Lustre file system that 
provides your home directories and working storage. When you log in
you will be placed in your home directory. 

The home directory for each user is located at:

    /home/[project code]/[group code]/[username]

where

   - `[project code]` is the code for your project (e.g., x01);
   - `[group code]` is the code for your project group, if your project has
     groups, (e.g. x01-a) or the same as the project code, if not;
   - `[username]` is your login name.

Each project is allocated a portion of the total storage available, and
the project PI will be able to sub-divide this quota among the groups
and users within the project. As is standard practice on UNIX and Linux
systems, the environment variable `$HOME` is automatically set to point
to your home directory.

## Sharing data with other Tursa users

How you share data with other Tursa users depends on whether or not
they belong to the same project as you. Each project has two shared
folders that can be used for sharing data.

### Sharing data with Tursa users in your project

Each project has an *inner* shared folder.

    /home/[project code]/[project code]/shared

This folder has read/write permissions for all project members. You can
place any data you wish to share with other project members in this
directory. For example, if your project code is x01 the inner shared
folder would be located at `/home/x01/x01/shared`.

### Sharing data with all Tursa users

Each project also has an *outer* shared folder.:

    /home/[project code]/shared

It is writable by all project members and readable by any user on the
system. You can place any data you wish to share with other Tursa
users who are not members of your project in this directory. For example,
if your project code is x01 the outer shared folder would be located
at `/home/x01/shared`.

### Permissions

You should check the permissions of any files that you place in the shared
area, especially if those files were created in your own Tursa account
Files of the latter type are likely to be readable by you only.

The `chmod` command below shows how to make sure that a file placed in the
outer shared folder is also readable by all Tursa users.

    chmod a+r /home/x01/shared/your-shared-file.txt

Similarly, for the inner shared folder, `chmod` can be called such that read
permission is granted to all users within the x01 project.

    chmod g+r /home/x01/x01/shared/your-shared-file.txt

If you're sharing a set of files stored within a folder hierarchy the
`chmod` is slightly more complicated.

    chmod -R a+Xr /home/x01/shared/my-shared-folder
    chmod -R g+Xr /home/x01/x01/shared/my-shared-folder

The `-R` option ensures that the read permission is enabled recursively and
the `+X` guarantees that the user(s) you're sharing the folder with can 
access the subdirectories below `my-shared-folder`.

## Archiving and data transfer

Data transfer speed may be limited by many different factors so the best
data transfer mechanism to use depends on the type of data being
transferred and where the data is going.

   - **Disk speed** - The Tursa  file system is highly parallel,
     consisting of a very large number
     of high performance disk drives. This allows it to support a
     very high data bandwidth. Unless the remote system has a similar
     parallel file-system you may find your transfer speed limited by
     disk performance.
   - **Meta-data performance** - Meta-data operations such as opening
     and closing files or listing the owner or size of a file are much
     less parallel than read/write operations. If your data consists of
     a very large number of small files you may find your transfer
     speed is limited by meta-data operations. Meta-data operations
     performed by other users of the system will interact strongly with
     those you perform so reducing the number of such operations you
     use, may reduce variability in your IO timings.
   - **Network speed** - Data transfer performance can be limited by
     network speed. More importantly it is limited by the slowest
     section of the network between source and destination.
   - **Firewall speed** - Most modern networks are protected by some
     form of firewall that filters out malicious traffic. This
     filtering has some overhead and can result in a reduction in data
     transfer performance. The needs of a general purpose network that
     hosts email/web-servers and desktop machines are quite different
     from a research network that needs to support high volume data
     transfers. If you are trying to transfer data to or from a host on
     a general purpose network you may find the firewall for that
     network will limit the transfer rate you can achieve.

The method you use to transfer data to/from Tursa will depend on how
much you want to transfer and where to. The methods we cover in this
guide are:

   - **scp/sftp/rsync** - These are the simplest methods of
     transferring data and can be used up to moderate amounts of data.
     If you are transferring data to your workstation/laptop then this
     is the method you will use.

Before discussing specific data transfer methods, we cover *archiving*
which is an essential process for transferring data efficiently.

### Archiving

If you have related data that consists of a large number of small files
it is strongly recommended to pack the files into a larger "archive"
file for ease of transfer and manipulation. A single large file makes
more efficient use of the file system and is easier to move and copy and
transfer because significantly fewer meta-data operations are required.
Archive files can be created using tools like `tar` and `zip`.

#### tar

The `tar` command packs files into a "tape archive" format. The command
has general form:

    tar [options] [file(s)]

Common options include:

   - `-c` create a new archive
   - `-v` verbosely list files processed
   - `-W` verify the archive after writing
   - `-l` confirm all file hard links are included in the archive
   - `-f` use an archive file (for historical reasons, tar writes its
     output to stdout by default rather than a file).

Putting these together:

    tar -cvWlf mydata.tar mydata

will create and verify an archive.

To extract files from a tar file, the option `-x` is used. For example:

    tar -xf mydata.tar

will recover the contents of `mydata.tar` to the current working
directory.

To verify an existing tar file against a set of data, the `-d` (diff)
option can be used. By default, no output will be given if a
verification succeeds and an example of a failed verification follows:

    $> tar -df mydata.tar mydata/*
    mydata/damaged_file: Mod time differs
    mydata/damaged_file: Size differs

!!! note
    tar files do not store checksums with their data, requiring
    the original data to be present during verification.

!!! tip
    Further information on using `tar` can be found in the `tar` manual
    (accessed via `man tar` or at [man
    tar](https://linux.die.net/man/1/tar)).

#### zip

The zip file format is widely used for archiving files and is supported
by most major operating systems. The utility to create zip files can be
run from the command line as:

    zip [options] mydata.zip [file(s)] 

Common options are:

   - `-r` used to zip up a directory
   - `-#` where "\#" represents a digit ranging from 0 to 9 to specify
     compression level, 0 being the least and 9 the most. Default
     compression is -6 but we recommend using -0 to speed up the
     archiving process.

Together:

    zip -0r mydata.zip mydata

will create an archive.

!!! note
    Unlike tar, zip files do not preserve hard links. File data will be
    copied on archive creation, *e.g.* an uncompressed zip archive of a
    100MB file and a hard link to that file will be approximately 200MB in
    size. This makes zip an unsuitable format if you wish to precisely
    reproduce the file system layout.

The corresponding `unzip` command is used to extract data from the
archive. The simplest use case is:

    unzip mydata.zip

which recovers the contents of the archive to the current working
directory.

Files in a zip archive are stored with a CRC checksum to help detect
data loss. `unzip` provides options for verifying this checksum against
the stored files. The relevant flag is `-t` and is used as follows:

    $> unzip -t mydata.zip
    Archive:  mydata.zip
        testing: mydata/                 OK
        testing: mydata/file             OK
    No errors detected in compressed data of mydata.zip.

!!! tip
    Further information on using `zip` can be found in the `zip` manual
    (accessed via `man zip` or at [man
    zip](https://linux.die.net/man/1/zip)).

### Data transfer via SSH

The easiest way of transferring data to/from Tursa is to use one of
the standard programs based on the SSH protocol such as `scp`, `sftp` or
`rsync`. These all use the same underlying mechanism (SSH) as you
normally use to log-in to Tursa. So, once the the command has been
executed via the command line, you will be prompted for your password
for the specified account on the *remote machine* (Tursa in this
case).

To avoid having to type in your password multiple times you can set up a
*SSH key pair* and use an *SSH agent* as documented in the User Guide at
`connecting`.

#### SSH data transfer performance considerations

The SSH protocol encrypts all traffic it sends. This means that file
transfer using SSH consumes a relatively large amount of CPU time at
both ends of the transfer (for encryption and decryption). The Tursa
login nodes have fairly fast processors that can sustain about 100 MB/s
transfer. The encryption algorithm used is negotiated between the SSH
client and the SSH server. There are command line flags that allow you
to specify a preference for which encryption algorithm should be used.
You may be able to improve transfer speeds by requesting a different
algorithm than the default. The `aes128-ctr` or `aes256-ctr` algorithms
are well supported and fast as they are implemented in hardware.
These are not usually the default choice when using `scp` so
you will need to manually specify them.

A single SSH based transfer will usually not be able to saturate the
available network bandwidth or the available disk bandwidth so you may
see an overall improvement by running several data transfer operations
in parallel. To reduce metadata interactions it is a good idea to
overlap transfers of files from different directories.

In addition, you should consider the following when transferring data:

   - Only transfer those files that are required. Consider which data
     you really need to keep.
   - Combine lots of small files into a single *tar* archive, to reduce
     the overheads associated in initiating many separate data
     transfers (over SSH, each file counts as an individual transfer).
   - Compress data before transferring it, *e.g.* using `gzip`.

#### scp

The `scp` command creates a copy of a file, or if given the `-r` flag, a
directory either from a local machine onto a remote machine or from a
remote machine onto a local machine.

For example, to transfer files to Tursa from a local machine:

    scp [options] source user@tursa.dirac.ed.ac.uk:[destination]

(Remember to replace `user` with your Tursa username in the example
above.)

In the above example, the `[destination]` is optional, as when left out
`scp` will copy the source into your home directory. Also, the `source`
should be the absolute path of the file/directory being copied or the
command should be executed in the directory containing the source
file/directory.

If you want to request a different encryption algorithm add the `-c
[algorithm-name]` flag to the `scp` options. For example, to use the
(usually faster) *arcfour* encryption algorithm you would
    use:

    scp [options] -c aes128-ctr source user@tursa.dirac.ed.ac.uk:[destination]

(Remember to replace `user` with your Tursa username in the example
above.)

#### rsync

The `rsync` command can also transfer data between hosts using a `ssh`
connection. It creates a copy of a file or, if given the `-r` flag, a
directory at the given destination, similar to `scp` above.

Given the `-a` option rsync can also make exact copies (including
permissions), this is referred to as *mirroring*. In this case the
`rsync` command is executed with `ssh` to create the copy on a remote
machine.

To transfer files to Tursa using `rsync` with `ssh` the command has
the form:

    rsync [options] -e ssh source user@tursa.dirac.ed.ac.uk:[destination]

(Remember to replace `user` with your Tursa username in the example
above.)

In the above example, the `[destination]` is optional, as when left out
rsync will copy the source into your home directory. Also the `source`
should be the absolute path of the file/directory being copied or the
command should be executed in the directory containing the source
file/directory.

Additional flags can be specified for the underlying `ssh` command by
using a quoted string as the argument of the `-e` flag.
    e.g.

    rsync [options] -e "ssh -c arcfour" source user@tursa.dirac.ed.ac.uk:[destination]

(Remember to replace `user` with your Tursa username in the example
above.)

!!! tip
    Further information on using `rsync` can be found in the `rsync` manual
    (accessed via `man rsync` or at [man
    rsync](https://linux.die.net/man/1/rsync)).

## SSH data transfer example: laptop/workstation to Tursa

Here we have a short example demonstrating transfer of data directly
from a laptop/workstation to Tursa.

!!! note
    This guide assumes you are using a command line interface to transfer
    data. This means the terminal on Linux or macOS, MobaXterm local terminal on Windows or Powershell.

Before we can transfer of data to Tursa we need to make sure we have an
SSH key setup to access Tursa from the system we are transferring data
from. If you are using the same system that you use to log into Tursa then
you should be all set. If you want to use a different system you will need
to generate a new SSH key there (or use SSH key forwarding) to allow you to 
connect to Tursa.
   
!!! tip
    Remember that you will need to use both a key and your password to
    transfer data to Tursa.

Once we know our keys are setup correctly, we are now ready to transfer 
data directly between the two machines. We begin by combining our important 
research data in to a single archive file using the following command:

    tar -czf all_my_files.tar.gz file1.txt file2.txt file3.txt

We then initiate the data transfer from our system to Tursa, here using
`rsync` to allow the transfer to be recommenced without needing to start
again, in the event of a loss of connection or other failure. For example, 
using the SSH key in the file `~/.ssh/id_RSA_A2` on our local system:

    rsync -Pv -e"ssh -c aes128-gcm@openssh.com -i $HOME/.ssh/id_RSA_A2" ./all_my_files.tar.gz otbz19@tursa.dirac.ed.ac.uk:/home/z19/z19/otbz19/

Note the use of the `-P` flag to allow partial transfer -- the same
command could be used to restart the transfer after a loss of
connection. The `-e` flag allows specification of the ssh command - we
have used this to add the location of the identity file. 
The `-c` option specifies the cipher to be used as `aes128-gcm` which has 
been found to increase performance. Unfortunately the `~` shortcut is not
correctly expanded, so we have specified the full path. We move our 
research archive to our project work directory on Tursa.

!!! note
    Remember to replace `otbz19` with your username on Tursa.

If we were unconcerned about being able to restart an interrupted
transfer, we could instead use the `scp` command,

    scp -c aes128-gcm@openssh.com -i ~/.ssh/id_RSA_A2 all_my_files.tar.gz otbz19@transfer.dyn.tursa.ac.uk:/home/z19/z19/otbz19/

but `rsync` is recommended for larger transfers.
