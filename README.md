# fuzzion2

fuzzion2 searches RNA and DNA to find read-pairs that match fusion patterns.

## Quick Start

### Docker

Coming soon.

### Developer's Build

```
$ git clone https://github.com/stjude/fuzzion2.git
$ cd fuzzion2
$ make
```

This builds executable files named `fuzzion2`, `fuzzort`, and `kmerank` and
puts them in `build/bin`.

#### Dependencies

These programs are written in C++ and compiled using [g++] version 6 or later.

[HTSlib] version 1.10.2 or later is used to read BAM files.
Set `CPATH` and `LIBRARY_PATH` before running `make` and set
`LD_LIBRARY_PATH` before running `fuzzion2`.

```
$ HTSLIB=HTSlib-installation-directory
$ export CPATH=$CPATH:$HTSLIB/include
$ export LIBRARY_PATH=$LIBRARY_PATH:$HTSLIB/lib
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HTSLIB/lib
```

[gunzip] is used to decompress gzipped FASTQ data.

[g++]: https://gcc.gnu.org/
[HTSlib]: https://github.com/samtools/htslib
[gunzip]: https://www.gnu.org/software/gzip/

## Usage

Running fuzzion2 with no command-line arguments displays its usage.

```
fuzzion2 OPTION ... [ubam_filename ...] > matches

These options are required:
  -pattern=filename  name of pattern input file
  -rank=filename     name of binary  input file containing the k-mer rank table

The following are optional:
  -fastq1=filename   name of FASTQ Read 1 input file
  -fastq2=filename   name of FASTQ Read 2 input file
  -ifastq=filename   name of interleaved FASTQ input file (may be /dev/stdin)
  -maxins=N          maximum insert size in bases, default is 500
  -maxrank=N         maximum rank percentile of minimizers, default is 95.0
  -minbases=N        minimum percentile of matching bases, default is 90.0
  -minmins=N         minimum number of matching minimizers, default is 3
  -minov=N           minimum overlap in number of bases, default is 5
  -threads=N         number of threads, default is 8
  -w=N               window length in number of bases, default is 5
```

`N` is a numeric value, e.g., `-threads=4`.

### Input

The `-pattern` option is required and specifies the name of a text file containing
RNA or DNA patterns.  Look in the `patterns` directory for pattern files provided
with this distribution.

The `-rank` option is also required and specifies the name of a binary file
containing a k-mer rank table.  Download this file from
`http://ftp.stjude.org/pub/software/fuzzion2_hg38_k15.krt`.
It holds a 4-GB 15-mer rank table that was constructed from the GRCh38 human
reference genome.  Use this file only when searching human RNA and DNA.
The kmerank program is provided to construct rank tables for other species.

*Since fuzzion2 reads the 4-GB rank table into memory, be sure to run fuzzion2 with
at least 5 GB of memory.*

fuzzion2 examines each read-pair to see if it matches any of the patterns in the
pattern file.  Read-pairs are obtained from:

1. a single interleaved FASTQ file named by the `-ifastq` option;
1. a pair of FASTQ files identified by the `-fastq1` and `-fastq2` options; or
1. one or more unaligned Bam files specified on the command line.

fuzzion2 expects that the mates of a read-pair are adjacent in interleaved FASTQ
files and unaligned Bam files.  If a pair of FASTQ files is specified, the mates
are separated; "Read 1" mates are in the first file and corresponding "Read 2"
mates are in the second file.  If the name of a FASTQ file ends with ".gz",
fuzzion2 assumes that the file is gzipped and uses gunzip to decompress the data.

### Output

Each read-pair that matches a pattern is a "hit" and is written along with the
pattern on three lines to the standard output stream.  In the following example,
`BCR-ABL1` is the name of the pattern and 208 is the insert size of the read-pair
aligned to the pattern.  The second column of the first line shows the substring
of the pattern sequence that matches the read-pair.  The second and third lines
show the read name and entire sequence of each mate.

```
pattern BCR-ABL1 208         CATCCGTGGAGCTGCAGATGCTGACCAACTCGTGTGTGAAACTCCAGACTGTCCACAGCATTCCGCTGACCATCAATAA]GGAAGA[AGCCCTTCAGCGGCCAGTAGCATCTGACTTTGAGCCTCAGGGTCTGAGTGAAGCCGCTCGTTGGAACTCCAAGGAAAACCTTCTCGCTGGACCCAGTGAAAATGACCCCAACCTTTTCGTTGC
EXAMPLE:1105:12909:66982/2   CATCCGTGGAGCTGCAGATGCTGACCAACTCGTGTGTGAAACTCCAGACTGTCCACAGCATTCCGCTGACCATCAATAAGGAAGAAGCCCTTCAGCGGCCA
EXAMPLE:1105:12909:66982/1                                                                                                                TCTGACTTTGAGCCTCAGGGTCTGAGTGAAGCCGCTCGTTGGAACTCCAAGGAAAACCTTCTCGCTGGACCCAGTGAAAATGACCCCAACCTTTTCGTTGC
```

A final line is written to the standard output stream showing the total number
of read-pairs processed by the program.

When multithreading is used (i.e., the value of the `-threads` option is greater
than 1), the order of the hits is indeterminate and a simple `diff` cannot be
used to compare output files.  It is therefore recommended to run the fuzzort
program to sort the hits.  This program sorts the hits by pattern name so that
the hits are in a determinate order (and `diff` can be used to compare files)
and the hits of a pattern are grouped together.

```
Usage: fuzzort < fuzzion2_hits > sorted_hits
```

The `test` directory contains some files you can use to run a simple test:

```
fuzzion2 -pattern=example_patterns.txt -rank=fuzzion2_hg38_k15.krt \
   -fastq1=example_input1.fq -fastq2=example_input2.fq | fuzzort > my_output.txt
```

The file named `my_output.txt` should match the provided file named
`example_output.txt`.

If the read-pairs for a sample are stored in multiple pairs of FASTQ files, run
fuzzion2 once for each pair of FASTQ files and concatenate the resulting output
files.  Then run fuzzort on the concatenated file to get a single sorted file
of hits.

## Patterns

(This section will describe the distributed pattern files, the format of patterns,
and the programs provided for creating patterns.)

## COPYRIGHT

Copyright 2021 St. Jude Children's Research Hospital

## LICENSE

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
