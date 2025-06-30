Cookbook of Common Tasks: ROOT and columnar analysis
====================================================


Here we'll provide a few code examples for some common tasks. 
It's also very useful to look at `scikithep <https://scikit-hep.org>`_ since
the documentation for packages like ``uproot`` (for interacting with data),
`hist <https://hist.readthedocs.io/>`_ (for visualizing data), `vector <https://vector.readthedocs.io/>`_ (for manipulating relativistic quantities) and `awkward <https://awkward-array.org/>`_ (for interacting with complex data structures) are littered with "common task" examples.

.. note::

   This cookbook was authored in 2019-2020 by Duke graduate Doug
   Davis. It's being updated, but currently doesn't make much use of
   the exceptionally useful ``awkward`` and ``vector`` libraries
   (these were very new at the time) so refer to the documentation of
   those packages to learn how to streamline the ``uproot`` examples
   below.

The file referenced in this "cookbook" can be downloaded with ``wget`` if
you'd like to actually execute some of this code and see it work:

.. code-block::

   wget http://phy.duke.edu/~ddavis/public/events.root

The examples on this page use ROOT and uproot. They both have very
thorough documentation. If you want to dive in with their docs, here
are some resources:

- `The ROOT reference documentation
  <https://root.cern/doc/v620/index.html>`_ (includes links
  to tutorials).
- `A detailed uproot tutorial
  <https://github.com/scikit-hep/uproot/blob/master/binder/tutorial.ipynb>`_.
- `The uproot reference documentation
  <https://uproot.readthedocs.io/en/latest/>`_.

Access ntuple branches
----------------------

Accessing branches in a ROOT TTree (we usually say "ntuple") is
something you'll do pretty much daily.

Let's say you have file called ``events.root`` and a tree inside
called ``nominal``. Let's say the ``nominal`` tree contains branches
for the electron transverse momentum (:math:`p_\mathrm{T}`, in MeV),
pseudorapidity (:math:`\eta`), and azimuthal angle (:math:`\phi`).

With ROOT TTreeReader
^^^^^^^^^^^^^^^^^^^^^

.. note:: As mentioned, ROOT has some pretty extensive documentation,
          but it can be a bit dense. This page should serve as a quick
          introduction to a few topics. If you'd like to dive into
          ROOT's ``TTreeReader`` documentation, then `start here
          <https://root.cern.ch/doc/master/classTTreeReader.html>`_.

In ROOT we'll use the ``TTreeReader`` and ``TTreeReaderValue<T>``
classes, in a file called ``read_electrons.cpp`` we can write:

.. code-block:: cpp

    #include "TFile.h"
    #include "TTreeReader.h"
    #include "TTreeReaderValue.h"
    #include "Math/GenVector/PtEtaPhiM4D.h"
    #include "Math/GenVector/LorentzVector.h"
    #include "Math/Vector4Dfwd.h"

    int main() {
      auto file = TFile::Open("events.root", "READ");
      TTreeReader reader("nominal", file);

      TTreeReaderValue<float> el_pt(reader, "el_pt");
      TTreeReaderValue<float> el_phi(reader, "el_phi");
      TTreeReaderValue<float> el_eta(reader, "el_eta");

      while (reader.Next()) {
        ROOT::Math::PtEtaPhiMVector el_fourvector;
        // the set coordinates function takes (pt, eta, phi, mass)
        el_fourvector.SetCoordinates(*el_pt, *el_eta, *el_phi, 0.511);
        // let's calculate the z-component of the momentum and print it.
        float el_pz = el_fourvector.pz();
        std::cout << el_pz << std::endl;

        // more code using the four vector...
      }

      file->Close();
      return 0;
    }

We can compile this code in our ``root6`` Anaconda environment like so:

.. code-block::

   (root6) $ $CXX read_electrons.cpp -o read_electrons $(root-config --cflags --glibs)

To create an executable called ``read_electrons``, to run it just enter

.. code-block::

   (root6) $ ./read_electrons


With uproot
^^^^^^^^^^^

.. note:: If you haven't already; take a look at the `uproot GitHub
          repository <https://github.com/scikit-hep/uproot>`_. It has
          a great introductory README.

With ``uproot`` we'll use the array access functionality, and as an
example we'll calculate an array for the :math:`z`-component of all
electron momenta (:math:`p_z = p_\mathrm{T}\sinh\eta`). we can make a
file called ``read_electrons.py``.

.. code-block:: python

   import uproot
   import numpy as np

   tree = uproot.open("events.root")["nominal"]

   el_pt = tree.array("el_pt")
   el_eta = tree.array("el_eta")
   el_phi = tree.array("el_phi")

   el_pz = el_pt * np.sinh(el_eta)

   print(el_pz.max()) # print out the maximum pz value

   ## now you can use the arrays for other things...

