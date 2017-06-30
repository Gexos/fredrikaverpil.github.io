---
layout: post
title: An alternative to building PySide2 from source
tags: [python, pyside, pyqt, qt.py]
---

I've received questions lately on the issues that people are having while attempting to build PySide2 on Windows, macOS and Linux. Instead of building PySide2, there's actually a workaround which works just as well for some people...

<!--more-->

### The alternative to building PySide2 from source; PyQt5

So you want to use PySide2 but experience issues when building it from source. In case you're really just interested in using Qt indirectly, I might have a solution for you. Meaning you're really just interested in using PySide or PyQt to access Qt5 functionality.

PySide2 is under the Qt Company umbrella and it would make sense to want to use it (they should know best how to conform to Qt, right?). However, the development of PySide2 seems not yet to be on par with e.g. the PyQt equivalent; PyQt5. Especially not when it comes to just installing it and getting on with life. Hopefully, it will be soon. But in the interim, this blog post could perhaps be of help.

What's so special about PyQt5 is they've created a standalone and portable Python wheel for Python 3. So all it takes to get up running is to be on Python 3 and pip-install PyQt5 and you're done. No compiling of Qt required and no linking to locally installed Qt libraries; it all comes pre-compiled inside the wheel!

```bash
# from a Python 3 installation
pip install PyQt5
python -c "import PyQt5; print(PyQt5)"  # aaaaaah, how refreshing
```

### Manage virtual environments using `conda`

Instead of using e.g. [virtualenv](https://virtualenv.pypa.io/en/stable/) to manage virtual environments, I use conda. Conda can actually manage individual Python distributions and pre-compiled dependencies which is super powerful.

Then I also use the conda distribution of the PyQt5 Python binding, available on [conda-forge](https://conda-forge.github.io). I prefer using [Miniconda](https://conda.io/miniconda.html), as then it doesn't come pre-bundled with a bunch of packages (which the bigger [Anaconda](https://anaconda.org) distribution does). But regardless of whether you choose Miniconda or Anaconda, they both install the `conda` command, which is what we need.

So basically, to get set up with a conda environment loaded with PyQt5, you first need to install conda (I would recommend Miniconda). Here are a couple of examples on how to install that:

```bash
# Windows
choco install miniconda3  # assumes you have chocolatey installed

# macOS
brew cask install miniconda3  # assumes you have homebrew installed

# CentOS 7
curl -O https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
./Miniconda-latest-Linux-x86_64.sh
```

Then create your virtual environment, loaded with PyQt5:

```bash
conda config --add channels conda-forge  # enable conda-forge channel, which contains the PyQt5 distributions
conda create --mkdir -p ~/myCondaEnv python=3.6 pyqt  # change the path into where you want your Python env
```

Please note the above gives you the most recent PyQt which is 5.6.2-1 as of writing this. You can pin the PyQt version in the `conda create` command so that the next time you install the environment you get the same PyQt version. You can read up on conda to manage your environment by recording its dependencies down into a specifications file (similar to a requirements.txt file).

Give it a test run:

```bash
~/myCondaEnv/bin/python -c "import PyQt5; print(PyQt5)"
```

I'd like to point out that you don't have to deal with the activate/deactivate stuff in order to use your environment. You just need to specify the absolute full path to the python binary.


### Writing an application for Maya, Nuke and standalone

So, you want to write an application using PySide/PyQt which needs to run in multiple environment such as in Maya, Nuke and even standalone?

This is when you need to look into the [Qt.py](https://github.com/mottosso/Qt.py) project!

Set up two different conda environments:

- Maya and Nuke: Python 2.7 without any Qt bindings installed
- Standalone: Python 3.6 with PyQt5 installed

Then you just make sure your application uses the proper environment ([`site`](https://docs.python.org/3/library/site.html) module) depending on whether it is running inside of Maya, Nuke or as a standalone application. Instead of importing PySide, PySide2 or PyQt5 - you import Qt, which will cleverly work out which one of the bindings is available and use that.

I have [a couple of articles](https://fredrikaverpil.github.io/blog/tag/qtpy/) written up on the subject.


### Stuck on Python 2.7?

In case you are stuck on Python 2.7... Ok, first I just want to say you *really* need to look into moving away from Python 2.7 which is actually referred to as "legacy Python" these days. Even the [VFXPlatform](https://www.vfxplatform.com) is drafting Python 3 for CY2019.

Anyways, if you really must, you can actually quite quickly get up and running with PySide and/or PyQt4 on Python 2.7 using conda too:

```bash
conda create --mkdir -p ~/myCondaEnv python=2.7 PySide=1.2.4 pyqt=4.11.4
```

Ever since Python 3.5, adoption has been soaring and more and more packages are moving away from legacy Python and starts supporting Python 3 only. You've been warned. This is why this paragraph didn't make it until the end of this blog post. :)
