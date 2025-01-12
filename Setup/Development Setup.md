# Python

## Python and pip

Check that python and pip are installed and their version.

https://developer.fedoraproject.org/tech/languages/python/python-installation.html

I decided to install `pip` using the Fedora repos, but there are alternative ways as described below:

https://pip.pypa.io/en/stable/installation/

## Managing Multiple Python Versions With pyenv

### Tutorial
https://realpython.com/intro-to-pyenv/#installing-pyenv
https://github.com/pyenv/pyenv#how-it-works

### pyenv installer

- Prerequisites
https://github.com/pyenv/pyenv/wiki#suggested-build-environment

- The installer
https://github.com/pyenv/pyenv-installer

```bash
dnf install make gcc patch zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel xz-devel libuuid-devel gdbm-devel libnsl2-devel

curl https://pyenv.run | bash

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
```


- Set up your shell environment for Pyenv (zsh)
```zsh
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

Define environment variable `PYENV_ROOT` to point to the path where Pyenv will store its data. `$HOME/.pyenv` is the default. If you installed Pyenv via Git checkout, we recommend to set it to the same location as where you cloned it.

### Usage

```
ls ~/.pyenv/versions/
```

## Install **pipenv:**

https://github.com/pypa/pipenv#installation

https://developer.fedoraproject.org/tech/languages/python/pipenv.html

https://pipenv-fork.readthedocs.io/en/latest/install.html#installing-pipenv

I choose install it using dnf repos

`$ sudo dnf install pipenv`

- Tutorial
https://www.rootstrap.com/blog/how-to-manage-your-python-projects-with-pipenv-pyenv

https://pipenv.pypa.io/en/latest/basics/


Download [.gitignore](https://djangowaves.com/tips-tricks/gitignore-for-a-django-project/) for Django projects

# Code

https://code.visualstudio.com/docs/setup/linux

Customize

- Sync settings and keybinds
- Install Python extension

## Install Fira Code Font

https://github.com/tonsky/FiraCode/wiki

`$ dnf install fira-code-fonts`

https://github.com/tonsky/FiraCode/wiki/VS-Code-Instructions

# JavaScript

## nvm

First install [**nvm**](https://github.com/nvm-sh/nvm)

`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash`

To verify that nvm has been installed, do:

`command -v nvm`

**Note:** On Linux, after running the install script, if you get `nvm: command not found` or see no feedback from your terminal after you type `command -v nvm`, simply close your current terminal, open a new terminal, and try verifying again.

## node & npm

https://docs.npmjs.com/downloading-and-installing-node-js-and-npm

Install **node LTS**

`nvm install --lts`

Update [**npm**](https://docs.npmjs.com/try-the-latest-stable-version-of-npm)

`npm install npm@lts -g`

And update using:
`nvm install-latest-npm`

Checkout these guides

[What version of npm to use with node LTS](https://stackoverflow.com/questions/66673012/should-i-update-my-npm-version-or-use-the-one-node-js-provides/66674123#66674123)

https://www.sitepoint.com/npm-guide/

## pnpm

https://pnpm.io/installation

### Fix for Angular

https://stackoverflow.com/questions/52948906/how-do-i-use-pnpm-in-my-angular-project-to-manage-packages

https://github.com/vaadin/flow/issues/9834

```
ng new test-pnpm --skip-install
pnpm install --shamefully-hoist
```

# Dart

https://dart.dev/get-dart

https://docs.flutter.dev/get-started/install/linux#install-flutter-using-snapd
https://i.stack.imgur.com/JAlKd.png

# Shortcuts

Setup alias in .bashrc

```
mkdir ~/.bashrc.d
chmod 700 ~/.bashrc.d
chmod +x ~/.bashrc.d/alias.bashrc
```

To set an alias in the file:

```
alias pyma="python manage.py"
alias glog="git log --oneline --graph --decorate --all"
```

Add this to your actual .bashrc (on top)

```
for file in ~/.bashrc.d/*.bashrc;
do
source “$file”
done
```

# Git

Setup mystuff

```
git config --global user.name "giraldiego"
git config --global user.email giraldiego@gmail.com

git config --global core.editor "vim --nofork"
```

Setup some aliases

```bash
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status

$ git config --global alias.last 'log -1 HEAD'
```


# .NET

https://docs.microsoft.com/en-us/dotnet/core/install/linux-fedora

`$ sudo dnf install dotnet-sdk-5.0`

# SQL
https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat?view=sql-server-linux-ver15
https://computingforgeeks.com/how-to-install-microsoft-sql-2019-on-centos-7-fedora/

Start the server:
`sudo systemctl start mssql-server`

Verify status:
`systemctl status mssql-server`

Connect to the server:
`sqlcmd -S localhost -U SA`

# Java

https://adoptium.net/installation/linux/

https://computingforgeeks.com/install-oracle-java-openjdk-fedora-linux/

`sudo alternatives --config java`

## Jenkins

https://www.youtube.com/watch?v=uSi17DYLjG8

# JetBrains


# Firefox Developer
https://www.addictivetips.com/ubuntu-linux-tips/install-firefox-developer-edition-on-linux/

# MiniConda

https://docs.conda.io/en/latest/miniconda.html

https://stackoverflow.com/questions/57640272/how-can-i-install-anaconda-aside-an-existing-pyenv-installation-on-osx

```bash
# virtual environments from pyenv
pyenv install 3.6.9
pyenv virtualenv 3.6.9 new-env
pyenv activate new-env
pyenv deactive
# You can also use `pyenv local`


# virtual environments from conda
conda create -n new-env python=3.6
conda env list
conda activate new-env
conda deactivate
```

### Extended Readign

- [Getting started with conda](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html)
- [Using Pip in a Conda Environment](https://www.anaconda.com/using-pip-in-a-conda-environment/), very important
- [How do I prevent Conda from activating the base environment by default?](https://stackoverflow.com/a/57974390/5101148)

This is for clean enviroment without `pyenv`

```bash
conda config --add channels conda-forge 
conda config --set channel_priority strict
conda install python=3.11
```