---
layout: default
---

## Introduction




The Linux kernel was originally written by Finn Linus Torvalds for personal hobbies while studying at the University of Helsinki.
Linux is a set of Unix-like operating systems that are free to use and freely disseminate. It is a multi-user, multi-tasking, multi-threading and multi-CPU operating system based on POSIX and UNIX.
Linux can run major UNIX tool software, applications and network protocols. It supports 32-bit and 64-bit hardware. Linux inherits Unix's network-centric design philosophy, and is a stable multi-user network operating system..

## Install

What I installed is the CentOS 8 in VMware Fusion in Mac. First you need to [download the CentOS][jekyll-docs2] and [download the VMware Fusion][jekyll-docs3] .

Run the VMware Fusion and select the DVD that you had downloaded and name it and Split space for it, the default is 20GB. And don't forget setting the share folder which can let Linux visit your Mac's folder by VMware tools.




## System startup

When Linux starts, we will see a lot of startup information.
The startup process of the Linux system is not as complicated as everyone thinks. 

The process can be divided into 5 stages:
The boot of the kernel;
Run init;
system initialization;
Establish a terminal and
The user logs into the system.

Shutdown command：
``` 
shutdown –h now 
``` 
if you want to drawback, input: 
``` 
# shutdown -c 
``` 



## System directory structure

Input:
 ```
 ls /
```

you can see the list of system directory structure.

Here are the explanations of those directories:
    
    /bin：As short of 'binary', which contains the most frequently used commands.
    /boot: Here are stored some core files used when starting Linux, including some connection files and image files.
    /dev: dev is the abbreviation of Device, and the external devices in Linux are stored in this directory. The way to access devices in Linux is the same as the way to access files.
    /etc: This directory is used to store all the configuration files and subdirectories needed for system management.
    /home: The user's home directory, in Linux, each user has its own directory, generally the directory name is named after the user's account.
    /lib: This directory stores the system's most basic dynamic link shared library, its role is similar to the DLL file in Windows. Almost all applications need to use these shared libraries.
    /opt: This is the directory where additional software is installed for the host. For example, if you install an ORACLE database, you can put it in this directory. The default is empty.
    /root: This directory is the system administrator's home directory, also known as the super administrator.
    /sbin: s is the meaning of Super User. Here is stored the system management program used by the system administrator.
    /selinux: This directory is unique to Redhat/CentOS. Selinux is a security mechanism similar to windows firewall, but this mechanism is more complicated. This directory stores selinux-related files. 
    /srv: This directory stores some data that needs to be extracted after the service is started. 
    /tmp: This directory is used to store some temporary files.
    /usr: This is a very important directory. Many user applications and files are placed in this directory, similar to the program files directory under windows. 
    /usr/bin: Applications used by system users.




## Basic file attributes

chgrp: change file group, input: 
``` 
# chgrp 
``` 
    ls: List directories and file names
    cd: switch directory
    pwd: display the current directory
    mkdir: create a new directory
    rmdir: delete an empty directory
    cp: copy files or directories
    rm: remove files or directories
    mv: move files and directories, or modify the names of files and directories


## User and user group management

if you want to add new user:
```
useradd [option] [username]
```
options:

    -c comment Specify a comment description.
    -d directory Specifies the user's home directory. If this directory does not exist, you can also use the -m option to create the home directory.
    -g user group Specifies the user group to which the user belongs.
    -G user group, user group Specifies the additional group to which the user belongs.
    -s Shell file Specify the user's login shell.
    -u user number Specify the user number of the user. If the -o option is also available, you can reuse the identification number of other users.

If you want to delete:
```$xslt
userdel -r [username]
```

## Disk management
df: List the overall disk usage of the file system
```$xslt
df [option] [name]
```
the options are listed down below:

    -a: List all file systems, including system-specific /proc file systems;
    -k: display each file system with the capacity of KBytes;
    -m: display each file system with the capacity of MBytes;
    -h: display in GBytes, MBytes, KBytes and other formats that people can read easily;
    -H: replace M=1024K with M=1000K;
    -T: display the file system type, along with the filesystem name of the partition (eg ext3) is also listed;
    -i: display the number of inodes without using hard disk capacity


fdisk: used for disk partition
```$xslt
fdisk [-l] [name]
```

## Vi/Vim
input:
```$xslt
vim [path]
```
to start,and then input 

    i Switch to input mode to enter characters.
    x Delete the character at the current cursor position.              
    : Switch to the bottom line command mode to enter commands at the bottom line
    
In the bottom line mode, input:

     q Exit the program
     w Save the file


## Thoughts
From learning the Linux system,  i realized that Linux and MacOS are both developed from Unix system, there are a lot of similar places. What's more, The Linux system tool chain is complete, and a suitable development environment can be configured with simple operations, which can simplify the development process, reduce the obstacles of simulation tools in development, and make the system more portable; I think that is the main reason that why nowadays most programmers prefer Linux to other systems.

<div style='display: none'>

_yay_

   Text can be **bold**, _italic_, or ~~strikethrough~~.
   
   > This is a blockquote following a header.
   >
   > When something is important enough, you do it even if the odds are not in your favor.

### Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```


</div>

[jekyll-docs2]: https://www.centos.org/download/
[jekyll-docs3]: https://www.vmware.com/products/fusion/fusion-evaluation.html

[back](./)