.. note:: When we use ``uproot`` we pull out the entire branch as an
          array. **We do not loop over the events**. This is a
          different style of programming compared to the C++ code we
          wrote with ROOT. With NumPy, we do operations *on the
          arrays*, There is no looping over an array and accessing
          individual elements. This style of programming is called
          `array programming
          <https://en.wikipedia.org/wiki/Array_programming>`_. Loops
          over NumPy arrays are very slow, but operations on the array
          are fast (hidden behind the nice python API NumPy operations
          are implemented in C and heavily optimized). You should
          almost *never* write a loop over a NumPy array!

This script can just be run with python:

.. code-block::

   (root6) $ python read_electrons.py


Counting Events
---------------

A very common task in HEP is just counting events. We frequently want
to know what happens to our yields when we do something like change a
Monte Carlo sample, or change a selection (set of cuts).

With ROOT
^^^^^^^^^




With uproot
^^^^^^^^^^^

.. code-block:: python

   import uproot
   import numpy as np

   tree = uproot.open("events.root")["nominal"]

   # give the raw number of events in the "nominal" ntuple
   num_events = len(tree)

   print("total events: ", num_events)

   # let's make a selection; how about el_pt > 20 GeV (20000 MeV)
   # we'll use a boolean array mask
   el_pt = tree.array("el_pt")
   # the initial size of the el_pt array is the full event set

   # this creates an array of bools
   mask = el_pt > 20000

   # if we call sum on the arrays, it gives us the sum of all elements
   # for an array of bools, we just have 0's (false) and 1's (true)
   print("events with el_pt > 20 GeV: ", sum(mask))


Histogram a single distribution
-------------------------------

With ROOT and TTreeReader
^^^^^^^^^^^^^^^^^^^^^^^^^

