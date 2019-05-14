Cookbook of Common Tasks
========================

Here we'll provide a few code examples for some common tasks. Thie
file referenced in this "cookbook" can be downloaded with ``wget`` if
you'd like to actually execute some of this code and see it work:

.. code-block::

   wget http://phy.duke.edu/~ddavis/public/events.root


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

In ROOT we'll use the ``TTreeReader`` and ``TTreeReaderValue<T>``
classes, in a file called ``read_electrons.cpp`` we can write:

.. code-block:: cpp

    #include "TFile.h"
    #include "TTreeReader.h"
    #include "TTreeReaderValue.h"
    #include "Math/GenVector/PtEtaPhiM4D.h"
    #include "Math/GenVector/LorentzVector.h"

    int main() {
      auto file = TFile::Open("events.root", "READ");
      TTreeReader reader("nominal", file);

      TTreeReaderValue<float> el_pt(reader, "el_pt");
      TTreeReaderValue<float> el_phi(reader, "el_phi");
      TTreeReaderValue<float> el_eta(reader, "el_eta");

      while (reader.Next()) {
        ROOT::Math::LorentzVector<ROOT::Math::PtEtaPhiM4D<float>> el_fourvector;
        // the set coordinates function takes (pt, eta, phi, mass)
        el_fourvector.SetCoordinates(*el_pt, *el_eta, *el_phi, 0.511);
        // let's calculate the z-component of the momentum and print it.
        float el_pz = el_fourvector.pz();
        std::cout << el_pz << std::endl;

        // more code using the four vector...
      }

      file->Close();
    }

We can compile this code in our ``root6`` Anaconda environment like so:

.. code-block::

   $ $CXX read_electrons.cpp -o read_electrons $(shell root-config --cflags --glibs)

To create an executable called ``read_electrons``, to run it just enter

.. code-block::

   $ ./read_electrons


With uproot
^^^^^^^^^^^

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

   ## now you can use the arrays for other things...


Histogram a single distribution
-------------------------------

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
        ROOT::Math::LorentzVector<ROOT::Math::PtEtaPhiM4D<float>> el_fourvector;
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
    }

Rerun the compilation step, run the executable again, and you'll have
a new file called ``pt_hist.pdf``, which includes the histogram we
created.


Histogram a single distribution with a cuts
-------------------------------------------

a
