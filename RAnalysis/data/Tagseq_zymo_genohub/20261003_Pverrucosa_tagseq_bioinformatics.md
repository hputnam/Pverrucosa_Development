#0. Setting up HPC folder for analysis

```
#set the main folder name
cd /work/pi_hputnam_uri_edu/Pverr_larvae
#make folders for analysis
mkdir scripts
mkdir qc
mkdir trimmed
mkdir aligned
mkdir Pverr_Genome

```

# Pocillopora verrucosa
#Reference Genome - https://academic.oup.com/gbe/article/12/10/1911/5898631

```
mkdir Pverr_Genome
cd Pverr_Genome/

#Genome scaffolds	fb4d03ba2a9016fabb284d10e513f873
wget http://pver.reefgenomics.org/download/Pver_genome_assembly_v1.0.fasta.gz

#Gene models (CDS)	019359071e3ab319cd10e8f08715fc71
wget http://pver.reefgenomics.org/download/Pver_genes_names_v1.0.fna.gz

#Gene models (proteins)	438f1d59b060144961d6a499de016f55
wget http://pver.reefgenomics.org/download/Pver_proteins_names_v1.0.faa.gz

#Gene models (GFF3)	614efffa87f6e8098b78490a5804c857
wget http://pver.reefgenomics.org/download/Pver_genome_assembly_v1.0.gff3.gz

#Full transcripts	76b5d8d405798d5ca7f6f8cc4b740eb2
wget http://pver.reefgenomics.org/download/Pver_transcriptome_v1.0.fasta.gz

md5sum *.gz > URI.Pverr_Genome.download.md5


019359071e3ab319cd10e8f08715fc71  Pver_genes_names_v1.0.fna.gz
fb4d03ba2a9016fabb284d10e513f873  Pver_genome_assembly_v1.0.fasta.gz
614efffa87f6e8098b78490a5804c857  Pver_genome_assembly_v1.0.gff3.gz
438f1d59b060144961d6a499de016f55  Pver_proteins_names_v1.0.faa.gz
76b5d8d405798d5ca7f6f8cc4b740eb2  Pver_transcriptome_v1.0.fasta.gz
```



# prep Genome gff3 structural annotation file



```
nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/fix_annotation.sh
```


```
#!/bin/bash
#SBATCH --job-name=Pverr_Larvae_Tagseq_fix_annotation
#SBATCH --nodes=1 --cpus-per-task=8
#SBATCH --mem=50G  # Requested Memory
#SBATCH -p gpu  # Partition
#SBATCH -G 1  # Number of GPUs
#SBATCH --time=12:00:00  # Job time limit
#SBATCH -o slurm-%j.out  # %j = job ID
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae

#load modules
module load python/3.12.3
module load gffread/0.12.7 

gunzip /work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0.gff3 


python /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/fix_annotation.py /work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0.gff3

### Sanity check on the modified gff3
wc -l /work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0.gff3
wc -l /work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0_modified.gff3

grep -c "Fixed misplaced attributes" /work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/annotation_repair.log
grep -c $'\tTo:' /work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/annotation_repair.log

gffread \
  /work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0_modified.gff3 \
  -T \
  -o /work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0_modified.gtf

```


```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/fix_annotation.sh
```
Submitted batch job 65205103


Now use this modified file for stringtie assembly
```
/work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0_modified.gff3
```


#1. Quality Control of sequence data

Zymo-Seq SwitchFree 3' mRNA library prep (Cat R3008) and NovaSeq X Plus 2 x 150bp seqeuncing

Raw data location
```
/project/pi_hputnam_uri_edu/raw_sequencing_data/20240424_Pverrucosa_larvae
```

Working data location
```
/work/pi_hputnam_uri_edu/Pverr_larvae
```

## 1.1 Fastqc of raw data

nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/Pverr_Larvae_fastqc.sh

