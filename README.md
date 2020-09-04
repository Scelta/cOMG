# cOMG (chaotic Omics - the MetaGenomics)
This pipeline is built to ease my pressure for Multiple omics analysis. In this version, I'm focused on the process of data polishing of metagenome-wide analysis.

# Dependency
- Linux OS (test on CentOS 6.9 and 7.2)  
- perl 5 (test by v5.26.2)

# Install

```
cd /path/to/your/dir/
clone https://github.com/Scelta/cOMG.git
# put it under PATH
ln -s /path/to/your/dir/cOMG/cOMG ~/bin/
# Or added to PATH
export PATH="/path/to/your/dir/cOMG":$PATH
```

# Usage:

```
cOMG
usage:
        cOMG <pe|se|config|cmd> [options]
mode
        pe|se           pair end | single end
        config          generate a config file template
        cmd             directely call a sub-script under bin/ or util/
options:
        -p|path         :[essential]sample path file (SampleID|fqID|fqPath)
        -i|ins          :[essential for pe mode]insert info file or a number of insert size
        -s|step         :functions,default 1234
                             1       trim+filter, OA method
                             2       remove host genomic reads
                             3       soap mapping to microbiotic genomics
                             4       combine samples' abun into a single profile table
        -o|outdir       :output directory path. Conatins the results and scripts.
        -c|config       :provide a configure file including needed database and parameters for each setp
        -h|help         :show help info
        -v|version      :show version and author info.
```

**path file**: used for record path location of raw data. Needs 3 columns:
1. *SampleID* : biological sample ID, generally the subject used in your study. Note: DO NOT start with numbers.
2. *SeqdataID*: Sequence data ID. Make sure it's unique in this batch run. Note: DO NOT start with numbers.
3. *SeqdataPath*: Sequence data ABSOLUTELY path location. For pair-end data, put `read1` and `read2` in 2 tandem lines , with same *SampleID* and *SeqdataID*.  

Note: For one sample sequenced multiple times, provide them with the same *sampleID* so they will be summed together in relative abundance calculation step.

e.g:
```
column -t test.5samples.path
t01        ERR260132  ./fastq/ERR260132_1.fastq.gz
t01        ERR260132  ./fastq/ERR260132_2.fastq.gz
t02.sth    ERR260133  ./fastq/ERR260133_1.fastq.gz
t02.sth    ERR260133  ./fastq/ERR260133_2.fastq.gz
t03_rep    ERR260134  ./fastq/ERR260134_1.fastq.gz
t03_rep    ERR260134  ./fastq/ERR260134_2.fastq.gz
t04_rep_2  ERR260135  ./fastq/ERR260135_1.fastq.gz
t04_rep_2  ERR260135  ./fastq/ERR260135_2.fastq.gz
t05        ERR260136  ./fastq/ERR260136_1.fastq.gz
t05        ERR260136  ./fastq/ERR260136_2.fastq.gz
```



**config file** example:

```
###### configuration

### Database location
db_host = $META_DB/human/hg19/Hg19.fa.index  # Prefix of host genome SOAP2 database
db_meta = $META_DB/1267sample_ICG_db/4Group_uniqGene.div_1.fa.index,$META_DB/1267sample_ICG_db/4Group_uniqGene.div_2.fa.index # Prefix of metagenomics gene set SOAP2 database. Use comma for multiple databases.

### reference gene length file
RGL  = $META_DB/IGC.annotation/IGC_9.9M_update.fa.len # Geneset length info mation. 3 columns needed. See below*.
### pipeline parameters
PhQ = 33            # reads Phred Quality system: 33 or 64.
mLen= 30            # minimal read length allowance
seedOA=0.9          # OA methd. Quality cutoff for seed [0,1]
fragOA=0.8          # OA methd. Quality cutoff for retained fragment [0,1]

qsub = 1234         # Following argment will enable only if qusb=on, otherwise you could commit it
q   = st.q          # queue id or group id (if PBS enabled) for qsub
P   = st_ms         # Project id for qsub
B   = 1             # Global setting of backup tasks number.
B1  = 3             # Backup tasks number specific for step 1.
p   = 6             # Global setting of cpu numbers for each step.
p1  = 1             # cpu numbers for step 1 (Quality-control)
p4  = 1             # cpu numbers for step 4 (Abundance calculation)
f1  = 0.5G          # virtual free for qsub in step 1 (trim & filter)
f2  = 6G            # virtual free for qsub in step 2 (remove host genes)
f3  = 14G           # virtual free for qsub in step 3 (aligned to gene set)
f4  = 8G            # virtual free for qsub in step 4 (calculate soap results to abundance)
s   = 120           # For qusbM. Time interval to check job status.
r   = 10            # Repeat time when job failed or interrupted

#### Denmark National Computerome 2.0 PBS parameters. See https://www.computerome.dk
PBS = 0                 #PBS support. [0] to turn off, [1] to turn on
walltime=7:00:00:00     #Requesting time - format is <days>:<hours>:<minutes>:<seconds> (here, 7 days)

```
* Geneset length info content:
1. Gene ID corresponding to the Geneset in `db_meta`.  
2. Gene Name.  
3. Gene length. For adjusting the calculation of relative abundance.  

After prepared above **configure** and **path file**. The workshop can be initiated. Command example:
```
cd t
cOMG se -p demo.input.lst -c demo.cfg -o demo.test
```

Before actually run or submit your task, finall Chcek the generated scripts.
I provide several strategy to run. Choose one of them. DO NOT run all! They will executing the same low-level scripts.
```
sh RUN.batch.sh        # Mode 1: Next step will start only When all samples' previous step done.
sh RUN.linear.1234.sh  # Mode 2: Each sample run parallel. Also available for qsub submit in Denmark National Computerome HPC.
sh RUN.qsubM.s         # Mode 3: For qsub in BGI HPC, monitor tasks by qsubM, a self-developed qusb task manager.
```

After finished, run `sh report.stat.sh` to print a report table.

### Feedback
Issue report is welcome.
