# Getting Started with Home Manager

As I mentioned earlier, NixOS can only manage system-level configuration. To manage user-level configuration in the Home directory, we need to install Home Manager.

According to the official [Home Manager Manual](https://nix-community.github.io/home-manager/index.html), to install Home Manager as a module of NixOS, we first need to create `/etc/nixos/home.nix`. Here's an example of its contents:

```nix
{ config, pkgs, ... }:

{
  # TODO please change the username & home directory to your own
  home.username = "ryan";
  home.homeDirectory = "/home/ryan";

  # link the configuration file in current directory to the specified location in home directory
  # home.file.".config/i3/wallpaper.jpg".source = ./wallpaper.jpg;

  # link all files in `./scripts` to `~/.config/i3/scripts`
  # home.file.".config/i3/scripts" = {
  #   source = ./scripts;
  #   recursive = true;   # link recursively
  #   executable = true;  # make all files executable
  # };

  # encode the file content in nix configuration file directly
  # home.file.".xxx".text = ''
  #     xxx
  # '';

  # set cursor size and dpi for 4k monitor
  xresources.properties = {
    "Xcursor.size" = 16;
    "Xft.dpi" = 172;
  };

  # basic configuration of git, please change to your own
  programs.git = {
    enable = true;
    userName = "Ryan Yin";
    userEmail = "xiaoyin_c@qq.com";
  };

  # Packages that should be installed to the user profile.
  home.packages = with pkgs; [
    # here is some command line tools I use frequently
    # feel free to add your own or remove some of them

    neofetch
    nnn # terminal file manager

    # archives
    zip
    xz
    unzip
    p7zip

    # utils
    ripgrep # recursively searches directories for a regex pattern
    jq # A lightweight and flexible command-line JSON processor
    yq-go # yaml processer https://github.com/mikefarah/yq
    exa # A modern replacement for ‘ls’
    fzf # A command-line fuzzy finder

    # networking tools
    mtr # A network diagnostic tool
    iperf3
    dnsutils  # `dig` + `nslookup`
    ldns # replacement of `dig`, it provide the command `drill`
    aria2 # A lightweight multi-protocol & multi-source command-line download utility
    socat # replacement of openbsd-netcat
    nmap # A utility for network discovery and security auditing
    ipcalc  # it is a calculator for the IPv4/v6 addresses

    # misc
    cowsay
    file
    which
    tree
    gnused
    gnutar
    gawk
    zstd
    gnupg

    # nix related
    #
    # it provides the command `nom` works just like `nix`
    # with more details log output
    nix-output-monitor

    # productivity
    hugo # static site generator
    glow # markdown previewer in terminal

    btop  # replacement of htop/nmon
    iotop # io monitoring
    iftop # network monitoring

    # system call monitoring
    strace # system call monitoring
    ltrace # library call monitoring
    lsof # list open files

    # system tools
    sysstat
    lm_sensors # for `sensors` command
    ethtool
    pciutils # lspci
    usbutils # lsusb
  ];

  # starship - an customizable prompt for any shell
  programs.starship = {
    enable = true;
    # custom settings
    settings = {
      add_newline = false;
      aws.disabled = true;
      gcloud.disabled = true;
      line_break.disabled = true;
    };
  };

  # alacritty - a cross-platform, GPU-accelerated terminal emulator
  programs.alacritty = {
    enable = true;
    # custom settings
    settings = {
      env.TERM = "xterm-256color";
      font = {
        size = 12;
        draw_bold_text_with_bright_colors = true;
      };
      scrolling.multiplier = 5;
      selection.save_to_clipboard = true;
    };
  };

  programs.bash = {
    enable = true;
    enableCompletion = true;
    # TODO add your cusotm bashrc here
    bashrcExtra = ''
      export PATH="$PATH:$HOME/bin:$HOME/.local/bin:$HOME/go/bin"
    '';

    # set some aliases, feel free to add more or remove some
    shellAliases = {
      k = "kubectl";
      urldecode = "python3 -c 'import sys, urllib.parse as ul; print(ul.unquote_plus(sys.stdin.read()))'";
      urlencode = "python3 -c 'import sys, urllib.parse as ul; print(ul.quote_plus(sys.stdin.read()))'";
    };
  };

  # This value determines the home Manager release that your
  # configuration is compatible with. This helps avoid breakage
  # when a new home Manager release introduces backwards
  # incompatible changes.
  #
  # You can update home Manager without changing this value. See
  # the home Manager release notes for a list of state version
  # changes in each release.
  home.stateVersion = "23.05";

  # Let home Manager install and manage itself.
  programs.home-manager.enable = true;
}
```

After adding `/etc/nixos/home.nix`, you need to import this new configuration file in `/etc/nixos/flake.nix` to make use of it, use the following command to generate an example in the current folder for reference:

```shell
nix flake new example -t github:nix-community/home-manager#nixos
```

After adjusting the parameters, the content of `/etc/nixos/flake.nix` is as follows:

```nix
{
  description = "NixOS configuration";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
    home-manager.url = "github:nix-community/home-manager";
    home-manager.inputs.nixpkgs.follows = "nixpkgs";
  };

  outputs = inputs@{ nixpkgs, home-manager, ... }: {
    nixosConfigurations = {
      # TODO please change the hostname to your own
      nixos-test = nixpkgs.lib.nixosSystem {
        system = "x86_64-linux";
        modules = [
          ./configuration.nix

          # make home-manager as a module of nixos
          # so that home-manager configuration will be deployed automatically when executing `nixos-rebuild switch`
          home-manager.nixosModules.home-manager
          {
            home-manager.useGlobalPkgs = true;
            home-manager.useUserPackages = true;

            # TODO replace ryan with your own username
            home-manager.users.ryan = import ./home.nix;

            # Optionally, use home-manager.extraSpecialArgs to pass arguments to home.nix
          }
        ];
      };
    };
  };
}
```

Then run `sudo nixos-rebuild switch` to apply the configuration, and home-manager will be installed automatically.

After the installation, all user-level packages and configuration can be managed through `/etc/nixos/home.nix`. When running `sudo nixos-rebuild switch`, the configuration of home-manager will be applied automatically. (**It's not necessary to run `home-manager switch` manually**!)

To find the options we can use in `home.nix`, referring to the following documents:

- [Home Manager - Appendix A. Configuration Options](https://nix-community.github.io/home-manager/options.html): A list of all options, it is recommended to search for keywords in it.
  - [Home Manager Option Search](https://mipmip.github.io/home-manager-option-search/) is another option search tool with better UI.
- [home-manager](https://github.com/nix-community/home-manager): Some options are not listed in the official documentation, or the documentation is not clear enough, you can directly search and read the corresponding source code in this home-manager repo.

## Home Manager vs NixOS

When it comes to managing software packages and configurations, you often have the choice of using either NixOS modules (`configuration.nix`) or Home Manager (`home.nix`). This poses a dilemma: **What are the differences between putting packages or configuration in NixOS modules vs Home Manager modules, and how should you decide?**

First, let's understand the differences. Packages and configuration installed through NixOS modules are global to the entire system. Global configurations are typically stored in `/etc`, and globally installed packages are linked accordingly. Regardless of the user you switch to, you can access and use these packages and configurations.

On the other hand, everything installed through Home Manager is specific to the corresponding user. Once you switch to another user, those configurations and packages become unavailable.

Based on these characteristics, here is a general recommended approach:

- NixOS modules: Install core system components and other software packages/configurations required by all users.
  - For example, if you want a package to be accessible even when you switch to the root user, or if you want a configuration to take effect globally on the system, you should install it through a NixOS module.
- Home Manager: Use Home Manager to install all other configurations and software specific to individual users.

## How to use packages installed by Home Manager with privileged access?

The first thing that comes to mind is to switch to `root`, but then any packages installed by the current user through `home.nix` will be unavailable.
let's take `kubectl` as an example(it's pre-installed via `home.nix`):

```sh
# 1. kubectl is available
› kubectl | head
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/
......

# 2. switch user to `root`
› sudo su

# 3. kubectl is no longer available
> kubectl
Error: nu::shell::external_command

  × External command failed
   ╭─[entry #1:1:1]
 1 │ kubectl
   · ───┬───
   ·    ╰── executable was not found
   ╰────
  help: No such file or directory (os error 2)


/home/ryan/nix-config> exit
```

But it's possible to run those packages with privileged access without switching to `root`,  by using `sudo`, we temporarily grant the current user privileged access to system resources:

```sh
› sudo kubectl
kubectl controls the Kubernetes cluster manager.
...
```


