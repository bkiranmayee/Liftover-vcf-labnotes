# Liftover-vcf-comparison-labnotes
This is my notes for the vcf liftover comparisons. I am comparing the liftover from the ARS.UCD.V14 to UMD3 and ARS.UCD.V14 to ARS.UCD.V1.2

I am comparing the liftover failed sites. How much do they overlap?

I need to combine all the liftover rejected vcf files and run a comparison...

Working directory: /mnt/nfs/nfs1/derek.bickhart/CDDR-Project/vcfs/condensed_vcfs

```bash
[kiranmayee.bakshy@assembler3 condensed_vcfs]$ ls liftover_to_v1.2/rejected/*.vcf.gz > liftover_to_v1.2/rejected/r.vcf.list
[kiranmayee.bakshy@assembler3 condensed_vcfs]$ ls liftover_to_umd3/liftover_rejected/*.vcf.gz > liftover_to_umd3/liftover_rejected/r.vcf.list
```

I manually edited the vcf.list files to contain only chromosomes 1 to 29 and X. I am ignoring the alternate haplotypes and the unplaced scaffolds for time being.

```bash
[kiranmayee.bakshy@assembler3 condensed_vcfs]$ bcftools concat -a -Oz -f liftover_to_v1.2/rejected/r.vcf.list -o liftover_to_v1.2/rejected/arsucdv1.2_combined.r.vcf.gz 
[kiranmayee.bakshy@assembler3 condensed_vcfs]$ bcftools concat -a -Oz -f liftover_to_umd3/liftover_rejected/r.vcf.list -o liftover_to_umd3/liftover_rejected/umd3_combined.r.vcf.gz

cat <(zcat liftover_to_v1.2/rejected/arsucdv1.2_combined.r.vcf.gz | grep -B10000 -m1 ^#CHROM) <(bcftools view -H liftover_to_v1.2/rejected/arsucdv1.2_combined.r.vcf.gz | sort -k1V,1 -k2n,2) | gzip > liftover_to_v1.2/rejected/arsucdv1.2_combined.r.sorted.vcf.gz 
gunzip liftover_to_v1.2/rejected/arsucdv1.2_combined.r.sorted.vcf.gz 
bgzip liftover_to_v1.2/rejected/arsucdv1.2_combined.r.sorted.vcf
tabix -p vcf liftover_to_v1.2/rejected/arsucdv1.2_combined.r.sorted.vcf.gz

cat <(zcat liftover_to_umd3/liftover_rejected/umd3_combined.r.vcf.gz | grep -B10000 -m1 ^#CHROM) <(bcftools view -H liftover_to_umd3/liftover_rejected/umd3_combined.r.vcf.gz | sort -k1V,1 -k2n,2) | gzip > liftover_to_umd3/liftover_rejected/umd3_combined.r.sorted.vcf.gz 
gunzip liftover_to_umd3/liftover_rejected/umd3_combined.r.sorted.vcf.gz 
bgzip liftover_to_umd3/liftover_rejected/umd3_combined.r.sorted.vcf
tabix -p vcf liftover_to_umd3/liftover_rejected/umd3_combined.r.sorted.vcf.gz


```

