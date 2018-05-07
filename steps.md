# Liftover-vcf-labnotes
This is my notes for the vcf liftover from the ARS.UCD.V14 to UMD3 

I am using GATK liftover for this project and the documentation I refered to is from https://software.broadinstitute.org/gatk/documentation/tooldocs/4.0.0.0/picard_vcf_LiftoverVcf.php

Working directory: /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover
```bash
mkdir liftover
First testing for chr1 vcf file:
module load  picard/2.9.2  gatk/3.7 java

ex:
java -jar picard.jar LiftoverVcf \ I=input.vcf \ O=lifted_over.vcf \ CHAIN=b37tohg38.chain \ REJECT=rejected_variants.vcf \ R=reference_sequence.fasta

java -jar picard.jar LiftoverVcf \ I=1.vcf.gz \ O=liftover/1.vcf.gz \ CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain \ REJECT=liftover/1.r.vcf.gz \ R=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa

OK, I got an error message: 
Error: Unable to access jarfile picard.jar

This was resolved by using module display command, it shows how the module was setup

[kiranmayee.bakshy@assembler2 condensed_vcfs]$ module display picard/2.9.2
-------------------------------------------------------------------
/usr/share/Modules/modulefiles/picard/2.9.2:

module-whatis    Makes Picard 2.9.2 available
conflict         picard
prepend-path     PICARDHOME /opt/agil_cluster/picard/2.9.2
setenv           PICARD /opt/agil_cluster/picard/2.9.2/picard.jar
-------------------------------------------------------------------
The new environment variable was setup for picard indicated by setenv, it can be invoked as $PICARD on the command line
So here it goes,
java -jar $PICARD LiftoverVcf \ I=1.vcf.gz \ O=liftover/1.vcf.gz \ CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain \ REJECT=liftover/1.r.vcf.gz \ R=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa
ERROR: Unrecognized option:  I
And the entire usage was printed
Yes, of course its the usage problem

create an empty file in the liftover directory for the lifted SNP positions before running the following command or else it throws an error: 1.vcf.gz

[kiranmayee.bakshy@assembler2 condensed_vcfs]$ java -jar $PICARD LiftoverVcf I=1.vcf.gz O=liftover/1.vcf.gz CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain REJECT=liftover/1.r.vcf.gz R=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa

[Fri Apr 27 15:26:48 EDT 2018] picard.vcf.LiftoverVcf INPUT=1.vcf.gz OUTPUT=liftover/1.vcf.gz CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain REJECT=liftover/1.r.vcf.gz REFERENCE_SEQUENCE=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa    WARN_ON_MISSING_CONTIG=false WRITE_ORIGINAL_POSITION=false LIFTOVER_MIN_MATCH=1.0 ALLOW_MISSING_FIELDS_IN_HEADER=false VERBOSITY=INFO QUIET=false VALIDATION_STRINGENCY=STRICT COMPRESSION_LEVEL=5 MAX_RECORDS_IN_RAM=500000 CREATE_INDEX=false CREATE_MD5_FILE=false GA4GH_CLIENT_SECRETS=client_secrets.json
[Fri Apr 27 15:26:48 EDT 2018] Executing as kiranmayee.bakshy@assembler2.agil.barc.ba.ars.usda.gov on Linux 4.8.13-100.fc23.x86_64 amd64; Java HotSpot(TM) 64-Bit Server VM 1.8.0_131-b11; Picard version: 2.9.2-SNAPSHOT
INFO    2018-04-27 15:26:50     LiftoverVcf     Loading up the target reference genome.
INFO    2018-04-27 15:27:14     LiftoverVcf     Lifting variants over and sorting.
INFO    2018-04-27 15:39:04     LiftoverVcf     read     1,000,000 records.  Elapsed time: 00:11:50s.  Time for last 1,000,000:  709s.  Last read position: 1:104,318,397
INFO    2018-04-27 15:42:45     LiftoverVcf     Processed 1495015 variants.
INFO    2018-04-27 15:42:45     LiftoverVcf     1495015 variants failed to liftover.
INFO    2018-04-27 15:42:45     LiftoverVcf     0 variants lifted over but had mismatching reference alleles after lift over.
INFO    2018-04-27 15:42:45     LiftoverVcf     100.0000% of variants were not successfully lifted over and written to the output.
INFO    2018-04-27 15:42:45     LiftoverVcf     Writing out sorted records to final VCF.
[Fri Apr 27 15:42:47 EDT 2018] picard.vcf.LiftoverVcf done. Elapsed time: 15.99 minutes.
```
Liftover failed!

