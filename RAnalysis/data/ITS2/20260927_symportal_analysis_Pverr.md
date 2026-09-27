scp -r hputnam_uri_edu@unity.rc.umass.edu://project/pi_hputnam_uri_edu/raw_sequencing_data/20240603_ITS2_Ashey/HP857* /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/

```
0320678392de683d67f4bb73d57b2454  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP823_S75_L001_R1_001.fastq.gz
47f053e8b76b0264177a2f198e2dfaba  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP823_S75_L001_R2_001.fastq.gz
a6ecfa782edbe0f2e1768f237c6a6fcb  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP824_S87_L001_R1_001.fastq.gz
f8fe101cd071531c4f7bb7f770c99aea  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP824_S87_L001_R2_001.fastq.gz
f16fc89afc7bc6eb5fc4b3a9daeaba25  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP828_S40_L001_R1_001.fastq.gz
8c4637d00bd657bcb8cdf657c5a3228b  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP828_S40_L001_R2_001.fastq.gz
53b449791efa178733beb33ca801e282  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP833_S5_L001_R1_001.fastq.gz
1929cc461dcc602987dea3615e2e7e5d  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP833_S5_L001_R2_001.fastq.gz
f906f03e10ea102e5f5a01710272ca40  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP837_S53_L001_R1_001.fastq.gz
5e77080e1a5448addd78fe73fd83f97a  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP837_S53_L001_R2_001.fastq.gz
a2be59f2db7b79f653f9c0525c8ca576  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP838_S65_L001_R1_001.fastq.gz
8393600dd85b9de03df7216a79df5e79  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP838_S65_L001_R2_001.fastq.gz
4014fc818651a65d29ec90b77f42e519  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP841_S6_L001_R1_001.fastq.gz
9a8a289dc6ecdd6968d08d31600e9e87  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP841_S6_L001_R2_001.fastq.gz
cc0e3d377a32f3461348b1a6a3c45ac1  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP856_S91_L001_R1_001.fastq.gz
4149a5a38d03b2a001f67e1c06a70ee2  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP856_S91_L001_R2_001.fastq.gz
91f082dd78528461910e307d222fbdb4  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP857_S8_L001_R1_001.fastq.gz
e2ce85480fd95079954d9ad21be1e439  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP857_S8_L001_R2_001.fastq.gz
469f36b325ee3e51cb2bca6d10750cd4  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP859_S32_L001_R1_001.fastq.gz
5edf4f7957dbbb6a340e528777cbfa5f  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP859_S32_L001_R2_001.fastq.gz
493fa5972823109801612f4810219371  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP860_S44_L001_R1_001.fastq.gz
3fb263ca120aac8d0dfdca66ea9c0367  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP860_S44_L001_R2_001.fastq.gz
a1c6024a495db878f338638b1391c9e9  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP861_S56_L001_R1_001.fastq.gz
3390757aa8cc61ffeea69a7994d047e8  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/HP861_S56_L001_R2_001.fastq.gz
```

