# Convert_Plinkfiles_Blupf90_files

#!/bin/bash
#BSUB -J KEIES_ALL      # LSF job name
#BSUB -o KEIESALL.%J.out     # Name of the job output file 
#BSUB -e KEIESALL.%J.error   # Name of the job error filei
#BSUB -n 8
#BSUB -M 10400 


# Module load plink

module load plink-1.90b3b

# extract snpID from chr 1-38 
awk '($1!=0)' KEIES_712k.map | awk '($1<39)' | awk '{print $2}' > snpID

# # extract map and ped files for chr 1-38 only

plink --file KEIES_712k --chr-set 43 --noweb --extract snpID --recode --make-bed --out New_KEIES_712k

# convert to blupf90 format/0125 for snp markers

plink --bfile New_KEIES_712k --allow-extra-chr --chr-set 38 --allow-no-sex --nonfounders --recode A --out gens

# remove header
sed -n '1p' gens.raw |fmt -1 > header.txt
sed -i 1d gens.raw

# remove the spaces between columns

paste -d' ' <(cut -d' ' -f2 gens.raw) <(cut -d' ' -f7- gens.raw |sed 's/ //g'|sed 's/9/5/g') >  gens.txt
mv gens.txt snp.txt

#Replace NA with 5
sed -i 's/NA/5/g' gens.txt

# extract the pedigree
cut -d' ' -f2-4 gens.raw > pedig.txt

# convert map file to blupf90 format
awk -F' '  '{print NR,$1,$4}' New_KEIES_712k.bim > mapfile.txt

#Add header to the mapfile

echo "SNP_ID" "CHR" "POS" >> mapcolname.txt
cat mapfile.txt >>  mapcolname.txt
mv mapcolname.txt mapfile.txt