The same was observed with the minimap2 liftover chain file.

Bed coordinates lifted over well but the vcf have not !!!

Should check what's going wrong...

One problem that I can think of is that the reference fasta has chr1 and the vcf file has 1 instead of chr1. Lets try to edit this and then do the liftover...

```bash
[kiranmayee.bakshy@assembler2 condensed_vcfs]$ zcat 1.vcf.gz | awk '{if($0 !~ /^#/) print "chr"$0; else print $0}' | gzip > 1.chr.vcf.gz
[kiranmayee.bakshy@assembler2 condensed_vcfs]$ java -jar $PICARD LiftoverVcf I=1.chr.vcf.gz O=liftover/1.vcf.gz CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain REJECT=liftover/1.r.vcf.gz R=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa
[Tue May 01 12:16:36 EDT 2018] picard.vcf.LiftoverVcf INPUT=1.chr.vcf.gz OUTPUT=liftover/1.vcf.gz CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain REJECT=liftover/1.r.vcf.gz REFERENCE_SEQUENCE=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa    WARN_ON_MISSING_CONTIG=false WRITE_ORIGINAL_POSITION=false LIFTOVER_MIN_MATCH=1.0 ALLOW_MISSING_FIELDS_IN_HEADER=false VERBOSITY=INFO QUIET=false VALIDATION_STRINGENCY=STRICT COMPRESSION_LEVEL=5 MAX_RECORDS_IN_RAM=500000 CREATE_INDEX=false CREATE_MD5_FILE=false GA4GH_CLIENT_SECRETS=client_secrets.json
[Tue May 01 12:16:36 EDT 2018] Executing as kiranmayee.bakshy@assembler2.agil.barc.ba.ars.usda.gov on Linux 4.8.13-100.fc23.x86_64 amd64; Java HotSpot(TM) 64-Bit Server VM 1.8.0_131-b11; Picard version: 2.9.2-SNAPSHOT
INFO    2018-05-01 12:16:37     LiftoverVcf     Loading up the target reference genome.
INFO    2018-05-01 12:16:58     LiftoverVcf     Lifting variants over and sorting.
ERROR   2018-05-01 12:16:58     LiftoverVcf     Encountered a contig, 1 that is not part of the target reference.
[Tue May 01 12:16:58 EDT 2018] picard.vcf.LiftoverVcf done. Elapsed time: 0.37 minutes.
Runtime.totalMemory()=6682050560
To get help, see http://broadinstitute.github.io/picard/index.html#GettingHelp
```
Ok I will try to change the contig names too...

```bash
zcat 1.vcf.gz | awk '{ if($0 !~ /^#/) print "chr"$0; else if(match($0,/(##contig=<ID=)) print "chr"$0; else print $0 }' | gzip > 1.chr.vcf.gz
[kiranmayee.bakshy@assembler2 condensed_vcfs]$ java -jar $PICARD LiftoverVcf I=1.chr.vcf.gz O=liftover/1.vcf.gz CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain REJECT=liftover/1.r.vcf.gz R=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa[Tue May 01 12:16:36 EDT 2018] picard.vcf.LiftoverVcf INPUT=1.chr.vcf.gz OUTPUT=liftover/1.vcf.gz CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain REJECT=liftover/1.r.vcf.gz REFERENCE_SEQUENCE=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa    WARN_ON_MISSING_CONTIG=false WRITE_ORIGINAL_POSITION=false LIFTOVER_MIN_MATCH=1.0 ALLOW_MISSING_FIELDS_IN_HEADER=false VERBOSITY=INFO QUIET=false VALIDATION_STRINGENCY=STRICT COMPRESSION_LEVEL=5 MAX_RECORDS_IN_RAM=500000 CREATE_INDEX=false CREATE_MD5_FILE=false GA4GH_CLIENT_SECRETS=client_secrets.json
[Tue May 01 12:16:36 EDT 2018] Executing as kiranmayee.bakshy@assembler2.agil.barc.ba.ars.usda.gov on Linux 4.8.13-100.fc23.x86_64 amd64; Java HotSpot(TM) 64-Bit Server VM 1.8.0_131-b11; Picard version: 2.9.2-SNAPSHOT
INFO    2018-05-01 12:16:37     LiftoverVcf     Loading up the target reference genome.
INFO    2018-05-01 12:16:58     LiftoverVcf     Lifting variants over and sorting.
ERROR   2018-05-01 12:16:58     LiftoverVcf     Encountered a contig, 1 that is not part of the target reference.
[Tue May 01 12:16:58 EDT 2018] picard.vcf.LiftoverVcf done. Elapsed time: 0.37 minutes.
Runtime.totalMemory()=6682050560
To get help, see http://broadinstitute.github.io/picard/index.html#GettingHelp
```
I get the same error, looks like it is an existing bug in picard. Try using the latest version of picard...

