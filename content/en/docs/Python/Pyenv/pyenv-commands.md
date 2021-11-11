---
title: "Pyenv cheat sheet"
linktitle: "Pyenv Commands"
date: 2021-11-09T17:26:49+01:00
draft: true
description: This is a collection of useful commands I usually use with pyenv.
---

## Install latest version of a python release

One-liner to install latest version of a python release. 

```shell
pyenv install $(pyenv install --list | grep "^\s*3.9." | tail -1)
```

With that command we install the latest version of python 3.9 using pyenv.





## Create virtualenv

Create a virtualenv with the name of the directory and its `.python-version` file so the virtualenv is activated when I get into the directory:

```shell
pyenv virtualenv 3.9.2 $(pwd | rev | cut -d"/" -f1 | rev) && \
pyenv local $(pwd | rev | cut -d"/" -f1 | rev) && \
pip install --upgrade pip
```


## Delete virtualenv

To delete the virtualenv called with the same name as the directory where you're working on:

```shell
pyenv virtualenv-delete $(pwd | rev | cut -d"/" -f1 | rev)
```

So, if you're working on the path `/home/gmoe/projects/rocketman/` and your virtualenv is called `rocketman` with that command you'll delete it. 
This is useful when you always name virtualenvs this way as you'll always use the same command to create and delete virtualenvs and the shell history will save your time.
