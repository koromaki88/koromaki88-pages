---
title: "Miniconda, Jupyter Notebook & VSCode setup on Arch Linux"
date: 2025-06-16T23:30:00+07:00
draft: false
tags: ["Arch", "Linux", "Miniconda", "Jupyter", "VSCode", "Python"]
categories: ["Guides"]
summary: "Personal setup notes for studying machine learning on Arch Linux."
---
I decided to dedicate a blog post for this, mostly because my preferred setup process on Arch was a lot more confusing that it needed to be. This is just what I find to give me the most control and flexibility, but feel free to argue against it. Otherwise, if you found this helpful, then I'm glad to have helped :D

# Miniconda
I picked Miniconda over Anaconda Distribution, because I want a minimal base environment and manually install what I need later on - mostly because I'm working with a 500GB SSD and am very nitpicky about disk space. The installation process should be the same regardless.

On Arch, Miniconda can be installed either through the [AUR](https://aur.archlinux.org/packages/miniconda3) or [official installation script](https://anaconda.com/docs/getting-started/miniconda/install#linux-terminal-installer). They're effectively the same where the AUR package installs Miniconda at root's `/opt/` by default and can be changed by just modifying the PKGBUILD by yourself. With multiple environments, Miniconda can take quite plenty of disk space in my root partition, so I use the official script and install Miniconda in home instead.

- Download the installation script
```sh
# This link should be different / can be changed for ARM based systems
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh 
```

- Verify its integrity by comparing the hash value to [here](https://repo.anaconda.com/miniconda/)
```sh
sha256sum ~/Downloads/Miniconda3-latest-Linux-x86_64.sh
```

- Follow the installation process until this point. Ensure that "no" is picked - we don't want to activate conda on every new shell.
```
Do you wish to update your shell profile to automatically initialize conda?
This will activate conda on startup and change the command prompt when activated.
If you'd prefer that conda's base environment not be activated on startup,
   run the following command when conda is activated:

conda config --set auto_activate_base false

You can undo this by running `conda init --reverse $SHELL`? [yes|no]
[no] >>> no
```

Miniconda is ready for use. I treat it similarly to Python's virtual environment by only activating the environment whenever I need to use it.
```sh
source miniconda3/bin/activate <env> # optionally specify an environment name
```

The default environment is labeled `base`, which I use as a template and not install anything extra on it. The commands creat a new environment with some packages (change/add whatever you like) and switch to that environment:
```sh
conda create -n myenv python numpy jupyter pandas
conda activate myenv
```

# Jupyter Notebook & VSCode
This section was where I had a bit of trouble due to Arch's open source VSCode quirks that I don't fully understand. This will be updated if the issue is fixed or if I found a proper fix.

```sh
code --enable-proposed-api ms-toolsai.jupyter
```
