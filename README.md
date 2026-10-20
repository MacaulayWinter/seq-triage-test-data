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

# Compress without a timestamp, so the same input gives the same .gz bytes
gzip -n illumina/*.fastq nanopore/*.fastq
```

Pairing was checked after cutting: read names in R1 and R2 match line for
line (0 mismatches of 120,000).

## Checksums (md5)

Uncompressed FASTQ, which is the stable check: it does not depend on the
gzip version used.

```
430c2fe958b0a1451ab1896428558979  illumina/SRR36800568_120k_R1.fastq
ff185e928a6d016a97911702bb06850c  illumina/SRR36800568_120k_R2.fastq
c8fd38f926193d759989c6fa04e35ca3  nanopore/SRR34410547_12k.fastq
```

Compressed files as stored here:

```
cf2d4386969fdeec7fc8f3876536c18d  illumina/SRR36800568_120k_R1.fastq.gz
2fd609bcf97eecce37f0ee4cd2123843  illumina/SRR36800568_120k_R2.fastq.gz
7c1e12285ab07f105111b896fadf7dbb  nanopore/SRR34410547_12k.fastq.gz
```

## Data terms

The reads are public data from the International Nucleotide Sequence Database
Collaboration (ENA / NCBI SRA / DDBJ), which makes submitted data available
without restriction on use or redistribution. If you use them, please cite the
original study (PRJNA1287288) and the run accessions above. No claim of
ownership is made over the data by this repository.
