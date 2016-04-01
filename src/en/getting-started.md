Title: Getting started with Juju 2.0
Todo: remove ppa/devel after release

# Getting started with Juju 2.0

The instructions here will get you up and running and deliver the best-possible
experience. At the moment, that means using the very latest release of 
Ubuntu, [16.04LTS(Xenial)](http://cdimage.ubuntu.com/releases/16.04/).

If you are using a different OS, please read the 
[general install instructions here](./getting-started-general.md).

To get the best experience, as well as Juju, this guide will also set up:
   
   - LXD - a hypervisor for LXC, providing fast, secure containers.
   - ZFS - a combined filesystem/LVM which gives great performance.

Both the above are provided with Ubuntu 16.04LTS.

## Installing the software

Run the following commands to install the required software:

```no-highlight  
  sudo add-apt-repository ppa:juju/devel
  sudo apt-get update
  sudo apt install zfsutils-linux
  sudo apt install lxd
  sudo apt install juju2
```

## Preparing ZFS

Using ZFS we can create sparse backing-storage for any of the containers which
LXD creates for Juju. You can create this storage anywhere (e.g. the fastest
drive you have). This example creates a 50G file to use:

  sudo mkdir /var/lib/zfs
  sudo truncate -s 50G /var/lib/zfs/lxd.img
  sudo zpool create lxd /var/lib/zfs/lxd.img
  
As this is sparse storage, it won't actually take up disk space until it is 
actually being used. You can check the file has been added to a ZFS pool with 
the following command:
  
  sudo zpool status

This should indicate that the newly created pool is 'ONLINE' and ready.

## Initialise LXD

Now we need to tell LXD about this storage

```bash
sudo lxd init --auto --storage-backend zfs --storage-pool lxd
newgrp - 
```

To have the group changes take effect, you will now need to logout of your
current session and log in once again, or execure the following in your shell:
  
```no-highlight
su -l $USER
```
## Createa controller

Juju needs to create a controller instance in the cloud to manage the models
you create. We use the `juju bootstrap` command to create that controller. For 
use with our LXD 'cloud', we will create a new controller called 'lxd-test':

```bash
juju bootstrap --config default-series=xenial lxd-test lxd
```

This may take a few minutes as initially, LXD needs to download an image for 
Xenial. 

Once the process has completed, Juju now has a controller. You can check this:

```bash
juju list-controllers 
```
...will return a list of the controllers known to Juju, which at the moment is
the one we just created:
  
```no-highlight
CONTROLLER       MODEL    USER         SERVER
local.lxd-test*  default  admin@local  10.0.3.124:17070
```

## Create a model

Before we deploy any services, we will create a model. A model in Juju is like a 
workspace where you can deploy and relate the services you want. A controller 
can create many models, and as you will see, Juju is able to share selected 
models with other users too. For now we will create a new model called 'test':

```bash
juju create-model test
```

The model is now created. Each time you create a new model, control is 
automatically switched to that model. You can find out which model you are 
currently using with the `juju switch` command:

```bash 
juju switch
```
...will return model and controller we are currently using:

```no-highlight
local.lxd-test:test
```
## Deploying services

Juju is now ready to deploy any services from the hundreds included in the
[https://jujucharms.com](juju charm store). It is a good idea to test your new 
model. How about a Mediawiki site?

```bash
juju deploy mediawiki-single
```
This will fetch a 'bundle' from the Juju store. A bundle is a pre=packaged set
of services, in this case the 'Mediawiki' service, and a database to run it 
with. Juju will install both these services and add a relation between them - 
this is part of the magic of Juju: it isn't just about deploying services, Juju 
also knows how to connect them together.

Installing shouldn't take long. you can check on how far Juju has got by running
the command:
 
```bash
juju status
```
When the services have been installed the output to the above command will look
something like this:

![juju status](./media/juju-mediawiki-status.png)





