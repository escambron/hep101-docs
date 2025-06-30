
Using Event Bookkeeping
-----------------------

Event bookkeeping is needed when you have multiple processes that would contribute to the data you select.
Combining simulated datasets into a realistic representation of data typically requires *weighting events,* if those datasets were created in different event generation steps.

Event weighting requires keeping track of information created during the Monte Carlo generation and subsequent processing of your data.


Combining MC datasets
^^^^^^^^^^^^^^^^^^^^^
If you are using Monte Carlo to model your data, you will probably have to combine simulated datasets.  
To be precise, "dataset" here refers specifically to a group of events (stored in one or more files) but sharing the same Monte Carlo channel number (the 6-digit number after the run conditions in the name of the Monte Carlo sample). 
This MC channel number identifies a specific, invariant set of event generation parameters.
Events with different MC channel numbers are usually not mixed together in files (at least, not until the very last stage of processing). 
To analyze them together, they need to be appropriately weighted.

What you need to know to weight your datasets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. The **luminosity** of the (real) dataset you want to model.  This is usually the luminosity of the real ATLAS data you will use for your analysis.
   You can get this information (or learn how to calculate it) from the ATLAS `luminosity for physics <https://twiki.cern.ch/twiki/bin/view/Atlas/LuminosityForPhysics>`_ pages.
2. The **cross-section** of each process contributing to your analysis.  The cross-section is reported by the Monte Carlo event generator, and stored as ATLAS metadata.
   You can look it up in many places, including the ATLAS Central Page in `TWiki <https://twiki.cern.ch/twiki/bin/view/AtlasProtected/CentralMC23ProductionListNew>`_ or shell-command format,
   or the web-based ATLAS Metadata Interface tool `AMI <https://atlassoftwaredocs.web.cern.ch/analysis-software/AnalysisSWTutorial/find_data_ami/>`_.
3. The **events created and used** to produce the dataset you will analyze for each process.  This is where bookkeeping comes in.
   
   Think of this as the number of events of each type that *had a chance* to make it into your final ntuples.
   It will differ from the number of events in your ntuples because Monte Carlo generation, derivation production, failed processing jobs, and your analysis framework may each remove events from your simulated sample, either intentionally or unintentionally.
   You need to know this number since the cross-section reported by the MC generator doesn't take any of these losses into account.

   .. note::
      Some MC generators produce weighted events.
      These weights are stored in the EventInfo (often appearing in ntuples as `generatorWeight`). Because of this "number of events" in the description above should actually be represented as "sum of event weights." 

Why bookkeeping?
^^^^^^^^^^^^^^^^
Bookkeeping is needed when events are filtered between event generation and your final analysis.
Here are some specific examples of filtering that might occur, and how to correct the generator cross-section in each case.
- MC generation of top pairs where one or more top quarks decays to a muon can be made by generating inclusive top pairs using POWHEG, and then filtering events without muons in the final state using Athena code.
  
The cross section for t-tbar production would need to br multiplied by the efficiency of this filtering step, typically called ``filtEff`` or ``GeneratorFilterEfficiency`` in ATLAS databases.  
- The **derivation framework** can apply a loose preselection on Monte Carlo events to save storage space.
  
  The derivation transform keeps track of the number of **input events** and the **sum of input events' weights** for every file it creates (when jobs crash and create partial files, this is also flagged somehow).
  These numbers represent the event/weight totals *after* the GeneratorFilter step of processing, but before any filtering. They're stored in metadata TTrees in each derivation file produced.
  
  Practically, you can calculate the denominator using all successfully processd files, and store it (for each sample) as the `sum_of_generatorWeights`.
  This sum can be used as a factor to rescale the generator weight assigned to each event in your data, so each rescaled event weight is equal to *the fraction of generated data represented by this event.*

- A few of your ntuple-making jobs could have crashed on the grid.  If they crash, the summed event counts and summed weight metadata should not have been produced. 
  .. warning::
  To avoid errors in weights, don't use partial ntuples produced from crashed jobs!!

- You might make tighter trigger or fiducial cuts in your DAOD-processing job before writing to your ntuple. This is taken into account as long as you are using the `sum_of_generatorWeights` described above:
  the events you keep still represent the fraction of initially generated data corresponding to their (rescaled) weight.
     

How to weight your datasets
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Remember that the number of events you expect in your final dataset from a process is the cross-section times the luminosty times the selection efficiency.
The procedure described below will scale your data to a luminosity one inverse picobarn; you can then rescale Monte Carlo histograms by the actual luminosity of the data to compare MC to data.

Some uproot-like code is written below. The function `retrieveFileMetadata` would be specific to your framework, so it isn't shown here,
but it could be created based on `these tools <https://gitlab.cern.ch/atlas/athena/-/tree/21.2/PhysicsAnalysis/AnalysisCommon/CutBookkeeperUtils>`_.

.. code-block:: python

   for channelNumber in [801169, 801170, 801171, 801172]:
       #original filtered cross-section for sample, based on dictionaries filled earlier from AMI query
       correction = ami_crosssection[channelNumber] *ami_effFilter[channelNumber]
       #sum of weights seen in input files, to correct for processing losses in sample
       SOW=0
      for rootfile in usedfilenames[channelNumber]: 
          SOW += retrieveFileMetadata(rootfile,"CutBookkeepers","sumOfEventWeights") 
      #set the denominator so that events from each process can be scaled to one inverse picobarn of luminosity.
      myTree[myTree.mcChannelNumber == channelNumber].mcFinalWeight = myTree.generatorWeight * correction / SOW


Key takeaway: *after* running over all DAODs (or other format containing CutBookkeepers metadata), add up the metadata from all of the files you processed.
This, combined with the generator cross section and event generation filter efficiency, allows you to change the generatorWeight of each event to correspond to a known luminosity.
For this to work, each file that you use in analysis has to contain events from only one MC channel (since bookeeping doesn't keep track of channel numbers).

   .. note :: 
  
      It's good to use Athena tools (including packages like EasyJet or TopCPToolkit) to read DAOD and write ntuples, since these usually copy the event bookkeeping metadata to your output ntuple files.
      Some other workflows read DAOD and write python, parquet, or other formats directly.  If you are doing this, you must copy the required metadata from the DAOD files that you process by hand.

      You should be sure to use consistent units when using the process cross sections.

   .. warning::
      If you later process more events from the same Monte Carlo process, you will need to adjust the weights stored in your data!

