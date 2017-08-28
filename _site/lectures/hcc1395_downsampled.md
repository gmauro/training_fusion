# Downloading and preparing HCC1395 sample data

> These instructions are for the .1% downsampled version with spiked in fusion reads

## Environment setup

We will create environment variables pointing to the data we download, and set a sample id for the data.  We will be working with a publically available HCC1395 breast cancer cell line sequenced for teaching and benchmarking purposes.  The dataset is available from washington university servers.

```{.bash}
SAMPLE_ID=HCC1395_downsampled
SAMPLE_ID_FULL=HCC1395
SAMPLE_FASTQ_1=$TUTORIAL_HOME/sampledata/$SAMPLE_ID/SAMPLE_R1.fastq
SAMPLE_FASTQ_2=$TUTORIAL_HOME/sampledata/$SAMPLE_ID/SAMPLE_R2.fastq
SAMPLE_SPIKEIN_FASTQ_1=$TUTORIAL_HOME/sampledata/$SAMPLE_ID/SPIKEIN_R1.fastq
SAMPLE_SPIKEIN_FASTQ_2=$TUTORIAL_HOME/sampledata/$SAMPLE_ID/SPIKEIN_R2.fastq
SAMPLE_DOWNSAMPLE_FASTQ_1=$TUTORIAL_HOME/sampledata/$SAMPLE_ID/gerald_C1TD1ACXX_8_ACAGTG_R1.fastq
SAMPLE_DOWNSAMPLE_FASTQ_2=$TUTORIAL_HOME/sampledata/$SAMPLE_ID/gerald_C1TD1ACXX_8_ACAGTG_R2.fastq
SAMPLE_RAW_BAM=$TUTORIAL_HOME/sampledata/$SAMPLE_ID/gerald_C1TD1ACXX_8_ACAGTG.bam
SAMPLE_URL=https://xfer.genome.wustl.edu/gxfer1/project/gms/testdata/bams/hcc1395_1tenth_percent/gerald_C1TD1ACXX_8_ACAGTG.bam
```

## Download and prepare

We will be working in a sample specific directory.

```{.bash}
mkdir -p $TUTORIAL_HOME/sampledata/$SAMPLE_ID
cd $TUTORIAL_HOME/sampledata/$SAMPLE_ID
```

Use wget to download the sample data.

```{.bash sentinal=$TUTORIAL_HOME/sentinal/sampledata/wget_downsampled}
wget $SAMPLE_URL
```

Convert to paired fastq using picard tools.

```{.bash sentinal=$TUTORIAL_HOME/sentinal/sampledata/samtofastq_downsampled}
java -Xmx2g -jar $PICARD_DIR/picard.jar SamToFastq \
    VALIDATION_STRINGENCY=LENIENT \
    INPUT=$SAMPLE_RAW_BAM \
    FASTQ=$SAMPLE_DOWNSAMPLE_FASTQ_1 \
    SECOND_END_FASTQ=$SAMPLE_DOWNSAMPLE_FASTQ_2
```

Assuming we have run deFuse on the full dataset, we will extract the fastq files
of high quality fusions and spike those into the downsampled data.

```{.bash sentinal=$TUTORIAL_HOME/sentinal/sampledata/createspikein}
python $TUTORIAL_HOME/packages/cbw_tutorial/gene_fusions/scripts/get_defuse_high_quality_ids.py \
    $TUTORIAL_HOME/analysis/defuse/$SAMPLE_ID_FULL/results.classify.tsv \
    $TUTORIAL_HOME/sampledata/$SAMPLE_ID/defuse_cluster_ids.txt

$DEFUSE_SCRIPT_DIR/get_fusion_fastq.pl \
    -o $TUTORIAL_HOME/analysis/defuse/$SAMPLE_ID_FULL/ \
    -l $TUTORIAL_HOME/sampledata/$SAMPLE_ID/defuse_cluster_ids.txt \
    --fastq1 $SAMPLE_SPIKEIN_FASTQ_1 \
    --fastq2 $SAMPLE_SPIKEIN_FASTQ_2
```

```{.bash sentinal=$TUTORIAL_HOME/sentinal/sampledata/catfastqs}
cat $SAMPLE_DOWNSAMPLE_FASTQ_1 $SAMPLE_SPIKEIN_FASTQ_1 > $SAMPLE_FASTQ_1
cat $SAMPLE_DOWNSAMPLE_FASTQ_2 $SAMPLE_SPIKEIN_FASTQ_2 > $SAMPLE_FASTQ_2
```