# Running Symportal Locally on Unity
[Symportal Setup Instructions](https://github.com/reefgenomics/SymPortal_framework/wiki/0_3_18_19_SymPortal-setup)


```
salloc

cd /work/pi_hputnam_uri_edu

git clone https://github.com/reefgenomics/SymPortal_framework

cd SymPortal_framework

mv settings_blank.py settings.py

base64 /dev/urandom | head -c50
kpkhwFIeLNXsOYwJyixMjZvodcxOJGcwHqW0vfl3f2GI2XNesZ

nano settings.py

mv sp_config_blank.py sp_config.py

nano sp_config.py

salloc
module load conda/latest
conda env create -f symportal_env.yml

conda activate symportal_env

python3 manage.py migrate
```

### make a database of sym ref seqeunces from the updates they provide from Symportal
Obtain locally on my computer 
[SymPortal ref seqs as of 20240213](https://symportal.org/static/resources/SymPortal_unique_DIVs.tar.gz)
Unzip and Upload to Unity becasue Unity will not recognize zip format
```
scp -r /Users/hputnam/Downloads/SymPortal_unique_DIVs.fasta unity://work/pi_hputnam_uri_edu/SymPortal_framework/symbiodiniaceaeDB
```

### Remove the default fasta file with the installation and replace it with the latest version from Symportal website (in this case [SymPortal ref seqs as of 20240213](https://symportal.org/static/resources/SymPortal_unique_DIVs.tar.gz))

```
cd /work/pi_hputnam_uri_edu/SymPortal_framework/symbiodiniaceaeDB   

rm refSeqDB.fa

mv SymPortal_unique_DIVs.fasta refSeqDB.fa

cd /work/pi_hputnam_uri_edu/SymPortal_framework/   

python3 populate_db_ref_seqs.py
```

## Test Installation

python3 -m tests.tests

Successful completion

### Upload sample information data in Symportal template xlsx

cd /work/pi_hputnam_uri_edu/

mkdir /work/pi_hputnam_uri_edu/Pverr_larvae_ITS2

cd /work/pi_hputnam_uri_edu/Pverr_larvae_ITS2

scp -r  /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/SymPortal_datasheet_20260927_HP.xlsx unity://work/pi_hputnam_uri_edu/Pverr_larvae_ITS2

### load sample sequence data 

example copy the samples R1 and R2 for all 12 to local file 

cp /project/pi_hputnam_uri_edu/raw_sequencing_data/20240603_ITS2_Ashey/HP851* .

```
nano SymPortal_Load.sh

#!/bin/bash
#SBATCH --job-name=SymPortal_Load_Pverr_Larvae
#SBATCH --nodes=1 --cpus-per-task=8
#SBATCH --mem=50G  # Requested Memory
#SBATCH -p gpu  # Partition
#SBATCH -G 1  # Number of GPUs
#SBATCH --time=12:00:00  # Job time limit
#SBATCH -o slurm-%j.out  # %j = job ID
#SBATCH -D /work/pi_hputnam_uri_edu/SymPortal_framework

module load conda/latest
conda activate symportal_env

python3 /work/pi_hputnam_uri_edu/SymPortal_framework/main.py --load /work/pi_hputnam_uri_edu/Pverr_larvae_ITS2 --name Pverr_larvae_ITS2 --num_proc $SLURM_CPUS_ON_NODE --data_sheet /work/pi_hputnam_uri_edu/Pverr_larvae_ITS2/SymPortal_datasheet_20260927_HP.xlsx


sbatch SymPortal_Load.sh
```
Submitted batch job 64966401

```
from within /work/pi_hputnam_uri_edu/SymPortal_framework 
python3 main.py --display_data_sets
```

```
nano SymPortal_Analysis.sh

#!/bin/bash
#SBATCH --job-name=SymPortal_Analysis_Pverr_Larvae
#SBATCH --nodes=1 --cpus-per-task=8
#SBATCH --mem=50G  # Requested Memory
#SBATCH -p gpu  # Partition
#SBATCH -G 1  # Number of GPUs
#SBATCH --time=12:00:00  # Job time limit
#SBATCH -o slurm-%j.out  # %j = job ID
#SBATCH -D /work/pi_hputnam_uri_edu/SymPortal_framework

module load conda/latest
conda activate symportal_env

python3 /work/pi_hputnam_uri_edu/SymPortal_framework/main.py --analyse 6 --name Pverr_larvae_ITS2 --num_proc 3 


sbatch SymPortal_Analysis.sh

```
Submitted batch job 64967848



 ANALYSIS COMPLETE: DataAnalysis:
        name: Pverr_larvae_ITS2
        UID: 5

DataSet analysis_complete_time_stamp: 20260927T203020


scp -r unity://work/pi_hputnam_uri_edu/SymPortal_framework/outputs/analyses/5/20260927T202911 /Users/hputnam/MyProjects/Pverrucosa_Development/RAnalysis/data/ITS2/Putnam_Run/