```
#!/bin/bash
#SBATCH --job-name=Pverr_Larvae_Tagseq_fastqc
#SBATCH --nodes=1 --cpus-per-task=8
#SBATCH --mem=50G  # Requested Memory
#SBATCH -p gpu  # Partition
#SBATCH -G 1  # Number of GPUs
#SBATCH --time=12:00:00  # Job time limit
#SBATCH -o slurm-%j.out  # %j = job ID
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae

module load fastqc/0.12.1

for file in /project/pi_hputnam_uri_edu/raw_sequencing_data/20240424_Pverrucosa_larvae/*.gz
do
fastqc $file --outdir /work/pi_hputnam_uri_edu/Pverr_larvae
done


```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/Pverr_Larvae_fastqc.sh

Submitted batch job 65204016

once complete move qc .html and .zip files to qc folder
mv *.html qc
mv *.zip qc


## 1.3 Trimming reads and Fastqc of trimmed data

The intial QC showed that Read1 had high unique reads and Read2 had high amount of duplicated reads. This is from the library prep process where the polyA priming generates reads that are characterized as having redundancy by FastQC, whereas the UMIs on Read1 result in FastQC characterizing them as unique. 

The Zymo library prep kit for the Zymo-Seq SwitchFree 3' mRNA library prep (Cat R3008) recommends to only anlyze Read 2. and using the following approach


```
cutadapt -a A{8}B{6}N{8}AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT -o adapterTrimmed.fastq.gz sample.fastq.gz
cutadapt -a A{100} -o completeTrimmed.fastq.gz adapterTrimmed.fastq.gz
```

nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/trimQC_R2_cutadapt.sh

```
#!/bin/bash
#SBATCH --job-name=Pverr_Larvae_Tagseq_trimQC_R2_cutadapt
#SBATCH --nodes=1 --cpus-per-task=8
#SBATCH --mem=50G  # Requested Memory
#SBATCH -p gpu  # Partition
#SBATCH -G 1  # Number of GPUs
#SBATCH --time=12:00:00  # Job time limit
#SBATCH -o slurm-%j.out  # %j = job ID
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae

# ------------------------------------------------------------
# P. verrucosa larvae TagSeq R2 trimming and FastQC
# ------------------------------------------------------------

#load modules
module load py-cutadapt/4.7
module load fastqc/0.12.1

# Directories
RAW_DIR="/project/pi_hputnam_uri_edu/raw_sequencing_data/20240424_Pverrucosa_larvae"
TRIM_DIR="/work/pi_hputnam_uri_edu/Pverr_larvae/trimmed"
QC_DIR="/work/pi_hputnam_uri_edu/Pverr_larvae/qc"

echo "Starting Cutadapt trimming"
echo "Raw data: ${RAW_DIR}"
echo "Trimmed reads: ${TRIM_DIR}"
echo "FastQC output: ${QC_DIR}"

# ------------------------------------------------------------
# Trim all R2 FASTQ files
# ------------------------------------------------------------

for i in "${RAW_DIR}"/*_R2_001.fastq.gz
do
    # Get filename only
    filename=$(basename "${i}")

    # Keep everything before the first underscore as the sample name
    sample_name="${filename%%_*}"

    echo "------------------------------------------------"
    echo "Processing: ${filename}"
    echo "Sample: ${sample_name}"

    # First trimming step
    cutadapt \
        -a "A{8}B{6}N{8}AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT" \
        -o "${TRIM_DIR}/${sample_name}_adapterTrimmed.fastq.gz" \
        "${i}"

    # Second trimming step
    cutadapt \
        -a "A{100}" \
        -m 40 \
        -o "${TRIM_DIR}/${sample_name}_completeTrimmed.fastq.gz" \
        "${TRIM_DIR}/${sample_name}_adapterTrimmed.fastq.gz"

done

# ------------------------------------------------------------
# FastQC
# ------------------------------------------------------------

echo "Running FastQC"

fastqc \
    --threads 8 \
    --outdir "${QC_DIR}" \
    "${TRIM_DIR}"/*adapterTrimmed.fastq.gz \
    "${TRIM_DIR}"/*completeTrimmed.fastq.gz

echo "------------------------------------------------"
echo "Trimming and FastQC complete."
echo "Trimmed reads: ${TRIM_DIR}"
echo "FastQC reports: ${QC_DIR}"


```


