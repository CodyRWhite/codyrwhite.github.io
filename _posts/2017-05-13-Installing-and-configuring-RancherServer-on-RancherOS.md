---
layout: post
title: "Installing and configuring RancherServer on RancherOS"
date: 2015-05-13 #hh:mm:ss -0000
categories: Docker FreeNAS RancherOS RancherServer
---


To install Rancher Server run the following command from your RancherOS installation.

sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server

Once the install process has completed wait a couple minutes while the OS boots and then you can browse to the website. Use the RancherOS IP address and port 8080. Once the site had loaded and you click Got It. There will be a warning about adding a host. Click Add a host. From there just click save, this is just letting you know that the IP address of your host is not public are you sure you want to continue. Rancher Server expects to be public and usually run in the Cloud so other RancherOS’s can all connect to the server. In this example, we are installing the server as a docker inside of the Rancher OS we are going to manage with the Server.

Then follow the next steps to install the Rancher Server Agent on the RancherOS installation. I normally leave everything as it is and just paste step 5 into the ssh for the RancherOS. Once the agent has come online and authenticated with the server the warning at the top will disappear.

If you hover over stacks and lick on infrastructure you will see the default containers that the server installs to monitor the RancherOS image. From here you can click on catalog and install some premade images for RancherServer. You are not able to launch a default Docker images from Docker Hub from the GUI, but you are able to run the command line to do so from the RancherOS install. Once a docker image has been installed I THINK you can manage it from the server. However, I have started a Rancher Server repository on GitHub, https://github.com/codyrwhite/rancher-catalog this can be added under settings and you will be able to search my catalog which is still under development.
