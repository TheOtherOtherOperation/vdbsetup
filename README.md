# VDBSetup

## License
Copyright (c) 2016 DeepStorage, LLC (deepstorage.net) and Ramon A. Lovato (ramonalovato.com).

See the file LICENSE for copying permission.

Author: Ramon A. Lovato (ramonalovato.com)
For: DeepStorage, LLC (deepstorage.net)
Version: 1.0

## Introduction
VDBSetup (vdbsetup.py) is a tool for automatically generating VDbench configuration files with complex hotspot distributions.

## Requirements
- Python 3.4 or later (http://www.python.org/)
- NumPy 1.11.0 or later (http://www.numpy.org/)
- matplotlib 1.5.1 or later (http://matplotlib.org/)
- Oracle VDbench 5.04.01 or later (http://www.oracle.com/technetwork/server-storage/vdbench-downloads-1901681.html)
- Java SE Runtime Environment (JRE) 1.6 or later (http://www.oracle.com/technetwork/java/javase/downloads/index.html)

Windows note: pre-built binaries for the scientific Python stack (including NumPy and matplotlib), as well as other extension libraries, are available from `http://www.lfd.uci.edu/~gohlke/pythonlibs/`.

## Configuration
Due to the complexity of VDbench configuration files, VDBSetup's configuration files are also complex.

Running vdbsetup.py --make-template will cause the script to output the following example file:


```
#
# vdbsetup input file example
#

#
# General
#
dedupratio=2
dedupunit=4k
compratio=1.5

#
# SDs
#
luns=lun1,lun2,lun3
# Optional: o_direct provided by default
# openflags=

#
# WDs
#
wdcount=1
xfersize=4k
seekpct=100
rdpct=75
percentdisk=100.0

#
# RDs
#
iorate=1000
format=yes
elapsed=60
interval=1
threads=2

#
# Distribution
#
hotspotnum=10
hotspotcap=25
hotspotiopct=10
disttype=gaussian
# Note: only required if disttype=gaussian
distribution=0.75,0.5
```

Empty lines and comments, those beginning with a hash ("#"), are ignored. Equal signs ("=", major delimiter) separate keys from values. If a parameter accepts multiple values, you may specify it as a comma-delimited (minor delimiter) list. Both the major and minor delimiter may be overridden using the --major-delimiter and --minor-delimiter command-line flags.

All parameters are required unless explicitly stated otherwise. Additionally, order matters for certain parameters (most notably, those under "Distribution"). If a parameter has a one-to-one mapping to a VDbench parameter (i.e. "dedupratio", "dedupunit", etc.), very little error checking is performed, so users should take care to check the VDbench specifications for formatting and valid inputs. Also note that, while VDbench does not complain about high-precision floating point values, it ignores fractional components beyond two (2) decimal places.

### Parameters

**General**
- `dedupratio`
Specify the deduplication ratio.
- `dedupunit`
Specify the deduplication unit size. This value is unchecked. Please see the VDbench documentation for proper formatting.
- `compratio`
Specify the compression ratio.

**Storage Definitions (SDs)**
- `luns` (list)
Specify the logical volumes (LUNs) to be used in the test as a comma-separated list. All LUNs will be used for all workloads. Variable names may be provided (e.g. $LUN1,$LUN2) and specified at runtime when calling VDbench.
- `openflags` (list)
Optional. Adds additional flags when opening the LUNs. The flag "o_direct" is provided by default and cannot be removed, as physical device identifiers may not be accessed under Linux without it.

**Workload Definitions (WDs)**
- `wdcount`
The number of non-hotspot workloads to generate. VDbench will automatically split any skew not assigned to hotspots among all remaining workloads.
- `xfersize`
The block size to use for IOs. Also affects hotspots.
- `seekpct`
The percentage of IOs that should utilize random access. A value of 100 is fully random, while a value of 0 is completely sequential. Also affects hotspots.
- `rdpct`
The percentage of IOs that should be reads (as opposed to writes). E.g. "rdpct=50" is 50% reads and 50% writes.
- `percentdisk`
Maximum percentage of disk to be used by any and all workloads. No general workload or hotspot will be generated outside the range [0, percentdisk].

**Run Definitions (RDs)**
- `iorate`
The number of IOPS to try for.
- `format`
Whether or not the storage devices should be formatted prior to use.
- `elapsed`
The duration of the test in seconds.
- `interval`
The reporting interval in seconds.
- `threads`
The number of threads/queue depth.

**Distribution**
- `hotspotnum`
The number of hotspots to generate.
- `hotspotcap`
The maximum disk capacity to be occupied by all hotspots as a percentage of total disk space.
- `hotspotiopct`
The maximum percentage of disk IOs to split among all hotspots.
- `disttype`
Specifies the type of hotspot distribution to generate --- one of "even", "gaussian", or "uniform".
  - "even": generates hotspotnum hotspots each of size hotspotcap/hotspotnum and skew hotspotiopct/hotspotnum.
  - "gaussian": generates a Gaussian (normal) distribution bell curve. Some hotspots will have a greater percentage (skew) of IOs than other hotspots. Hotspot sizes (ranges) will be randomly generated, and a second Gaussian will be used to distribute hotspot starting positions so more hotspots are near the beginning of the disk than the end. Standard deviations for the Gaussians of both the skew and range starting positions must be provided using the "distribution" parameter.
  - "uniform": generates hotspots with uniform random skews and ranges, honoring the "hotspotcap", "hotspotiopct", and "capacity" parameters.
- `distribution` (list, required if "disttype=gaussian")
Specifies the standard deviations of the two Gaussian distributions for IO percentage (skew) and range starting position as a pair of floating point values. E.g. "distribution=0.75,0.5" means the Gaussian for the skew will have a standard deviation of 0.75 and the Gaussian for the starting positions will have a standard deviation of 0.5.

## Usage
```
usage: vdbsetup.py [-h] [--make-template] [-v] [-gs] [-gr] [--no-overwrite]
                   [--no-shuffle] [--header HEADER] [-M MAJOR_DELIMITER]
                   [-m MINOR_DELIMITER] [-c SAMPLE_COUNT]
                   [inPath] [outPath]

create VDbench hotspot-distribution configuration files

positional arguments:
  inPath                where to find the input file
  outPath               where to output the configuration file

optional arguments:
  -h, --help            show this help message and exit
  --make-template       create an example input file and exit
  -v, --verbose         enable verbose mode
  -gs, --graph-skews    enable graph display of the hotspot skews
  -gr, --graph-ranges   enale graph display of the hotspots ranges
  --no-overwrite        don't overwrite output file if it already exists
  --no-shuffle          disable random hotspot permutation
  --header HEADER       add a comment header
  -M MAJOR_DELIMITER, --major-delimiter MAJOR_DELIMITER
                        major delimiter regex used in configuration file
                        (default " *= *")
  -m MINOR_DELIMITER, --minor-delimiter MINOR_DELIMITER
                        minor delimiter regex used in configuration file
                        (default " *, *")
  -c SAMPLE_COUNT, --sample-count SAMPLE_COUNT
                        number of samples to generate when using Monte Carlo
                        method to compute distributions (default 200000);
                        setting sample size < 10000 is strongly discourage
```

### Example
```bash
vdbsetup.py input_file.txt output_file.txt
```

### Optional Parameters
- `--make-template`
Causes VDBSetup to output a configuration file template called "vdbsetup_input_template.txt". This flag overrides the input and output file parameters.
- `-gs, --graph-skews`
Uses matplotlib to generate a bar chart showing the percentage of IOs assigned to each hotspot. May be specified alongside --graph-ranges.
- `-gr, --graph-ranges`
Uses matplotlib to generate a bar chart showing the percentage of IOs assigned to each hotspot on the dependent axis with the hotspots' starting positions as a percentage of the disk on the independent axis and bar widths showing the hotspot sizes. May be specified alongside --graph-skews.
- `--no-overwrite`
Prevents overwriting the output file. If the output file already exists, a suffix of the form " (n)", where **n** is an integer, will be appended to the output path. This is especially useful for generating multiple VDbench configurations.
- `--no-shuffle`
Disable generation of random permutations of hotspot range-IO rate combinations. Enabling --no-shuffle when generating Gaussian distributions will cause hotspots near the mean of the distribution to have higher IO rates than those near the extrema, preserving the bell curve visible when using --graph-skews. This is generally only useful for debugging.
- `--header HEADER`
Causes a block comment header to be added at the beginning of the file. The comment is automatically line wrapped to 70 characters.
- `-M MAJOR_DELIMITER`
Overrides the major delimiter regex used in the configuration file (default " *= *").
- `-m MINOR_DELIMITER`
Overrides the minor delimiter regex used in the configuration file (default " *, *").
- `-c SAMPLE_COUNT`
Override the number of samples generated when using Monte Carlo method to compute the Gaussian distribution. The default value of 200,000 is usually sufficient. Setting this number too small has a negative effect on precision and can, in extreme cases, cause hotspot generation to fail.

## Version History
1.0 - Initial release.



This document was last updated on 03/29/16.