```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/trimQC_R2_cutadapt.sh
```

Submitted batch job 65213479


#3. Aligning Read2 data with STAR

## 3.1 STAR genome index


```
nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/STAR_index.sh
```
```
#!/bin/bash
#SBATCH --job-name=Pverr_Larvae_Tagseq_STAR_index
#SBATCH --nodes=1 --cpus-per-task=8
#SBATCH --mem=50G  # Requested Memory
#SBATCH -p gpu  # Partition
#SBATCH -G 1  # Number of GPUs
#SBATCH --time=12:00:00  # Job time limit
#SBATCH -o slurm-%j.out  # %j = job ID
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae


# ------------------------------------------------------------
# Directories
# ------------------------------------------------------------

GENOME_DIR="/work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome"
TRIM_DIR="/work/pi_hputnam_uri_edu/Pverr_larvae/trimmed"
OUT_DIR="/work/pi_hputnam_uri_edu/Pverr_larvae/aligned"

FASTA="${GENOME_DIR}/Pver_genome_assembly_v1.0.fasta"
GTF="${GENOME_DIR}/Pver_genome_assembly_v1.0_modified.gtf"


# ------------------------------------------------------------
# Load STAR
# ------------------------------------------------------------

module load star/2.7.11a

# ------------------------------------------------------------
# Generate STAR genome index
# ------------------------------------------------------------

STAR \
    --runMode genomeGenerate \
    --runThreadN 8 \
    --genomeDir "${GENOME_DIR}" \
    --genomeFastaFiles "${FASTA}" \
    --sjdbGTFfile "${GTF}" \
    --sjdbOverhang 149 \
    --genomeSAindexNbases 13
```
```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/STAR_index.sh
```

Submitted batch job 65212732


## 3.2 STAR align

```
nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/STAR_align.sh
```
```
#!/bin/bash
#SBATCH --job-name=Pverr_Larvae_Tagseq_STAR_align
#SBATCH --nodes=1 --cpus-per-task=8
#SBATCH --mem=50G  # Requested Memory
#SBATCH -p gpu  # Partition
#SBATCH -G 1  # Number of GPUs
#SBATCH --time=12:00:00  # Job time limit
#SBATCH -o slurm-%j.out  # %j = job ID
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae

# ------------------------------------------------------------
# Directories
# ------------------------------------------------------------

GENOME_DIR="/work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome"
TRIM_DIR="/work/pi_hputnam_uri_edu/Pverr_larvae/trimmed"
OUT_DIR="/work/pi_hputnam_uri_edu/Pverr_larvae/aligned"


# ------------------------------------------------------------
# Load modules
# ------------------------------------------------------------

module load conda/latest
conda create -n star_env -c conda-forge -c bioconda star
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate star_env

echo "STAR version:"
STAR --version

# ------------------------------------------------------------
# Align complete-trimmed R2 reads
# ------------------------------------------------------------

for R2_file in "${TRIM_DIR}"/*_completeTrimmed.fastq.gz
do

    # Extract sample name
    filename=$(basename "${R2_file}")
    sample_name="${filename%_completeTrimmed.fastq.gz}"

    echo "------------------------------------------------"
    echo "Aligning: ${sample_name}"
    echo "Input: ${R2_file}"

    STAR \
        --runMode alignReads \
        --genomeDir "${GENOME_DIR}" \
        --runThreadN 8 \
        --readFilesCommand zcat \
        --readFilesIn "${R2_file}" \
        --outSAMtype BAM SortedByCoordinate \
        --outSAMunmapped Within \
        --outSAMattributes Standard \
        --outFileNamePrefix "${OUT_DIR}/${sample_name}_" \
        --quantMode GeneCounts
done

echo "------------------------------------------------"
echo "STAR alignment complete."

```


