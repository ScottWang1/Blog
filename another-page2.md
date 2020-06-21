---
layout: default
---

## Introduction
Git is an open source distributed version control system for agile and efficient processing of any small or large projects. Git is different from the commonly used version control tools CVS, Subversion, etc. It uses a distributed version library without server-side software support.

##Git installation and configuration
The work of Git needs to call the code of libraries such as curl, zlib, openssl, expat, libiconv, etc., so you need to install these dependent tools first.
On a system with yum (such as Fedora) or a system with apt-get (such as Debian), you can install it with the following command:
Each Linux system can be installed using its installation package management tools (apt-get, yum, etc.):



## Git workflow
What I used is CentOS 8, so my install steps are:

    $ yum install curl-devel expat-devel gettext-devel \
       openssl-devel zlib-devel     
     $ yum -y install git-core     
     $ git --version
     git version 1.7.1

## Git basic operations
Initializing a Repository in an Existing Directory:
```$xslt
$ git init
```
 You can accomplish that with a few git add commands that specify the files you want to track, followed by a git commit:
 ```$xslt
$ git add *.c
$ git add LICENSE
$ git commit -m 'Initial project version'
```
if you want to clone the Git linkable library called libgit2, you can do so like this:
```$xslt
$ git clone https://github.com/libgit2/libgit2
```
There is the lifecycle of the status of your files.
![图片pic1]({{ "/assets/images/lifecycle.png" | absolute_url }})


```$xslt
Checking the Status of Your Files:     $ git status
Moving Files:                          $ git mv file_from file_to
Viewing the Commit History:            $ git log
Fetching and Pulling from Your Remotes:$ git fetch <remote>
Pushing to Your Remotes:               $ git push origin master
Tagging:                               $ git tag
see the tag data along with the commit:$ git show [version]
With the rebase command, you can take all the changes that were committed on one branch 
and replay them on a different branch: $ git rebase master                       
```


## Git branch management
```$xslt
Create new branch: $ git branch (branchname)
Switch branch:     $ git checkout (branchname)
Create and Switch :$ git checkout -b (branchname)
Merge:             $ git merge hotfix
```


## Distributed Git 
##### Centralized Workflow
In centralized systems, there is generally a single collaboration model — the centralized workflow. One central hub, or repository, can accept code, and everyone synchronizes their work with it. A number of developers are nodes — consumers of that hub — and synchronize with that centralized location.
![图片pic1]({{ "/assets/images/centralized_workflow.png" | absolute_url }})

This means that if two developers clone from the hub and both make changes, the first developer to push their changes back up can do so with no problems. The second developer must merge in the first one’s work before pushing changes up, so as not to overwrite the first developer’s changes. This concept is as true in Git as it is in Subversion (or any CVCS), and this model works perfectly well in Git.
##### Integration-Manager Workflow
Because Git allows you to have multiple remote repositories, it’s possible to have a workflow where each developer has write access to their own public repository and read access to everyone else’s. This scenario often includes a canonical repository that represents the “official” project. To contribute to that project, you create your own public clone of the project and push your changes to it. Then, you can send a request to the maintainer of the main project to pull in your changes. The maintainer can then add your repository as a remote, test your changes locally, merge them into their branch, and push back to their repository. The process works as follows (see Integration-manager workflow.):
![图片pic1]({{ "/assets/images/integration-manager.png" | absolute_url }})
```
1.The project maintainer pushes to their public repository.
2.A contributor clones that repository and makes changes.
The contributor pushes to their own public copy.
3.The contributor sends the maintainer an email asking them to pull changes.
4.The maintainer adds the contributor’s repository as a remote and merges locally.
5.The maintainer pushes merged changes to the main repository.
```
##### Dictator and Lieutenants Workflow
This is a variant of a multiple-repository workflow. It’s generally used by huge projects with hundreds of collaborators; one famous example is the Linux kernel. Various integration managers are in charge of certain parts of the repository; they’re called lieutenants. All the lieutenants have one integration manager known as the benevolent dictator. The benevolent dictator pushes from their directory to a reference repository from which all the collaborators need to pull. The process works like this (see Benevolent dictator workflow.):
![图片pic1]({{ "/assets/images/benevolent-dictator.png" | absolute_url }})

```$xslt
1.Regular developers work on their topic branch and rebase their work on top of master. 
The master branch is that of the reference repository to which the dictator pushes.
2.Lieutenants merge the developers' topic branches into their master branch.
3.The dictator merges the lieutenants' master branches into the dictator’s master branch.
4.Finally, the dictator pushes that master branch to the reference repository so the other 
developers can rebase on it.
```

## Thoughts
After learning Git, I think it is very powerful and it does can give us a lot of help by its distributed version control system. For example, If you have written long articles in Microsoft Word, then you must have this experience: If I am using Word and i want to delete a paragraph, but I am afraid I won’t find it in the future, So i have to  to "save the current file as..." a new Word file, and then change it to a certain degree, and then "save as..." a new file, so keep changing, and finally when i  want to retrieve the deleted text, i can’t remember which file was saved before the deletion. I have to find each file one by one, which is a really troublesome.
In this case, it can be solved easily by Git. The only thing i need do is to create a new branch at the point which i want to delete a paragraph and it can be found seaily in the future, which saves a lot of trouble.

[back](./)