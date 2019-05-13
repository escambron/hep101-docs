ROOT Quick Start
================

`ROOT <https://root.cern/>`_ is the main piece of software used by
high energy experimentalists. It's been this way for the last two
decades. You'll find use of ROOT in the majority of ATLAS
software. Luckily, ROOT can be used independent of ATLAS software on
any Linux or macOS machine. If you do not have to use ATLAS software
(that is, you have datasets that are in a pure ROOT format without any
ties to the ATLAS data format), just running ROOT on a laptop is the
fastest way to develop code for analyzing ROOT data (commonly called
"ROOT ntuples").

Installing Anaconda
-------------------

The fastest way to get up and running with ROOT is to install it via
the `conda package manager <https://docs.conda.io/en/latest/>`_. To
use the ``conda`` package manager we need to install the `Anaconda
Distribution <https://www.anaconda.com/distribution/>`_.

ROOT can be installed in a number of ways. One of the benefits of
installing it via Anaconda is we get a pre-built binary (we do not
have to compile ROOT from scratch ourselves), and our ROOT
installation will be out-of-the-box compatible with a Python
distribution (we can use the native C++ ROOT libraries and interpreter
or we can use the python bindings). And "for free" we get the
incredibly useful scientific python stack provided by Anaconda. The
main purpose of Anaconda is to provide the scientific python
ecosystem.

If you're on a Linux machine download the latest installer script via
``curl``:

.. code-block::

   $ curl -O https://repo.anaconda.com/archive/Anaconda3-2019.03-Linux-x86_64.sh

If you're on a Mac we'll grab the installer for macOS:

.. code-block::

   $ curl -O https://repo.anaconda.com/archive/Anaconda3-2019.03-MacOSX-x86_64.sh

Once we have the installer script let's prepare an isolated
environment where we can install the Anaconda distribution. A clean
place can be in ``~/Software/Python``; let's make the directory and
run the installer script:

.. code-block::

   $ mkdir -p ~/Software/Python
   $ bash Anaconda3-2019.03-*.sh

You'll be prompted to agree with the their license agreement. Press
enter, then you can press ``q`` then type ``yes`` and press enter
again to agree. Now it asks where you'd like to install Anaconda; by
default it wants to install it to ``~/anaconda3`` (in your home
directory), but we'll give it our already created (better organized)
home. Let's tell it ``~/Software/Python3/anaconda3``; at the prompt
we'll give the path

.. code-block::

   [...] >>> ~/Software/Python/anaconda3

and press enter. After the installation process is done it will ask if
you'd like to run ``conda init`` to modify your shell init script. The
default option should be ``no``, this is the cleanest thing to do (to
prevent the ``conda`` installation from clashing with other
applications).

Let's create an alias in your shell's init script to make it wasy to
spin on ``conda``. Add the following line to your ``~/.bashrc`` file
(this assumes you use ``bash`` as your default shell, which is the
default shell for almost all Linux distributions and macOS, if you're
not using ``bash``, I expect you know what you're doing):

.. code-block:: bash

   alias setupConda='eval "$(~/Software/Python/anaconda3/bin/conda shell.bash hook)"'

Now when you open a fresh shell (like a new terminal), you can enter
``setupConda`` and you should see ``(base)`` prepend the prompt
now. This means you are in the ``base`` (default) Anaconda
environment. In the next section we'll talk about creating a new
environment for a particular purpose.

Creating a Conda Environment
----------------------------

Let's spin up a fresh shell (terminal) and use our new alias, and look
at the ``conda`` help menu.

.. code-block::

   $ setupConda
   (base) $ conda --help

You'll see a whole lot of options that steer the ``conda`` package
manager. We're going to use the ``create`` option to create a new
environment. You can read all about ``conda`` environments `here
<https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html>`_. We're
going to focus on setting up a new one with ROOT.

.. note::

   The installer can get a bit stale (they update it a few times a
   year). To make sure everything is up to date we can run ``conda
   update -n base --all`` to update the base installation.

Here we'll mention the ``conda-forge`` *channel*. The ``conda``
package manager has the ability to access different channels for
installing various packages. The default channel is run by the
Anaconda Inc. company and it has many of the most popular scientific
python packages. The ``conda-forge`` channel is a community organized
and supported set of packages, and ROOT is one of them. You can read
more about it at `conda-forge.org <https://conda-forge.org/>`_.

.. code-block::

   (base) $ conda create -n root6 python=3.6 root -c conda-forge

Proceed by entering ``y``. Let's disect this a bit:

- ``-n root6`` gives the name ``root6`` to the new envionrment we're
  creating.
- ``python=3.6`` is telling ``conda`` that we want the environment to
  use version 3.6 of Python.
- ``root`` is saying while creating the environment also install the
  ``root`` package.
- ``-c conda-forge`` is telling the ``conda`` package manager to use
  the ``conda-forge`` channel.

Now it'll take some time for all of the necessary dependencies to be
installed. Once it's done it'll tell you how to activate the
environment.

.. code-block::

   (base) $ conda activate root6

Now if we want to spin up our new environment from a fresh shell, we just have to enter

.. code-block::

   $ setupConda
   (base) $ conda activate root6

And we're set to go. Now you have a full-featured ROOT installation at
our fingertips that you can play with:

.. code-block::

   (root6) $ root
   root [0] int x = 5;
   root [1] cout << x << endl;
   5
   root [2] .q
   (root6) $ exit

At this point we point you to the `ROOT documentation
<https://root.cern/documentation>`_ for more.

Extending your Conda Environment
--------------------------------

Conda environments are good places to isolate installed software for
specific purposes. We've obviously set one up for when we want to use
ROOT. If you find yourself needing another piece of software to tackle
a problem, we can use ``conda`` or ``pip`` to install more
packages. Since we installed ROOT from the ``conda-forge`` channel,
it's probably a good idea to start there. With your ``root6``
environment activated we can install more packages like so:

.. code-block::

   (root6) $ conda install <package> -c conda-forge

We can search for available packages on the command line like so:

.. code-block::

   (root6) $ conda search <package> -c conda-forge


Or you can search `<https://anaconda.org/>`_.

A very useful package for analyzing data stored in ROOT files with the
scientific python stack is `uproot
<https://github.com/scikit-hep/uproot/>`_. We can install it like so:

.. code-block::

   (root6) $ conda install uproot -c conda-forge

The ``uproot`` python package is great because it allows us to turn
ntuples in ROOT files into ``numpy`` arrays in one line, check out
their documentation.
