201809

To blast on a custom database, use BLAST+ on baobab.

1 - Building a BLAST database with local sequences

Doc
https://www.ncbi.nlm.nih.gov/books/NBK279688/

database = two-lined caecilian genome downloaded on https://vgp.github.io/genomeark/Rhinatrema_bivittatum/
file = aRhiBiv1_t4.p.fasta

mkdir CaecDB

Copy files from local to baobab
Caecilian genome, from which we will build the database (blastn subject)
MacBook-Pro-de-Hintermann:blastCaecilian Hintermann$ scp aRhiBiv1_t4.p.fasta hinterma@baobab.unige.ch:~/CaecilianBLAST/CaecDB/

HCNEs mouse/chicken that were found in Caecilian (blastn query)
scp HCNEs_Caecilian.fasta hinterma@baobab.unige.ch:~/CaecilianBLAST/

Write mkdb.sh (-rw-r--r-- 1 hinterma unige        297 Oct  3 14:27 slurm-10055082.out)

#!/bin/sh

#SBATCH --time=00:10:00
#SBATCH --cpus-per-task=1
#SBATCH --partition=debug
#SBATCH --mail-type=ALL


module load GCC/4.9.3-2.25 OpenMPI/1.10.2 BLAST+/2.3.0-Python-2.7.11

srun makeblastdb -in aRhiBiv1_t4.p.fasta -dbtype='nucl' -title aR_DB -out aR_DB.full -parse_seqids

Use our custom database to blast the mouse/chicken HCNEs

Write blastHCNE.sh ( 0 -rw-r--r-- 1 hinterma unige     0 Oct  3 14:56 slurm-10057290.out)

#!/bin/sh

#SBATCH --time=00:14:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=20G
#SBATCH --partition=debug
#SBATCH --mail-type=ALL


module load GCC/4.9.3-2.25 OpenMPI/1.10.2 BLAST+/2.3.0-Python-2.7.11

srun blastn -db CaecDB/aR_DB.full -query HCNEs_Caecilian.fasta -out CaecBlast.txt -outfmt '6 qseqid sseqid sseq'

The -outfmt '6 qseqid sseqid sseq' is required if you want to transform your output file from .txt to .fasta

TXTtoFASTA.sh (-rw-r--r--  1 hinterma unige     0 Oct  3 14:58 slurm-10057361.out)

#!/bin/sh

#SBATCH --time=00:14:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=20G
#SBATCH --partition=debug
#SBATCH --mail-type=ALL


module load GCC/4.9.3-2.25 OpenMPI/1.10.2 BLAST+/2.3.0-Python-2.7.11

srun awk 'BEGIN { OFS = "\n" } { print ">"$2, $3 }' CaecBlast.txt > CaecBlast.fasta


