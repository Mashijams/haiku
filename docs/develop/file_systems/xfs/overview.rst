The XFS File System
===================

This document describes How to test XFS file system, XFS file system API for Haiku and Its current status on Haiku.

Testing XFS File System
-----------------------

There are three ways we can test XFS : 

- Using xfs_shell.
- Using userlandfs.
- Building a version of Haiku with XFS support and then mounting file system.

But before that we will need to create XFS images for all testing purposes. 

Creating File system images
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Currently only linux has full XFS support so we will use linux for generating file system images.

First we need to create an empty sparse image using command:: 

   $ dd if=/dev/zero of=fs.img count=0 bs=1 seek=5G
   
The output will be:: 

     0+0 records in
     0+0 records out
     0 bytes (0 B) copied, 0.000133533 s, 0.0 kB/s 
     
Do note that we can create image of whatever size or name we want, for example above command creates fs.img of size 5 GB, if we alter seek = 10G it will create fs.img with size 10 GB.

XFS file system on linux supports two versions, V4 and V5.

To put XFS V5 file system on our sparse image run command::

    $ /sbin/mkfs.xfs fs.img
    
The output will be::

    meta-data=fs.img                 isize=512    agcount=4, agsize=65536 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=1        finobt=1, sparse=1, rmapbt=0
             =                       reflink=1
    data     =                       bsize=4096   blocks=262144, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
    log      =internal log           bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0
    
To put XFS V4 file system on our sparse image run command::

    $ /sbin/mkfs.xfs -m crc=0 file.img
    
The output will be::

    meta-data=fs.img                 isize=256    agcount=4, agsize=327680 blks
             =                       sectsz=512   attr=2, projid32bit=0
    data     =                       bsize=4096   blocks=1310720, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0
    log      =internal log           bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0
    
**The Linux kernel will support older XFS v4 filesystems by default until 2025 and Support for the V4 format will be removed entirely in September 2030**

Now we can mount our file system image and create entries for testing XFS Haiku driver.

Test using xfs_shell
^^^^^^^^^^^^^^^^^^^^^^^

The idea of fs_shell is to run the file system code outside of Haiku. We can run it as an application,
it provides a simple command line interface to perform various operations on the file system (list
directories, read and display files, etc).

First we have to compile it::

  jam "<build>xfs_shell"

Then run it::

  jam run ":<build>xfs_shell" fs.img
  
Where fs.img is the file system image we created from linux kernel.

Test directly inside Haiku
^^^^^^^^^^^^^^^^^^^^^^^^^^

First build a version of Haiku with XFS support, To do this we need to add "xfs" to the image definition `here <https://git.haiku-os.org/haiku/tree/build/jam/images/definitions/minimum#n239>`__.

Then compile Haiku as usual and run the resulting system in a virtual machine or on real hardware.

We can then try to mount an XFS file system using command on Haiku::

  mount -t xfs <path to image> <path to mount folder>
  
for example::

  mount -t xfs /boot/home/Desktop/fs.img /boot/home/Desktop/Testing

Here fs.img is file system image and Testing is mount point.

Test using userlandfs
^^^^^^^^^^^^^^^^^^^^^

To be updated


Haiku XFS API
-------------


Current Status of XFS
---------------------

Currently we only have read support for XFS, below briefly summarises read support for all formats.  


Directories
^^^^^^^^^^^

Short-Directory
   Stable read support for both V4 and V5 inside Haiku.

Block-Directory
   Stable read support for both V4 and V5 inside Haiku.
   
Leaf-Directory
   Stable read support for both V4 and V5 inside Haiku.

Node-Directory
   Stable read support for both V4 and V5 inside Haiku.
   
B+Tree-Directory
   Unstable read support for both V4 and V5, due to so many read from disk entire process inside Haiku is too slow.
 
 
Files
^^^^^

Extent based Files
 | *xfs_shell* - stable read support for both V4 and V5.
 | *Haiku* - Unstable, Cat command doesn't print entire file and never terminates process.
   
B+Tree based Files
 | *xfs_shell* - stable read support for both V4 and V5.
 | *Haiku* - Unstable, Cat command doesn't print entire file and never terminates process.
 

Attributes
^^^^^^^^^^

Currently we have no extended attributes support for xfs.


Symlinks
^^^^^^^^

Currently we have no symlinks support for xfs.


XFS V5 exclusive features
^^^^^^^^^^^^^^^^^^^^^^^^^

MetaData Checksumming
   Metadata checksums for superblock, Inodes, and data headers are implemented.
   
Big Timestamps
   Currently we have no support.

Reverse mapping btree
   Currently we have no support, this data structure is still under construction and testing inside linux kernel.

Refrence count btree
   Currently we have no support, this data structure is still under construction and testing inside linux kernel.
   

Write Support
^^^^^^^^^^^^^

Currently we have no write support for xfs.


References
----------

The best and only reference for xfs is latest version of "xfs_filesystem_structure".

The pdf version of above Doc is `here <http://ftp.ntu.edu.tw/linux/utils/fs/xfs/docs/xfs_filesystem_structure.pdf>`_