```bash
git clone https://github.com/broadinstitute/picard.git
cd picard/
./gradlew shadowJar

Build successful

[kiranmayee.bakshy@assembler2 condensed_vcfs]$  java -jar /mnt/nfs/nfs1/kiranmayee.bakshy/picard/build/libs/picard.jar LiftoverVcf I=1.chr.vcf.gz O=liftover/1.vcf.gz CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain REJECT=liftover/1.r.vcf.gz R=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa
15:39:51.863 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/mnt/nfs/nfs1/kiranmayee.bakshy/picard/build/libs/picard.jar!/com/intel/gkl/native/libgkl_compression.so
[Tue May 01 15:39:51 EDT 2018] LiftoverVcf INPUT=1.chr.vcf.gz OUTPUT=liftover/1.vcf.gz CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/umd3_psl/combined_chain_blat.R2/combined.ars.v14_to_umd3.liftover.chain REJECT=liftover/1.r.vcf.gz REFERENCE_SEQUENCE=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa    WARN_ON_MISSING_CONTIG=false LOG_FAILED_INTERVALS=true WRITE_ORIGINAL_POSITION=false WRITE_ORIGINAL_ALLELES=false LIFTOVER_MIN_MATCH=1.0 ALLOW_MISSING_FIELDS_IN_HEADER=false RECOVER_SWAPPED_REF_ALT=false TAGS_TO_REVERSE=[AF] TAGS_TO_DROP=[MAX_AF] VERBOSITY=INFO QUIET=false VALIDATION_STRINGENCY=STRICT COMPRESSION_LEVEL=5 MAX_RECORDS_IN_RAM=500000 CREATE_INDEX=false CREATE_MD5_FILE=false GA4GH_CLIENT_SECRETS=client_secrets.json USE_JDK_DEFLATER=false USE_JDK_INFLATER=false
[Tue May 01 15:39:51 EDT 2018] Executing as kiranmayee.bakshy@assembler2.agil.barc.ba.ars.usda.gov on Linux 4.8.13-100.fc23.x86_64 amd64; OpenJDK 64-Bit Server VM 1.8.0_111-b16; Deflater: Intel; Inflater: Intel; Provider GCS is not available; Picard version: 2.18.4-SNAPSHOT
INFO    2018-05-01 15:39:54     LiftoverVcf     Loading up the target reference genome.
INFO    2018-05-01 15:40:12     LiftoverVcf     Lifting variants over and sorting (not yet writing the output file.)
ERROR   2018-05-01 15:40:13     LiftoverVcf     Encountered a contig, 1 that is not part of the target reference.
[Tue May 01 15:40:13 EDT 2018] picard.vcf.LiftoverVcf done. Elapsed time: 0.35 minutes.
Runtime.totalMemory()=6759645184
To get help, see http://broadinstitute.github.io/picard/index.html#GettingHelp
```
Even after running with updated version of picard, there is still the error...
Now I suspect that the chain file is wrong...

