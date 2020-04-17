---
layout: post
title: "Installing Rancher OS on FreeNAS 11"
image: "https://image.slidesharecdn.com/rancheros-170731045638/95/rancher-os-a-simplified-linux-distribution-built-from-containers-for-containers-1-638.jpg?cb=1501477377"
date: "2017-05-11 12:53:08"
---
<h1>Installing Rancher OS on FreeNAS 11</h1>

Hello,

Since the deprecation of FreeNAS Corral, many users have found themselves with the lack of docker images when moving back to the latest releases of 11.x or are stuck on the release version of Corral wondering how long this release will last. Will it make it to August when the docker add-ons are scheduled to be released in 11.x. I was in the same boat, I started reading some forums on FreeNAS to try and figure out when and if dockers will be implemented. From what I read I started to see a spec of light in the far distance, but I wanted to know more about what was coming so I moved to the bug tracker. <!--more-->

In the bug tracker, there are 2 bugs slotted in 11.1 which is currently due in August, these 2 bugs seem to be the first iteration of docker on FreeNAS 11 using the RancherOS. I can only assume that when it is released it will be a raw docker install, automatic VM creation but command line from there, will they leverage the FreeNAS Docker Images repository or will they try and leverage some built in Rancher functionality I wait to find out more as we get closer to the deadline.

Regardless August is too far for me to wait around, I decided to try and get Dockers running on my install as soon as possible. I have now set out to provide this tutorial on how to get RancherOS installed and running on FreeNAS 11.x. My install is not perfect I am still trying to get NFS shares working on my install but without luck, if anyone is able to shed some light on this issue please post in the comments below.

tl;dr Oh noes Corral what now!!! What I can't wait.... Here's what I am doing now while I wait.
<h2>To the fun stuff... The dreadful command line...</h2>
So first thing is first we are going to need to get to the command line of your FreeNAS install. If you have never done this you will need to get familiar with a few things which I am not going into today. There are many posts around that will explain this stuff is possibly better detail than I can.

