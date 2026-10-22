# seq-triage-test-data

Small public test data for [seq-triage](https://github.com/MacaulayWinter/seq-triage).
Every file here is a subsample of a public ENA run, cut with the command
shown, so it can be checked or rebuilt from the original.

The seq-triage test samplesheet points at these files **by commit**, so the
test inputs never change underneath the pipeline. Files here are never edited
in place: a new subsample is a new file.

## Files

Both files come from the same bacterial isolate, *Escherichia coli* ECE107
(BioSample [SAMN49834648](https://www.ebi.ac.uk/ena/browser/view/SAMN49834648)),
from study [PRJNA1287288](https://www.ebi.ac.uk/ena/browser/view/PRJNA1287288)
("R10.4.1 WGS of bacterial isolate for accuracy benchmarking", public 2025-07-09).

| File | Source run | Platform | Subsample | Reads | Read N50 | Q30 |
|---|---|---|---|---|---|---|
| `illumina/SRR36800568_120k_R1.fastq.gz` | [SRR36800568](https://www.ebi.ac.uk/ena/browser/view/SRR36800568) | Illumina NovaSeq 6000, paired | first 120,000 pairs | 120,000 | 151 | 87% |
| `illumina/SRR36800568_120k_R2.fastq.gz` | (as above) | | | 120,000 | 151 | 88% |
| `nanopore/SRR34410547_12k.fastq.gz` | [SRR34410547](https://www.ebi.ac.uk/ena/browser/view/SRR34410547) | ONT MinION, R10.4.1 (per the submitter's metadata) | first 12,000 reads | 12,000 | 15,463 | 72% |

### MGI/DNBSEQ and Element (added 2026-10-22)

For seq-triage's MGI/DNBSEQ and Element QC routes (seq-triage CLAUDE.md
D69): one bacterial isolate per platform. The DNBSEQ run is one that
seq-triage's validation round 3 called DNBSEQ at high confidence; the Element
run is a small, paired *S. aureus* isolate run (a species well covered by
Kraken2 databases), new to seq-triage. Read names keep the instrument markers seq-triage
recognises: an MGI flowcell name (`V350117074L4C001R0381353996/1`) and an
Element instrument ID (`AV224802:...`).

| File | Source run | Platform | Subsample | Reads | Read length | Q30 |
|---|---|---|---|---|---|---|
| `dnbseq/SRR31532556_60k_R1.fastq.gz` | [SRR31532556](https://www.ebi.ac.uk/ena/browser/view/SRR31532556) (*Enterococcus faecium*, study [PRJNA1190376](https://www.ebi.ac.uk/ena/browser/view/PRJNA1190376), public 2025-06-23) | BGISEQ-500, paired | first 60,000 pairs | 60,000 | 150 | 95% |
| `dnbseq/SRR31532556_60k_R2.fastq.gz` | (as above) | | | 60,000 | 150 | 92% |
| `element/SRR32132937_60k_R1.fastq.gz` | [SRR32132937](https://www.ebi.ac.uk/ena/browser/view/SRR32132937) (*Staphylococcus aureus*, study [PRJNA1215987](https://www.ebi.ac.uk/ena/browser/view/PRJNA1215987), public 2025-01-29) | Element AVITI, paired | first 60,000 pairs | 60,000 | 151 | 98% |
| `element/SRR32132937_60k_R2.fastq.gz` | (as above) | | | 60,000 | 151 | 98% |

Statistics from `seqkit stats -a` (seqkit 2.13.0).

### Why these files

- **Real, current instruments.** NovaSeq 6000 is the most common Illumina
  instrument today and records quality in four binned values. R10.4.1 is the
  current nanopore chemistry.
- **Big enough to PASS.** Both clear every provisional threshold in
  seq-triage's `assets/metric_guide.tsv`, so the pipeline's test can show a
  PASS verdict, not only WARN and FAIL. A nanopore sample that passes needs
  at least 10,000 reads with a read N50 of 5,000 or more, which is why the
  nanopore file is about 90 MB.
- **Same isolate on both platforms**, for testing hybrid-data handling later.

### Caveats

- These are the **first** N reads of each run in file order, not a random
  sample. The first Illumina reads come from the first tiles imaged, which
  are often slightly lower quality than the run average. That is acceptable
  for testing triage, but they are not a fair estimate of the whole run's quality.
- ENA serves these Illumina reads with some already shortened (mean length
  143 of a 151-cycle run). The subsample keeps them as served.

## How they were made

```bash
# Illumina: same number of lines from the top of both mates keeps pairs in step
for m in 1 2; do
  curl -sS "https://ftp.sra.ebi.ac.uk/vol1/fastq/SRR368/068/SRR36800568/SRR36800568_${m}.fastq.gz" \
    | gunzip -c | head -n 480000 > illumina/SRR36800568_120k_R${m}.fastq
done

# Nanopore
curl -sS "https://ftp.sra.ebi.ac.uk/vol1/fastq/SRR344/047/SRR34410547/SRR34410547_1.fastq.gz" \
  | gunzip -c | head -n 48000 > nanopore/SRR34410547_12k.fastq

# MGI/DNBSEQ and Element (2026-10-22): first 60,000 pairs of each
for m in 1 2; do
  curl -sS "https://ftp.sra.ebi.ac.uk/vol1/fastq/SRR315/056/SRR31532556/SRR31532556_${m}.fastq.gz" \
    | gunzip -c | head -n 240000 > dnbseq/SRR31532556_60k_R${m}.fastq
  curl -sS "https://ftp.sra.ebi.ac.uk/vol1/fastq/SRR321/037/SRR32132937/SRR32132937_${m}.fastq.gz" \
    | gunzip -c | head -n 240000 > element/SRR32132937_60k_R${m}.fastq
done

# Compress without a timestamp, so the same input gives the same .gz bytes
gzip -n illumina/*.fastq nanopore/*.fastq dnbseq/*.fastq element/*.fastq
```

Pairing was checked after cutting: read names in R1 and R2 match line for
line (0 mismatches of 120,000). The same check on the DNBSEQ and Element
pairs: 0 mismatches of 60,000 each.

## Checksums (md5)

Uncompressed FASTQ, which is the stable check: it does not depend on the
gzip version used.

```
430c2fe958b0a1451ab1896428558979  illumina/SRR36800568_120k_R1.fastq
ff185e928a6d016a97911702bb06850c  illumina/SRR36800568_120k_R2.fastq
c8fd38f926193d759989c6fa04e35ca3  nanopore/SRR34410547_12k.fastq
0c3ca4314c5ca884afd52ef1cb8cd40f  dnbseq/SRR31532556_60k_R1.fastq
d3e1e19d73ef2971661f4cb77c046293  dnbseq/SRR31532556_60k_R2.fastq
d70607acc260cae003980135482b0213  element/SRR32132937_60k_R1.fastq
3c47901216a6fcc1ec964aa07adee1e1  element/SRR32132937_60k_R2.fastq
```

Compressed files as stored here:

```
cf2d4386969fdeec7fc8f3876536c18d  illumina/SRR36800568_120k_R1.fastq.gz
2fd609bcf97eecce37f0ee4cd2123843  illumina/SRR36800568_120k_R2.fastq.gz
7c1e12285ab07f105111b896fadf7dbb  nanopore/SRR34410547_12k.fastq.gz
a180497f1ca19cbeb55108c05b880172  dnbseq/SRR31532556_60k_R1.fastq.gz
4cb8839e175626b65ef52b291816cc5a  dnbseq/SRR31532556_60k_R2.fastq.gz
99ffb7b5a1565bf2ff8bbab8bdc93617  element/SRR32132937_60k_R1.fastq.gz
fdf8773a774490d8feccb0e03738bafb  element/SRR32132937_60k_R2.fastq.gz
```

## Data terms

The reads are public data from the International Nucleotide Sequence Database
Collaboration (ENA / NCBI SRA / DDBJ), which makes submitted data available
without restriction on use or redistribution. If you use them, please cite the
original studies (PRJNA1287288, PRJNA1190376, PRJNA1215987) and the run
accessions above. No claim of
ownership is made over the data by this repository.
