+++
date = "2017-06-23"
draft = false
title = "Deploying a python script"
slug = "deploying-a-python-script"
tags = ["python", "sysadmin", "devops", "trick", "scripting", "dev"]
author = "Miguel de la Cruz"
+++

Lately I've been writing a lot of python to script some common processes in a large infrastructure. Those scripts require some python libraries which are not part of the standard library nor installed by default in all systems. For example, I love how [docopt](http://docopt.org/) manages the arguments of a python script, or using the `YAML` format for my config files instead of `INI`.

During the development stage, I usually manage those dependencies using `pip` and `virtualenv`, but I would like my deployment to require only `python3` installed in the remote machine.

`pip` allows us to download the dependencies to a local directory running the following command:

```sh
pip3 install -r requirements.txt --upgrade --system --target python_deps
```

  - `upgrade`: we use this flag to download not only the required dependency, but all the transitive dependencies that this one requires.
  - `system`: by default, `pip` installs the packages following the *user scheme*, which is a specific directory structure for the interpreter to find the packages when running as a specific user. This flag modifies this behaviour, using a scheme that we point the interpreter to.
  - `target`: we use this flag to tell `pip` where we want the dependencies to be downloaded.

After installing the packages to a specific directory, we need to add that directory to our `PYTHONPATH` environment variable when running the interpreter:

```sh
PYTHONPATH=`pwd`/python_deps python3 main.py
```

To wrap all of this, we can create a simple script to run our python program, that searchs for the dependencies directory, installs them if they are not there and then runs the program with the `PYTHONPATH` variable set:

```sh
#!/usr/bin/env bash

# Set the PYTHONPATH environment variable. Using export will make the variable
# available for the rest of the script
export PYTHONPATH=`pwd`/python_deps

# If the python_deps directory doesn't exist, we download the dependencies there
if [[ ! -d "$PYTHONPATH" ]]; then
    pip3 install -r requirements.txt --upgrade --system --target $PYTHONPATH
fi

python3 src/main.py $@
```

We can now upload the script and run it from any system knowing that the only thing we need is the `python3` interpreter to be installed.

An improvement for this method would be to directly create a debian package, but that's for the next post.