SSH into your server and the first thing you want to do is set up the iohyve environment. Replace with the ZFS Pool you want to have iohyve use for its guests and for the network adapter you want to use for guests. When setting the con parameter below this sets the console path to use you can run iohyve conlist to show a list of console ports in use, just use pick a number that is not in use, in my example, I am using 0.
<pre lang="bash">iohyve setup pool=DATASTORE kmod=1 net=em2
</pre>
Now we are going to configure the RancherOS container.
<pre lang="bash">iohyve create RancherOS 100G
iohyve set RancherOS loader=grub-bhyve boot=1 ram=8G cpu=1 con=nmdm0 os=debian
</pre>
Now let's download the latest release of RancherOS from their website check the list to make sure it is listed before we proceed.
<pre lang="bash">iohyve fetch https://releases.rancher.com/os/latest/rancheros.iso
iohyve isolist
</pre>
Once the ISO has been downloaded, moved and verified we just want to verify that the container is listed for verification and then launch the container with the ISO attached for install.
<pre lang="bash">iohyve list
iohyve install RancherOS rancheros.iso
</pre>
Alright, this is where I messed up on the install part, grub is not able to detect how to boot everything up (there is a way to pre-map this but I do that later for the installed image). You should see a prompt showing you that the container might be stuck at a grub screen... its right!! Best now is to open a new SSH window to run console commands in. I found this was easiest while debugging all of the issues I was running into.
<blockquote>Installing Rancher...
GRUB Process does not run in the background....
If your terminal appears to be hanging, check iohyve console Rancher in second terminal to complete GRUB process...</blockquote>
<pre lang="grub">iohyve console RancherOS 
</pre>
Now we should see a grub screen. Run the following and the RancherOS should boot. Note: the initrd and the vmlinuz might not have these revision numbers please run<strong> <em>ls (cd0,1)/boot</em> </strong>to verify the versions.
<pre lang="grub">set root=(cd0,msdos1)
linux /boot/vmlinuz-4.9.24-rancher rancher.password=rancher rancher.state.autoformat=[/dev/sda,/dev/vda]
initrd /boot/initrd-v1.0.1
boot
</pre>
Voila!! Wait we're not done, this is just booting directly from CD, we need to install it to the local drive.
<pre lang="bash">              ,        , ______                 _                 _____ _____TM
  ,------------|'------'| | ___ \               | |               /  _  /  ___|
 / .           '-'    |-  | |_/ /__ _ _ __   ___| |__   ___ _ __  | | | \ '--.
 \/|             |    |   |    // _' | '_ \ / __| '_ \ / _ \ '__' | | | |'--. \
   |   .________.'----'   | |\ \ (_| | | | | (__| | | |  __/ |    | \_/ /\__/ /
   |   |        |   |     \_| \_\__,_|_| |_|\___|_| |_|\___|_|     \___/\____/
   \___/        \___/     Linux 4.9.24-rancher

         RancherOS v1.0.1 rancher ttyS0
         docker-sys: 172.18.42.2 lo: 127.0.0.1
rancher login:
</pre>
At this point, we can now log in using rancher/rancher and we can start the install. The tutorial I used for this was for a cloud-based install so it wanted me to set up a cloud-config.yml and include my SSH keys. If this is required I am sure you will be able to locate it in the RancherOS install documentation, I just use the grub mapping on the final install to set the rancher password, I know this is not a secure way to do this but for me it's a home/development machine and I am not concerned at this point. Ahh what the hey I'll show it anyway.

So from here, you want to log in and create a new file using vi.
<pre lang="bash"> vi cloud-config.yml
</pre>
Once you have launched vi you can press the insert key and then paste in the below with your information (The Dash between desc and key is important) and save the file (Esc, : wq enter)
<pre lang="bash"># cloud-config.yml
ssh_authorized_keys:
Your_Name_Here - Your_Public_SSH_Key_Here
</pre>
Now let's launch the installer using the yml config.
<pre lang="bash">sudo ros install -c cloud-config.yml -d /dev/sda
</pre>
<blockquote>&gt; INFO[0000] No install type specified...defaulting to generic
Installing from rancher/os:v1.0.1
Continue [y/N]: y
&gt; INFO[0018] start !isoinstallerloaded
&gt; INFO[0018] trying to load /bootiso/rancheros/installer.tar.gz
23b9c7b43573: Loading layer 4.23 MB/4.23 MB
f90562450ed7: Loading layer 14.96 MB/14.96 MB
94dbf06f5a27: Loading layer 4.608 kB/4.608 kB
52125c070e5e: Loading layer 18.08 MB/18.08 MB
051a95b6eaa9: Loading layer 1.636 MB/1.636 MB
a6fd95b21434: Loading layer 1.536 kB/1.536 kB
c52e27553689: Loading layer 2.56 kB/2.56 kB
7b425eeedf72: Loading layer 3.072 kB/3.072 kB
&gt; INFO[0021] Loaded images from /bootiso/rancheros/installer.tar.gz
&gt; INFO[0021] starting installer container for rancher/os-installer:latest (new)
Installing from rancher/os-installer:latest
mount: /dev/sr0 is write-protected, mounting read-only
Continue with reboot [y/N]: y
&gt; INFO[0076] Rebooting</blockquote>
When you say yes to the reboot it will just hang, but the system really powered off not rebooted. Now let's move back to the original ssh window that you had opened if we run <em>iohyve list
</em>we will set that the RancherOS is not running before we start it up we want to make a grub map so we do not always have to run grub commands to start it up.
<pre lang="bash">cd /mnt/iohyve/RancherOS/
nano grub.cfg
</pre>
Paste the following into nano and save it. (Ctrl+Shift+X
UNSECURE PASSWORD SET IN GRUB
<pre lang="bash">set root=(hd0,msdos1)
linux /boot/vmlinuz-4.9.24-rancher rancher.password=rancher printk.devkmsg=on rancher.state.dev=LABEL=RANCHER_STATE rancher.state.wait console=tty0
initrd /boot/initrd-v1.0.1
boot
</pre>
SECURE, SSH WITH PRIVATE KEY ACCESS ONLY
<pre lang="bash">set root=(hd0,msdos1)
linux /boot/vmlinuz-4.9.24-rancher printk.devkmsg=on rancher.state.dev=LABEL=RANCHER_STATE rancher.state.wait console=tty0
initrd /boot/initrd-v1.0.1
boot
</pre>
Now we need to configure the container to read the grub.cfg file I did this by changing the container os to custom
<pre lang="bash">iohyve set RancherOS os=custom
</pre>
Now let's start the container
<pre lang="bash">iohyve start RancherOS
</pre>
You can launch the console in another window just like before and log in with rancher/rancher if you chose to load the unsecured grub file, otherwise, you will have to ssh into the ip address that you see in the console view.

That's it that is installing RancherOS on FreeNAS 11.

Now that we are done installing we should configure FreeNAS to auto start iohyve on boot, we do this using Tunables in FreeNAS. Set up the following tunable in your FreeNAS UI.

<img class="size-full wp-image-30 alignnone" src="/assets/uploads/2017/05/iohyve_flags_tunables_freenas-1.png" alt="" width="299" height="290" /><img class="size-medium wp-image-29 alignnone" src="/assets/uploads/2017/05/iohyve_enable_tuneable_freenas.png" alt="" width="299" height="291" />

I will be posting another article on how to install Rancher Server to get a web based UI to manage docker containers. Otherwise, you can add dockers using the command line. The only missing link is NFS at this point, something I am still working toward finding a solution.