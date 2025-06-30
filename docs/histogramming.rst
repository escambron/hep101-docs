Using Histograms for Data Analysis in Particle Physics
=========================================================

What is a Histogram?
--------------------

A **histogram** is a graphical representation of the distribution of a dataset. In particle physics, histograms are used to:
visualize the number of events (counts) expected within specific ranges (bins) of a variable or set of variables.
Each bar (or "bin") in the histogram corresponds to a **range of values**, and the height of the bar represents, in some way, the probability of that variable being in that range.
We usually examine one- or two-dimensional histograms visually; higher dimensional histograms can be used for mathematical analysis although they are harder to visualize.


Uncertainties in Histograms
---------------------------

Every bin in a histogram comes with a **statistical uncertainty**, often due to the finite number of events used to create it. This is typically:

- Represented as an **error bar** on the bin height,
- Modeled as **Poisson** uncertainty for event counts: for count :math:`N`, the standard deviation is approximately :math:`\sqrt{N}`.

In cases with weighted events (common in simulations), uncertainty is calculated using:

.. math::

   \sigma_{\text{bin}} = \sqrt{\sum_i w_i^2}

where :math:`w_i` are the weights of the individual events in that bin.

Stacked Histograms
------------------

A **stacked histogram** is used when multiple processes contribute to the same observable. Instead of superimposing the plots, each process is shown as a segment of a bar **stacked vertically**.

This helps to visually display how much of the total comes from each process, and in many analyses, is used to show locally how background contributions compare to the signal.
Stacked histograms are particularly useful when estimating the **expected composition** of an observable in experimental data.

Comparing True and Reconstructed Values
---------------------------------------

In high-energy experiments, we often want to compare the **true** value of a physical quantity (from simulation) with the **reconstructed** value (from detector output). A naive approach is to overlay the two histograms, but this is not ideal:
it's possible for the two distributions to agree, even when there is a systematic difference between the two quantities.  

Instead, consider these better strategies:

1. **Histogram of Differences**:

   - Plot :math:`\Delta x = x_{\text{reco}} - x_{\text{true}}`
   - Reveals biases (nonzero mean) and resolutions (spread).

2. **Histogram of Relative Differences**:

   - Plot :math:`\Delta x / x_{\text{true}}`
   - Useful for quantities with large dynamic range.

3. **2D Histograms (Scatter Plots or Heatmaps)**:

   - Plot :math:`x_{\text{true}}` on one axis and :math:`x_{\text{reco}}` on the other.
   - Perfect reconstruction would lie along the line :math:`x_{\text{reco}} = x_{\text{true}}`.
   - Reveals what values of the parameter are most affected by bias or resolution.


Data Sufficiency: How Much Is Enough?
-------------------------------------

Relative statistical uncertainties decrease with more data.  When you don't have enough, you will see noisy, misleading histograms. You can reduce this noise by increasing the number of events per bin:

- Smaller bin sizes reduce the number of events per bin, leading to large fluctuations, but
- Larger bin sizes oversimplify the shape and hinder analysis.

Ask yourself:

- Are the **error bars** smaller than the **differences** you are trying to observe?
- Would rebinning (fewer, larger bins) give more meaningful trends, or do you need to create more data?
- Could **bootstrapping** or **toy experiments** help estimate robustness?

Summary
-------

Histograms are a core part of data analysis in particle physics. To use them effectively:

- Represent uncertainties correctly,
- Choose comparisons that reveal rather than obscure differences,
- Think carefully about binning and data sufficiency.

As always, good histogram practice involves **critical reasoning**, not just plotting.


Further resources
-----------------

If you want to use histograms in python, you should consider `hist<https://hist.readthedocs.io/en/latest/>`.


