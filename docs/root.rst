ROOT files on your Laptop
=========================

`ROOT <https://root.cern/>`_ has been used by high-energy experimentalists
for most of the last two decades. You'll find use of ROOT in the majority of ATLAS
software. ROOT can be used independently of ATLAS software on any Linux or macOS machine. 
Many students prefer using  `uproot <https://github.com/scikit-hep/uproot/>`_ to read and manipulate ROOT files instead, since they can use python and array-based approaches to the data.

Installing uproot
-----------------

The ``uproot`` python package is great because it allows us to turn
ntuples in ROOT files into ``numpy`` or ``awkward`` arrays in one line. 
Getting is as easy as installing any other python package: just type 
``python -m pip install uproot.`` You can then ``import uproot`` in 
Jupyter notebooks or python code.


Installing Anaconda
-------------------

If you'd like to use CERN ROOT instead, the easiest way is to install it via
the `conda package manager <https://docs.conda.io/en/latest/>`_. To
use the ``conda`` package manager we need to install the `Anaconda
Distribution <https://www.anaconda.com/distribution/>`_. The
instructions below will guide you through installing the ``conda``
package manager with the Miniconda distribution. This is a lighter
weight installation (the full Anaconda distribution can always be
installed later).

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

   $ curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

If you're on a Mac we'll grab the installer for macOS:

.. code-block::

   $ curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh

Once we have the installer script let's prepare an isolated
environment where we can install the Anaconda distribution. A clean
place can be in ``~/Software/Python``; let's make the directory and
run the installer script:

.. code-block::

   $ mkdir -p ~/Software/Python
   $ bash Miniconda3-latest*.sh -b -p ~/Software/Python/anaconda3

The flag ``-b`` tells t he installer script to run in `batch` mode
(the default is interactive, where it will ask you to accept the
license agreement and ask for an installation location). The ``-p``
option is how we define the installation `prefix`, which is the
directory where we want to install Miniconda.

.. note::

   You don't have to use ``-b`` and ``-p``. If you don't, you'll be
   prompted to agree with the their license agreement. Press enter,
   then you can press ``q`` then type ``yes`` and press enter again to
   agree. Now it asks where you'd like to install Anaconda; by default
   it wants to install it to ``~/anaconda3`` (in your home directory),
   but we'll give it our already created (better organized) directory
   location. Let's tell it ``~/Software/Python3/anaconda3``; at the
   prompt give that path and press enter. After the installation
   process is done it will ask if you'd like to run ``conda init`` to
   modify your shell init script. The default option should be ``no``,
   this is the cleanest thing to do (to prevent the ``conda``
   installation from clashing with other applications).

Let's create an alias in your shell's init script to make it wasy to
spin on ``conda``. Add the following line to your ``~/.bashrc`` file
or ``~/.zshrc`` file (this assumes you use ``bash`` or ``zsh`` as your
default shell, which is the default shell for almost all Linux
distributions, if you're not using ``bash`` or ``zsh``, I expect you
know what you're doing):

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

   (base) $ conda create -n root6 python=3.7 root -c conda-forge

Proceed by entering ``y``. Let's disect this a bit:

- ``-n root6`` gives the name ``root6`` to the new envionrment we're
  creating.
- ``python=3.7`` is telling ``conda`` that we want the environment to
  use version 3.7 of Python. Checkout the ``conda`` documentation to
  see which versions of Python you can choose from (3.7 quite recent
  and all major Python packages support it).
- ``root`` is saying while creating the environment also install the
  ``root`` package.
- ``-c conda-forge`` is telling the ``conda`` package manager to use
  the ``conda-forge`` channel.

Now it'll take some time for all of the necessary dependencies to be
installed. Once it's done it'll tell you how to activate the
environment.

.. code-block::

   (base) $ conda activate root6

Now if you want to spin up our new environment from a fresh shell, you just have to enter

.. code-block::

   $ setupConda
   (base) $ conda activate root6

And you're set to go. Now you have a full-featured ROOT installation at
our fingertips that you can play with:

.. code-block::

   (root6) $ root
   root [0] int x = 5;
   root [1] cout << x << endl;
   5
   root [2] cout << x * 3 << endl;
   15
   root [3] .q
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

If you want both ``conda`` and `uproot
<https://github.com/scikit-hep/uproot/>`_ you can install it like so:

.. code-block::

   (root6) $ conda install uproot -c conda-forge


