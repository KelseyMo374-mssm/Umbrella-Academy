# Command Line Cheat Sheet
This is a cheat sheet for the codes learned during the [Command Line Exercise](Command-Line_Exercise.md).

## HPC File Structure
As a refresher, this is an example of the HPC file structure: 
```
└── Zamarin Lab Home Base (/sc/arion/projects/NGSCRC)/
    ├── master_data (Raw data, organized by project)/
    │   ├── Project 1
    │   └── Project 2
    ├── Scripts (Scripts, organized by project and by pipeline step)/
    │   ├── Project 1/
    │   │   ├── fastp Script
    │   │   └── Genome Alignment Script
    │   └── Project 2/
    │       ├── fastp Script
    │       └── Genome Alignment Script
    ├── Work (Output from scripts, organized by project and by pipeline step)/
    │   ├── Project 1/
    │   │   ├── Trimmed Reads (fastp output)
    │   │   └── Aligned Files (Genome Alignment output)
    │   └── Project 2/
    │       ├── Trimmed Reads
    │       └── Aligned Files
    ├── Tools (Shared modules for specialty projects not already available on Minerva)/
    │   ├── MultiQC
    │   └── MiXCR
    └── Resources (Shared resources such as reference genomes)/
        ├── Reference Genome (WGS)
        └── Reference Genome (WES)
```
This structure becomes particularly useful as projects become more complex.

<details>
<summary>Expand for a more detailed example to illustrate this complexity:</summary>
  
### Complex file structures

In this example, we have `Project 1` with two samples (`Sample 1` and `Sample 2`), and `Project 2` with 1 sample (`Sample A`). 

```
.
└── Zamarin Lab Home Base (/sc/arion/projects/NGSCRC)/
    ├── master_data/
    │   ├── Project 1/
    │   │   ├── Sample 1/
    │   │   │   ├── Sample_1_raw_read_1.fasq
    │   │   │   └── Sample_1_raw_read_2.fasq
    │   │   └── Sample 2/
    │   │       ├── Sample_2_raw_read_1.fasq
    │   │       └── Sample_2_raw_read_2.fasq
    │   └── Project 2/
    │       └── Sample A/
    │           ├── Sample_A_raw_read_1.fasq
    │           └── Sample_A_raw_read_2.fasq
    ├── Scripts/
    │   ├── Project 1/
    │   │   ├── fastp Script/
    │   │   │   ├── project_1_sample_list.txt
    │   │   │   └── project_1_fastp.sh
    │   │   └── Genome Alignment Script/
    │   │       ├── project_1_trimmed_sample_list.txt
    │   │       └── project_1_align.sh
    │   └── Project 2/
    │       ├── fastp Script/
    │       │   ├── project_2_sample_list.txt
    │       │   └── project_2_fastp.sh
    │       └── Genome Alignment Script/
    │           ├── project_2_trimmed_sample_list.txt
    │           └── project_2_align.sh
    ├── Work/
    │   ├── Project 1/
    │   │   ├── Trimmed Reads/
    │   │   │   ├── Sample 1/
    │   │   │   │   ├── Sample_1_trimmed_1.fastq
    │   │   │   │   └── Sample_1_trimmed_2.fastq
    │   │   │   └── Sample 2/
    │   │   │       ├── Sample_2_trimmed_1.fastq
    │   │   │       └── Sample_2_trimmed_2.fastq
    │   │   └── Aligned Files/
    │   │       ├── Sample 1/
    │   │       │   └── Sample_1_aligned.bam
    │   │       └── Sample 2/
    │   │           └── Sample_2_aligned.bam
    │   └── Project 2/
    │       ├── Trimmed Reads/
    │       │   └── Sample A/
    │       │       ├── Sample_A_trimmed_1.fastq
    │       │       └── Sample_A_trimmed_2.fastq
    │       └── Aligned Files/
    │           └── Sample A/
    │               └── Sample_A_aligned.bam
    ├── Tools/
    │   ├── MultiQC
    │   └── MiXCR
    └── Resources/
        ├── Reference Genome (WGS)
        └── Reference Genome (WES)

```
</details>

*File trees🌲created using [this cool tool](https://tree.nathanfriend.com/)*

Visualizing the structure of your directories can be useful when thinking about navigating the command line.

## 🗺️Navigating the Command Line
These are the codes we learned for basic navigation around the command line environment:

### Change directory: `cd`
Use this code to move from your current directory to another directory.

For example this code brings us to the `master_data` folder where the raw .fastqs for the [Command Line Exercise](Command-Line_Exercise.md) are stored:
```
cd /sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy
```
If you don't specify a destination (as we did when we included the path to the `master_data` above), typing just `cd` will send you to your login directory.

Let's say you want to move up two folders to the `/sc/arion/projects/NGSCRC/master_data/` folder. 
- You could type out the whole path (`cd /sc/arion/projects/NGSCRC/master_data/`).
- But you can also use "dot dot" (`..`), which is recognized as the directory immediately above the current directory (called the "parent" directory).
- If you know how many folders up you want to travel, you can separate your dot dots with a slash to get to the desired destination.

Since we are moving up two directories, we would type
```
cd ../../
```
