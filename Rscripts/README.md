# Title of Dataset: Data Archive for Sexually discordant selection is associated with trait specific morphological changes and a complex genomic response

[Access this dataset on Dryad] DOI:10.5061/dryad.6t1g1jx6k

*brief summary of dataset contents, contextualized in experimental procedures and results.

Data Archive for:
Sexually discordant selection is associated with trait specific morphological changes and a complex genomic response
by T Audet, J Krol, K Pelletier, AD Stewart, I Dworkin (2024)

This folder contains all associated Rscripts for analysis done after data files are generated via bash scripts. For scripts to run, the bash outputs must be placed in the data folder, which is called in each script with a relative path backwards once and in to /data. For information on experimental details, or information on the upstream bash scripts, please see the main README.


## Description of the data and file structure

All scripts in this folder are in R markdown or R scripts, and detailed information of the steps are included as comments within the script.

### Rscript details

Between_treatment_logistic_regression.Rmd
The csv that is read in is not provided but is outputted from 'freq_table_grenedalf.sh' with the input being a sync file containing all SNPs in all treatment and the output being a csv file of the minor and major allele frequencies for each treatment.

BinomialSampling_FST.Rmd
On line 119 a csv file is read in that is output from 'freq_table_grenedalf.sh' with the input being a sync file of males and females chromosome 3L and the output being a csv of major and minor allele frequencies.

CMH_analysis.Rmd
This script loads in a sync file generated after SNP calling with 'poolSNP_callSNP.sh' with an mpileup with the sexes merged, and using those SNP positions extracted from a sync file of all positions output from 'sync_grenedalf.sh'.

Extracting_majAllele_frequencies_from_sync.Rmd
This script was used to extract the letter code of the major allele from our C1 SNP called sync file so that it can be appended to our frequency table as input to 'Between_treatment_logistic_regression.Rmd'.

Extracting_sites_and_plotting.Rmd
This script is used to extract sites of interest and to plot from various output files. Files read in to this script are Fst estimate within and between treatments outputted from 'grenedalf_fst.sh', between treatment pi values output from 'pi_grenedalf.sh', and between treatment CMH values outputted from 'CMH_analysis.Rmd'.

F100_plots_andTopPercents.R
This script does the same as 'Extracting_sites_and_plotting.Rmd' however the inputs are generated with the F100 data outputted from the same grenedalf script.

Phenotype_analysis.Rmd
This  script contains all phenotypic analysis for morphology and the input is '../data/JK_Feb2020_legsThorax_SSD.csv'

SexRatio_F1_F2.Rmd
This script contains all analysis done on the sex ratio cross and the input data is '../data/sex_ratio_cross.csv'.

Sex_comparison_general_linear_model.Rmd
This script is an example of the linear modelling for the male vs. female modelling. This was run for each treatment independantly on a remote computer. The inputs are frequency tables between sexes within treatment generated with 'grenedalf_fst.sh'

Simulation_Fst_analysis.Rmd
This script is used to calculate quantiles from our simulations. The input is Fst calculations output by 'grenedalf_fst.sh' for 100 generated simulated genomes placed within a simulation folder inside the data folder. This script takes all simulated Fst files and reads them in, then calculates quantiles. Simulated genomes are created using the two scripts in the simulationScripts folder.

plotting_Fst_vs_model_Estimates.Rmd
This script just reads in the output from 'Sex_comparison_general_linear_model.Rmd' and 'Extracting_sites_and_plotting.Rmd' and plots one on the X axis and one on the Y axis for comparison.

plotting_between_sex_models.Rmd
This script reads in the output for each treatment from 'Sex_comparison_general_linear_model.Rmd' and plots it. It also has lines to extract low p-values or the highest 99th percentile of odds ratio which was used for plotting.