```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/STAR_align.sh
```

Submitted batch job 65214620


## 3.2 STAR align Qualimap

```
nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/alignment_qc.sh
```
```
#!/bin/bash
#SBATCH --job-name=Pverr_Larvae_Tagseq_alignment_qc
#SBATCH --nodes=1 --cpus-per-task=8
#SBATCH --mem=50G  # Requested Memory
#SBATCH -p gpu  # Partition
#SBATCH -G 1  # Number of GPUs
#SBATCH --time=12:00:00  # Job time limit
#SBATCH -o slurm-%j.out  # %j = job ID
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae


# ------------------------------------------------------------
# Directories
# ------------------------------------------------------------

alignments_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/aligned"
qc_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/qc"
gtf="/work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0_modified.gtf"

# ------------------------------------------------------------
# Load modules
# ------------------------------------------------------------

module load qualimap/2.2.1

# ------------------------------------------------------------
# Run Qualimap RNA-seq QC
# ------------------------------------------------------------

for f in "${alignments_dir}"/*Aligned.sortedByCoord.out.bam
do
    filename=$(basename "${f}")
    sample_name="${filename%_Aligned.sortedByCoord.out.bam}"

    echo "------------------------------------------------"
    echo "Running Qualimap on ${sample_name}"
    echo "BAM: ${f}"

    qualimap rnaseq \
        --java-mem-size=16G \
        -bam "${f}" \
        -gtf "${gtf}" \
        -p strand-specific-forward \
        -outdir "${qc_dir}/qualimap_${sample_name}"

done

echo "------------------------------------------------"
echo "Qualimap RNA-seq QC complete."

```

```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/alignment_qc.sh
```

Submitted batch job 65218171


“Missing chromosome in annotation” = mapping to rRNA from the over represented sequences identified in the fastqc results


### Testing stranded settings with one sample

```
nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/alignment_test.sh
```

```
#!/bin/bash
#SBATCH --job-name=Pverr_106_strand_test
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=50G
#SBATCH -p gpu
#SBATCH -G 1
#SBATCH --time=12:00:00
#SBATCH -o slurm-%j.out
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae

set -euo pipefail

# ------------------------------------------------------------
# P. verrucosa sample 106
# Qualimap strandedness comparison
# ------------------------------------------------------------

# Load modules
module load qualimap/2.2.1

# ------------------------------------------------------------
# Files and directories
# ------------------------------------------------------------

bam="/work/pi_hputnam_uri_edu/Pverr_larvae/aligned/106_Aligned.sortedByCoord.out.bam"

gtf="/work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0_modified.gtf"

qc_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/qc"

# ------------------------------------------------------------
# Non-strand-specific
# ------------------------------------------------------------

echo "------------------------------------------------"
echo "Running sample 106 as non-strand-specific"

qualimap rnaseq \
    --java-mem-size=16G \
    -bam "${bam}" \
    -gtf "${gtf}" \
    -p non-strand-specific \
    -outdir "${qc_dir}/qualimap_106_non_strand_specific"

# ------------------------------------------------------------
# Strand-specific reverse
# ------------------------------------------------------------

echo "------------------------------------------------"
echo "Running sample 106 as strand-specific-reverse"

qualimap rnaseq \
    --java-mem-size=16G \
    -bam "${bam}" \
    -gtf "${gtf}" \
    -p strand-specific-reverse \
    -outdir "${qc_dir}/qualimap_106_strand_specific_reverse"

echo "------------------------------------------------"
echo "Qualimap strandedness comparison complete."


```

```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/alignment_test.sh
```
Submitted batch job 65218233



