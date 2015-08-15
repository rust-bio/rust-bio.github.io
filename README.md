# Rust-Bio, a bioinformatics library for Rust.

This library provides implementations of many algorithms and data structures
that are useful for bioinformatics.
All provided implementations are rigorously tested via continuous
integration.

Currently, rust-bio provides

* most major pattern matching algorithms,
* a convenient alphabet implementation,
* pairwise alignment,
* suffix arrays,
* BWT and FM-Index,
* FMD-Index for finding supermaximal exact matches,
* a q-gram index,
* a rank/select data structure,
* FASTQ and FASTA and BED readers and writers,
* helper functions for combinatorics and dealing with log probabilities.

For reading and writing BAM and BCF files, have a look at https://github.com/christopher-schroeder/rust-htslib.

## Resources

* [Documentation](https://rust-bio.github.io/rust-bio)
* [Continuous integration tests](https://travis-ci.org/rust-bio/rust-bio)
* [Roadmap](https://github.com/rust-bio/rust-bio/issues/3)
