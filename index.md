[![Crates.io](https://img.shields.io/crates/d/bio.svg?style=flat-square)](https://crates.io/crates/bio)
[![Crates.io](https://img.shields.io/crates/v/bio.svg?style=flat-square)](https://crates.io/crates/bio)
[![Crates.io](https://img.shields.io/crates/l/bio.svg?style=flat-square)](https://crates.io/crates/bio)
[![Travis](https://img.shields.io/travis/rust-bio/rust-bio.svg?style=flat-square)](https://travis-ci.org/rust-bio/rust-bio)

[![GitHub watchers](https://img.shields.io/github/watchers/rust-bio/rust-bio.svg?style=social&label=Watch)](https://github.com/rust-bio/rust-bio/watchers)
[![GitHub stars](https://img.shields.io/github/stars/rust-bio/rust-bio.svg?style=social&label=Star)](https://github.com/rust-bio/rust-bio/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/rust-bio/rust-bio.svg?style=social&label=Fork)](https://github.com/rust-bio/rust-bio/network)

This library makes many algorithms and data structures that are useful for bioinformatics available for the [Rust language](https://www.rust-lang.org).
All provided implementations are rigorously tested via continuous
integration.

Currently, Rust-Bio provides

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

A detailed overview of the supported algorithms and data structures can be found in the [API Documentation](https://docs.rs/bio).
For reading and writing SAM/BAM, CRAM and VCF/BCF files, Rust-Bio is complemented by [Rust-Htslib](https://github.com/rust-bio/rust-htslib).

When using Rust-Bio, please cite the following article:

[Köster, J. (2015). Rust-Bio: a fast and safe bioinformatics library. Bioinformatics.](http://bioinformatics.oxfordjournals.org/content/early/2015/10/06/bioinformatics.btv573.short?rss=1)

## Resources

* [News](https://github.com/rust-bio/rust-bio/releases)
* [API Documentation](https://rust-bio.github.io/rust-bio)
* [Issues](https://github.com/rust-bio/rust-bio/issues)
* [Roadmap](https://github.com/rust-bio/rust-bio/issues/3)
* [Continuous integration tests](https://travis-ci.org/rust-bio/rust-bio)


## Usage

We explain how to use Rust-Bio step-by-step. Users who already have experience with Rust can skip the first two steps.

### Step 1: Setting up Rust

Rust-Bio needs the stable version (>=1.3.0) of Rust.
Rust can be installed or updated by executing

```bash
curl -sSf https://static.rust-lang.org/rustup.sh | sh
```
in your terminal. Depending on the age of your system, the registered CA certificates might not be new enough and curl will display an error about being unable to verify the certificate. In that case, you can add a flag -k to proceed.
For details or alternative ways of installation, have a look at the [Rust download page](https://www.rust-lang.org/install.html).

### Step 2: Setting up a new Rust project

Since Rust-Bio is a library, you need to setup your own new Rust project to use Rust-Bio.
With Rust, projects and their dependencies are managed with the builtin package manager [Cargo](https://crates.io/).
To create a new Rust project, issue

```bash
cargo new hello_world --bin
cd hello_world
```
in your terminal. The flag `--bin` tells Cargo to create an executable project instead of a library.
In [this section](https://doc.rust-lang.org/nightly/book/hello-cargo.html#a-new-project) of the Rust docs, you find details about what Cargo just created for you.

Your new project can be compiled with
```bash
cargo build
```
If dependencies in your project are out of date, update with
```bash
cargo update
```
Execute the compiled code with
```bash
cargo run
```
If you are new to Rust, we suggest to proceed with [learning Rust](https://doc.rust-lang.org/nightly/book/learn-rust.html) via the Rust docs.

### Step 3: Use Rust-Bio in your project

To use Rust-Bio in your Rust project, add the following to your `Cargo.toml`

```toml
[dependencies]
bio = "*"
```

and import the crate from your source code:

```rust
extern crate bio;
```

### Example

An example usage of Rust-Bio is presented in the following:
```rust
// Import some modules
use bio::alphabets;
use bio::data_structures::suffix_array::suffix_array;
use bio::data_structures::bwt::{bwt, less, Occ};
use bio::data_structures::fmindex::{FMIndex, FMIndexable};
use bio::io::fastq;


// a given text
let text = b"ACAGCTCGATCGGTA";

// Create an FM-Index for the given text.

// instantiate an alphabet
let alphabet = alphabets::dna::iupac_alphabet();
// calculate a suffix array
let pos = suffix_array(text);
// calculate BWT
let bwt = bwt(text, &pos);
// calculate less and Occ
let less = less(&bwt, &alphabet);
let occ = Occ::new(&bwt, 3, &alphabet);
// setup FMIndex
let fmindex = FMIndex::new(&bwt, &less, &occ);


// Iterate over a FASTQ file, use the alphabet to validate read
// sequences and search for exact matches in the FM-Index.

// obtain reader or fail with error (via the unwrap method)
let reader = fastq::Reader::from_file("reads.fastq").unwrap();
for result in reader.records() {
    // obtain record or fail with error
    let record = result.unwrap();
    // obtain sequence
    let seq = record.seq();
    if alphabet.is_word(seq) {
        let interval = fmindex.backward_search(seq.iter());
        let positions = interval.occ(&pos);
    }
}
```

This example shall only illustrate the API (e.g. we don't provide the ``reads.fastq`` file). For more information and working examples for each provided algorithm, please read the [documentation](https://rust-bio.github.io/rust-bio).

## Benchmarks

Since Rust-Bio is based on a compiled language, similar performance to C/C++ based libraries can be expected. Indeed, we find the pattern matching algorithms of Rust-Bio to perform in the range of the C++ library Seqan:

| Algorithm | Rust-Bio | Seqan   |
| --------- | -------: | ------: |
| BNDM      | 77ms     | 80ms    |
| Horspool  | 122ms    | 125ms   |
| BOM       | 103ms    | 107ms   |
| Shift-And | 241ms    | 545ms   |

We measured 10000 iterations of searching pattern `GCGCGTACACACCGCCCG` in the sequence of the hg38 MT chromosome.
Initialization time of each algorithm for the given pattern was included in each iteration. Benchmarks were conducted with *Cargo bench* for Rust-Bio and *Python timeit* for Seqan on an Intel Core i5-3427U CPU.
Benchmarking Seqan from *Python timeit* entails an overhead of 1.46ms for calling a C++ binary. This overhead was subtracted from above Seqan run times.
Note that this benchmark only compares the two libraries to exemplify that Rust-Bio has comparable speed to C++ libraries: all used algorithms have their advantages for specific text and pattern structures and lengths (see [the pattern matching section in the documentation](https://rust-bio.github.io/rust-bio/bio/pattern_matching/index.html)).

## Author

[Johannes Köster](https://github.com/johanneskoester)

## Contributors

* [Christopher Schröder](https://github.com/christopher-schroeder)
* [Peer Aramillo Irizar](https://github.com/parir)
* [Fedor Gusev](https://github.com/gusevfe)
* [Vadim Nazarov](https://github.com/vadimnazarov)
* [Brad Chapman](https://github.com/chapmanb)
* [Florian Gilcher](https://github.com/skade)
* [Erik Clarke](https://github.com/eclarke)
* [Rizky Luthfianto](https://github.com/rilut)
* [Adam Perry](https://github.com/dikaiosune)
* [Taylor Cramer](https://github.com/cramertj)
* [Andre Bogus](https://github.com/llogiq)
* Philipp Angerer
* [Pierre Marijon](https://github.com/natir)
* [Franklin Delehelle](https://github.com/delehef)

The next name in this list could be you! If you are interested in joining the effort to build a general purpose Rust bioinformatics library, just introduce yourself [here](https://github.com/rust-bio/rust-bio/issues/27), or issue a pull request with your first contribution.

## License

Licensed under the MIT license http://opensource.org/licenses/MIT. This project may not be copied, modified, or distributed except according to those terms.
