# RMATS_PIPELINE

## About

This rMATS_pipeline is the C/Cython version of rMATS (refer to http://rnaseq-mats.sourceforge.net/user_guide.htm). The main difference between rMATS_pipeline and rMATS is speed and space usage. The speed of rMATS_pipeline is 100 times faster and the output file is 1000 times smaller than rMATS. These advantages make analysis and storage of large scale dataset easy and convenient.

|  | Counting part | Statistical part |
|---------------|----------------|--------------|
| Speed (C/Cython version vs Python version) | 20~100 times faster (one thread) | 300 times faster (6 threads) |
| Storage usage (C/Cython version vs Python version) | 1000 times smaller | - |

## Dependencies

- Python 2.6/2.7 # Python3 is Ok, but you have to copy the built shared library by yourself. See **Build**.
- Cython >= (0.23.4)

You can use the instruction below if you're using pip,

    (sudo) pip install Cython

## Build

    make # After executing this command, there will be a rmatsrmatspipeline.so in working directory.
         # If not, please check **build** folder and copy the .so file by yourself.

## Usage

    python rmats.py -h
    
    usage: usage: rmats.py [options] arg1 arg2
	
    optional arguments:
		  -h, --help            show this help message and exit
		  --version             Version.
		  --gtf GTF             An annotation of genes and transcripts in GTF format.
		  --b1 B1               BAM configuration file.
		  --b2 B2               BAM configuration file.
		  --s1 S1               FASTQ configuration file.
		  --s2 S2               FASTQ configuration file.
		  --od OD               output folder of post step.
		  --tmp TMP             output folder of prep step. Each run will generate a
		                        .rmats file which contain essential information of
		                        each BAM file.
		  -t {paired,single}    readtype, single or paired.
		  --libType {fr-unstranded,fr-firststrand,fr-secondstrand}
		                        Library type. Default is unstranded (fr-unstranded).
		                        Use fr-firststrand or fr-secondstrand for strand-
		                        specific data.
		  --readLength READLENGTH
		                        The length of each read.
		  --anchorLength ANCHORLENGTH
		                        The anchor length.
		  --tophatAnchor TOPHATANCHOR
		                        The "anchor length" or "overhang length" used in the
		                        aligner. At least “anchor length” NT must be
		                        mapped to each end of a given junction. The default is
		                        1. (This parameter applies only if using fastq).
		  --bi BINDEX           The folder name of the STAR binary indexes (i.e., the
		                        name of the folder that contains SA file). For
		                        example, use ~/STARindex/hg19 for hg19. (Only if using
		                        fastq)
		  --nthread NTHREAD     The number of thread. The optimal number of thread
		                        should be equal to the number of CPU core.
		  --tstat TSTAT         the number of thread for statistical model.
		  --cstat CSTAT         The cutoff splicing difference. The cutoff used in the
		                        null hypothesis test for differential splicing. The
		                        default is 0.0001 for 0.01% difference. Valid: 0 ≤
		                        cutoff < 1.
		  --task {prep,post,both,inte}
		                        Preprocessing BAMs or Performing ASE detection and
		                        read Counting. prep: preprocess BAMs and generate a
		                        .rmats file. post: load .rmats file into memory, get
		                        the read count for each BAM and calculate P value if
		                        --stat is on. both: prep + post. inte: To check if
		                        every BAM file is inputed exactly once. (integrity).
		  --statoff             Turn statistical analysis off.

## Output

--od read count generated by the post step:

- fromGTF.AS_Event.txt: all possible alternative splicing (AS) events derived from GTF and RNA.

- JC.raw.input.AS_Event.txt evaluates splicing with only reads that span splicing junctions
	- IJC_SAMPLE_1: inclusion junction counts for SAMPLE_1, replicates are separated by comma
	- SJC_SAMPLE_1: skipping junction counts for SAMPLE_1, replicates are separated by comma
	- IJC_SAMPLE_2: inclusion junction counts for SAMPLE_2, replicates are separated by comma
	- SJC_SAMPLE_2: skipping junction counts for SAMPLE_2, replicates are separated by comma
	- IncFormLen: length of inclusion form, used for normalization
	- SkipFormLen: length of skipping form, used for normalization

- JCEC.raw.input.AS_Event.txt evaluates splicing with reads that span splicing junctions and reads on target (striped regions on home page figure)
	- IC_SAMPLE_1: inclusion counts for SAMPLE_1, replicates are separated by comma
	- SC_SAMPLE_1: skipping counts for SAMPLE_1, replicates are separated by comma
	- IC_SAMPLE_2: inclusion counts for SAMPLE_2, replicates are separated by comma
	- SC_SAMPLE_2: skipping counts for SAMPLE_2, replicates are separated by comma
	- IncFormLen: length of inclusion form, used for normalization
	- SkipFormLen: length of skipping form, used for normalization

- AS_Event.MATS.JC.txt evaluates splicing with only reads that span splicing junctions
	- IC_SAMPLE_1: inclusion counts for SAMPLE_1, replicates are separated by comma
	- SC_SAMPLE_1: skipping counts for SAMPLE_1, replicates are separated by comma
	- IC_SAMPLE_2: inclusion counts for SAMPLE_2, replicates are separated by comma
	- SC_SAMPLE_2: skipping counts for SAMPLE_2, replicates are separated by comma

- AS_Event.MATS.JCEC.txt evaluates splicing with reads that span splicing junctions and reads on target (striped regions on home page figure)
	- IC_SAMPLE_1: inclusion counts for SAMPLE_1, replicates are separated by comma
	- SC_SAMPLE_1: skipping counts for SAMPLE_1, replicates are separated by comma
	- IC_SAMPLE_2: inclusion counts for SAMPLE_2, replicates are separated by comma
	- SC_SAMPLE_2: skipping counts for SAMPLE_2, replicates are separated by comma

- Important columns contained in both types of output files
	- IncFormLen: length of inclusion form, used for normalization
	- SkipFormLen: length of skipping form, used for normalization
	- P-Value: (The meaning of p value???)
	- FDR: (The meaning of FDR???)
	- IncLevel1: inclusion level for SAMPLE_1 replicates (comma separated) calculated from normalized counts
	- IncLevel2: inclusion level for SAMPLE_2 replicates (comma separated) calculated from normalized counts
	- IncLevelDifference: average(IncLevel1) - average(IncLevel2)

--tmp intermediate file generated by the prep step

- filename.rmats Output of the prep step. Input of the post step.
- bamX_Y STAR mapping result.

	                        
## Examples for 

### scenario one:

Suppose we have 4 samples, and our machine is able to process these 4 samples once for all.

b1.txt: /xxx/xxx/1.bam,/xxx/xxx/2.bam

b2.txt: /xxx/xxx/3.bam,/xxx/xxx/4.bam

    python rmats/asevent.py --b1 bi.txt --b2 b2.txt --gtf /xxx/xxx/5.gtf --od output -t paired --nthread 4 --readLength 101 --anchorLength 8 --task both --tmp tmp

### scenario two:

Suppose we have 8 samples, and our machine has only 4 CPU threads to process these samples.

b1.txt: /xxx/xxx/1.bam,/xxx/xxx/2.bam

b2.txt: /xxx/xxx/3.bam,/xxx/xxx/4.bam

b3.txt: /xxx/xxx/5.bam,/xxx/xxx/6.bam

b4.txt: /xxx/xxx/7.bam,/xxx/xxx/8.bam

    python rmats/asevent.py --b1 b1.txt --b2 b2.txt --gtf /xxx/xxx/9.gtf --od output -t paired --nthread 4 --readLength 101 --anchorLength 8 --task prep --tmp tmp
    ...
    python rmats/asevent.py --b1 b3.txt --b2 b4.txt --gtf /xxx/xxx/9.gtf --od output -t paired --nthread 4 --readLength 101 --anchorLength 8 --task prep --tmp tmp

And then,

b5.txt: /xxx/xxx/1.bam,/xxx/xxx/2.bam,/xxx/xxx/5.bam,/xxx/xxx/6.bam

b6.txt: /xxx/xxx/3.bam,/xxx/xxx/4.bam,/xxx/xxx/7.bam,/xxx/xxx/8.bam

    python rmats/asevent.py --b1 b5.txt --b2 b6.txt --gtf /xxx/xxx/9.gtf --od output -t paired --nthread 1 --readLength 101 --anchorLength 8 --task post --tmp tmp

**Important Note**:

Since our program can preprocess BAM files group by group, it is easy for the user to utilize their cluster (multiple computation nodes) to speed up the progress. For example, the user can preprocess their BAM files on hundreds of machines (prep step), and then combine the splice graph and generate output on another machine (post step).

```diff

- Caveat

```
------

In the 'post' step, the program traverses the folder 'tmp' and considers files ended with '.rmats' as input. It is users’ responsibility to ensure that the folder 'tmp' only contains the files their are analyzing.

BAM file name should be in the same format in both prep and post step. For example, if you use /xxx/xxx/1.bam in prep step, then you cannot use 1.bam in post step. 


## Exception

**Exception ValueError: ValueError('list.index(x): x not in list',) in 'asevent_core._load_job' ignored**

The filename used in post step must be the same as that which is used in prep step.
The BAM filename which the user provide to the post step must be contained by .rmats file we're analyzing. Refer to **Caveat**.

**Zero read count**

IF a single BAM file is provided multiple times during the post step, the program might output zero read count for this BAM file due to multithreading.

## Best practice for --task pipe

prep step:

- Separate your BAM file into groups, say 10 groups.
- Make sure that every BAM filename only occur once.
- run prep step.

post step:

- Gather all .rmats file which you are going to analyze, and put them into same folder.
- The first line of .rmats indicate the original BAM file from which this .rmats file generated.
- Make sure that every BAM filename only occur once.
- run post step.

------
OR

If your dataset is small enough to fit into memory, you can just run --task both and get all the result all at once.

## Docker image

Please run the command below to install/run the Docker image:

```sh
docker load -i rmats.tar # install
docker run rmats:release -h # print help page.
```