### testing distance of intergenic alignments from the strand-specific annotated 3′ ends


```
nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/distancefrom3primeend.sh
```

```

#!/bin/bash
#SBATCH --job-name=Pverr_106_3prime_distance
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=50G
#SBATCH -p gpu
#SBATCH -G 1
#SBATCH --time=12:00:00
#SBATCH -o slurm-%j.out
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae

set -euo pipefail

# ============================================================
# P. verrucosa sample 106
#
# Determine whether intergenic TagSeq alignments occur
# immediately downstream of annotated transcript 3' ends.
#
# GTF contains:
#   transcript
#   exon
#   CDS
#
# Library:
#   R2-only
#   forward stranded
#   Qualimap estimated 96% forward / 4% reverse
# ============================================================


module load samtools/1.19.2
module load bedtools2/2.31.1

# ------------------------------------------------------------
# Input
# ------------------------------------------------------------

BAM="/work/pi_hputnam_uri_edu/Pverr_larvae/aligned/106_Aligned.sortedByCoord.out.bam"

GTF="/work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0_modified.gtf"

OUT="/work/pi_hputnam_uri_edu/Pverr_larvae/qc/106_3prime_distance"

# Clear previous incorrect output
rm -rf "${OUT}"
mkdir -p "${OUT}"


# ============================================================
# 1. Build transcript BED
#
# GTF:
#   1-based, inclusive
#
# BED:
#   0-based, half-open
#
# Extract:
#   chromosome
#   start
#   end
#   transcript_id
#   score
#   strand
# ============================================================

awk 'BEGIN{OFS="\t"}
     $3=="transcript" {

         transcript_id="NA"

         if (match($9,/transcript_id "[^"]+"/)) {
             transcript_id=substr($9,RSTART+15,RLENGTH-16)
         }

         print $1,$4-1,$5,transcript_id,".",$7
     }' "${GTF}" \
    | sort -k1,1 -k2,2n \
    > "${OUT}/transcripts.bed"


echo "Transcript models:"
wc -l "${OUT}/transcripts.bed"

echo
echo "First three transcript models:"
head -3 "${OUT}/transcripts.bed"


# ============================================================
# 2. Generate 1-bp transcript 3' ends
#
# + strand:
#
#       transcript --------->
#                           3'
#
#   BED 3' coordinate = end-1 : end
#
#
# - strand:
#
#       <--------- transcript
#       3'
#
#   BED 3' coordinate = start : start+1
# ============================================================

awk 'BEGIN{OFS="\t"}

     $6=="+" {
         print $1,$3-1,$3,$4,$5,$6
     }

     $6=="-" {
         print $1,$2,$2+1,$4,$5,$6
     }

     ' "${OUT}/transcripts.bed" \
    | sort -k1,1 -k2,2n \
    > "${OUT}/transcript_3prime_ends.bed"


echo
echo "3-prime ends:"
wc -l "${OUT}/transcript_3prime_ends.bed"


# ============================================================
# 3. Extract primary mapped alignments
#
# Exclude:
#
#   4     unmapped
#   256   secondary
#   2048  supplementary
#
# Sum = 2308
# ============================================================

samtools view \
    -bh \
    -F 2308 \
    "${BAM}" \
    > "${OUT}/106_primary_mapped.bam"


# ============================================================
# 4. Convert alignments to BED
#
# BED column 6 preserves read strand.
# ============================================================

bedtools bamtobed \
    -i "${OUT}/106_primary_mapped.bam" \
    | sort -k1,1 -k2,2n \
    > "${OUT}/106_reads.bed"


# ============================================================
# 5. Identify reads outside ALL annotated transcript spans
#
# IMPORTANT:
#
# Do NOT use -s here.
#
# We want reads that are genuinely outside the existing
# annotation regardless of transcript strand.
#
# This prevents a read overlapping an antisense transcript
# from being incorrectly called intergenic.
# ============================================================

bedtools intersect \
    -a "${OUT}/106_reads.bed" \
    -b "${OUT}/transcripts.bed" \
    -v \
    > "${OUT}/106_intergenic_reads.bed"


# ============================================================
# 6. Find closest SAME-STRAND transcript 3' end
#
# -s:
#   same strand
#
# We do not rely on the signed distance produced by BEDTools
# for determining upstream/downstream. We calculate direction
# explicitly below.
# ============================================================

bedtools closest \
    -a "${OUT}/106_intergenic_reads.bed" \
    -b "${OUT}/transcript_3prime_ends.bed" \
    -s \
    -d \
    -t first \
    > "${OUT}/106_intergenic_nearest_3prime.tsv"


# ============================================================
# 7. Explicitly calculate distance DOWNSTREAM from 3' end
#
# Columns 1-6:
#   intergenic read
#
# Columns 7-12:
#   nearest transcript 3' end
#
# Column 13:
#   BEDTools unsigned distance
#
#
# + transcript:
#
# transcript -------->
#                    3'        READ -------->
#
# distance = read_start - transcript_end
#
#
# - transcript:
#
# <-------- READ       3'<-------- transcript
#
# distance = transcript_start - read_end
#
#
# d >= 0 means downstream.
# d < 0 means the read is on the upstream side of the
# transcript rather than beyond its 3' end.
# ============================================================

awk 'BEGIN{OFS="\t"}

     {

         read_chr=$1
         read_start=$2
         read_end=$3
         read_name=$4
         read_strand=$6

         end_start=$8
         end_end=$9
         transcript_id=$10
         transcript_strand=$12


         if (transcript_strand == "+") {

             d = read_start - end_end

         } else if (transcript_strand == "-") {

             d = end_start - read_end

         } else {

             next

         }


         print read_chr,
               read_start,
               read_end,
               read_name,
               read_strand,
               transcript_id,
               transcript_strand,
               d
     }

     ' "${OUT}/106_intergenic_nearest_3prime.tsv" \
    > "${OUT}/106_intergenic_3prime_distances.tsv"


# ============================================================
# 8. Keep only reads DOWNSTREAM of transcript 3' ends
# ============================================================

awk '$8 >= 0' \
    "${OUT}/106_intergenic_3prime_distances.tsv" \
    > "${OUT}/106_downstream_3prime_reads.tsv"


# ============================================================
# 9. Bin downstream distances
# ============================================================

awk '

BEGIN {

    b100=0
    b250=0
    b500=0
    b1000=0
    b2000=0
    b5000=0
    over5000=0

    total=0
}

{

    d=$8
    total++

    if (d <= 100)
        b100++

    else if (d <= 250)
        b250++

    else if (d <= 500)
        b500++

    else if (d <= 1000)
        b1000++

    else if (d <= 2000)
        b2000++

    else if (d <= 5000)
        b5000++

    else
        over5000++

}

END {

    print "Distance_bin\tReads\tPercent"

    if (total > 0) {

        printf "0-100\t%d\t%.2f\n",
            b100,100*b100/total

        printf "101-250\t%d\t%.2f\n",
            b250,100*b250/total

        printf "251-500\t%d\t%.2f\n",
            b500,100*b500/total

        printf "501-1000\t%d\t%.2f\n",
            b1000,100*b1000/total

        printf "1001-2000\t%d\t%.2f\n",
            b2000,100*b2000/total

        printf "2001-5000\t%d\t%.2f\n",
            b5000,100*b5000/total

        printf ">5000\t%d\t%.2f\n",
            over5000,100*over5000/total

        printf "TOTAL\t%d\t100.00\n",
            total
    }

}

' "${OUT}/106_downstream_3prime_reads.tsv" \
    > "${OUT}/106_3prime_distance_bins.tsv"


# ============================================================
# 10. Summary
# ============================================================

total_reads=$(wc -l < "${OUT}/106_reads.bed")

intergenic_reads=$(wc -l < \
    "${OUT}/106_intergenic_reads.bed")

downstream_reads=$(wc -l < \
    "${OUT}/106_downstream_3prime_reads.tsv")


echo
echo "============================================================"
echo "Sample 106 3-prime distance analysis"
echo "============================================================"

echo
echo "Primary mapped alignments: ${total_reads}"
echo "Intergenic alignments:     ${intergenic_reads}"
echo "Downstream of 3' end:      ${downstream_reads}"

echo
echo "Percent intergenic:"
awk -v i="${intergenic_reads}" \
    -v t="${total_reads}" \
    'BEGIN {printf "%.2f%%\n",100*i/t}'

echo
echo "Percent of intergenic reads downstream of a same-strand 3-prime end:"
awk -v d="${downstream_reads}" \
    -v i="${intergenic_reads}" \
    'BEGIN {printf "%.2f%%\n",100*d/i}'

echo
echo "Distance distribution:"
echo

column -t "${OUT}/106_3prime_distance_bins.tsv"

echo
echo "Results written to:"
echo "${OUT}"
echo
```