Now let's histogram the transverse momentum distribution. We'll use
the ``TH1F`` class and the ``TCanvas`` class for saving a PDF of the
histogram. We only have to add a few lines to make this happen (marked
with ``// new`` comments.

.. code-block:: cpp

    #include "TFile.h"
    #include "TTreeReader.h"
    #include "TTreeReaderValue.h"
    #include "Math/GenVector/PtEtaPhiM4D.h"
    #include "Math/GenVector/LorentzVector.h"
    #include "Math/Vector4Dfwd.h"

    #include "TH1F.h" // new
    #include "TCanvas.h" // new

    int main() {
      auto file = TFile::Open("events.root", "READ");
      TTreeReader reader("nominal", file);

      TTreeReaderValue<float> el_pt(reader, "el_pt");
      TTreeReaderValue<float> el_phi(reader, "el_phi");
      TTreeReaderValue<float> el_eta(reader, "el_eta");

      // give the histogram 20 bins from 0 to 20 GeV.
      TH1F el_pt_hist("el_pt_hist", ";electron #it{p}_{T} [GeV];Events", 20, 0, 100); // new

      while (reader.Next()) {
        ROOT::Math::PtEtaPhiMVector el_fourvector;
        // the set coordinates function takes (pt, eta, phi, mass)
        el_fourvector.SetCoordinates(*el_pt, *el_eta, *el_phi, 0.511);
        // let's calculate the z-component of the momentum and print it.
        float el_pz = el_fourvector.pz();
        std::cout << el_pz << std::endl;

        el_pt_hist.Fill(*el_pt * 0.001); // new [we convert MeV to GeV, pt variable is in MeV]

        // more code using the four vector...
      }

      TCanvas c; // new
      el_pt_hist.Draw(); //  new
      c.SaveAs("pt_hist.pdf"); // new

      file->Close();
      return 0;
    }

Rerun the compilation step, run the executable again, and you'll have
a new file called ``pt_hist.pdf``, which includes the histogram we
created.

With uproot via matplotlib
^^^^^^^^^^^^^^^^^^^^^^^^^^

Now let's do the same this in ``uproot`` with ``matplotlib``. If you
don't have ``matplotlib`` installed in your ``root6`` Anaconda
environment, let's grab it:

.. code-block::

   (root6) $ conda install matplotlib -c conda-forge

Now let's see that histogram, update our ``read_electrons.py`` script to have:

.. code-block:: python

   import uproot
   import numpy as np
   import matplotlib # new
   matplotlib.use("pdf") # new
   import matplotlib.pyplot as plt # new

   tree = uproot.open("events.root")["nominal"]

   el_pt = tree.array("el_pt")
   el_eta = tree.array("el_eta")
   el_phi = tree.array("el_phi")

   el_pz = el_pt * np.sinh(el_eta)

   plt.hist(el_pt * 0.001, bins=20, range=(0, 100), histtype="step") # new, convert MeV to GeV
   plt.savefig("pt_hist_mpl.pdf") # new

   ## now you can use the arrays for other things...

Now if you run the script

.. code-block::

   (root6) $ python read_electrons.py

You'll see a new PDF ``pt_hist_mpl.pdf`` with the histogrammed data.

Histogram a single distribution with a cut
------------------------------------------

You'll find that we like to apply selections ("cuts") to various
datasets. Let's apply a cut and make our histograms again. Let's only
histogram electron transverse momentum if the electron pseudorapidity
satisfies a particular selection. I'll let you figure out what's going
on yourself by reading the code this time!

In our ROOT analysis
^^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

    #include "TFile.h"
    #include "TTreeReader.h"
    #include "TTreeReaderValue.h"
    #include "Math/GenVector/PtEtaPhiM4D.h"
    #include "Math/GenVector/LorentzVector.h"
    #include "Math/Vector4Dfwd.h"

    #include "TH1F.h"
    #include "TCanvas.h"

    #include <cmath> // new

    int main() {
      auto file = TFile::Open("events.root", "READ");
      TTreeReader reader("nominal", file);

      TTreeReaderValue<float> el_pt(reader, "el_pt");
      TTreeReaderValue<float> el_phi(reader, "el_phi");
      TTreeReaderValue<float> el_eta(reader, "el_eta");

      TH1F el_pt_hist("el_pt_hist", ";electron #it{p}_{T} [GeV];Events", 20, 0, 100);

      while (reader.Next()) {
        ROOT::Math::PtEtaPhiMVector el_fourvector;
        // the set coordinates function takes (pt, eta, phi, mass)
        el_fourvector.SetCoordinates(*el_pt, *el_eta, *el_phi, 0.511);
        // let's calculate the z-component of the momentum and print it.
        float el_pz = el_fourvector.pz();
        std::cout << el_pz << std::endl;

        if (std::abs(*el_eta) < 1.0) {
          el_pt_hist.Fill(*el_pt * 0.001);
        }

      }

      TCanvas c;
      el_pt_hist.Draw();
      c.SaveAs("pt_hist.pdf");

      file->Close();
      return 0;
    }

Re-compile and re-run to see the new histogram.

In our uproot analysis
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   import uproot
   import numpy as np
   import matplotlib
   matplotlib.use("pdf")
   import matplotlib.pyplot as plt

   tree = uproot.open("events.root")["nominal"]

   el_pt = tree.array("el_pt")
   el_eta = tree.array("el_eta")
   el_phi = tree.array("el_phi")

   el_pz = el_pt * np.sinh(el_eta)

   el_pt_selected = el_pt[np.abs(el_eta) < 1.0]

   plt.hist(el_pt_selected * 0.001, bins=20, range=(0, 100), histtype="step")
   plt.savefig("pt_hist_mpl.pdf")

Re-run the script to see the new histogram.

Overlaying (Plotting Multiple) Histograms
-----------------------------------------

Comparing distributions is very useful in many studies. Let's see how
we can plot two histograms at the same time. We'll also include a
legend to make the plot easier to read.

We're going to add two new histograms:

- histogram the :math:`p_\mathrm{T}` for :math:`|\eta| < 1.5`
- histogram the :math:`p_\mathrm{T}` for :math:`|\eta| > 1.5`


With ROOT
^^^^^^^^^

.. code-block:: cpp

   #include "TFile.h"
   #include "TTreeReader.h"
   #include "TTreeReaderValue.h"
   #include "Math/GenVector/PtEtaPhiM4D.h"
   #include "Math/GenVector/LorentzVector.h"
   #include "Math/Vector4Dfwd.h"

   #include "TH1F.h"
   #include "TCanvas.h"
   #include "TLegend.h" // new
   #include "TStyle.h"  // new

   #include <cmath>

   int main() {
     auto file = TFile::Open("events.root", "READ");
     TTreeReader reader("nominal", file);

     TTreeReaderValue<float> el_pt(reader, "el_pt");
     TTreeReaderValue<float> el_phi(reader, "el_phi");
     TTreeReaderValue<float> el_eta(reader, "el_eta");

     TH1F el_pt_hist("el_pt_hist", ";electron #it{p}_{T} [GeV];Events", 20, 0, 100);

     // new histograms
     TH1F el_pt_hist_lowEta("el_pt_hist_lowEta", ";electron #it{p}_{T} [GeV];Events", 20, 0, 100);
     TH1F el_pt_hist_hiEta("el_pt_hist_hiEta", ";electron #it{p}_{T} [GeV];Events", 20, 0, 100);
     // change the line colors
     el_pt_hist_lowEta.SetLineColor(kRed);
     el_pt_hist_hiEta.SetLineColor(kBlack);

     while (reader.Next()) {
       ROOT::Math::PtEtaPhiMVector el_fourvector;
       // the set coordinates function takes (pt, eta, phi, mass)
       el_fourvector.SetCoordinates(*el_pt, *el_eta, *el_phi, 0.511);
       // let's calculate the z-component of the momentum and print it.
       float el_pz = el_fourvector.pz();
       std::cout << el_pz << std::endl;

       // always plot pt
       el_pt_hist.Fill(*el_pt * 0.001);

       if (std::abs(*el_eta) < 1.5) {
         el_pt_hist_lowEta.Fill(*el_pt * 0.001);
       }
       else {
         el_pt_hist_hiEta.Fill(*el_pt * 0.001);
       }
     }

     TCanvas c1;
     el_pt_hist.Draw();
     c1.SaveAs("pt_hist.pdf");

     // turn off the stat box by default
     gStyle->SetOptStat(0);
     // now draw two histograms together; notice we use the "same" argument
     TCanvas c2;
     el_pt_hist_lowEta.Draw();
     el_pt_hist_hiEta.Draw("same");

     // now we make a legend. see ROOT documentation for API reference;
     // the numbers are associated with the size and location of the
     // legned on the canvas
     TLegend legend(0.6, 0.7, 0.88, 0.9);
     legend.AddEntry(&el_pt_hist_lowEta, "low eta");
     legend.AddEntry(&el_pt_hist_hiEta, "hi eta");
     legend.Draw("same");

     c2.SaveAs("pt_hist_overlay.pdf");

     file->Close();
     return 0;
   }

With uproot
^^^^^^^^^^^

.. code-block:: python

   import uproot
   import numpy as np
   import matplotlib
   matplotlib.use("pdf")
   import matplotlib.pyplot as plt

   tree = uproot.open("events.root")["nominal"]

   el_pt = tree.array("el_pt")
   el_eta = tree.array("el_eta")
   el_phi = tree.array("el_phi")

   el_pz = el_pt * np.sinh(el_eta)
   # lets plot el_pt using matplotlib's figure
   fig, ax = plt.subplots()
   ax.hist(el_pz * 0.001, bins=20, range=(0, 100), histtype="step")
   ax.set_xlabel(r"Electron $p_z$ [GeV]")
   fig.savefig("pz_hist_from_mpl.pdf")

   # now lets plot pt based on our eta selections
   fig, ax = plt.subplots()
   ax.hist(el_pt[np.abs(el_eta) < 1.5] * 0.001, bins=20, range=(0, 100), histtype="step", label=r"$|\eta| < 1.5$")
   ax.hist(el_pt[np.abs(el_eta) > 1.5] * 0.001, bins=20, range=(0, 100), histtype="step", label=r"$|\eta| > 1.5$")
   ax.set_xlabel(r"Electron $p_\mathrm{T}$ [GeV]")
   ax.legend(loc="best")
   fig.savefig("pt_in_eta_regions.pdf")


.. _col-ana:

Columnar Analysis
-----------------

So far we've looked at how to analyze ROOT ntuples with ROOT's builtin
``TTreeReader`` and also with the ``uproot`` python library to
interface with NumPy. With ROOT's ``TTreeReader``, we were doing
classic serial programming, performing the same logic but in a hand
written loop. With ``uproot`` and NumPy we switched over to using
array programming, where we don't write loops; we write instructions
to be executed over the array (behind the scenes highly optimized C
code is actually executing a loop over the data structures, with
multiple operations being executed simultaneously; this is what makes
array programming so powerful).

Another programming paradigm for analyzing data in the form of a ROOT
ntuple (which essentially a set of columns), can be called "columnar
analysis". There are a number of software packages which implement a
so-called "data frame": a structured set of columnar data where the
operations have been optimized for the structure. This is quite
similar to NumPy, but on steroids in terms of the higher level
functionality.

With ROOT's RDataFrame
^^^^^^^^^^^^^^^^^^^^^^

ROOT uses the ``RDataFrame`` class. The documentation can be `found
here
<https://root.cern/doc/v620/classROOT_1_1RDataFrame.html>`_. You'll
notice they compare to ``TTreeReader`` usage, which you should now be
familiar with.

With a pandas DataFrame
^^^^^^^^^^^^^^^^^^^^^^^

In the Scientific Python (SciPy) ecosystem (SciPy is used to describe
an `ecosystem <https://www.scipy.org/>`_ and a `library
<https://scipy.org/scipylib/index.html>`_) the core library for
dataframes is called ``pandas``. For a simple introduction checkout
`this YouTube video
<https://www.youtube.com/watch?v=3qDhDXNRgHE>`_. ``uproot`` has some
nice functionality to go straight from a ROOT file to a ``pandas``
dataframe: `see here
<https://github.com/scikit-hep/uproot#connectors-to-other-packages>`_.

