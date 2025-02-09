#!/bin/bash

# 환경 변수 설정
source ~/.bashrc


BAM="/mnt/e/CSW/GALC"
REF_PATH="/mnt/e/CSW/reference/hg38_chromosome/chr14.fa"
DICT_PATH="/mnt/e/CSW/reference/hg38_chromosome/chr14.dict"
VCF_FILE="/mnt/e/CSW/reference/hg38_00-All.vcf.gz" #hg38_known_site.vcf.gz
GATK_JAR="/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar"
PICARD_JAR="/home/rare1/gene/SNUM/picard-2.25.0/picard.jar"
THREADS=10


if [ ! -f $DICT_PATH ]; then
    echo "Creating .dict file for reference..."
    java -jar $PICARD_JAR CreateSequenceDictionary R=$REF_PATH O=$DICT_PATH
fi


if [ ! -f $VCF_FILE ]; then
    echo "Error: The VCF file ($VCF_FILE) does not exist. Please provide a valid VCF file."
    exit 1
fi


for BAM_FILE in $BAM/*_region.bam; do
    # 파일명에서 샘플 이름 추출
    BASENAME=$(basename $BAM_FILE _region.bam)
    BAM_FILE_RG="$BAM/${BASENAME}_rg.bam"
    DEDUP_BAM="$BAM/${BASENAME}_dedup.bam"
    RECAL_TABLE="$BAM/${BASENAME}_recal_data.table"
    RECAL_BAM="$BAM/${BASENAME}_recal.bam"
    GVCF_OUTPUT="$BAM/${BASENAME}.g.vcf.gz"
    FINAL_VCF_OUTPUT="$BAM/${BASENAME}_final.vcf.gz"

    # 1. BAM 파일에 Read Group 추가
    echo "Adding Read Group information to $BASENAME..."
    java -jar $PICARD_JAR AddOrReplaceReadGroups \
       I=$BAM_FILE \
       O=$BAM_FILE_RG \
       RGID=$BASENAME \
       RGLB="lib1" \
       RGPL="ILLUMINA" \
       RGPU="unit1" \
       RGSM=$BASENAME

    # 중복 제거
    echo "Running GATK MarkDuplicates for $BASENAME..."
    java -jar $GATK_JAR MarkDuplicates \
        -I $BAM_FILE_RG \
        -O $DEDUP_BAM \
        -M $BAM/${BASENAME}_markdup_metrics.txt \
        --CREATE_INDEX true

    # GATK BaseRecalibrator 
    echo "Running GATK BaseRecalibrator for $BASENAME..."
    java -Xmx16g -Xms8g -jar $GATK_JAR BaseRecalibrator \
        -I $DEDUP_BAM \
        -R $REF_PATH \
        --known-sites $VCF_FILE \
        -O $RECAL_TABLE

    # 재교정 테이블이 생성되었는지 확인
    if [ ! -f $RECAL_TABLE ]; then
        echo "Error: Failed to create recalibration table for $BASENAME. Skipping..."
        continue
    fi

    # GATK ApplyBQSR 
    echo "Applying BQSR for $BASENAME..."
    java -Xmx16g -Xms8g -jar $GATK_JAR ApplyBQSR \
        -I $DEDUP_BAM \
        -R $REF_PATH \
        --bqsr-recal-file $RECAL_TABLE \
        -O $RECAL_BAM

    # GATK HaplotypeCaller 
    echo "Running HaplotypeCaller for $BASENAME..."
    java -Xmx16g -Xms8g -jar $GATK_JAR HaplotypeCaller \
        -R $REF_PATH \
        -I $RECAL_BAM \
        -O $GVCF_OUTPUT \
        -ERC GVCF

    # GATK GenotypeGVCFs 
    echo "Running GenotypeGVCFs for $BASENAME..."
    java -Xmx16g -Xms8g -jar $GATK_JAR GenotypeGVCFs \
        -R $REF_PATH \
        -V $GVCF_OUTPUT \
        -O $FINAL_VCF_OUTPUT

    echo "$BASENAME GATK analysis complete. Final VCF saved to $FINAL_VCF_OUTPUT"
done

echo "All GATK analyses are complete."

을 했는데

(base) rare1@DESKTOP-Q8E0MN3:/mnt/e/CSW/GALC/GATK$ ./GATK_test.sh
Adding Read Group information to G2110067F_GALC...
INFO    2025-02-09 05:05:47     AddOrReplaceReadGroups

********** NOTE: Picard's command line syntax is changing.
**********
********** For more information, please see:
********** https://github.com/broadinstitute/picard/wiki/Command-Line-Syntax-Transition-For-Users-(Pre-Transition)
**********
********** The command line looks like this in the new syntax:
**********
**********    AddOrReplaceReadGroups -I /mnt/e/CSW/GALC/G2110067F_GALC_region.bam -O /mnt/e/CSW/GALC/G2110067F_GALC_rg.bam -RGID G2110067F_GALC -RGLB lib1 -RGPL ILLUMINA -RGPU unit1 -RGSM G2110067F_GALC
**********


05:05:48.338 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/gene/SNUM/picard-2.25.0/picard.jar!/com/intel/gkl/native/libgkl_compression.so
[Sun Feb 09 05:05:48 GMT 2025] AddOrReplaceReadGroups INPUT=/mnt/e/CSW/GALC/G2110067F_GALC_region.bam OUTPUT=/mnt/e/CSW/GALC/G2110067F_GALC_rg.bam RGID=G2110067F_GALC RGLB=lib1 RGPL=ILLUMINA RGPU=unit1 RGSM=G2110067F_GALC    VERBOSITY=INFO QUIET=false VALIDATION_STRINGENCY=STRICT COMPRESSION_LEVEL=5 MAX_RECORDS_IN_RAM=500000 CREATE_INDEX=false CREATE_MD5_FILE=false GA4GH_CLIENT_SECRETS=client_secrets.json USE_JDK_DEFLATER=false USE_JDK_INFLATER=false
[Sun Feb 09 05:05:48 GMT 2025] Executing as rare1@DESKTOP-Q8E0MN3 on Linux 4.4.0-17134-Microsoft amd64; OpenJDK 64-Bit Server VM 22.0.1-internal-adhoc.conda.src; Deflater: Intel; Inflater: Intel; Provider GCS is not available; Picard version: 2.25.0
INFO    2025-02-09 05:05:48     AddOrReplaceReadGroups  Created read-group ID=G2110067F_GALC PL=ILLUMINA LB=lib1 SM=G2110067F_GALC

[Sun Feb 09 05:05:48 GMT 2025] picard.sam.AddOrReplaceReadGroups done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=536870912
Running GATK MarkDuplicates for G2110067F_GALC...
05:05:50.674 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
[Sun Feb 09 05:05:50 GMT 2025] MarkDuplicates --INPUT /mnt/e/CSW/GALC/G2110067F_GALC_rg.bam --OUTPUT /mnt/e/CSW/GALC/G2110067F_GALC_dedup.bam --METRICS_FILE /mnt/e/CSW/GALC/G2110067F_GALC_markdup_metrics.txt --CREATE_INDEX true --MAX_SEQUENCES_FOR_DISK_READ_ENDS_MAP 50000 --MAX_FILE_HANDLES_FOR_READ_ENDS_MAP 8000 --SORTING_COLLECTION_SIZE_RATIO 0.25 --TAG_DUPLICATE_SET_MEMBERS false --REMOVE_SEQUENCING_DUPLICATES false --TAGGING_POLICY DontTag --CLEAR_DT true --DUPLEX_UMI false --ADD_PG_TAG_TO_READS true --REMOVE_DUPLICATES false --ASSUME_SORTED false --DUPLICATE_SCORING_STRATEGY SUM_OF_BASE_QUALITIES --PROGRAM_RECORD_ID MarkDuplicates --PROGRAM_GROUP_NAME MarkDuplicates --READ_NAME_REGEX <optimized capture of last three ':' separated fields as numeric values> --OPTICAL_DUPLICATE_PIXEL_DISTANCE 100 --MAX_OPTICAL_DUPLICATE_SET_SIZE 300000 --VERBOSITY INFO --QUIET false --VALIDATION_STRINGENCY STRICT --COMPRESSION_LEVEL 2 --MAX_RECORDS_IN_RAM 500000 --CREATE_MD5_FILE false --GA4GH_CLIENT_SECRETS client_secrets.json --help false --version false --showHidden false --USE_JDK_DEFLATER false --USE_JDK_INFLATER false
Feb 09, 2025 5:05:50 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
[Sun Feb 09 05:05:50 GMT 2025] Executing as rare1@DESKTOP-Q8E0MN3 on Linux 4.4.0-17134-Microsoft amd64; OpenJDK 64-Bit Server VM 22.0.1-internal-adhoc.conda.src; Deflater: Intel; Inflater: Intel; Provider GCS is available; Picard version: Version:4.2.0.0
INFO    2025-02-09 05:05:50     MarkDuplicates  Start of doWork freeMemory: 83616952; totalMemory: 113246208; maxMemory: 8522825728
INFO    2025-02-09 05:05:50     MarkDuplicates  Reading input file and constructing read end information.
INFO    2025-02-09 05:05:50     MarkDuplicates  Will retain up to 30879803 data points before spilling to disk.
INFO    2025-02-09 05:05:51     MarkDuplicates  Read 14336 records. 186 pairs never matched.
INFO    2025-02-09 05:05:51     MarkDuplicates  After buildSortedReadEndLists freeMemory: 219280128; totalMemory: 503316480; maxMemory: 8522825728
INFO    2025-02-09 05:05:51     MarkDuplicates  Will retain up to 266338304 duplicate indices before spilling to disk.
INFO    2025-02-09 05:05:52     MarkDuplicates  Traversing read pair information and detecting duplicates.
INFO    2025-02-09 05:05:52     MarkDuplicates  Traversing fragment information and detecting duplicates.
INFO    2025-02-09 05:05:52     MarkDuplicates  Sorting list of duplicate records.
INFO    2025-02-09 05:05:52     MarkDuplicates  After generateDuplicateIndexes freeMemory: 1902719672; totalMemory: 4068474880; maxMemory: 8522825728
INFO    2025-02-09 05:05:52     MarkDuplicates  Marking 0 records as duplicates.
INFO    2025-02-09 05:05:52     MarkDuplicates  Found 0 optical duplicate clusters.
INFO    2025-02-09 05:05:52     MarkDuplicates  Reads are assumed to be ordered by: coordinate
INFO    2025-02-09 05:05:53     MarkDuplicates  Writing complete. Closing input iterator.
INFO    2025-02-09 05:05:53     MarkDuplicates  Duplicate Index cleanup.
INFO    2025-02-09 05:05:53     MarkDuplicates  Getting Memory Stats.
INFO    2025-02-09 05:05:53     MarkDuplicates  Before output close freeMemory: 257154504; totalMemory: 293601280; maxMemory: 8522825728
INFO    2025-02-09 05:05:53     MarkDuplicates  Closed outputs. Getting more Memory Stats.
INFO    2025-02-09 05:05:53     MarkDuplicates  After output close freeMemory: 260305808; totalMemory: 293601280; maxMemory: 8522825728
[Sun Feb 09 05:05:53 GMT 2025] picard.sam.markduplicates.MarkDuplicates done. Elapsed time: 0.04 minutes.
Runtime.totalMemory()=293601280
Tool returned:
0
Running GATK BaseRecalibrator for G2110067F_GALC...
05:05:55.222 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:05:55 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:05:55.381 INFO  BaseRecalibrator - ------------------------------------------------------------
05:05:55.382 INFO  BaseRecalibrator - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:05:55.382 INFO  BaseRecalibrator - For support and documentation go to https://software.broadinstitute.org/gatk/
05:05:55.383 INFO  BaseRecalibrator - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:05:55.383 INFO  BaseRecalibrator - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:05:55.384 INFO  BaseRecalibrator - Start Date/Time: February 9, 2025, 5:05:55 AM GMT
05:05:55.384 INFO  BaseRecalibrator - ------------------------------------------------------------
05:05:55.384 INFO  BaseRecalibrator - ------------------------------------------------------------
05:05:55.385 INFO  BaseRecalibrator - HTSJDK Version: 2.24.0
05:05:55.386 INFO  BaseRecalibrator - Picard Version: 2.25.0
05:05:55.386 INFO  BaseRecalibrator - Built for Spark Version: 2.4.5
05:05:55.388 INFO  BaseRecalibrator - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:05:55.389 INFO  BaseRecalibrator - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:05:55.389 INFO  BaseRecalibrator - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:05:55.389 INFO  BaseRecalibrator - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:05:55.390 INFO  BaseRecalibrator - Deflater: IntelDeflater
05:05:55.390 INFO  BaseRecalibrator - Inflater: IntelInflater
05:05:55.391 INFO  BaseRecalibrator - GCS max retries/reopens: 20
05:05:55.392 INFO  BaseRecalibrator - Requester pays: disabled
05:05:55.392 INFO  BaseRecalibrator - Initializing engine
05:05:55.674 INFO  FeatureManager - Using codec VCFCodec to read file file:///mnt/e/CSW/reference/hg38_00-All.vcf.gz
05:05:55.739 WARN  IndexUtils - Feature file "file:///mnt/e/CSW/reference/hg38_00-All.vcf.gz" appears to contain no sequence dictionary. Attempting to retrieve a sequence dictionary from the associated index file
05:05:55.831 INFO  BaseRecalibrator - Done initializing engine
05:05:55.835 INFO  BaseRecalibrationEngine - The covariates being used here:
05:05:55.835 INFO  BaseRecalibrationEngine -    ReadGroupCovariate
05:05:55.836 INFO  BaseRecalibrationEngine -    QualityScoreCovariate
05:05:55.836 INFO  BaseRecalibrationEngine -    ContextCovariate
05:05:55.837 INFO  BaseRecalibrationEngine -    CycleCovariate
05:05:55.841 INFO  ProgressMeter - Starting traversal
05:05:55.842 INFO  ProgressMeter -        Current Locus  Elapsed Minutes       Reads Processed     Reads/Minute
05:05:57.071 INFO  BaseRecalibrator - 15 read(s) filtered by: MappingQualityNotZeroReadFilter
0 read(s) filtered by: MappingQualityAvailableReadFilter
0 read(s) filtered by: MappedReadFilter
14 read(s) filtered by: NotSecondaryAlignmentReadFilter
0 read(s) filtered by: NotDuplicateReadFilter
0 read(s) filtered by: PassesVendorQualityCheckReadFilter
0 read(s) filtered by: WellformedReadFilter
29 total reads filtered
05:05:57.075 INFO  ProgressMeter -       chr14:87992431              0.0                 14307         696204.4
05:05:57.076 INFO  ProgressMeter - Traversal complete. Processed 14307 total reads in 0.0 minutes.
05:05:57.099 INFO  BaseRecalibrator - Calculating quantized quality scores...
05:05:57.107 INFO  BaseRecalibrator - Writing recalibration report...
05:05:58.177 INFO  BaseRecalibrator - ...done!
05:05:58.177 INFO  BaseRecalibrator - BaseRecalibrator was able to recalibrate 14307 reads
05:05:58.178 INFO  BaseRecalibrator - Shutting down engine
[February 9, 2025, 5:05:58 AM GMT] org.broadinstitute.hellbender.tools.walkers.bqsr.BaseRecalibrator done. Elapsed time: 0.05 minutes.
Runtime.totalMemory()=8589934592
Tool returned:
SUCCESS
Applying BQSR for G2110067F_GALC...
05:05:59.885 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:00 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:00.062 INFO  ApplyBQSR - ------------------------------------------------------------
05:06:00.063 INFO  ApplyBQSR - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:00.063 INFO  ApplyBQSR - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:00.064 INFO  ApplyBQSR - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:00.064 INFO  ApplyBQSR - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:00.065 INFO  ApplyBQSR - Start Date/Time: February 9, 2025, 5:05:59 AM GMT
05:06:00.065 INFO  ApplyBQSR - ------------------------------------------------------------
05:06:00.066 INFO  ApplyBQSR - ------------------------------------------------------------
05:06:00.068 INFO  ApplyBQSR - HTSJDK Version: 2.24.0
05:06:00.068 INFO  ApplyBQSR - Picard Version: 2.25.0
05:06:00.068 INFO  ApplyBQSR - Built for Spark Version: 2.4.5
05:06:00.069 INFO  ApplyBQSR - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:00.069 INFO  ApplyBQSR - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:00.069 INFO  ApplyBQSR - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:00.070 INFO  ApplyBQSR - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:00.070 INFO  ApplyBQSR - Deflater: IntelDeflater
05:06:00.071 INFO  ApplyBQSR - Inflater: IntelInflater
05:06:00.071 INFO  ApplyBQSR - GCS max retries/reopens: 20
05:06:00.071 INFO  ApplyBQSR - Requester pays: disabled
05:06:00.072 INFO  ApplyBQSR - Initializing engine
05:06:00.204 INFO  ApplyBQSR - Done initializing engine
05:06:00.240 INFO  ProgressMeter - Starting traversal
05:06:00.241 INFO  ProgressMeter -        Current Locus  Elapsed Minutes       Reads Processed     Reads/Minute
05:06:00.680 INFO  ApplyBQSR - 0 read(s) filtered by: WellformedReadFilter

05:06:00.681 INFO  ProgressMeter -       chr14:87992295              0.0                 14336        1954909.1
05:06:00.682 INFO  ProgressMeter - Traversal complete. Processed 14336 total reads in 0.0 minutes.
05:06:01.082 INFO  ApplyBQSR - Shutting down engine
[February 9, 2025, 5:06:01 AM GMT] org.broadinstitute.hellbender.tools.walkers.bqsr.ApplyBQSR done. Elapsed time: 0.02 minutes.
Runtime.totalMemory()=8589934592
Running HaplotypeCaller for G2110067F_GALC...
05:06:02.854 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:03 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:03.019 INFO  HaplotypeCaller - ------------------------------------------------------------
05:06:03.020 INFO  HaplotypeCaller - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:03.020 INFO  HaplotypeCaller - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:03.021 INFO  HaplotypeCaller - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:03.021 INFO  HaplotypeCaller - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:03.022 INFO  HaplotypeCaller - Start Date/Time: February 9, 2025, 5:06:02 AM GMT
05:06:03.022 INFO  HaplotypeCaller - ------------------------------------------------------------
05:06:03.023 INFO  HaplotypeCaller - ------------------------------------------------------------
05:06:03.024 INFO  HaplotypeCaller - HTSJDK Version: 2.24.0
05:06:03.024 INFO  HaplotypeCaller - Picard Version: 2.25.0
05:06:03.025 INFO  HaplotypeCaller - Built for Spark Version: 2.4.5
05:06:03.027 INFO  HaplotypeCaller - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:03.027 INFO  HaplotypeCaller - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:03.028 INFO  HaplotypeCaller - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:03.028 INFO  HaplotypeCaller - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:03.029 INFO  HaplotypeCaller - Deflater: IntelDeflater
05:06:03.029 INFO  HaplotypeCaller - Inflater: IntelInflater
05:06:03.030 INFO  HaplotypeCaller - GCS max retries/reopens: 20
05:06:03.030 INFO  HaplotypeCaller - Requester pays: disabled
05:06:03.030 INFO  HaplotypeCaller - Initializing engine
05:06:03.210 INFO  HaplotypeCaller - Done initializing engine
05:06:03.212 INFO  HaplotypeCallerEngine - Tool is in reference confidence mode and the annotation, the following changes will be made to any specified annotations: 'StrandBiasBySample' will be enabled. 'ChromosomeCounts', 'FisherStrand', 'StrandOddsRatio' and 'QualByDepth' annotations have been disabled
05:06:03.219 INFO  HaplotypeCallerEngine - Standard Emitting and Calling confidence set to 0.0 for reference-model confidence output
05:06:03.219 INFO  HaplotypeCallerEngine - All sites annotated with PLs forced to true for reference-model confidence output
05:06:03.231 INFO  NativeLibraryLoader - Loading libgkl_utils.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_utils.so
05:06:03.242 INFO  NativeLibraryLoader - Loading libgkl_pairhmm_omp.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_pairhmm_omp.so
05:06:03.274 INFO  IntelPairHmm - Flush-to-zero (FTZ) is enabled when running PairHMM
05:06:03.275 INFO  IntelPairHmm - Available threads: 20
05:06:03.275 INFO  IntelPairHmm - Requested threads: 4
05:06:03.275 INFO  PairHMM - Using the OpenMP multi-threaded AVX-accelerated native PairHMM implementation
05:06:03.341 INFO  ProgressMeter - Starting traversal
05:06:03.341 INFO  ProgressMeter -        Current Locus  Elapsed Minutes     Regions Processed   Regions/Minute
05:06:03.386 INFO  VectorLoglessPairHMM - Time spent in setup for JNI call : 0.0
05:06:03.387 INFO  PairHMM - Total compute time in PairHMM computeLogLikelihoods() : 0.005:06:03.388 INFO  SmithWatermanAligner - Total compute time in java Smith-Waterman : 0.00 sec
05:06:03.388 INFO  HaplotypeCaller - Shutting down engine
[February 9, 2025, 5:06:03 AM GMT] org.broadinstitute.hellbender.tools.walkers.haplotypecaller.HaplotypeCaller done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=8589934592
***********************************************************************

A USER ERROR has occurred: Contig: chr1 not found in reference dictionary.
Please check that you are using a compatible reference for your data.
Reference Contigs: [chr14]

***********************************************************************
Set the system property GATK_STACKTRACE_ON_USER_EXCEPTION (--java-options '-DGATK_STACKTRACE_ON_USER_EXCEPTION=true') to print the stack trace.
Running GenotypeGVCFs for G2110067F_GALC...
05:06:05.145 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:05 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:05.322 INFO  GenotypeGVCFs - ------------------------------------------------------------
05:06:05.322 INFO  GenotypeGVCFs - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:05.323 INFO  GenotypeGVCFs - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:05.324 INFO  GenotypeGVCFs - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:05.324 INFO  GenotypeGVCFs - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:05.325 INFO  GenotypeGVCFs - Start Date/Time: February 9, 2025, 5:06:05 AM GMT
05:06:05.326 INFO  GenotypeGVCFs - ------------------------------------------------------------
05:06:05.326 INFO  GenotypeGVCFs - ------------------------------------------------------------
05:06:05.327 INFO  GenotypeGVCFs - HTSJDK Version: 2.24.0
05:06:05.328 INFO  GenotypeGVCFs - Picard Version: 2.25.0
05:06:05.330 INFO  GenotypeGVCFs - Built for Spark Version: 2.4.5
05:06:05.330 INFO  GenotypeGVCFs - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:05.331 INFO  GenotypeGVCFs - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:05.332 INFO  GenotypeGVCFs - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:05.333 INFO  GenotypeGVCFs - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:05.333 INFO  GenotypeGVCFs - Deflater: IntelDeflater
05:06:05.334 INFO  GenotypeGVCFs - Inflater: IntelInflater
05:06:05.335 INFO  GenotypeGVCFs - GCS max retries/reopens: 20
05:06:05.335 INFO  GenotypeGVCFs - Requester pays: disabled
05:06:05.336 INFO  GenotypeGVCFs - Initializing engine
05:06:05.430 INFO  FeatureManager - Using codec VCFCodec to read file file:///mnt/e/CSW/GALC/G2110067F_GALC.g.vcf.gz
05:06:05.507 INFO  GenotypeGVCFs - Done initializing engine
05:06:05.557 INFO  ProgressMeter - Starting traversal
05:06:05.559 INFO  ProgressMeter -        Current Locus  Elapsed Minutes    Variants Processed  Variants/Minute
05:06:05.565 INFO  ProgressMeter -             unmapped              0.0                     0              0.0
05:06:05.565 INFO  ProgressMeter - Traversal complete. Processed 0 total variants in 0.0 minutes.
05:06:05.570 INFO  GenotypeGVCFs - Shutting down engine
[February 9, 2025, 5:06:05 AM GMT] org.broadinstitute.hellbender.tools.walkers.GenotypeGVCFs done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=8589934592
G2110067F_GALC GATK analysis complete. Final VCF saved to /mnt/e/CSW/GALC/G2110067F_GALC_final.vcf.gz
Adding Read Group information to G2110067M_GALC...
INFO    2025-02-09 05:06:06     AddOrReplaceReadGroups

********** NOTE: Picard's command line syntax is changing.
**********
********** For more information, please see:
********** https://github.com/broadinstitute/picard/wiki/Command-Line-Syntax-Transition-For-Users-(Pre-Transition)
**********
********** The command line looks like this in the new syntax:
**********
**********    AddOrReplaceReadGroups -I /mnt/e/CSW/GALC/G2110067M_GALC_region.bam -O /mnt/e/CSW/GALC/G2110067M_GALC_rg.bam -RGID G2110067M_GALC -RGLB lib1 -RGPL ILLUMINA -RGPU unit1 -RGSM G2110067M_GALC
**********


05:06:06.451 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/gene/SNUM/picard-2.25.0/picard.jar!/com/intel/gkl/native/libgkl_compression.so
[Sun Feb 09 05:06:06 GMT 2025] AddOrReplaceReadGroups INPUT=/mnt/e/CSW/GALC/G2110067M_GALC_region.bam OUTPUT=/mnt/e/CSW/GALC/G2110067M_GALC_rg.bam RGID=G2110067M_GALC RGLB=lib1 RGPL=ILLUMINA RGPU=unit1 RGSM=G2110067M_GALC    VERBOSITY=INFO QUIET=false VALIDATION_STRINGENCY=STRICT COMPRESSION_LEVEL=5 MAX_RECORDS_IN_RAM=500000 CREATE_INDEX=false CREATE_MD5_FILE=false GA4GH_CLIENT_SECRETS=client_secrets.json USE_JDK_DEFLATER=false USE_JDK_INFLATER=false
[Sun Feb 09 05:06:06 GMT 2025] Executing as rare1@DESKTOP-Q8E0MN3 on Linux 4.4.0-17134-Microsoft amd64; OpenJDK 64-Bit Server VM 22.0.1-internal-adhoc.conda.src; Deflater: Intel; Inflater: Intel; Provider GCS is not available; Picard version: 2.25.0
INFO    2025-02-09 05:06:06     AddOrReplaceReadGroups  Created read-group ID=G2110067M_GALC PL=ILLUMINA LB=lib1 SM=G2110067M_GALC

[Sun Feb 09 05:06:06 GMT 2025] picard.sam.AddOrReplaceReadGroups done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=536870912
Running GATK MarkDuplicates for G2110067M_GALC...
05:06:08.557 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
[Sun Feb 09 05:06:08 GMT 2025] MarkDuplicates --INPUT /mnt/e/CSW/GALC/G2110067M_GALC_rg.bam --OUTPUT /mnt/e/CSW/GALC/G2110067M_GALC_dedup.bam --METRICS_FILE /mnt/e/CSW/GALC/G2110067M_GALC_markdup_metrics.txt --CREATE_INDEX true --MAX_SEQUENCES_FOR_DISK_READ_ENDS_MAP 50000 --MAX_FILE_HANDLES_FOR_READ_ENDS_MAP 8000 --SORTING_COLLECTION_SIZE_RATIO 0.25 --TAG_DUPLICATE_SET_MEMBERS false --REMOVE_SEQUENCING_DUPLICATES false --TAGGING_POLICY DontTag --CLEAR_DT true --DUPLEX_UMI false --ADD_PG_TAG_TO_READS true --REMOVE_DUPLICATES false --ASSUME_SORTED false --DUPLICATE_SCORING_STRATEGY SUM_OF_BASE_QUALITIES --PROGRAM_RECORD_ID MarkDuplicates --PROGRAM_GROUP_NAME MarkDuplicates --READ_NAME_REGEX <optimized capture of last three ':' separated fields as numeric values> --OPTICAL_DUPLICATE_PIXEL_DISTANCE 100 --MAX_OPTICAL_DUPLICATE_SET_SIZE 300000 --VERBOSITY INFO --QUIET false --VALIDATION_STRINGENCY STRICT --COMPRESSION_LEVEL 2 --MAX_RECORDS_IN_RAM 500000 --CREATE_MD5_FILE false --GA4GH_CLIENT_SECRETS client_secrets.json --help false --version false --showHidden false --USE_JDK_DEFLATER false --USE_JDK_INFLATER false
Feb 09, 2025 5:06:08 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
[Sun Feb 09 05:06:08 GMT 2025] Executing as rare1@DESKTOP-Q8E0MN3 on Linux 4.4.0-17134-Microsoft amd64; OpenJDK 64-Bit Server VM 22.0.1-internal-adhoc.conda.src; Deflater: Intel; Inflater: Intel; Provider GCS is available; Picard version: Version:4.2.0.0
INFO    2025-02-09 05:06:08     MarkDuplicates  Start of doWork freeMemory: 83758088; totalMemory: 113246208; maxMemory: 8522825728
INFO    2025-02-09 05:06:08     MarkDuplicates  Reading input file and constructing read end information.
INFO    2025-02-09 05:06:08     MarkDuplicates  Will retain up to 30879803 data points before spilling to disk.
INFO    2025-02-09 05:06:09     MarkDuplicates  Read 16735 records. 376 pairs never matched.
INFO    2025-02-09 05:06:09     MarkDuplicates  After buildSortedReadEndLists freeMemory: 235945336; totalMemory: 520093696; maxMemory: 8522825728
INFO    2025-02-09 05:06:09     MarkDuplicates  Will retain up to 266338304 duplicate indices before spilling to disk.
INFO    2025-02-09 05:06:10     MarkDuplicates  Traversing read pair information and detecting duplicates.
INFO    2025-02-09 05:06:10     MarkDuplicates  Traversing fragment information and detecting duplicates.
INFO    2025-02-09 05:06:10     MarkDuplicates  Sorting list of duplicate records.
INFO    2025-02-09 05:06:10     MarkDuplicates  After generateDuplicateIndexes freeMemory: 1919636976; totalMemory: 4085252096; maxMemory: 8522825728
INFO    2025-02-09 05:06:10     MarkDuplicates  Marking 0 records as duplicates.
INFO    2025-02-09 05:06:10     MarkDuplicates  Found 0 optical duplicate clusters.
INFO    2025-02-09 05:06:10     MarkDuplicates  Reads are assumed to be ordered by: coordinate
INFO    2025-02-09 05:06:10     MarkDuplicates  Writing complete. Closing input iterator.
INFO    2025-02-09 05:06:10     MarkDuplicates  Duplicate Index cleanup.
INFO    2025-02-09 05:06:10     MarkDuplicates  Getting Memory Stats.
INFO    2025-02-09 05:06:10     MarkDuplicates  Before output close freeMemory: 259833512; totalMemory: 293601280; maxMemory: 8522825728
INFO    2025-02-09 05:06:11     MarkDuplicates  Closed outputs. Getting more Memory Stats.
INFO    2025-02-09 05:06:11     MarkDuplicates  After output close freeMemory: 260446544; totalMemory: 293601280; maxMemory: 8522825728
[Sun Feb 09 05:06:11 GMT 2025] picard.sam.markduplicates.MarkDuplicates done. Elapsed time: 0.05 minutes.
Runtime.totalMemory()=293601280
Tool returned:
0
Running GATK BaseRecalibrator for G2110067M_GALC...
05:06:13.106 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:13 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:13.278 INFO  BaseRecalibrator - ------------------------------------------------------------
05:06:13.278 INFO  BaseRecalibrator - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:13.279 INFO  BaseRecalibrator - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:13.280 INFO  BaseRecalibrator - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:13.280 INFO  BaseRecalibrator - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:13.281 INFO  BaseRecalibrator - Start Date/Time: February 9, 2025, 5:06:13 AM GMT
05:06:13.282 INFO  BaseRecalibrator - ------------------------------------------------------------
05:06:13.282 INFO  BaseRecalibrator - ------------------------------------------------------------
05:06:13.285 INFO  BaseRecalibrator - HTSJDK Version: 2.24.0
05:06:13.285 INFO  BaseRecalibrator - Picard Version: 2.25.0
05:06:13.286 INFO  BaseRecalibrator - Built for Spark Version: 2.4.5
05:06:13.286 INFO  BaseRecalibrator - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:13.287 INFO  BaseRecalibrator - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:13.287 INFO  BaseRecalibrator - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:13.288 INFO  BaseRecalibrator - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:13.291 INFO  BaseRecalibrator - Deflater: IntelDeflater
05:06:13.292 INFO  BaseRecalibrator - Inflater: IntelInflater
05:06:13.292 INFO  BaseRecalibrator - GCS max retries/reopens: 20
05:06:13.294 INFO  BaseRecalibrator - Requester pays: disabled
05:06:13.295 INFO  BaseRecalibrator - Initializing engine
05:06:13.438 INFO  FeatureManager - Using codec VCFCodec to read file file:///mnt/e/CSW/reference/hg38_00-All.vcf.gz
05:06:13.503 WARN  IndexUtils - Feature file "file:///mnt/e/CSW/reference/hg38_00-All.vcf.gz" appears to contain no sequence dictionary. Attempting to retrieve a sequence dictionary from the associated index file
05:06:13.602 INFO  BaseRecalibrator - Done initializing engine
05:06:13.605 INFO  BaseRecalibrationEngine - The covariates being used here:
05:06:13.606 INFO  BaseRecalibrationEngine -    ReadGroupCovariate
05:06:13.607 INFO  BaseRecalibrationEngine -    QualityScoreCovariate
05:06:13.607 INFO  BaseRecalibrationEngine -    ContextCovariate
05:06:13.608 INFO  BaseRecalibrationEngine -    CycleCovariate
05:06:13.613 INFO  ProgressMeter - Starting traversal
05:06:13.614 INFO  ProgressMeter -        Current Locus  Elapsed Minutes       Reads Processed     Reads/Minute
05:06:14.880 INFO  BaseRecalibrator - 31 read(s) filtered by: MappingQualityNotZeroReadFilter
0 read(s) filtered by: MappingQualityAvailableReadFilter
0 read(s) filtered by: MappedReadFilter
3 read(s) filtered by: NotSecondaryAlignmentReadFilter
0 read(s) filtered by: NotDuplicateReadFilter
0 read(s) filtered by: PassesVendorQualityCheckReadFilter
0 read(s) filtered by: WellformedReadFilter
34 total reads filtered
05:06:14.884 INFO  ProgressMeter -       chr14:87991259              0.0                 16701         789645.4
05:06:14.885 INFO  ProgressMeter - Traversal complete. Processed 16701 total reads in 0.0 minutes.
05:06:14.907 INFO  BaseRecalibrator - Calculating quantized quality scores...
05:06:14.914 INFO  BaseRecalibrator - Writing recalibration report...
05:06:15.694 INFO  BaseRecalibrator - ...done!
05:06:15.695 INFO  BaseRecalibrator - BaseRecalibrator was able to recalibrate 16701 reads
05:06:15.696 INFO  BaseRecalibrator - Shutting down engine
[February 9, 2025, 5:06:15 AM GMT] org.broadinstitute.hellbender.tools.walkers.bqsr.BaseRecalibrator done. Elapsed time: 0.04 minutes.
Runtime.totalMemory()=8589934592
Tool returned:
SUCCESS
Applying BQSR for G2110067M_GALC...
05:06:17.398 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:17 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:17.577 INFO  ApplyBQSR - ------------------------------------------------------------
05:06:17.578 INFO  ApplyBQSR - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:17.579 INFO  ApplyBQSR - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:17.579 INFO  ApplyBQSR - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:17.580 INFO  ApplyBQSR - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:17.581 INFO  ApplyBQSR - Start Date/Time: February 9, 2025, 5:06:17 AM GMT
05:06:17.581 INFO  ApplyBQSR - ------------------------------------------------------------
05:06:17.582 INFO  ApplyBQSR - ------------------------------------------------------------
05:06:17.585 INFO  ApplyBQSR - HTSJDK Version: 2.24.0
05:06:17.585 INFO  ApplyBQSR - Picard Version: 2.25.0
05:06:17.586 INFO  ApplyBQSR - Built for Spark Version: 2.4.5
05:06:17.587 INFO  ApplyBQSR - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:17.587 INFO  ApplyBQSR - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:17.588 INFO  ApplyBQSR - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:17.589 INFO  ApplyBQSR - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:17.591 INFO  ApplyBQSR - Deflater: IntelDeflater
05:06:17.591 INFO  ApplyBQSR - Inflater: IntelInflater
05:06:17.592 INFO  ApplyBQSR - GCS max retries/reopens: 20
05:06:17.594 INFO  ApplyBQSR - Requester pays: disabled
05:06:17.594 INFO  ApplyBQSR - Initializing engine
05:06:17.722 INFO  ApplyBQSR - Done initializing engine
05:06:17.755 INFO  ProgressMeter - Starting traversal
05:06:17.756 INFO  ProgressMeter -        Current Locus  Elapsed Minutes       Reads Processed     Reads/Minute
05:06:18.265 INFO  ApplyBQSR - 0 read(s) filtered by: WellformedReadFilter

05:06:18.266 INFO  ProgressMeter -       chr14:87991167              0.0                 16735        1972691.6
05:06:18.266 INFO  ProgressMeter - Traversal complete. Processed 16735 total reads in 0.0 minutes.
05:06:18.667 INFO  ApplyBQSR - Shutting down engine
[February 9, 2025, 5:06:18 AM GMT] org.broadinstitute.hellbender.tools.walkers.bqsr.ApplyBQSR done. Elapsed time: 0.02 minutes.
Runtime.totalMemory()=8589934592
Running HaplotypeCaller for G2110067M_GALC...
05:06:20.465 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:20 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:20.647 INFO  HaplotypeCaller - ------------------------------------------------------------
05:06:20.647 INFO  HaplotypeCaller - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:20.647 INFO  HaplotypeCaller - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:20.648 INFO  HaplotypeCaller - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:20.649 INFO  HaplotypeCaller - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:20.649 INFO  HaplotypeCaller - Start Date/Time: February 9, 2025, 5:06:20 AM GMT
05:06:20.650 INFO  HaplotypeCaller - ------------------------------------------------------------
05:06:20.650 INFO  HaplotypeCaller - ------------------------------------------------------------
05:06:20.651 INFO  HaplotypeCaller - HTSJDK Version: 2.24.0
05:06:20.653 INFO  HaplotypeCaller - Picard Version: 2.25.0
05:06:20.653 INFO  HaplotypeCaller - Built for Spark Version: 2.4.5
05:06:20.654 INFO  HaplotypeCaller - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:20.654 INFO  HaplotypeCaller - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:20.656 INFO  HaplotypeCaller - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:20.656 INFO  HaplotypeCaller - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:20.657 INFO  HaplotypeCaller - Deflater: IntelDeflater
05:06:20.657 INFO  HaplotypeCaller - Inflater: IntelInflater
05:06:20.658 INFO  HaplotypeCaller - GCS max retries/reopens: 20
05:06:20.659 INFO  HaplotypeCaller - Requester pays: disabled
05:06:20.659 INFO  HaplotypeCaller - Initializing engine
05:06:20.837 INFO  HaplotypeCaller - Done initializing engine
05:06:20.839 INFO  HaplotypeCallerEngine - Tool is in reference confidence mode and the annotation, the following changes will be made to any specified annotations: 'StrandBiasBySample' will be enabled. 'ChromosomeCounts', 'FisherStrand', 'StrandOddsRatio' and 'QualByDepth' annotations have been disabled
05:06:20.846 INFO  HaplotypeCallerEngine - Standard Emitting and Calling confidence set to 0.0 for reference-model confidence output
05:06:20.846 INFO  HaplotypeCallerEngine - All sites annotated with PLs forced to true for reference-model confidence output
05:06:20.856 INFO  NativeLibraryLoader - Loading libgkl_utils.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_utils.so
05:06:20.862 INFO  NativeLibraryLoader - Loading libgkl_pairhmm_omp.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_pairhmm_omp.so
05:06:20.883 INFO  IntelPairHmm - Flush-to-zero (FTZ) is enabled when running PairHMM
05:06:20.884 INFO  IntelPairHmm - Available threads: 20
05:06:20.884 INFO  IntelPairHmm - Requested threads: 4
05:06:20.885 INFO  PairHMM - Using the OpenMP multi-threaded AVX-accelerated native PairHMM implementation
05:06:20.941 INFO  ProgressMeter - Starting traversal
05:06:20.942 INFO  ProgressMeter -        Current Locus  Elapsed Minutes     Regions Processed   Regions/Minute
05:06:20.987 INFO  VectorLoglessPairHMM - Time spent in setup for JNI call : 0.0
05:06:20.988 INFO  PairHMM - Total compute time in PairHMM computeLogLikelihoods() : 0.005:06:20.989 INFO  SmithWatermanAligner - Total compute time in java Smith-Waterman : 0.00 sec
05:06:20.990 INFO  HaplotypeCaller - Shutting down engine
[February 9, 2025, 5:06:20 AM GMT] org.broadinstitute.hellbender.tools.walkers.haplotypecaller.HaplotypeCaller done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=8589934592
***********************************************************************

A USER ERROR has occurred: Contig: chr1 not found in reference dictionary.
Please check that you are using a compatible reference for your data.
Reference Contigs: [chr14]

***********************************************************************
Set the system property GATK_STACKTRACE_ON_USER_EXCEPTION (--java-options '-DGATK_STACKTRACE_ON_USER_EXCEPTION=true') to print the stack trace.
Running GenotypeGVCFs for G2110067M_GALC...
05:06:22.764 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:22 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:22.934 INFO  GenotypeGVCFs - ------------------------------------------------------------
05:06:22.935 INFO  GenotypeGVCFs - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:22.936 INFO  GenotypeGVCFs - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:22.936 INFO  GenotypeGVCFs - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:22.937 INFO  GenotypeGVCFs - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:22.938 INFO  GenotypeGVCFs - Start Date/Time: February 9, 2025, 5:06:22 AM GMT
05:06:22.938 INFO  GenotypeGVCFs - ------------------------------------------------------------
05:06:22.939 INFO  GenotypeGVCFs - ------------------------------------------------------------
05:06:22.940 INFO  GenotypeGVCFs - HTSJDK Version: 2.24.0
05:06:22.942 INFO  GenotypeGVCFs - Picard Version: 2.25.0
05:06:22.943 INFO  GenotypeGVCFs - Built for Spark Version: 2.4.5
05:06:22.943 INFO  GenotypeGVCFs - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:22.944 INFO  GenotypeGVCFs - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:22.945 INFO  GenotypeGVCFs - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:22.945 INFO  GenotypeGVCFs - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:22.946 INFO  GenotypeGVCFs - Deflater: IntelDeflater
05:06:22.946 INFO  GenotypeGVCFs - Inflater: IntelInflater
05:06:22.947 INFO  GenotypeGVCFs - GCS max retries/reopens: 20
05:06:22.948 INFO  GenotypeGVCFs - Requester pays: disabled
05:06:22.948 INFO  GenotypeGVCFs - Initializing engine
05:06:23.038 INFO  FeatureManager - Using codec VCFCodec to read file file:///mnt/e/CSW/GALC/G2110067M_GALC.g.vcf.gz
05:06:23.108 INFO  GenotypeGVCFs - Done initializing engine
05:06:23.161 INFO  ProgressMeter - Starting traversal
05:06:23.162 INFO  ProgressMeter -        Current Locus  Elapsed Minutes    Variants Processed  Variants/Minute
05:06:23.167 INFO  ProgressMeter -             unmapped              0.0                     0              0.0
05:06:23.167 INFO  ProgressMeter - Traversal complete. Processed 0 total variants in 0.0 minutes.
05:06:23.173 INFO  GenotypeGVCFs - Shutting down engine
[February 9, 2025, 5:06:23 AM GMT] org.broadinstitute.hellbender.tools.walkers.GenotypeGVCFs done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=8589934592
G2110067M_GALC GATK analysis complete. Final VCF saved to /mnt/e/CSW/GALC/G2110067M_GALC_final.vcf.gz
Adding Read Group information to G2110067_GALC...
INFO    2025-02-09 05:06:23     AddOrReplaceReadGroups

********** NOTE: Picard's command line syntax is changing.
**********
********** For more information, please see:
********** https://github.com/broadinstitute/picard/wiki/Command-Line-Syntax-Transition-For-Users-(Pre-Transition)
**********
********** The command line looks like this in the new syntax:
**********
**********    AddOrReplaceReadGroups -I /mnt/e/CSW/GALC/G2110067_GALC_region.bam -O /mnt/e/CSW/GALC/G2110067_GALC_rg.bam -RGID G2110067_GALC -RGLB lib1 -RGPL ILLUMINA -RGPU unit1 -RGSM G2110067_GALC
**********


05:06:24.000 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/gene/SNUM/picard-2.25.0/picard.jar!/com/intel/gkl/native/libgkl_compression.so
[Sun Feb 09 05:06:24 GMT 2025] AddOrReplaceReadGroups INPUT=/mnt/e/CSW/GALC/G2110067_GALC_region.bam OUTPUT=/mnt/e/CSW/GALC/G2110067_GALC_rg.bam RGID=G2110067_GALC RGLB=lib1 RGPL=ILLUMINA RGPU=unit1 RGSM=G2110067_GALC    VERBOSITY=INFO QUIET=false VALIDATION_STRINGENCY=STRICT COMPRESSION_LEVEL=5 MAX_RECORDS_IN_RAM=500000 CREATE_INDEX=false CREATE_MD5_FILE=false GA4GH_CLIENT_SECRETS=client_secrets.json USE_JDK_DEFLATER=false USE_JDK_INFLATER=false
[Sun Feb 09 05:06:24 GMT 2025] Executing as rare1@DESKTOP-Q8E0MN3 on Linux 4.4.0-17134-Microsoft amd64; OpenJDK 64-Bit Server VM 22.0.1-internal-adhoc.conda.src; Deflater: Intel; Inflater: Intel; Provider GCS is not available; Picard version: 2.25.0
INFO    2025-02-09 05:06:24     AddOrReplaceReadGroups  Created read-group ID=G2110067_GALC PL=ILLUMINA LB=lib1 SM=G2110067_GALC

[Sun Feb 09 05:06:24 GMT 2025] picard.sam.AddOrReplaceReadGroups done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=536870912
Running GATK MarkDuplicates for G2110067_GALC...
05:06:25.931 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
[Sun Feb 09 05:06:25 GMT 2025] MarkDuplicates --INPUT /mnt/e/CSW/GALC/G2110067_GALC_rg.bam --OUTPUT /mnt/e/CSW/GALC/G2110067_GALC_dedup.bam --METRICS_FILE /mnt/e/CSW/GALC/G2110067_GALC_markdup_metrics.txt --CREATE_INDEX true --MAX_SEQUENCES_FOR_DISK_READ_ENDS_MAP 50000 --MAX_FILE_HANDLES_FOR_READ_ENDS_MAP 8000 --SORTING_COLLECTION_SIZE_RATIO 0.25 --TAG_DUPLICATE_SET_MEMBERS false --REMOVE_SEQUENCING_DUPLICATES false --TAGGING_POLICY DontTag --CLEAR_DT true --DUPLEX_UMI false --ADD_PG_TAG_TO_READS true --REMOVE_DUPLICATES false --ASSUME_SORTED false --DUPLICATE_SCORING_STRATEGY SUM_OF_BASE_QUALITIES --PROGRAM_RECORD_ID MarkDuplicates --PROGRAM_GROUP_NAME MarkDuplicates --READ_NAME_REGEX <optimized capture of last three ':' separated fields as numeric values> --OPTICAL_DUPLICATE_PIXEL_DISTANCE 100 --MAX_OPTICAL_DUPLICATE_SET_SIZE 300000 --VERBOSITY INFO --QUIET false --VALIDATION_STRINGENCY STRICT --COMPRESSION_LEVEL 2 --MAX_RECORDS_IN_RAM 500000 --CREATE_MD5_FILE false --GA4GH_CLIENT_SECRETS client_secrets.json --help false --version false --showHidden false --USE_JDK_DEFLATER false --USE_JDK_INFLATER false
Feb 09, 2025 5:06:26 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
[Sun Feb 09 05:06:26 GMT 2025] Executing as rare1@DESKTOP-Q8E0MN3 on Linux 4.4.0-17134-Microsoft amd64; OpenJDK 64-Bit Server VM 22.0.1-internal-adhoc.conda.src; Deflater: Intel; Inflater: Intel; Provider GCS is available; Picard version: Version:4.2.0.0
INFO    2025-02-09 05:06:26     MarkDuplicates  Start of doWork freeMemory: 83779648; totalMemory: 113246208; maxMemory: 8522825728
INFO    2025-02-09 05:06:26     MarkDuplicates  Reading input file and constructing read end information.
INFO    2025-02-09 05:06:26     MarkDuplicates  Will retain up to 30879803 data points before spilling to disk.
INFO    2025-02-09 05:06:26     MarkDuplicates  Read 14411 records. 240 pairs never matched.
INFO    2025-02-09 05:06:26     MarkDuplicates  After buildSortedReadEndLists freeMemory: 227823120; totalMemory: 511705088; maxMemory: 8522825728
INFO    2025-02-09 05:06:26     MarkDuplicates  Will retain up to 266338304 duplicate indices before spilling to disk.
INFO    2025-02-09 05:06:27     MarkDuplicates  Traversing read pair information and detecting duplicates.
INFO    2025-02-09 05:06:27     MarkDuplicates  Traversing fragment information and detecting duplicates.
INFO    2025-02-09 05:06:27     MarkDuplicates  Sorting list of duplicate records.
INFO    2025-02-09 05:06:27     MarkDuplicates  After generateDuplicateIndexes freeMemory: 1911269608; totalMemory: 4076863488; maxMemory: 8522825728
INFO    2025-02-09 05:06:27     MarkDuplicates  Marking 0 records as duplicates.
INFO    2025-02-09 05:06:27     MarkDuplicates  Found 0 optical duplicate clusters.
INFO    2025-02-09 05:06:28     MarkDuplicates  Reads are assumed to be ordered by: coordinate
INFO    2025-02-09 05:06:28     MarkDuplicates  Writing complete. Closing input iterator.
INFO    2025-02-09 05:06:28     MarkDuplicates  Duplicate Index cleanup.
INFO    2025-02-09 05:06:28     MarkDuplicates  Getting Memory Stats.
INFO    2025-02-09 05:06:28     MarkDuplicates  Before output close freeMemory: 258266888; totalMemory: 293601280; maxMemory: 8522825728
INFO    2025-02-09 05:06:28     MarkDuplicates  Closed outputs. Getting more Memory Stats.
INFO    2025-02-09 05:06:28     MarkDuplicates  After output close freeMemory: 260467640; totalMemory: 293601280; maxMemory: 8522825728
[Sun Feb 09 05:06:28 GMT 2025] picard.sam.markduplicates.MarkDuplicates done. Elapsed time: 0.04 minutes.
Runtime.totalMemory()=293601280
Tool returned:
0
Running GATK BaseRecalibrator for G2110067_GALC...
05:06:30.244 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:30 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:30.426 INFO  BaseRecalibrator - ------------------------------------------------------------
05:06:30.426 INFO  BaseRecalibrator - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:30.427 INFO  BaseRecalibrator - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:30.427 INFO  BaseRecalibrator - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:30.428 INFO  BaseRecalibrator - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:30.429 INFO  BaseRecalibrator - Start Date/Time: February 9, 2025, 5:06:30 AM GMT
05:06:30.429 INFO  BaseRecalibrator - ------------------------------------------------------------
05:06:30.430 INFO  BaseRecalibrator - ------------------------------------------------------------
05:06:30.431 INFO  BaseRecalibrator - HTSJDK Version: 2.24.0
05:06:30.433 INFO  BaseRecalibrator - Picard Version: 2.25.0
05:06:30.433 INFO  BaseRecalibrator - Built for Spark Version: 2.4.5
05:06:30.434 INFO  BaseRecalibrator - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:30.435 INFO  BaseRecalibrator - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:30.435 INFO  BaseRecalibrator - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:30.436 INFO  BaseRecalibrator - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:30.437 INFO  BaseRecalibrator - Deflater: IntelDeflater
05:06:30.438 INFO  BaseRecalibrator - Inflater: IntelInflater
05:06:30.439 INFO  BaseRecalibrator - GCS max retries/reopens: 20
05:06:30.439 INFO  BaseRecalibrator - Requester pays: disabled
05:06:30.440 INFO  BaseRecalibrator - Initializing engine
05:06:30.589 INFO  FeatureManager - Using codec VCFCodec to read file file:///mnt/e/CSW/reference/hg38_00-All.vcf.gz
05:06:30.664 WARN  IndexUtils - Feature file "file:///mnt/e/CSW/reference/hg38_00-All.vcf.gz" appears to contain no sequence dictionary. Attempting to retrieve a sequence dictionary from the associated index file
05:06:30.766 INFO  BaseRecalibrator - Done initializing engine
05:06:30.769 INFO  BaseRecalibrationEngine - The covariates being used here:
05:06:30.769 INFO  BaseRecalibrationEngine -    ReadGroupCovariate
05:06:30.770 INFO  BaseRecalibrationEngine -    QualityScoreCovariate
05:06:30.771 INFO  BaseRecalibrationEngine -    ContextCovariate
05:06:30.771 INFO  BaseRecalibrationEngine -    CycleCovariate
05:06:30.775 INFO  ProgressMeter - Starting traversal
05:06:30.776 INFO  ProgressMeter -        Current Locus  Elapsed Minutes       Reads Processed     Reads/Minute
05:06:32.033 INFO  BaseRecalibrator - 21 read(s) filtered by: MappingQualityNotZeroReadFilter
0 read(s) filtered by: MappingQualityAvailableReadFilter
0 read(s) filtered by: MappedReadFilter
2 read(s) filtered by: NotSecondaryAlignmentReadFilter
0 read(s) filtered by: NotDuplicateReadFilter
0 read(s) filtered by: PassesVendorQualityCheckReadFilter
0 read(s) filtered by: WellformedReadFilter
23 total reads filtered
05:06:32.038 INFO  ProgressMeter -       chr14:87992044              0.0                 14388         685142.9
05:06:32.038 INFO  ProgressMeter - Traversal complete. Processed 14388 total reads in 0.0 minutes.
05:06:32.061 INFO  BaseRecalibrator - Calculating quantized quality scores...
05:06:32.070 INFO  BaseRecalibrator - Writing recalibration report...
05:06:33.057 INFO  BaseRecalibrator - ...done!
05:06:33.058 INFO  BaseRecalibrator - BaseRecalibrator was able to recalibrate 14388 reads
05:06:33.058 INFO  BaseRecalibrator - Shutting down engine
[February 9, 2025, 5:06:33 AM GMT] org.broadinstitute.hellbender.tools.walkers.bqsr.BaseRecalibrator done. Elapsed time: 0.05 minutes.
Runtime.totalMemory()=8589934592
Tool returned:
SUCCESS
Applying BQSR for G2110067_GALC...
05:06:34.860 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:35 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:35.067 INFO  ApplyBQSR - ------------------------------------------------------------
05:06:35.073 INFO  ApplyBQSR - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:35.080 INFO  ApplyBQSR - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:35.085 INFO  ApplyBQSR - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:35.087 INFO  ApplyBQSR - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:35.097 INFO  ApplyBQSR - Start Date/Time: February 9, 2025, 5:06:34 AM GMT
05:06:35.100 INFO  ApplyBQSR - ------------------------------------------------------------
05:06:35.100 INFO  ApplyBQSR - ------------------------------------------------------------
05:06:35.113 INFO  ApplyBQSR - HTSJDK Version: 2.24.0
05:06:35.116 INFO  ApplyBQSR - Picard Version: 2.25.0
05:06:35.128 INFO  ApplyBQSR - Built for Spark Version: 2.4.5
05:06:35.128 INFO  ApplyBQSR - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:35.128 INFO  ApplyBQSR - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:35.143 INFO  ApplyBQSR - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:35.144 INFO  ApplyBQSR - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:35.159 INFO  ApplyBQSR - Deflater: IntelDeflater
05:06:35.160 INFO  ApplyBQSR - Inflater: IntelInflater
05:06:35.160 INFO  ApplyBQSR - GCS max retries/reopens: 20
05:06:35.163 INFO  ApplyBQSR - Requester pays: disabled
05:06:35.164 INFO  ApplyBQSR - Initializing engine
05:06:35.292 INFO  ApplyBQSR - Done initializing engine
05:06:35.329 INFO  ProgressMeter - Starting traversal
05:06:35.330 INFO  ProgressMeter -        Current Locus  Elapsed Minutes       Reads Processed     Reads/Minute
05:06:35.789 INFO  ApplyBQSR - 0 read(s) filtered by: WellformedReadFilter

05:06:35.791 INFO  ProgressMeter -       chr14:87991947              0.0                 14411        1883790.8
05:06:35.791 INFO  ProgressMeter - Traversal complete. Processed 14411 total reads in 0.0 minutes.
05:06:36.216 INFO  ApplyBQSR - Shutting down engine
[February 9, 2025, 5:06:36 AM GMT] org.broadinstitute.hellbender.tools.walkers.bqsr.ApplyBQSR done. Elapsed time: 0.02 minutes.
Runtime.totalMemory()=8589934592
Running HaplotypeCaller for G2110067_GALC...
05:06:38.057 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:38 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:38.233 INFO  HaplotypeCaller - ------------------------------------------------------------
05:06:38.234 INFO  HaplotypeCaller - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:38.235 INFO  HaplotypeCaller - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:38.236 INFO  HaplotypeCaller - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:38.237 INFO  HaplotypeCaller - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:38.243 INFO  HaplotypeCaller - Start Date/Time: February 9, 2025, 5:06:38 AM GMT
05:06:38.244 INFO  HaplotypeCaller - ------------------------------------------------------------
05:06:38.245 INFO  HaplotypeCaller - ------------------------------------------------------------
05:06:38.247 INFO  HaplotypeCaller - HTSJDK Version: 2.24.0
05:06:38.247 INFO  HaplotypeCaller - Picard Version: 2.25.0
05:06:38.248 INFO  HaplotypeCaller - Built for Spark Version: 2.4.5
05:06:38.249 INFO  HaplotypeCaller - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:38.251 INFO  HaplotypeCaller - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:38.254 INFO  HaplotypeCaller - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:38.254 INFO  HaplotypeCaller - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:38.255 INFO  HaplotypeCaller - Deflater: IntelDeflater
05:06:38.256 INFO  HaplotypeCaller - Inflater: IntelInflater
05:06:38.257 INFO  HaplotypeCaller - GCS max retries/reopens: 20
05:06:38.257 INFO  HaplotypeCaller - Requester pays: disabled
05:06:38.258 INFO  HaplotypeCaller - Initializing engine
05:06:38.466 INFO  HaplotypeCaller - Done initializing engine
05:06:38.468 INFO  HaplotypeCallerEngine - Tool is in reference confidence mode and the annotation, the following changes will be made to any specified annotations: 'StrandBiasBySample' will be enabled. 'ChromosomeCounts', 'FisherStrand', 'StrandOddsRatio' and 'QualByDepth' annotations have been disabled
05:06:38.475 INFO  HaplotypeCallerEngine - Standard Emitting and Calling confidence set to 0.0 for reference-model confidence output
05:06:38.475 INFO  HaplotypeCallerEngine - All sites annotated with PLs forced to true for reference-model confidence output
05:06:38.484 INFO  NativeLibraryLoader - Loading libgkl_utils.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_utils.so
05:06:38.489 INFO  NativeLibraryLoader - Loading libgkl_pairhmm_omp.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_pairhmm_omp.so
05:06:38.522 INFO  IntelPairHmm - Flush-to-zero (FTZ) is enabled when running PairHMM
05:06:38.523 INFO  IntelPairHmm - Available threads: 20
05:06:38.523 INFO  IntelPairHmm - Requested threads: 4
05:06:38.524 INFO  PairHMM - Using the OpenMP multi-threaded AVX-accelerated native PairHMM implementation
05:06:38.586 INFO  ProgressMeter - Starting traversal
05:06:38.587 INFO  ProgressMeter -        Current Locus  Elapsed Minutes     Regions Processed   Regions/Minute
05:06:38.631 INFO  VectorLoglessPairHMM - Time spent in setup for JNI call : 0.0
05:06:38.631 INFO  PairHMM - Total compute time in PairHMM computeLogLikelihoods() : 0.005:06:38.632 INFO  SmithWatermanAligner - Total compute time in java Smith-Waterman : 0.00 sec
05:06:38.633 INFO  HaplotypeCaller - Shutting down engine
[February 9, 2025, 5:06:38 AM GMT] org.broadinstitute.hellbender.tools.walkers.haplotypecaller.HaplotypeCaller done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=8589934592
***********************************************************************

A USER ERROR has occurred: Contig: chr1 not found in reference dictionary.
Please check that you are using a compatible reference for your data.
Reference Contigs: [chr14]

***********************************************************************
Set the system property GATK_STACKTRACE_ON_USER_EXCEPTION (--java-options '-DGATK_STACKTRACE_ON_USER_EXCEPTION=true') to print the stack trace.
Running GenotypeGVCFs for G2110067_GALC...
05:06:40.425 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/rare1/tool/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
Feb 09, 2025 5:06:40 AM shaded.cloud_nio.com.google.auth.oauth2.ComputeEngineCredentials runningOnComputeEngine
INFO: Failed to detect whether we are running on Google Compute Engine.
05:06:40.606 INFO  GenotypeGVCFs - ------------------------------------------------------------
05:06:40.607 INFO  GenotypeGVCFs - The Genome Analysis Toolkit (GATK) v4.2.0.0
05:06:40.607 INFO  GenotypeGVCFs - For support and documentation go to https://software.broadinstitute.org/gatk/
05:06:40.608 INFO  GenotypeGVCFs - Executing as rare1@DESKTOP-Q8E0MN3 on Linux v4.4.0-17134-Microsoft amd64
05:06:40.608 INFO  GenotypeGVCFs - Java runtime: OpenJDK 64-Bit Server VM v22.0.1-internal-adhoc.conda.src
05:06:40.609 INFO  GenotypeGVCFs - Start Date/Time: February 9, 2025, 5:06:40 AM GMT
05:06:40.609 INFO  GenotypeGVCFs - ------------------------------------------------------------
05:06:40.610 INFO  GenotypeGVCFs - ------------------------------------------------------------
05:06:40.611 INFO  GenotypeGVCFs - HTSJDK Version: 2.24.0
05:06:40.611 INFO  GenotypeGVCFs - Picard Version: 2.25.0
05:06:40.612 INFO  GenotypeGVCFs - Built for Spark Version: 2.4.5
05:06:40.613 INFO  GenotypeGVCFs - HTSJDK Defaults.COMPRESSION_LEVEL : 2
05:06:40.614 INFO  GenotypeGVCFs - HTSJDK Defaults.USE_ASYNC_IO_READ_FOR_SAMTOOLS : false
05:06:40.615 INFO  GenotypeGVCFs - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_SAMTOOLS : true
05:06:40.615 INFO  GenotypeGVCFs - HTSJDK Defaults.USE_ASYNC_IO_WRITE_FOR_TRIBBLE : false
05:06:40.616 INFO  GenotypeGVCFs - Deflater: IntelDeflater
05:06:40.617 INFO  GenotypeGVCFs - Inflater: IntelInflater
05:06:40.617 INFO  GenotypeGVCFs - GCS max retries/reopens: 20
05:06:40.618 INFO  GenotypeGVCFs - Requester pays: disabled
05:06:40.618 INFO  GenotypeGVCFs - Initializing engine
05:06:40.713 INFO  FeatureManager - Using codec VCFCodec to read file file:///mnt/e/CSW/GALC/G2110067_GALC.g.vcf.gz
05:06:40.783 INFO  GenotypeGVCFs - Done initializing engine
05:06:40.837 INFO  ProgressMeter - Starting traversal
05:06:40.838 INFO  ProgressMeter -        Current Locus  Elapsed Minutes    Variants Processed  Variants/Minute
05:06:40.842 INFO  ProgressMeter -             unmapped              0.0                     0              0.0
05:06:40.843 INFO  ProgressMeter - Traversal complete. Processed 0 total variants in 0.0 minutes.
05:06:40.848 INFO  GenotypeGVCFs - Shutting down engine
[February 9, 2025, 5:06:40 AM GMT] org.broadinstitute.hellbender.tools.walkers.GenotypeGVCFs done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=8589934592
G2110067_GALC GATK analysis complete. Final VCF saved to /mnt/e/CSW/GALC/G2110067_GALC_final.vcf.gz
All GATK analyses are complete.

vcf에 아무것도 call된게 없어 