```bash
[kiranmayee.bakshy@assembler2 condensed_vcfs]$  java -jar /mnt/nfs/nfs1/kiranmayee.bakshy/picard/build/libs/picard.jar LiftoverVcf I=1.vcf.gz O=liftover/1.vcf.gz CHAIN=/mnt/nfs/nfs2/bickhart-users/cattle_asms/ars_ucd_114_igc/minimap_liftover/r1/umd3_ars.v14_mmap.liftover.chain REJECT=liftover/1.r.vcf.gz R=/mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa
INFO    2018-05-02 12:11:51     LiftoverVcf     Processed 1495015 variants.
INFO    2018-05-02 12:11:51     LiftoverVcf     65006 variants failed to liftover.
INFO    2018-05-02 12:11:51     LiftoverVcf     120890 variants lifted over but had mismatching reference alleles after lift over.
INFO    2018-05-02 12:11:51     LiftoverVcf     12.4344% of variants were not successfully lifted over and written to the output.
INFO    2018-05-02 12:11:51     LiftoverVcf     liftover success by source contig:
INFO    2018-05-02 12:11:51     LiftoverVcf     1: 1309119 / 1495015 (87.5656%)
INFO    2018-05-02 12:11:51     LiftoverVcf     lifted variants by target contig:
INFO    2018-05-02 12:11:51     LiftoverVcf     chr1: 1301143
INFO    2018-05-02 12:11:51     LiftoverVcf     chr10: 97
INFO    2018-05-02 12:11:51     LiftoverVcf     chr11: 323
INFO    2018-05-02 12:11:51     LiftoverVcf     chr12: 432
INFO    2018-05-02 12:11:51     LiftoverVcf     chr13: 1104
INFO    2018-05-02 12:11:51     LiftoverVcf     chr14: 29
INFO    2018-05-02 12:11:51     LiftoverVcf     chr15: 2
INFO    2018-05-02 12:11:51     LiftoverVcf     chr17: 57
INFO    2018-05-02 12:11:51     LiftoverVcf     chr18: 280
INFO    2018-05-02 12:11:51     LiftoverVcf     chr19: 34
INFO    2018-05-02 12:11:51     LiftoverVcf     chr2: 1317
INFO    2018-05-02 12:11:51     LiftoverVcf     chr20: 46
INFO    2018-05-02 12:11:51     LiftoverVcf     chr21: 31
INFO    2018-05-02 12:11:51     LiftoverVcf     chr23: 11
INFO    2018-05-02 12:11:51     LiftoverVcf     chr27: 1107
INFO    2018-05-02 12:11:51     LiftoverVcf     chr3: 1195
INFO    2018-05-02 12:11:51     LiftoverVcf     chr4: 446
INFO    2018-05-02 12:11:51     LiftoverVcf     chr5: 10
INFO    2018-05-02 12:11:51     LiftoverVcf     chr6: 642
INFO    2018-05-02 12:11:51     LiftoverVcf     chr7: 126
INFO    2018-05-02 12:11:51     LiftoverVcf     chr8: 487
INFO    2018-05-02 12:11:51     LiftoverVcf     chr9: 57
INFO    2018-05-02 12:11:51     LiftoverVcf     chrX: 143
WARNING 2018-05-02 12:11:51     LiftoverVcf     111833 variants with a swapped REF/ALT were identified, but were not recovered.  See RECOVER_SWAPPED_REF_ALT and associated caveats.
INFO    2018-05-02 12:12:42     LiftoverVcf     Writing out sorted records to final VCF.
INFO    2018-05-02 12:17:42     LiftoverVcf     written     1,000,000 records.  Elapsed time: 00:04:59s.  Time for last 1,000,000:  299s.  Last read position: chr1:120,384,709
[Wed May 02 12:19:14 EDT 2018] picard.vcf.LiftoverVcf done. Elapsed time: 24.42 minutes.
Runtime.totalMemory()=17024679936
```
The liftover was successful with some warning which has to be checked...

This process was done for all the vcf files using the script liftover_vcf.sh
The slurm job output was saved to slurm-815853.out

These lifted_over files have to be concatenated using bcftools concat

