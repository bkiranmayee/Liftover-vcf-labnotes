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







