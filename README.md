# POKEMON
pokemon is a structure based variance component test for studying rare variants

## Setup
**Highly recommand to use through docker image**  
dockerhub repo: docker.io/bushlab/pokemon:latest
```bash
dir=/home/myhomespace
gene=ENST00000373113
docker run -it --rm -v ${dir}:${dir} -w=/POKEMON docker.io/bushlab/pokemon \
python run_pokemon.py --gene_name ${gene} --genotype ${gene}.raw --phenotype test.pheno --annotation ${gene}.csq \
                     --cov_file test.cov --cov_list APOE4_dose,APOE2_dose --alpha 0 --use_blosum  \
                     --out_file ${dir}/results --figures --out_fig_dir=${dir}  
```

## Dependencies
python > 3.7  

gcc > 6.3.0

Required package:(can be installed via pip)
- fastlmmclib=0.0.1
see the instruction guide here: https://pypi.org/project/fastlmmclib/  
- biopython=1.77
see the instruction here: https://biopython.org/wiki/Download  
- sklearn=0.22.2
see the instruction here: https://scikit-learn.org/stable/install.html  

Optional pacakges if visualization is on:  
-  pymol   
pymol must be installed to use the flag --figures, see instruction here: https://pymol.org/2/  

```bash
git clone https://github.com/bushlab-genomics/POKEMON.git  
cd POKEMON 
```

## Example:
```bash
gene=ENST00000373113
python run_pokemon.py --gene_name ${gene} --genotype ${gene}.raw --phenotype test.pheno --annotation ${gene}.csq \
                     --cov_file test.cov --cov_list APOE4_dose,APOE2_dose --alpha 0 --use_blosum  \
                     --out_file results 
```
## Flags:
**--gene_name**: required  
   A label used as gene name in the output result file 
   
**--genotype**: required  
   plink output with recode A option(The 'transpose' modifier).    
   **note1: snp must be named as chr:pos:alt:ref (e.g., 6:41129275:G:C)**   
   **note2: snps must be unique**  
   
   The columns for genotype file is      
| FID  | IID | PAT | MAT | SEX | PHENOTYPE | 6:41129275:G:C | ... | ... |
| --- | --- | --- | --- |--- | --- | --- | --- | --- |
| ... | ... | ... | ... | ... | ... | ... | ... | ... |   

   A typical command to generate the genotype file:
```bash
bcftools annotate --set-id '%CHROM:%POS:%REF:%FIRST_ALT' <vcf file with genotype>  

plink --vcf <vcf file with genotype> --snps-only  --allow-no-sex --max-maf 0.05 --recode A --threads 4 --out test_gene
   
# change the formart from chr:pos:ref:alt_alt -> chr:pos:ref:alt
sed -i 's/_[A-Z]//g' test_gene.raw
```

**--phenotype**: required  
   phenotype file  
   **note: phenotypes must be unique**  
   
   format1 for testing with single phenotype: column1 for individuals, column2 for phenotype, seperated by space. 
| ID  | pheno |
| --- | ---- |
| name1  | 0  |
| name2  | 1  | 
| ...  | ...  |
| name1000  | 0  |  

   format2 for testing with multiple phenotypes: row for phenotypes, columns for individuals, seperated by space.   
| pheno  | name1 | name2 | ... | name1000 |
| --- | --- |--- |--- |---|
| pheno1  | 1  | 0 | ... | 1 |
| pheno2  | 1  | 0 | ... | 1 |
| ...  | ...  | ... | ... | ... |
| pheno50  | 1  | 0 | ... | 1 |


 
**--annotation**: required  
    Consequence annotations from Ensembl VEP __with vcf format__  
    INFO columns must contains CANONICAL|SWISSPROT|Amino_acids|Protein_position(can be easily achieved when run vep with outputing everything)    
    A typical script to generate the annotation file:  

```    
${dir_to_vep}/vep -i <vcf file with genotype> --format vcf --cache --offline --dir <dir to cache> \
--check_existing --symbol --protein --uniprot --domains --canonical --biotype --pubmed --coding_only --assembly GRCh37 \
--buffer_size 50000  --fork 8 --vcf -o test_gene.csq --no_stats
```
    
**--alpha**:  required    
    alpha = 0: using structural kernel only  
    alpha = 0.5: using combined kernel of frequency and structure  
    alpha = 1: using frequency kernel only(equivalent to a standard SKAT test)  

**--out_file**: required   
    output file where the POKEMON will write  

**--cov_file**: *optional*
    covariate file.  
    the columns for covariate file are: FID IID <cov1> ... <cov2>
    A typical command to generate the covariate file:  

``` 
${dir_to_plink}/plink --vcf <vcf file with genotype> --allow-no-sex --covar <vcf file with genotype> \  
 --prune --snps-only  --allow-no-sex --max-maf 0.05 --recode A --threads 4 --out test_gene 
```

**--cov_list**: *optional, but required if --cov_file is flagged*   
    covariates to be used  
    **covariate must be present in the columns for covariate file**  

**--pdb**: *optional*   
    e.g., --pdb 5eli, POKEMON will run on the specificed protein:5eli rather the optimal one  
   
**--maf**: *optional*      
    e.g., --maf 0.01, POKEMON will only run on variants with minor allele frequency < 0.01  
  
**--database**: *optional*      
    can only be pdb or alphafold, default as pdb  
                                                                                         
**--use_aa**: *optional*  
    if explicitly flagged, the kernel will be further scaled by AA change weight from BLOSUM62 matrix  

**--figures**: *optional*    
    if explicitly flagged, POKEMON will save pymol figures  
    **pymol must be installed to use this flag**  
  
**--out_fig_dir**: *optional but required if --figures is flagged*   
    Directory where POKEMON will write figures to                                                                                         

### Reference:  
Jin, B., Capra, J.A., Benchek, P., Wheeler, N., Naj, A.C., Hamilton-Nelson, K.L., Farrell, J.J., Leung, Y.Y., Kunkle, B., Vadarajan, B., et al. (2021). An Association Test of the Spatial Distribution of Rare Missense Variants within Protein Structures Improves Statistical Power of Sequencing Studies. BioRxiv 2021.08.09.455695.    

### License:  
MIT   
