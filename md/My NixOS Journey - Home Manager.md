> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [tech.aufomm.com](https://tech.aufomm.com/my-nixos-journey-home-manager/#Install-in-standalone-mode)

> In the last post I talked about installing NixOS and how you can find the packages and settings to be......

In the [last post](https://tech.aufomm.com/my-nixos-journey-intro-and-installation/) I talked about installing NixOS and how you can find the packages and settings to be used on `/etc/nixos/configurations.nix`. If you care about setting up user profile declaratively, you will find [home manager](https://github.com/nix-community/home-manager) very quickly. Home manager can be used to not only install user specific packages but also manage application configurations. Some of them are not configurable via nixos options. ([dconf.settings](https://rycee.gitlab.io/home-manager/options.html#opt-dconf.settings) is a good example).

> You can find home manager official doc [here](https://rycee.gitlab.io/home-manager/).

In today’s post, I will show you how to start using home manager. Let’s get started.

[](#Installation "Installation")Installation
--------------------------------------------

On NixOS you have two choices to install Home manager, either as NixOS module or in standalone mode. From my understanding, here are the pros/cons to use home manager as nixos module.

Pros:

*   You can easily backup, rollback and set up the entire system with a single `nixos` command.

Cons:

*   Making changes to home manager settings results in a whole system level rebuild, users end up having too many generations.

### [](#Install-as-Nix-Module "Install as Nix Module")Install as Nix Module

If you want to install home manager as standalone mode, you can go [here](#Install-in-standalone-mode).

> I will follow what is on the [official documentation](https://rycee.gitlab.io/home-manager) first and then show you how to optimize your config.

#### [](#Add-home-manager-channel "Add home manager channel")Add home manager channel

Let’s run below command to add `22.11` channel first.

```
sudo nix-channel --add https://github.com/nix-community/home-manager/archive/release-22.11.tar.gz home-manager
sudo nix-channel --update
```

#### [](#Add-hm-to-config "Add hm to config")Add hm to config

Next we need to add home manager to `/etc/nixos/configuration.nix`. Let’s open this file in text editor and

*   Add `<home-manager/nixos>` under _**imports**_
*   Set `users.users.<name>.isNormalUser = true;` (in my case this is already **true**)
*   Add `home-manager.users.<name> = { pkgs, ... }: {` related config as below

```
...
  home-manager.users.<name> = { pkgs, ... }: {
    home.packages = [ pkgs.atool pkgs.httpie ];
    home.stateVersion = "22.11";
  };
```

Let’s rebuild and the command should work this time. We can use `http` to verify _httpie_ is installed.

```
[fomm@nixos:~]$ http --version
3.2.1
```

Above example is pretty self-explanatory. It use home-manager for `user.<name>` and installs [atool](https://linux.die.net/man/1/atool) and [httpie](https://httpie.io/) for this user.

If we take a look at where _http_ is installed, we should see it is installed under my user’s home directory.

```
[fomm@nixos:~]$ which http
/home/fomm/.nix-profile/bin/http
```

As home manager configs are for a specific user, let’s split its configs from the main system configuration.

#### [](#Separate-configs "Separate configs")Separate configs

On the official documentation it mentioned there should be a `home.nix` but we don’t have this file when we install home manager as NixOS module. Let’s manually create `home.nix` with below config in `/etc/nixos/` folder.

```
{ config, pkgs, ... }:

{
  home.username = "<username>";
  home.homeDirectory = "/home/<username>";
  home.stateVersion = "22.11";
}
```

Then we need to change `home-manager` part on `/etc/nixos/configuration.nix` to below. I also enable [useGlobalPkgs](https://rycee.gitlab.io/home-manager/nixos-options.html#nixos-opt-home-manager.useGlobalPkgs) and [useUserPackages](https://rycee.gitlab.io/home-manager/nixos-options.html#nixos-opt-home-manager.useUserPackages) here. Please have a read if you want to know what these two variables do.

```
...
  home-manager = {
    useGlobalPkgs = true;
    useUserPackages = true;
    users.<name> = import ./home.nix;
  };
```

Let’s run `sudo nixos-rebuild switch` to rebuild. Because we removed `home.packages = [ pkgs.atool pkgs.httpie ];` with above changes, _atool_ and _httpie_ will also be removed. We can verify this as below.

```
[fomm@nixos:~]$ which http
which: no http in (/run/wrappers/bin:/home/fomm/.nix-profile/bin:/etc/profiles/per-user/fomm/bin:/nix/var/nix/profiles/default/bin:/run/current-system/sw/bin)
```

Let’s move on to [the next section](#Use-home-manager) to learn how to use home manager.

### [](#Install-in-standalone-mode "Install in standalone mode")Install in standalone mode

Running home manager in standalone mode means you will use `home-manager` command to mange home manager related configs.

#### [](#Add-home-manager-channel-1 "Add home manager channel")Add home manager channel

Let’s run below command to add `22.11` channel first.

```
sudo nix-channel --add https://github.com/nix-community/home-manager/archive/release-22.11.tar.gz home-manager
sudo nix-channel --update
```

#### [](#Install-Home-manager "Install Home manager")Install Home manager

To install, we just need to run below command.

```
nix-shell '<home-manager>' -A install
```

[](#Use-home-manager "Use home manager")Use home manager
--------------------------------------------------------

Config folders (where we can find `home.nix`):

*   NixOS module: `/etc/nixos`
*   Standalone: `~/.config/nixpkgs/`

Update nix channel:

*   NixOS: `sudo nix-channel --update`
*   Home manager: `nix-channel --update`

Apply home manager changes command:

*   NixOS module: `sudo nixos-rebuild switch`
*   Standalone: `home-manager switch`

### [](#Install-Nix-Packages "Install Nix Packages")Install Nix Packages

We can install nix package by putting the package name under `home.packages` on `home.nix`. Let’s open this file and add below, this will install _htop_.

```
...
  home.packages = with pkgs; [
    htop
  ];
```

Once it we rebuild, we should have _htop_.

```
[fomm@nixos:~]$ htop --version
htop 3.2.1
```

### [](#Home-manager-configs "Home manager configs")Home manager configs

Now that we know how to install packages, let’s explore how to use Home manager to manage application configurations .

Let’s add below to `home.nix` and then run [switch command](#Use-home-manager) to install.

```
...
  programs.zsh = {
    enable = true;
    enableCompletion = true;
    enableAutosuggestions = true;
    enableSyntaxHighlighting = true;
    oh-my-zsh = {
      enable = true;
      plugins = [ "docker-compose" "docker" ];
      theme = "dst";
    };
    initExtra = ''
      bindkey '^f' autosuggest-accept
    '';
  };

  programs.fzf = {
    enable = true;
    enableZshIntegration = true;
  };
```

This is pretty self-explanatory.

1.  Install [zsh](https://www.zsh.org/), [fzf](https://github.com/junegunn/fzf).
2.  Enable [syntax highlighting](https://github.com/zsh-users/zsh-syntax-highlighting), [completion](https://github.com/zsh-users/zsh-completions) and [auto suggestion](https://github.com/zsh-users/zsh-autosuggestions).
3.  Install [oh-my-zsh](https://ohmyz.sh/), use _dst_ theme and _docker-compose_, _docker_ plugin.
4.  `Ctrl + f` to accept autosuggestion.

To find more home manager configs, the first place we can check is the [manual page](https://rycee.gitlab.io/home-manager/options.html). We may also find some useful information from [NixOS Wiki](https://nixos.wiki/wiki/Main_Page).

Now that zsh is installed, we can run `zsh` and we should see all the config above are working. We might also want to set zsh as our default shell. From [NixOS Wiki page](https://nixos.wiki/wiki/Command_Shell), we know w need to add below to `/etc/nixos/configuration.nix`.

```
users.users.<name>.shell = pkgs.zsh;
environment.shells = with pkgs; [ zsh ];
```

Let’s add these config and run `sudo nixos-rebuild switch`. The next time we log in to our terminal, we should see zsh is used.

### [](#Split-HM-configs "Split HM configs")Split HM configs

As you can imagine, the more home manager configs we need to use, the longer `home.nix` will be which makes managing this file very difficult in the future. What we can do is to split `home.nix` and put programs configs to different files and then import it back to `home.nix`.

Let’s create `/apps` folder in our [config folder](#Use-home-manager). Then we can create a `zsh.nix` file and put below content on it.

```
{
  programs.zsh = {
    enable = true;
    enableCompletion = true;
    enableAutosuggestions = true;
    enableSyntaxHighlighting = true;
    oh-my-zsh = {
      enable = true;
      plugins = [ "docker-compose" "docker" ];
      theme = "dst";
    };
    initExtra = ''
      bindkey '^f' autosuggest-accept
    '';
  };

  programs.fzf = {
    enable = true;
    enableZshIntegration = true;
  };
}
```

Next let’s go back to `home.nix` file and remove the `programs.zsh` section and import `zsh.nix` back.

```
{ config, pkgs, ... }:

{
  imports = [
    ./apps/zsh.nix
  ];
```

Let’s rebuild again and we should see zsh is still running fine.

Let’s do one more thing by removing micro editor from system package and use home manager to manage it.

1.  Let’s comment out **micro** under `environment.systemPackages` on `/etc/nixos/configuration.nix`.
    
2.  Then we can create a _micro.nix_ file under home manager `/apps` folder with below content.
    
    ```
    {
      programs.micro = {
        enable = true;
        settings = {
          colorscheme = "material-tc";
          mkparents = true;
          softwrap = true;
          tabmovement = true;
          tabsize = 2;
          tabstospaces = true;
          autosu = true;
        };
      };
    }
    ```
    
3.  Then we can can add `./apps/micro.nix` to `home.nix` and rebuild. We will get below error message. This is because we have `~/.config/micro/settings.json` from previous installation, home manager wantss you to confirm if you need to backup.
    
    ```
    Existing file '/home/fomm/.config/micro/settings.json' is in the way of '/nix/store/78gyysv4v3wdkds5l50nv2vbiqj2avnk-home-manager-files/.config/micro/settings.json'
    Please move the above files and try again or use 'home-manager switch -b backup' to back up existing files automatically.
    ```
    
4.  Since I will use home manager to manage configs for micro, I delete the file and rebuild again. Rebuild should be successful this time. If we use micro again, we can see the indentation, colour theme are working exactly as our configs above.
    

That’s all I want to share with you today. I hope you can start building your nix configuration using home manager.

See you next time.