```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/distancefrom3primeend.sh
```

Submitted batch job 65219293

For sample 106, the QC high intergenic percentage is largely explained by abundant rRNA sequences that exist in the genome but aren't represented in the protein-coding/mRNA GTF. That's a library prep efficiency issue, not necessarily a quantification problem. Moving forward with Stringtie quantification in genome informed mode with NO novel variants





# 4. Stringtie quantification

```
nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/stringtie_quant.sh
```

```
#!/bin/bash
#SBATCH --job-name=Pverr_Larvae_Tagseq_stringtie_quant
#SBATCH --nodes=1 --cpus-per-task=8
#SBATCH --mem=50G  # Requested Memory
#SBATCH -p gpu  # Partition
#SBATCH -G 1  # Number of GPUs
#SBATCH --time=12:00:00  # Job time limit
#SBATCH -o slurm-%j.out  # %j = job ID
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae


# ------------------------------------------------------------
# Directories
# ------------------------------------------------------------

alignments_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/aligned"
qc_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/qc"
gtf="/work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0_modified.gtf"

# ------------------------------------------------------------
# Load modules
# ------------------------------------------------------------

module load uri/main StringTie/2.2.1-GCC-11.2.0

# ------------------------------------------------------------
# Paths
# ------------------------------------------------------------

bam_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/aligned"

gtf_path="/work/pi_hputnam_uri_edu/Pverr_larvae/Pverr_Genome/Pver_genome_assembly_v1.0_modified.gtf"

out_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/stringtie_quant"

mkdir -p "${out_dir}"

# ------------------------------------------------------------
# BAM files
# ------------------------------------------------------------

bams=("${bam_dir}"/*_Aligned.sortedByCoord.out.bam)

echo "Found ${#bams[@]} BAM files"

# ------------------------------------------------------------
# StringTie quantification
# ------------------------------------------------------------

for f in "${bams[@]}"; do

    filename=$(basename "${f}")

    sample_name="${filename%_Aligned.sortedByCoord.out.bam}"

    echo
    echo "============================================================"
    echo "Starting StringTie: ${sample_name}"
    echo "BAM: ${f}"
    echo "============================================================"
    echo

    # -p 16 : use 16 CPU threads
    #
    # --fr   : forward-stranded library
    #          appropriate for these R2-only libraries based
    #          on the Qualimap strandedness analysis
    #
    # -e     : estimate abundance only for transcripts in -G;
    #          do not assemble novel transcripts
    #
    # -B     : create Ballgown-compatible output
    #
    # -v     : verbose output
    #
    # -G     : reference GTF
    #
    # -A     : gene-level abundance output
    #
    # -o     : transcript-level GTF output

    stringtie \
        -p 16 \
        --fr \
        -e \
        -B \
        -v \
        -G "${gtf_path}" \
        -A "${out_dir}/${sample_name}.gene_abund.tab" \
        -o "${out_dir}/${sample_name}.gtf" \
        "${f}"

    echo
    echo "Finished StringTie for ${sample_name}: $(date)"
    echo

done

echo
echo "============================================================"
echo "All StringTie quantification complete: $(date)"
echo "============================================================"


```

