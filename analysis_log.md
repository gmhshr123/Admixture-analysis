
# SNP and Admixture Analysis Log

## Generate Input File in PLINK

First, perform LD pruning using PLINK with a 50kb window, a 10 step, and an r2 threshold of 0.1:

```bash
plink --vcf ../../vcf/nigra.only.vcf -double-id --allow-extra-chr --set-missing-var-ids @:# --indep-pairwise 50 10 0.1 --out nigra_only_LDpruned
```

Next, use the pruned SNPs to generate the input files:

```bash
plink --vcf ../../vcf/nigra.only.vcf --double-id --allow-extra-chr --set-missing-var-ids @:# --geno 0.1 --extract nigra_only_LDpruned.prune.in --make-bed --out nigra.snp
```

> `--geno 0.1` will remove loci where 99.9% of genotypes are missing.

This will generate the following output files:
- `nigra.snp.bed`
- `nigra.snp.bim`
- `nigra.snp.fam`

## Modify BIM File for Admixture

ADMIXTURE does not accept chromosome names that are not human chromosomes. Replace the first column of the `.bim` file with `0`:

```bash
awk '{$1=0;print $0}' nigra.snp.bim > nigra.snp.bim.tmp
mv nigra.snp.bim.tmp nigra.snp.bim
```

## Run Admixture with 5-Fold CV

Use the following SLURM script to run ADMIXTURE with cross-validation (K=2 to K=7):

```bash
#!/bin/bash
#SBATCH -J admixture
#SBATCH -N 2
#SBATCH --ntasks-per-node=64
#SBATCH -o log/%x.%j.out
#SBATCH -e log/%x.%j.err
#SBATCH -p nocona

for i in {2..7}
do
  admixture --cv nigra.snp.bed $i > log${i}.out
done
```

## Check CV Errors for Different K

To extract and check the CV errors for each value of K, use the following command:

```bash
awk '{split($1,name,"."); print $1,name[2]}' nigra.snp.nosex > nigra.population.list
```

## Transfer Files to Local Machine

Transfer the generated `.Q` files and the `nigra.population.list` file to your local machine for further analysis.

## Further Analysis in R

Use the R script in the directory for additional analysis.

## Upload to StructuRly

Generate a CSV file from the analysis and upload it to the [StructuRly](https://nicocriscuolo.shinyapps.io/StructuRly/) web app for visualization.
