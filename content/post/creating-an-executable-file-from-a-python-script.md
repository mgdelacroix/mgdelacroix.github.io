+++
date = "2017-07-23"
draft = false
title = "Creating an executable file from a python script"
slug = "creating-an-executable-file-from-a-python-script"
tags = ["python", "sysadmin", "devops", "trick", "scripting", "dev"]
author = "Miguel de la Cruz"
+++

Recently I wrote about how
to
[deploy a python script without virtualenv](/post/deploying-a-python-script-without-virtualenv). This
method takes advantage of the idea of installing the script's
dependencies in a local directory using `pip` and wraps them and our
code with a simple `shell` script. The result was good, but you still
needed to deploy a directory with your source code, another one with
the dependencies and the wrapper script with those paths fixed, which
will be the one to run. Good, but not ideal.

Right after publishing that post, a colleague showed
me
[the zipapp module](https://docs.python.org/3.6/library/zipapp.html),
recently added in Python 3.5, which allows you to easily create an
executable zipfile from your source code. The feature of running
python code from a zipfile was added long time ago (refer
to
[this article](https://blogs.gnome.org/jamesh/2012/05/21/python-zip-files/) to
find out more about the specific PEPs), but I wasn't aware of its
existence. As soon as I read through the module's documentation, I
decided to try and mix both ideas, bundling in a zipfile both the
source code of a script and its dependencies.

All the literature I found achieved this modifying the `sys.path` of
the script, but my goal was to do it without explicitly touching the
dependencies, just playing with the packaging process. The only
requirement for the `zipapp` module to work is to have a `__main__.py`
file on your sources folder with a valid entrypoint, which is the one
that python will run when you execute the file.

As running Python code from a zipfile can be done since Python 2.6,
Python 3 (in fact, 3.5 or greater) is only required to be used in the
line that calls the `zipapp` module. Through this article I'm going to
call `python3` in the code examples for consistency, but it is only
needed at that precise step.

So imagine we have a script with the following layout:

```sh
repo
├── myscript
│   ├── __main__.py
│   └── myscript.py
└── requirements.txt
```

If the main function of the script is called `entrypoint`, for
example, the contents of the `__main__.py` would be:

```python
from myscript import entrypoint

entrypoint()
```

For the script to be able to find the dependencies, we are going to
put them right in the sources directory. This way, they will be in the
default `PYTHONPATH`, and we will not need to touch the code's
`sys.path`.

To keep the original source code clean, we will create a `dist`
temporal folder, that we can remove after creating the executable
file. In this folder we will copy our source code, and we will install
our dependencies locally:

```sh
mkdir dist
cp -r myscript/* dist
pip3 install -r requirements.txt --upgrade --system --target dist
```

At this point, we should be able to run our program directly using the
following line, without needing to touch the `PYTHONPATH`:

```sh
python3 dist
```

Finally, to create the executable zipfile, just run the following:

```sh
python3 -m zipapp -p '/usr/bin/env python3' -o myscript.bin dist
```

- The `-m` tells Python to run the `zipapp` module.
- The `-p` specifies
  the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) of the
  header of the generated executable zipfile. If your script uses
  Python 2 instead of Python 3, just modify that line.
- The `-o` specifies the name of the output file. The extension is
  totally optional. I prefer to use no extension at all.
- The last parameter is the directory of the code you want to bundle.

The output of this command should leave you with a `myscript.bin`
file, perfectly portable and whose only requirement is the python
interpreter that you specified in the `-p` argument on its creation.

You can even open the file with any text editor to see how it is
basically a text file with our shebang, source code files and
dependencies, all concatenated.
