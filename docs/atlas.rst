ATLAS Software & Datasets
=========================

Centralized ATLAS Tutorial
--------------------------

The ATLAS Analysis Software Group (ASG) maintains a very detailed
tutorial for getting started with ATLAS specific analysis software. If
you have to analyze ATLAS analysis-level datasets, so-called DAODs
(derived analysis object data), then going through the central
tutorial **is absolutely necessary**. This tutorial can be found `here
<https://atlassoftwaredocs.web.cern.ch/ABtutorial/>`_.

Downstream Datasets
-------------------

The defition of "downstream datasets" is relative to a specific task,
but usually in an ATLAS analysis the final steps are performed on
so-called "flat ntuple" style datasets. These are datasets which are
independent of the complex collaboration specific software stack. The
tools described in the later sections of this website are perfect for
analyzing flat ntuple datasets. The ATLAS DAOD datasets are far from
flat (they have a complex structure that is required to maintain
flexibility to be used by many collaborators, while flat ntuples are
usually fine tuned for a handful of people in an analysis).

If you need to analyze DAODs, the rest of these docs can still be
useful for later steps in your work, but the ASG tutorial is your best
friend. If you're analyzing very downstream datasets that are in a
plain ROOT ntuple format, then the rest of these docs should
definitely be useful (most analyzers generate plain ROOT ntuples from
DAODs).
