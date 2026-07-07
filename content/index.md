---
title: MSense Lab fMRI Tutorial
---

A hands-on, reproducible guide to analysing functional MRI — from a raw scanner
export all the way to a thresholded group activation map and task-based
connectivity. The material grew out of the [MSense Lab](https://msense.de)
seminar series and is updated as tools and best practices evolve.

**Who it is for.** Master's and early-PhD researchers who have fMRI data (or soon
will) and want to understand not just *which button to press* but *why* each step
matters. We assume some Python and basic statistics; everything MRI-specific is
built up from the ground.

**What it covers.** The tutorial walks through the whole task-fMRI pipeline, one
step at a time, with runnable code (FSL, Nilearn, Nipype) and worked examples:

- **Foundations** — how MRI forms an image, and where the BOLD signal and the
  haemodynamic response function come from ([[1. MRI Basics]], [[1.1 BOLD Signal and HRF]]).
- **From scanner to analysis-ready data** — DICOM conversion and formats, the
  BIDS standard, preprocessing with fMRIPrep, and quality control with MRIQC
  ([[2.1 MRI Data Acquisition and Format]] → [[2.5 fMRIPrep and FitLins]]).
- **Modelling the signal** — experimental design, then the general linear model:
  design matrices, HRF convolution, contrasts, and statistical inference
  ([[3.1 Experimental Design]], [[3.2 General Linear Model]]).
- **First- and group-level analysis** — running the GLM in FSL and in
  Python/Nilearn, selecting confounds, and thresholding honestly for the
  multiple-comparisons problem ([[4.1 First-level Analysis with FSL]] →
  [[5.3 Visualization and Statistical Thresholding with Nilearn]]).
- **Connectivity** — resting-state functional connectivity, and task-modulated
  connectivity with PPI and RSA ([[6. Functional Connectivity and Resting State]],
  [[7. Task-fMRI Connectivity (PPI and RSA)]]).

**How to use it.** Read top to bottom for a full course, or open a single chapter
as a standalone reference — each is self-contained. The companion **seminar
slides** ([[Session 1 Slides]], [[Session 2 Slides]]) give the same material in a
lecture format, and the *Server and Software* section covers the computing setup.

**A note on philosophy.** We favour **reproducible, well-documented choices** over
black-box defaults. Wherever a decision changes the result — a confound set, a
smoothing kernel, a correction method — we try to lay out the trade-off so you can
justify it in your Methods section. Same data plus the same documented choices
should give the same map.