```bash
module load bcftools
bcftools concat -a -Ob -f vcf.list -o combinedvcfs_new
# set up ID column
sbatch --nodes=1 --partition=assemble2 --ntasks-per-node=1 --mem=5000 --wrap="bcftools annotate -I 'bakshy_%CHROM\_%POS\_%REF\_%FIRST_ALT' combinedvcfs.gz > combinedvcfs_id"
gzip combinedvcfs_id
sbatch --nodes=1 --partition=assemble2 --ntasks-per-node=1 --mem=5000 --wrap="bcftools index -t combinedvcfs_id.gz"
# download snpEff database
java -jar snpEff.jar download UMD3.1.86
# Start annotating
sbatch --nodes=1 --partition=assemble2 --ntasks-per-node=1 --mem=20000 --wrap="java -Xmx4g -jar /home/kiranmayee.bakshy/snpEff/snpEff.jar -v -stats ann_stats/combined.html UMD3.1.86 combinedvcfs_id.gz > annotated/combined.ann.vcf"

[kiranmayee.bakshy@assembler2 annotated]$ java -jar ~/snpEff/SnpSift.jar filter "(ANN[*].EFFECT has 'stop_gained')" combined.ann.vcf > combined_st.gn.vcf
[kiranmayee.bakshy@assembler2 annotated]$ module load bcftools
[kiranmayee.bakshy@assembler2 annotated]$ bcftools query -f'%CHROM\t%POS\t%REF\t%ALT\t%INFO/ANN\n' combined_st.gn.vcf > combined_st.gn.txt
[kiranmayee.bakshy@assembler2 annotated]$ module load R

> data<-read.table(file="/mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/ann_stats/combined.genes.txt",sep="\t",header=F,stringsAsFactors=F)
> data_stgn<-data[,c(1,2,3,4,30)]
> colnames(data_stgn)<-c("GeneName","GeneId","TranscriptId","BioType","stop_gain")
> df<-data_stgn[data_stgn$stop_gain > 0,]
> dim(df)
[1] 737   5
> dim(data_stgn)
[1] 26396     5
> head(df)
    GeneName             GeneId       TranscriptId        BioType stop_gain
517   ABCA13 ENSBTAG00000003531 ENSBTAT00000061018 protein_coding         3
523    ABCA6 ENSBTAG00000006921 ENSBTAT00000009089 protein_coding         1
567  ABHD16B ENSBTAG00000031906 ENSBTAT00000045249 protein_coding         1
653   ACOT12 ENSBTAG00000011110 ENSBTAT00000014753 protein_coding         1
682    ACSM3 ENSBTAG00000006447 ENSBTAT00000008455 protein_coding         1
770  ADAMTS7 ENSBTAG00000001396 ENSBTAT00000038814 protein_coding         1

> write.table(df, file="stgn_per_gene.txt", sep="\t", row.names=F,quote=F)
> q()
Save workspace image? [y/n/c]: y
[kiranmayee.bakshy@assembler2 annotated]$ grep "^#CHROM" combined.ann.vcf > sample.list

# I manually edited the sample file to include population id column

[kiranmayee.bakshy@gateway1 ~]$ java -jar /mnt/nfs/nfs2/bickhart-users/binaries/LargeVCFParser/store/LargeVCFParser.jar popstats -p /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/annotated/sample.list -v /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/annotated/combined.ann.vcf -o /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/annotated/combined
[MAIN] Command line arguments supplied:
popstats -p /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/annotated/sample.list -v /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/annotated/combined.ann.vcf -o /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/annotated/combined
[MAIN] Debug flag set to: true
LargeVCFParser  version: 0.0.2
Mode: popstats
Cmd line options: popstats -p /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/annotated/sample.list -v /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/annotated/combined.ann.vcf -o /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/annotated/combined
[POPSTATS] VCF sample count: 172
```
The stats are ready...

Now try to separate the homozygous variants...
```bash
 java -jar ~/snpEff/SnpSift.jar filter "(countHom()=172)" combined.ann.vcf > allHom.vcf
 ```


Generate some stats and plot:
```bash
# Calculating stats
sbatch --nodes=1 --partition=assemble2 --ntasks-per-node=1 --mem=10000 --wrap="bcftools stats --debug -F /mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa --af-bins '(0.05,0.1,0.5,1)' combined.ann.vcf.gz > combined.d.vstats"

mkdir vcfstatplots
plot-vcfstats -r -p vcfstatplots combined.vstats
```

```bash

# sort the vcf file according to chr and position:
# cat <(zcat $vcf | grep -B10000 -m1 ^#CHROM) <(bcftools view -H $vcf | sort -k1V,1 -k2n,2) | bgzip > out.vcf.gz
cat <(zcat combined_st.gn.vcf.gz | grep -B10000 -m1 ^#CHROM) <(bcftools view -H combined_st.gn.vcf.gz | sort -k1V,1 -k2n,2) | gzip > combined_st.gn.sorted.vcf.gz
gunzip combined_st.gn.sorted.vcf.gz
bgzip combined_st.gn.sorted.vcf
module load samtools
tabix -p vcf combined_st.gn.sorted.vcf.gz

module load gatk/3.7
# Calculate the genotype count in a sorted and indexed vcf file using GATK (VariantsToTable):
GenomeAnalysisTK -R /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs/liftover/test_files/umd3_kary_unmask_ngap.fa -T VariantsToTable -V combined_st.gn.sorted.vcf.gz -F CHROM -F POS -F ID -F QUAL -F HET -F HOM-REF -F HOM-VAR -F NO-CALL -F NSAMPLES -o test

# Calculate sample level stats using verbose option 
sbatch --nodes=1 --partition=assemble2 --ntasks-per-node=1 --mem=10000 --wrap="bcftools stats -v -s - -F /mnt/nfs/nfs2/Genomes/umd3_kary_unmask_ngap.fa --af-bins '(0.05,0.1,0.5,1)' combined.ann.vcf.gz > combined.d.vstats"




