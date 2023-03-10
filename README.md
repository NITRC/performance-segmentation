# NITRC tool performance: segmentation

This repository contains code for preparing segmentation data for
[NITRC's tool performance
project](https://www.nitrc.org/projects/performance/).

## Installation

Start a [NITRC Computational Environment](https://www.nitrc.org/ce/)
(CE) instance, v0.56-2.  Set up the CE according to the documentation
and add a FreeSurfer license.

Install Python 3.10 from source.  (Code for this project requires
a version of Python newer than that supported by the base system
of the CE.)  Install packages given in `requirements.txt`:

```python3 -m pip install -r requirements.txt```

## Running

First download the source images.

From [The Internet Brain Segmentation
Repository](https://www.nitrc.org/projects/ibsr) (IBSR), download
`IBSR_##_ANALYZE.tgz` from the IBSR_V2.0 release and put them in
`raw/ibsr/` relative to this directory.

From [Brain Segmentation Testing
Protocol](https://www.nitrc.org/projects/bstp) (BSTP), download
`reproducibilility_test_retest.zip` (from "Same scanner and sequence"),
`reproducibility_different_scanner.zip` (from "Different scanner
and pulse sequence"), and
`OASIS_Baseline_scan_144_healthy_participants.zip` and
`OASIS_Follow_up_scan_144_healthy_participants.zip` (from "Longitudinal
images") and put them under `raw/bstp/`.

Checksums for these source files can be found in `MD5SUMS`.

Prepare the source data by running `bin/prepare_source`.  This will
create `tmp/` and `source/`.

Run the segmentations using `bin/run`.  This will create `runs/`
containing run data.

Run `bin/compile` to compile run data and create images and plots.
This creates `data/`, which is packaged and released [on
NITRC](https://www.nitrc.org/frs/?group_id=1591).  Note that `compile`
checks for and excludes failed runs; watch the output for reports.

`bin/build` is used internally by NITRC to prepare content displayed
on NITRC.

## Internals

`bin/run` runs analyses in series so runtimes are measured on a
dedicated machine.  It is possible to run analyses on separate
machines by using the `-r` option to `bin/run`.

Each tool in the analysis is configured by files under `tools/`.
Each tool (or a run configuration of each tool) has a subdirectory
containing the following files:

* `command_line`: The command line to be displayed on NITRC's report.
  This file is not actually used to run the analysis.
* `name`: The tool name to be displayed on NITRC's report.
* `nitrc_id`: The NITRC ID of the tool.
* `version`: An executable that reports the version of the tool.
* `run`: An executable that performs the analysis.
* `preprocess` (optional): An executable that preprocesses the data
  to prepare it for the run.  The time taken by `preprocess` is not
  counted in the runtime.
* `postprocess` (optional): An executable that postprocesses the
  data after the run.  The time taken by `postprocess` is not counted
  in the runtime.

In addition, tool-specific files can be stored in each subdirectory.

`bin/run` creates `runs/`, under which is a directory for each tool.
Under each of these are directories, one per run, coded by subject
IDs.  Under each run directory is written:

* `data/`: The data directory containing data from the run.
* `output`: The output (stdout and stderr) from the run.
* `status`: The exit status from the run.
* `time`: The runtime.
* `version`: The tool version (the output of the version script).

If `preprocess` or `postprocess` are defined for a tool,
`output.preprocess`, `status.preprocess`, `output.postprocess`, and
`status.postprocess` are written.

`data/` is the working directory for `run`, `preprocess`, and
`postprocess`, so each can limit itself to writing to its current
directory.  The following environment variables are defined and
available to the scripts:

* `TP_HOME`: This top-level directory.  This can be used to access
  global configuration files under `lib/`.
* `TOOL_DIR`: The tool directory under `tools/`, used to access
  tool-specific configuration files.
* `SOURCE_DIR`: The subject's directory under `source/`, used to
  access the source data.

## Space needs

Run Version 1.0 uses the following disk space:

| Directory | Space |
|---|---|
|data|654M|
|raw|3.1G|
|runs|67G|
|source|2.9G|
|tmp|7.6G|
