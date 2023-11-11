> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [tech.aufomm.com](https://tech.aufomm.com/my-nixos-journey-intro-and-installation/#Why-Nix)

> IntroI switched back to Linux from Mac a couple years ago when I needed to use MetalLB to provision I......

[](#Intro "Intro")Intro
-----------------------

I switched back to Linux from Mac a couple years ago when I needed to use [MetalLB](https://metallb.universe.tf/) to provision IPs for Kubernetes LoadBalancer Service. PopOS was the distro I settled with. This distro is just SO EASY to use and all the tiny details make it the best distro for anything making a switch.

> The big tech companies are collecting outrageous amount of user data nowadays. You should not be forced to register accounts or answer so many questions before using your computer. In my opinion Linux is good enough for everyone even for gamers (Thanks to [Proton](https://en.wikipedia.org/wiki/Proton_(software))). Please have a try when you can.

### [](#Why-Nix "Why Nix")Why Nix

PopOS uses APT package manager by default. I was also using [Homebrew](https://brew.sh/) to manage software that are not on APT. Homebrew is so easy to use, one single CLI interface handles everything. Because it is more and more popular, many software developers choose to publish their software on homebrew. As much as I want to love homebrew, I notice that it would install and OVERRIDE my default software without me knowing it.

For example, I was trying to integrate [gsettings](https://manpages.ubuntu.com/manpages/trusty/man1/gsettings.1.html) to my scripts for setting up my new VMs and I noticed the commands did not work. After doing some digging, I found that the default _gsettings_ somehow pointing to the one managed by homebrew. After deleting it, the commands worked as expected.

_**The feeling of losing control of my machines is REALLY REALLY scary.**_ I was looking for alternative right away and that was how I found Nix.

### [](#Good "Good")Good

> I don’t think I can explain the benefits better than this [blog post](https://sioodmy.dev/blog/nixos-guide/#tldr).

[Nix](https://en.wikipedia.org/wiki/Nix_(package_manager)) is a very special package manager that allows user to install and configure software declaratively. This means these configuration files can be version controlled and everything is reproducible. Currently I am using dotfile and some scripts to install software and symlink configurations as a disaster recovery plan. Nix can do a much better job here.

Another reason is how packages are built. Ever since Apple released their computers with M1 chip, there are more demands for applications to be built for ARM based processor. With Nix, even when developers do not provide a binary for ARM, you can install and use the app on ARM based computer the same way as you would with Intel or AMD.

### [](#Bad "Bad")Bad

*   Nix is **HARD** to learn and its documentation is a mess. Not many people are willing to read the whole manual (especially it is so big) to start using something. IMO a good software should have an easy entry point to get people to start. These users will learn whatever they need when they use it.
*   Most docs, blog posts or YouTube videos are either too simple, too difficult, outdated or use Nix the wrong way. For example, you may find a lot of docs talking about using `nix-env` to install packages. However, `nix-env` should be avoided according to this [discussion](https://discourse.nixos.org/t/difference-between-installing-via-nix-env-and-configuration-nix/7473/3).
*   Being declarative is good but it also means the GitHub repos you can find are someone’s _COMPLETE_ solution. It is very hard for new users to understand and learn from those repos.

Most likely you will end up being frustrated over and over again. There are many discussions out there, this [Hacker news post](https://news.ycombinator.com/item?id=30057287) should give you some ideas.

### [](#Why-am-I-still-using-Nix "Why am I still using Nix")Why am I still using Nix

> I am a fighter and not a quitter! 🤡

Seriously though, the benefits Nix brings are just too tempting.

Enough talking, let’s start using Nix.

[](#Installation "Installation")Installation
--------------------------------------------

> In the demo I will install NixOS 22.11 on a Proxmox VM.

Installation is pretty straight forward. I downloaded the ISO with GUI from [official site](https://nixos.org/download.html), you should be able to follow the prompt to install and set up your initial user successfully.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/1.png)

It doesn’t matter what Desktop you want to use, in the demo I am going to install with option `No Desktop`.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/2.png)

[](#Getting-Start "Getting Start")Getting Start
-----------------------------------------------

### [](#Initial-set-up "Initial set up")Initial set up

The first time we log in to NixOS, we need to install a few packages to help us use the system. Let’s Navigate to `/etc/nixos` directory and open the `configurations.nix` file with the nano editor as root.

We should see a lot of staff on this file. Here are the things we will do.

*   Install editor  
    I will install [Micro](https://micro-editor.github.io/) editor. Let’s put micro under `environment.systemPackages`.
*   Enable sshd  
    This allows me to ssh into the VM via our own terminal. We just need to uncomment `services.openssh.enable = true;`.
*   (Optional) Enable QEMU Guest Agent  
    This helps us to control VM via Proxmox UI. Let’s add `services.qemuGuest.enable = true;` to `configurations.nix`.

Once we’ve done, we should see something similar like below.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/3.png)

Let’s press `Ctrl + s` to save and then `Ctrl + x` to exit. Next we need to run `sudo nixos-rebuild switch`. This command will apply and activate the changes.

Once it is done we can ssh to our server.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/4.png)

### [](#Packages-and-configs "Packages and configs")Packages and configs

Let’s take a look at `/etc/nixos` folder first.

```
.
├── configuration.nix
└── hardware-configuration.nix
```

We can see we’ve got two files here, `configuration.nix` and `hardware-configuration.nix`. Normally you don’t need to touch `hardware-configuration.nix`. We already learn that we can put packages names in `configuration.nix` to install them. How do we know the right package names to use? What if I need to change some user settings like adding SSH key?

> Packages name are normally consistent with what we know them for, for example, micro, htop, vim etc. However, there are cases that the package names are different. For example, helm (k8s package manager) is called kubernetes-helm on Nix.

Let’s bookmark [Nix package search page](https://search.nixos.org/packages). This is where you can find nix packages names. For example, if I want to install **htop**, I can search htop and see if it is available. After confirming the package name, we can put it under `environment.systemPackages` to install it.

> Please note, packages we install under `environment.systemPackages` are automatically available to all users. For user specific packages, we can put it under `users.users.<name>.packages` or you can also use [Home manager](https://github.com/nix-community/home-manager).

You may also notice the **NixOS options** tab on the [website](https://search.nixos.org/options). This is where you can find NixOS settings.

#### [](#Add-authorized-SSH-keys "Add authorized SSH keys")Add authorized SSH keys

To add SSH public key to my user, let’s search _users_ on the Options page and I can find `users.users.<name>.openssh.authorizedKeys.keys`. Let’s expand this option and we can see this is exactly what we are looking for and there is even an example on the doc.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/5.png)

Let’s open `/etc/nixos/configuration.nix` again and we can see there is a `users.users.<name>` section already. Let’s add our public key under `openssh.authorizedKeys.keys`.

```
...
  users.users.<name> = {
    extraGroups = [ "networkmanager" "wheel" "docker" ];
    packages = with pkgs; [];
  
    openssh.authorizedKeys.keys = [
      "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHAKe2Dv58MInaZ9oy7R5m6OIgVDsYPIxWbTDtYPH0m3 pve"
    ];
  };
```

#### [](#Disable-SSH-password-login "Disable SSH password login")Disable SSH password login

Now that we’ve added our keys for ssh log in, we probably want to disable SSH password log in. If we search _openssh_, we can find `services.openssh.passwordAuthentication` and this seems to be the right option. Let’s modify `/etc/nixos/configurations.nix` and add `services.openssh.passwordAuthentication = false;`.

If you still remember, we uncommented `services.openssh.enable = true;` in the [initial set up](#Initial-set-up). Because both options start with `services.openssh`, we can group them together as below

```
...
  services.openssh = {
    enable = true;
  
    passwordAuthentication = false;
  };
```

#### [](#Install-Docker "Install Docker")Install Docker

I am a big fan of docker and I need docker on all machines that I use. You might be thinking _docker_, _docker-compose_ are both software so we probably should search them from the packages site, right? That’s correct but we also want docker to be running as a service on our machine. Let’s try searching _docker_ on the [Options page](https://search.nixos.org/options) and we will find `virtualisation.docker.enable` there. Let’s try this option and see what it does.

We can add below at the bottom of `/etc/nixos/configurations.nix`.

```
...
  
  virtualisation.docker.enable = true;
```

#### [](#Disable-password-for-sudo "Disable password for sudo")Disable password for sudo

If you’ve used some VPS from cloud provider before, you might notice you don’t need to put in a password when you run commands with _sudo_. How do we achieve the same on Nix? I can’t seem to find anything on NixOS package/options page. Let’s try [NixOS forum](https://discourse.nixos.org/) and see if anyone asked this before.

Luckily someone asked the same question before and it is marked solved which means there is a solution there. Great!

![](https://raw.githubusercontent.com/lslz627/PicGo/master/6.png)

The solution is to add below to our config on `/etc/nix/configurations.nix`. (Please remember to change `<name>` to your user name)

```
...
  security.sudo.extraRules= [
    {
      users = [ "<name>" ];
      commands = [
        { command = "ALL" ;
          options= [ "NOPASSWD" ];
        }
      ];
    }
  ];
```

Now that we have all we need, let’s run `sudo nixos-rebuild switch` to apply the changes.

### [](#Verify-config "Verify config")Verify config

Once it is done, let’s verify our settings above.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/7.png)

We can see password log in is disable, ssh with certificate works but docker does not. The error message suggests we need to make sure our user has the privilege to use docker. This is mentioned on docker official doc [Docker Engine post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/). What we need is to add user to `docker` group.

It is very simple to add users to a group on nix. What we need is to add _docker_ in `users.users.<name>.extraGroups`.

```
...
  users.users.<name> = {
  
    extraGroups = [ "networkmanager" "wheel" "docker" ];
    packages = with pkgs; [];
    openssh.authorizedKeys.keys = [
      "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHAKe2Dv58MInaZ9oy7R5m6OIgVDsYPIxWbTDtYPH0m3 pve"
    ];
  };
```

Let’s rebuild and verify again.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/8.png)

As we can see everything works as expected now, docker error is gone and both **docker** and **docker compose** are installed.

### [](#Upgrade-packages "Upgrade packages")Upgrade packages

You might be wondering how you can update packages on NixOS. By default nixos use nixos-22.11 channel. We can find this info with `sudo nix-channel --list`.

```
[fomm@nixos:~]$ sudo nix-channel --list
nixos https://nixos.org/channels/nixos-22.11
```

To update, we simple need to run `sudo nix-channel --update` to update channel packages first and then we use `sudo nixos-rebuild switch` to update all packages.

That’s all I want to cover today, see you next time.