# Run fastp on Hydra in a loop

### Job Summary

This job will run fastp on Hydra in a loop. User will provide genomic raw reads. Trimmed reads will be outputed to a directory named 'fastp_trimmed'. To run the script, see 'TO RUN THIS JOB' below.

```
#!/bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -pe mthread 6
#$ -q sThC.q
#$ -l mres=6G,h_data=1G,h_vmem=1G
#$ -cwd
#$ -j y
#$ -N fastp_loop
#$ -o fastp_loop.log
#
# ----------------Modules------------------------- #
module load bio/fastp
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS

# Create output directory if it doesn't exist
mkdir -p fastp_trimmed

# Set sample directory path to raw reads
SAMPLEDIR_RAW="path to raw reads"

# Set sample directory path to base project directory
SAMPLEDIR_BASE="path to base project directory"

# Loop over each R1 file matching the pattern
for GETSAMPLENAME in ${SAMPLEDIR_RAW}/*_R1_001.fastq.gz; do
    # Check if the file exists to avoid errors
    if [ ! -f "$GETSAMPLENAME" ]; then
        echo "No R1 file found for processing in $SAMPLEDIR."
        continue
    fi

    SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_001.fastq.gz)

    # Construct R2 file path
    R2_FILE="${SAMPLEDIR_RAW}/${SAMPLENAME}_R2_001.fastq.gz"

    # Check if R2 file exists
    if [ ! -f "$R2_FILE" ]; then
        echo "Warning: R2 file ${R2_FILE} not found for sample ${SAMPLENAME}. Skipping."
        continue
    fi

    # Run fastp with specified parameters
    fastp \
        -i "$SAMPLEDIR_RAW/${SAMPLENAME}_R1_001.fastq.gz" \
        -I "$SAMPLEDIR_RAW/${SAMPLENAME}_R2_001.fastq.gz"  \
        -o "${SAMPLEDIR_BASE}/fastp_trimmed/${SAMPLENAME}_R1_PE_trimmed.fastq.gz" \
        -O "${SAMPLEDIR_BASE}/fastp_trimmed/${SAMPLENAME}_R2_PE_trimmed.fastq.gz" \
        --unpaired1 "${SAMPLEDIR_BASE}/fastp_trimmed/${SAMPLENAME}_R0_SE_trimmed.fastq.gz" \
        --unpaired2 "${SAMPLEDIR_BASE}/fastp_trimmed/${SAMPLENAME}_R0_SE_trimmed.fastq.gz" \
        -h "${SAMPLEDIR_BASE}/fastp_trimmed/${SAMPLENAME}_fastp.html" \
        -j "${SAMPLEDIR_BASE}/fastp_trimmed/${SAMPLENAME}_fastp.json" \
        -l 30 -f 3 -F 3 -t 2 -T 2 \
        --cut_tail --cut_tail_window_size 5 --cut_tail_mean_quality 15 \
        --trim_poly_g \
        --adapter_fasta /scratch/nmnh_lab/macdonaldk/primers/itru_adapters.fa \
        --thread "$NSLOTS"
done
#
echo = `date` job $JOB_NAME done

```
