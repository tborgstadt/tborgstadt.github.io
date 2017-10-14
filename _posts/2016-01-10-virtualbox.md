---
layout: post
title: VirtualBox 4.3 Shared Folders Setup
description: Setup for using Google Drive with VirtualBox on Ubuntu
tags: [virtualbox, google drive, ubuntu]
comments: True
---
Virtualbox is a valuable tool for prototyping new applications or testing unfamiliar applications without risk to your production host. I use it for these reasons but also to run different operating systems when needed. For example, I have a Windows guest for Microsoft applications that I use. I also use guest machines to stand up development database servers. For example, I have a postgreSQL guest and a MySQL guest. 

This post explains how to share a host file folder with a VirtualBox guest. I have also included my experience using insync and Google Drive.

Objectives

1. Synchronize select host folders with Google Drive for backup and accessiblity from the web
2. Access selected host folders from a VirtualBox guest(s) as needed

### Synchronizing Google Drive
I use [insync](https://www.insynchq.com/){:target="_blank"} to synchronize folders from Google Drive to my local Ubuntu host. Insync is available for Windows, Mac, and Linux. My experience with insync is very positive. The key features I like include:

* sychronize Google Drive folders from up to 3 separate Google Accounts
* ability to select exactly which Google Drive folders you want to synchronize
* it runs on Linux
* insync is customer focused, interested in feedback and suggestions for their product, not too much but in touch and responsive which is nice


### Configuring VirtualBox Shared Folders

The following can be done to share any host file folder with a guest. I have shared my synchronized Google Drive folder here only for example.

#### From the VirtualBox host:

* Open VirtualBox Manager
* Select the guest needing access to a host folder (e.g. 'postgre')
* Under settings select 'Shared Folders'
* Click on 'Machine Folders' and click 'Add new shared folder definition'
* Browse to the local path of the desired folder
* Select 'Auto-mount' and 'Make Permanent'
* Click 'OK' and 'OK'

<figure><a><img src="/images/vbxshrdfldr1.png"></a>
<figcaption></figcaption>
</figure>

#### From a VirtualBox guest:

* Login
* Add your user to the vbox group 
{% highlight bash %}
tom@postgre:~$ sudo usermod -aG vboxsf tom 
{% endhighlight %}
* Logout and log back in
* Browse to the system '/media' folder and verify the mapped folder is present. By default VirtualBox will mount the shared folder in the '/media' folder with prefix 'sf_'. We can see it below both from Nautilus and the terminal.

<figure><a><img src="/images/vbxshrdfldr2.png"></a>
<figcaption></figcaption>
</figure>

#### Change the guest mount directory:

I prefer to have the shared folder accessbile from my home directory (instead of folder '/media') and this is easily achieved by setting the VirtualBox shared folders mount directory option for the guest using the command line on the host.

From the host:
{% highlight bash %}
# Note this is set per guest system, it is not a global setting
tom@host:~$ vboxmanage guestproperty set "postgre" /VirtualBox/GuestAdd/SharedFolders/MountDir /home/tom
{% endhighlight %}

Restart the guest and now the shared folder is available from the home location.

<figure><a><img src="/images/vbxshrdfldr3.png"></a>
<figcaption></figcaption>
</figure>

There you have it.