```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/stringtie_quant.sh
```

Submitted batch job 65219616


# 5. Generate gene counts matrix with PrepDE
[prepDE.py](https://raw.githubusercontent.com/gpertea/stringtie/master/prepDE.py3)
nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/prepDE.py3


```
nano /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/prepDE_matrix.sh
```


```
#!/bin/bash
#SBATCH --job-name=Pverr_Larvae_prepDE
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=10G
#SBATCH --time=01:00:00
#SBATCH -o slurm-%j.out
#SBATCH -D /work/pi_hputnam_uri_edu/Pverr_larvae

set -euo pipefail

# ============================================================
# Compile StringTie gene counts using prepDE.py3
# ============================================================

# ------------------------------------------------------------
# Variables
# ------------------------------------------------------------

species="Pverrucosa"
genome="Pver_genome_assembly_v1.0"

stringtie_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/stringtie_quant"

out_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/counts"

script_dir="/work/pi_hputnam_uri_edu/Pverr_larvae/scripts"

# ------------------------------------------------------------
# Make output directory
# ------------------------------------------------------------

mkdir -p "${out_dir}"

# ------------------------------------------------------------
# Move into StringTie output directory
# ------------------------------------------------------------

cd "${stringtie_dir}"

# ------------------------------------------------------------
# Create prepDE input file
#
# Format:
#
# sample_name    /full/path/to/sample.gtf
#
# Example:
#
# 6      /work/.../stringtie/6.gtf
# 7      /work/.../stringtie/7.gtf
# 106    /work/.../stringtie/106.gtf
# ------------------------------------------------------------

> listGTF.txt

for filename in *.gtf; do

    sample_name=$(basename "${filename}" .gtf)

    printf "%s\t%s/%s\n" \
        "${sample_name}" \
        "${PWD}" \
        "${filename}"

done > listGTF.txt


# ------------------------------------------------------------
# Check input
# ------------------------------------------------------------

echo
echo "============================================================"
echo "prepDE input files"
echo "============================================================"
echo

cat listGTF.txt

echo
echo "Number of samples:"
wc -l < listGTF.txt

echo


# ------------------------------------------------------------
# Compile gene and transcript count matrices
# ------------------------------------------------------------

python "${script_dir}/prepDE.py3" \
    -g "${out_dir}/${species}_${genome}_gene_count_matrix.csv" \
    -t "${out_dir}/${species}_${genome}_transcript_count_matrix.csv" \
    -i listGTF.txt


# ------------------------------------------------------------
# Report output
# ------------------------------------------------------------

echo
echo "============================================================"
echo "Gene count matrix compiled: $(date)"
echo "============================================================"
echo

echo "Gene counts:"
echo "${out_dir}/${species}_${genome}_gene_count_matrix.csv"

echo
echo "Transcript counts:"
echo "${out_dir}/${species}_${genome}_transcript_count_matrix.csv"

echo
echo "Dimensions:"
echo -n "Gene matrix: "
awk -F',' 'END {print NR-1 " genes x " NF-1 " samples"}' \
    "${out_dir}/${species}_${genome}_gene_count_matrix.csv"

echo -n "Transcript matrix: "
awk -F',' 'END {print NR-1 " transcripts x " NF-1 " samples"}' \
    "${out_dir}/${species}_${genome}_transcript_count_matrix.csv"
```


```
sbatch /work/pi_hputnam_uri_edu/Pverr_larvae/scripts/prepDE_matrix.sh
```

Submitted batch job 65220042
