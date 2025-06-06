# Real-world Debugging Example
These are three real `.out` (*output*) and/or `.err` (*error*) files that I generated while writing a script to run [MiXCR](https://mixcr.com/mixcr/about/), an algorithm to assemble and quantify TCR sequences, on **Minerva** (Mount Sinai's HPC).
![MiXCR](https://mixcr.com/mixcr/about-light.svg#only-light)

I'll walk you step-by-step through my process for identifying and fixing each of these bugs 🪲

> **ℹ️Note:** You will encounter many different kinds of errors as you write your own scripts and conduct your own analyses, so I can't cover all possible scenarios. 
>
> But these three examples serve as a good illustration of the categories of bugs you will encounter and strategies for addressing them.

## 🪲 Example 1: The fix is in the `.out` file
**These are the best kinds of errors to debug, because the error message tells you (roughly) what you need to do.**

Here is the output from the first time I ran my script:

<details>
  <summary> Full output file </summary>
  
```
Sender: LSF System <lsfadmin@lh05g17>
Subject: Job 137582052[1]: <MyArrayJob[1-1]> in cluster <chimera> Done

Job <MyArrayJob[1-1]> was submitted from host <li03c03.chimera.hpc.mssm.edu> by user <monsok03> in cluster <chimera> at Thu Aug 22 10:14:37 2024
Job was executed on host(s) <lh05g17>, in queue <premium>, as user <monsok03> in cluster <chimera> at Thu Aug 22 10:15:51 2024
</hpc/users/monsok03> was used as the home directory.
</sc/arion/projects/NGSCRC/Scripts/NRG-GY003/TCRseq> was used as the working directory.
Started at Thu Aug 22 10:15:51 2024
Terminated at Thu Aug 22 10:15:52 2024
Results reported at Thu Aug 22 10:15:52 2024

Your job looked like:

------------------------------------------------------------
# LSBATCH: User input
#!/bin/bash
#BSUB -P acc_NGSCRC 		   # Zamarin Lab allocation account
#BSUB -n 1	    		       # total number of compute cores
#BSUB -W 08:00	    		   # walltime in HH:MM
#BSUB -q premium    		   # queue 
#BSUB -R "rusage[mem=32GB]"        # 32 GB of memory (32 GB per core)
#BSUB -J "MiXCR_FC7[1-94]"         # Job name (batch job for flow cell 7 TCR-seq)
#BSUB -o out.%J.%Im		     # output file name (%I=job array index, %J=jobID)
#BSUB -e err.%J.%I  		   # error file name (same nomenclature as above)
#BSUB -L /bin/bash		     # Initialize the execution environment


export PATH=/sc/arion/projects/NGSCRC/Tools/MiXCR/mixcr-4.7.0-0/bin:$PATH

FLOW_CELL="FC07"
SAMPLE_LIST=${FLOW_CELL}_sample_list.txt
SAMPLE_DIR=$(head -n "${LSB_JOBINDEX}" "${SAMPLE_LIST}" | tail -n 1)
SAMPLE_ID=$(basename ${SAMPLE_DIR})
OUTPUT_FOLDER="/sc/arion/projects/NGSCRC/Work/NRG-GY003/TCRseq/MiXCR"

echo `date`
echo "Starting MiXCR for ${SAMPLE_ID}"

## MiXCR has a pre-set for Cellecta DNA sequencing so we will use that

mkdir -p ${OUTPUT_FOLDER}/${SAMPLE_ID}
mkdir -p ${OUTPUT_FOLDER}/${SAMPLE_ID}/figs


mixcr analyze cellecta-human-dna-xcr-umi-drivermap-air \
    ${SAMPLE_DIR}/${SAMPLE_ID}_R1_001.fastq.gz \
    ${SAMPLE_DIR}/${SAMPLE_ID}_R2_001.fastq.gz \
    ${OUTPUT_FOLDER}/${SAMPLE_ID}

mv ${SAMPLE_ID}.* ${OUTPUT_FOLDER}/${SAMPLE_ID}
cd ${OUTPUT_FOLDER}/${SAMPLE_ID}

mixcr exportQc align \
    ${SAMPLE_ID}.clns \
    figs/${SAMPLE_ID}_alignQc.pdf

mixcr exportQc chainUsage \
    ${SAMPLE_ID}.clns \
    figs/${SAMPLE_ID}_chainUsage.pdf

mixcr exportQc tags \
    ${SAMPLE_ID}.clns \
    figs/${SAMPLE_ID}_tags.pdf

(... more ...)
------------------------------------------------------------

Successfully completed.

Resource usage summary:

    CPU time :                                   0.14 sec.
    Max Memory :                                 -
    Average Memory :                             -
    Total Requested Memory :                     32768.00 MB
    Delta Memory :                               -
    Max Swap :                                   -
    Max Processes :                              -
    Max Threads :                                -
    Run time :                                   1 sec.
    Turnaround time :                            75 sec.

The output (if any) follows:

Thu Aug 22 10:15:52 EDT 2024
Starting MiXCR for P2A10_GADCIWn2_S10
Wrong version of java. Please use Java 7 or higher.
Wrong version of java. Please use Java 7 or higher.
Wrong version of java. Please use Java 7 or higher.
Wrong version of java. Please use Java 7 or higher.
Finished MiXCR for P2A10_GADCIWn2_S10
Thu Aug 22 10:15:52 EDT 2024


PS:

Read file <err.137582052.1> for stderr output of this job.

```

</details>

Here is the relevant section:
```
Successfully completed.

The output (if any) follows:

Thu Aug 22 10:15:52 EDT 2024
Starting MiXCR for P2A10_GADCIWn2_S10
Wrong version of java. Please use Java 7 or higher.
Wrong version of java. Please use Java 7 or higher.
Wrong version of java. Please use Java 7 or higher.
Wrong version of java. Please use Java 7 or higher.
Finished MiXCR for P2A10_GADCIWn2_S10
Thu Aug 22 10:15:52 EDT 2024
```
You'll notice it says `Successfully completed` -- but then it tells me that I used the `Wrong version of java. Please use Java 7 or higher.`

>⚠️ **This is important**
>
>Just because you see `Successfully completed` in your `.out` file does **not** necessarily mean your script did what you wanted it to do!
>
>That's why it's important to check your `.err` files, as well as your output folders to see if you have gotten what you expect.

In this case, my script couldn't actually run, so there was no output.

**This was because MiXCR requires Java to run, and I hadn't loaded Java in my script.**

Now, I will admit, this error didn't tell me *exactly* what I needed to do to fix it. 

I could have wasted time trying to figure out what version of Java was installed on Minerva.

But I knew I hadn't loaded *any* version of Java, so I knew the fix was to add that to my script. 

### Summary:
* **Error:** "Wrong" version of Java -- a.k.a. *no* version of Java
* **Fix:** Add `module load java/11.0.2` to my script so that Java is loaded

## 🪲 Example 2: The fix is in the `.err` file
**This error message is also relatively straightforward, but if you only look at the `.out` file, you won't know there's something wrong**

After updating my script to load Java and re-running it, I didn't get any errors in my `.out` file -- so far so good!
<details>
  <summary>Full output file</summary>

```
Sender: LSF System <lsfadmin@lc01e25>
Subject: Job 137529137[1]: <MyArrayJob[1-1]> in cluster <chimera> Done

Job <MyArrayJob[1-1]> was submitted from host <li03c04.chimera.hpc.mssm.edu> by user <monsok03> in cluster <chimera> at Wed Aug 21 17:49:58 2024
Job was executed on host(s) <lc01e25>, in queue <premium>, as user <monsok03> in cluster <chimera> at Wed Aug 21 17:50:00 2024
</hpc/users/monsok03> was used as the home directory.
</sc/arion/projects/NGSCRC/Scripts/NRG-GY003/TCRseq> was used as the working directory.
Started at Wed Aug 21 17:50:00 2024
Terminated at Wed Aug 21 17:50:04 2024
Results reported at Wed Aug 21 17:50:04 2024

Your job looked like:

------------------------------------------------------------
# LSBATCH: User input
#!/bin/bash
#BSUB -P acc_NGSCRC 		   # Zamarin Lab allocation account
#BSUB -n 1	    		   # total number of compute cores
#BSUB -W 08:00	    		   # walltime in HH:MM
#BSUB -q premium    		   # queue 
#BSUB -R "rusage[mem=32GB]"        # 32 GB of memory (32 GB per core)
#BSUB -J "MiXCR_FC7[1-94]"         # Job name (batch job for flow cell 7 TCR-seq)
#BSUB -o out.%J.%Im		   # output file name (%I=job array index, %J=jobID)
#BSUB -e err.%J.%I  		   # error file name (same nomenclature as above)
#BSUB -L /bin/bash		   # Initialize the execution environment


## Load MiXCR and Java (needs Java to run)
module load java/11.0.2
export PATH=/sc/arion/projects/NGSCRC/Tools/MiXCR/mixcr-4.7.0-0/bin:$PATH

FLOW_CELL="FC07"
SAMPLE_LIST=${FLOW_CELL}_sample_list.txt
SAMPLE_DIR=$(head -n "${LSB_JOBINDEX}" "${SAMPLE_LIST}" | tail -n 1)
SAMPLE_ID=$(basename ${SAMPLE_DIR})
OUTPUT_FOLDER="/sc/arion/projects/NGSCRC/Work/NRG-GY003/TCRseq/MiXCR"

echo `date`
echo "Starting MiXCR for ${SAMPLE_ID}"

## MiXCR has a pre-set for Cellecta DNA sequencing so we will use that
mixcr analyze cellecta-human-dna-xcr-umi-drivermap-air \
    ${SAMPLE_DIR}/${SAMPLE_ID}_R1_001.fastq.gz \
    ${SAMPLE_DIR}/${SAMPLE_ID}_R2_001.fastq.gz \
    ${OUTPUT_DIR}/${SAMPLE_ID}

echo "Finished MiXCR for ${SAMPLE_ID}"
echo `date`


------------------------------------------------------------

Successfully completed.

Resource usage summary:

    CPU time :                                   3.61 sec.
    Max Memory :                                 129 MB
    Average Memory :                             92.00 MB
    Total Requested Memory :                     32768.00 MB
    Delta Memory :                               32639.00 MB
    Max Swap :                                   -
    Max Processes :                              6
    Max Threads :                                19
    Run time :                                   4 sec.
    Turnaround time :                            6 sec.

The output (if any) follows:

Wed Aug 21 17:50:01 EDT 2024
Starting MiXCR for P2A10_GADCIWn2_S10
Finished MiXCR for P2A10_GADCIWn2_S10
Wed Aug 21 17:50:04 EDT 2024


PS:

Read file <err.137529137.1> for stderr output of this job.

```
</details>

But I did get a short and sweet message in my `.err` file:

```
License error: ConnectionError
License manager thread died.
```

This was another case of me knowing what I *didn't* do, which led me to figure out what I needed to do. 

**In this case, the MiXCR software requires an academic license to run.**

While the error message said there was a `ConnectionError`, I knew this was because I hadn't yet generated a license, so there was nothing to connect to!

So the fix was to go to the MiXCR website and generate a license. 

### Summary:
* **Error:** "License error" -- I knew I didn't yet have a license  
* **Fix:** Generate a MiXCR license so the software could run

## 🪲 Example 3: The fix is not obvious
**In this example, the error message is very misleading -- this happens more often than you think!** 

At this point, I've loaded Java, I've gotten my license, and I'm ready to rock. 

I again have an `.out` file that looks fine -- and this one even shows output for MiXCR actually running!
<details>
  <summary>Full output file</summary>

```
Sender: LSF System <lsfadmin@lc01g15>
Subject: Job 137530828[1]: <MyArrayJob[1-1]> in cluster <chimera> Done

Job <MyArrayJob[1-1]> was submitted from host <li03c04.chimera.hpc.mssm.edu> by user <monsok03> in cluster <chimera> at Wed Aug 21 17:57:59 2024
Job was executed on host(s) <lc01g15>, in queue <premium>, as user <monsok03> in cluster <chimera> at Wed Aug 21 17:58:05 2024
</hpc/users/monsok03> was used as the home directory.
</sc/arion/projects/NGSCRC/Scripts/NRG-GY003/TCRseq> was used as the working directory.
Started at Wed Aug 21 17:58:05 2024
Terminated at Wed Aug 21 17:58:16 2024
Results reported at Wed Aug 21 17:58:16 2024

Your job looked like:

------------------------------------------------------------
# LSBATCH: User input
#!/bin/bash
#BSUB -P acc_NGSCRC 		   # Zamarin Lab allocation account
#BSUB -n 1	    		   # total number of compute cores
#BSUB -W 08:00	    		   # walltime in HH:MM
#BSUB -q premium    		   # queue 
#BSUB -R "rusage[mem=32GB]"        # 32 GB of memory (32 GB per core)
#BSUB -J "MiXCR_FC7[1-94]"         # Job name (batch job for flow cell 7 TCR-seq)
#BSUB -o out.%J.%Im		   # output file name (%I=job array index, %J=jobID)
#BSUB -e err.%J.%I  		   # error file name (same nomenclature as above)
#BSUB -L /bin/bash		   # Initialize the execution environment


## Load MiXCR and Java (needs Java to run)
module load java/11.0.2
export PATH=/sc/arion/projects/NGSCRC/Tools/MiXCR/mixcr-4.7.0-0/bin:$PATH

FLOW_CELL="FC07"
SAMPLE_LIST=${FLOW_CELL}_sample_list.txt
SAMPLE_DIR=$(head -n "${LSB_JOBINDEX}" "${SAMPLE_LIST}" | tail -n 1)
SAMPLE_ID=$(basename ${SAMPLE_DIR})
OUTPUT_FOLDER="/sc/arion/projects/NGSCRC/Work/NRG-GY003/TCRseq/MiXCR"

echo `date`
echo "Starting MiXCR for ${SAMPLE_ID}"

## MiXCR has a pre-set for Cellecta DNA sequencing so we will use that

mkdir -p ${OUTPUT_FOLDER}/${SAMPLE_ID}

mixcr analyze cellecta-human-dna-xcr-umi-drivermap-air \
    ${SAMPLE_DIR}/${SAMPLE_ID}_R1_001.fastq.gz \
    ${SAMPLE_DIR}/${SAMPLE_ID}_R2_001.fastq.gz \
    ${OUTPUT_DIR}/${SAMPLE_ID}

echo "Finished MiXCR for ${SAMPLE_ID}"
echo `date`


------------------------------------------------------------

Successfully completed.

Resource usage summary:

    CPU time :                                   10.01 sec.
    Max Memory :                                 391 MB
    Average Memory :                             242.50 MB
    Total Requested Memory :                     32768.00 MB
    Delta Memory :                               32377.00 MB
    Max Swap :                                   -
    Max Processes :                              5
    Max Threads :                                20
    Run time :                                   11 sec.
    Turnaround time :                            17 sec.

The output (if any) follows:

Wed Aug 21 17:58:06 EDT 2024
Starting MiXCR for P2A10_GADCIWn2_S10

>>>>>>>>>>>>>>>>>>>>>>> mixcr align <<<<<<<<<<<<<<<<<<<<<<<
Running:
mixcr align --report /P2A10_GADCIWn2_S10.align.report.txt --json-report /P2A10_GADCIWn2_S10.align.report.json --preset cellecta-human-dna-xcr-umi-drivermap-air --save-output-file-names /P2A10_GADCIWn2_S10.align.list.tsv /sc/arion/projects/NGSCRC/master_data/NRG-GY003/TCRseq/Cellecta_DM_101799_FC09_data_release_240723/240723_101799_FC09/FC09/P2A10_GADCIWn2_S10/P2A10_GADCIWn2_S10_R1_001.fastq.gz /sc/arion/projects/NGSCRC/master_data/NRG-GY003/TCRseq/Cellecta_DM_101799_FC09_data_release_240723/240723_101799_FC09/FC09/P2A10_GADCIWn2_S10/P2A10_GADCIWn2_S10_R2_001.fastq.gz /P2A10_GADCIWn2_S10.alignments.vdjca
The following tags and their roles will be associated with each output alignment:
  Payload tags: R1, R2
  Molecule tags: UMI(SQ)
Finished MiXCR for P2A10_GADCIWn2_S10
Wed Aug 21 17:58:16 EDT 2024


PS:

Read file <err.137530828.1> for stderr output of this job.


```
</details>

Here's MiXCR running, yay!
```
>>>>>>>>>>>>>>>>>>>>>>> mixcr align <<<<<<<<<<<<<<<<<<<<<<<
Running:
mixcr align --report /P2A10_GADCIWn2_S10.align.report.txt --json-report /P2A10_GADCIWn2_S10.align.report.json --preset cellecta-human-dna-xcr-umi-drivermap-air --save-output-file-names /P2A10_GADCIWn2_S10.align.list.tsv /sc/arion/projects/NGSCRC/master_data/NRG-GY003/TCRseq/Cellecta_DM_101799_FC09_data_release_240723/240723_101799_FC09/FC09/P2A10_GADCIWn2_S10/P2A10_GADCIWn2_S10_R1_001.fastq.gz /sc/arion/projects/NGSCRC/master_data/NRG-GY003/TCRseq/Cellecta_DM_101799_FC09_data_release_240723/240723_101799_FC09/FC09/P2A10_GADCIWn2_S10/P2A10_GADCIWn2_S10_R2_001.fastq.gz /P2A10_GADCIWn2_S10.alignments.vdjca
The following tags and their roles will be associated with each output alignment:
  Payload tags: R1, R2
  Molecule tags: UMI(SQ)
Finished MiXCR for P2A10_GADCIWn2_S10
Wed Aug 21 17:58:16 EDT 2024
```
But then I got this horrible-looking `.err` file:
```
Please copy the following information along with the stacktrace:
   Version: 4.7.0; built=Wed Aug 07 15:19:48 EDT 2024; rev=976ba14139; lib=repseqio.v5.1
        OS: Linux
      Java: 11.0.2
  Abs path: /sc/arion/projects/NGSCRC/Scripts/NRG-GY003/TCRseq
  Cmd args: align --report /P2A10_GADCIWn2_S10.align.report.txt --json-report /P2A10_GADCIWn2_S10.align.report.json --preset cellecta-human-dna-xcr-umi-drivermap-air --save-output-file-names /P2A10_GADCIWn2_S10.align.list.tsv /sc/arion/projects/NGSCRC/master_data/NRG-GY003/TCRseq/Cellecta_DM_101799_FC09_data_release_240723/240723_101799_FC09/FC09/P2A10_GADCIWn2_S10/P2A10_GADCIWn2_S10_R1_001.fastq.gz /sc/arion/projects/NGSCRC/master_data/NRG-GY003/TCRseq/Cellecta_DM_101799_FC09_data_release_240723/240723_101799_FC09/FC09/P2A10_GADCIWn2_S10/P2A10_GADCIWn2_S10_R2_001.fastq.gz /P2A10_GADCIWn2_S10.alignments.vdjca
picocli.CommandLine$ExecutionException: Error while running command align java.nio.file.AccessDeniedException: /P2A10_GADCIWn2_S10.align.list.tsv
	at com.milaboratory.mixcr.cli.Main.registerExceptionHandlers$lambda-17(SourceFile:420)
	at picocli.CommandLine.execute(CommandLine.java:2088)
	at com.milaboratory.mixcr.cli.Main.execute(SourceFile:105)
	at com.milaboratory.mixcr.cli.CommandAnalyze$Cmd$PlanBuilder.executeSteps(SourceFile:543)
	at com.milaboratory.mixcr.cli.CommandAnalyze$Cmd.run0(SourceFile:444)
	at com.milaboratory.mixcr.cli.MiXCRCommand.run(SourceFile:37)
	at picocli.CommandLine.executeUserObject(CommandLine.java:1939)
	at picocli.CommandLine.access$1300(CommandLine.java:145)
	at picocli.CommandLine$RunLast.executeUserObjectOfLastSubcommandWithSameParent(CommandLine.java:2358)
	at picocli.CommandLine$RunLast.handle(CommandLine.java:2352)
	at picocli.CommandLine$RunLast.handle(CommandLine.java:2314)
	at picocli.CommandLine$AbstractParseResultHandler.execute(CommandLine.java:2179)
	at picocli.CommandLine$RunLast.execute(CommandLine.java:2316)
	at com.milaboratory.mixcr.cli.Main.registerLogger$lambda-32(SourceFile:539)
	at picocli.CommandLine.execute(CommandLine.java:2078)
	at com.milaboratory.mixcr.cli.Main.execute(SourceFile:105)
	at com.milaboratory.mixcr.cli.Main.main(SourceFile:101)
Caused by: java.nio.file.AccessDeniedException: /P2A10_GADCIWn2_S10.align.list.tsv
	at java.base/sun.nio.fs.UnixException.translateToIOException(UnixException.java:90)
	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:111)
	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:116)
	at java.base/sun.nio.fs.UnixFileSystemProvider.newByteChannel(UnixFileSystemProvider.java:215)
	at java.base/java.nio.file.spi.FileSystemProvider.newOutputStream(FileSystemProvider.java:478)
	at java.base/java.nio.file.Files.newOutputStream(Files.java:219)
	at com.milaboratory.o.e$a.<init>(SourceFile:85)
	at com.milaboratory.mixcr.cli.CommandAlign$Cmd.alignedWriter(SourceFile:1564)
	at com.milaboratory.mixcr.cli.CommandAlign$Cmd.run1(SourceFile:1305)
	at com.milaboratory.mixcr.cli.MiXCRCommandWithOutputs.run0(SourceFile:69)
	at com.milaboratory.mixcr.cli.MiXCRCommand.run(SourceFile:37)
	at picocli.CommandLine.executeUserObject(CommandLine.java:1939)
	at picocli.CommandLine.access$1300(CommandLine.java:145)
	at picocli.CommandLine$RunLast.executeUserObjectOfLastSubcommandWithSameParent(CommandLine.java:2358)
	at picocli.CommandLine$RunLast.handle(CommandLine.java:2352)
	at picocli.CommandLine$RunLast.handle(CommandLine.java:2314)
	at picocli.CommandLine$AbstractParseResultHandler.execute(CommandLine.java:2179)
	at picocli.CommandLine$RunLast.execute(CommandLine.java:2316)
	at com.milaboratory.mixcr.cli.Main.registerLogger$lambda-32(SourceFile:539)
	at picocli.CommandLine.execute(CommandLine.java:2078)
	... 15 more

```
I'll be honest, when I first started coding, I would be totally freaked out by an error message like this 🤯

But, with a bit of practice, I now know what to look for 🔎

Here are the steps to debugging this one, which is a bit more complicated.

### Step 1: Find the error
Here's the key bit:
```
CommandLine$ExecutionException: Error while running command align java.nio.file.AccessDeniedException
```
If you notice, this is the only place in the message where the word **error** actually appears
* So I know that whatever is going on in this line is what's causing the problem.
* And the actually important bit is the `AccessDeniedException`.
* Even if I don't know what the rest of it means (which I don't), this means that I'm getting some kind of Access Denied error. 

Now, this is a bit strange. 

Typically, an Access Denied error happens when you are trying to save a file or navigate to a directory that you don't have permission to access.
* *Remember, we learned about read, write, execute (`rwx`) permissions in the Command Line class. You must have permission to interact with files on the cluster.*
* This was strange because I knew that I was the one running the script and generating the output files and directories.
* So it was unlikely to *really* be an issue of incorrect permissions, because I owned all the files and folders that I was interacting with, so I automatically have permission to access them. 

### Step 2: Find the source
The next step was to go back to the script to see if I could find any errors or typos. 

If it's not immediately clear what the problem is, it's most likely the result of a typo. 

Here is the script -- take a look to see if you can find the issue.

```
## Load MiXCR and Java (needs Java to run)
module load java/11.0.2
export PATH=/sc/arion/projects/NGSCRC/Tools/MiXCR/mixcr-4.7.0-0/bin:$PATH

FLOW_CELL="FC07"
SAMPLE_LIST=${FLOW_CELL}_sample_list.txt
SAMPLE_DIR=$(head -n "${LSB_JOBINDEX}" "${SAMPLE_LIST}" | tail -n 1)
SAMPLE_ID=$(basename ${SAMPLE_DIR})
OUTPUT_FOLDER="/sc/arion/projects/NGSCRC/Work/NRG-GY003/TCRseq/MiXCR"

echo `date`
echo "Starting MiXCR for ${SAMPLE_ID}"

## MiXCR has a pre-set for Cellecta DNA sequencing so we will use that

mkdir -p ${OUTPUT_FOLDER}/${SAMPLE_ID}

mixcr analyze cellecta-human-dna-xcr-umi-drivermap-air \
    ${SAMPLE_DIR}/${SAMPLE_ID}_R1_001.fastq.gz \
    ${SAMPLE_DIR}/${SAMPLE_ID}_R2_001.fastq.gz \
    ${OUTPUT_DIR}/${SAMPLE_ID}

echo "Finished MiXCR for ${SAMPLE_ID}"
echo `date`
```
I'll give you a moment 🕰️...

If you didn't immediately spot it, no worries -- I'll show you a tool later that can help with this.

If you notice, I took my own advice of not "hard-coding" my script. 

That is, I defined variables at the beginning of my script that I reference later. 

But, I made a typo in the variable name that I *defined* vs the variable name that I *called* later in my script.
* I defined a variable called `${OUTPUT_FOLDER}` pointing to a subfolder in my Work directory where the output from this analysis will go.
* But the `${OUTPUT_FOLDER}` variable doesn't appear in the `mixcr` code!
* Instead, I had the idea to streamline the naming of the variables, and called a (non-existent) variable named `${OUTPUT_DIR}` in the `mixcr` statement -- but forgot to rename the variable from `${OUTPUT_FOLDER}` to `${OUTPUT_DIR}` above!

### Step 3: Figure out the error
So the issue was just that I created a variable but then incorrectly called a new one that I hadn't yet defined.

Why was I getting an "Access Denied" error, then? 
* The script was looking for a path corresponding to the `${OUTPUT_DIR}` folder (which didn't exist because I didn't define the variable)
* It assumed it didn't have access to the `${OUTPUT_DIR}` path because it couldn't find it
* It generated an Access Denied error because it couldn't find `${OUTPUT_DIR}`

The fix is easy, in that I just have to streamline my variable names. I updated it so that both were called `${OUTPUT_DIR}`

### Summary:
* **Error:** "Access Denied."
	* This didn't make sense since I should have access to all the files I am reading and writing.
 	* In reviewing the script, I found it was a simple typo. 
* **Fix:** Fix the typo! I removed `${OUTPUT_FOLDER}` and had everything be `${OUTPUT_DIR}`

> **💡Tip:** I mentioned that there's a tool that can help debug these cases.
>
> Text editors* like [VS Code](https://code.visualstudio.com/) are great for writing your scripts.  
> **Technically, VS Code is an Integrated Development Environment, like RStudio. But it's a great text editor on its own.*
>
> They color-code your text and auto-highlight variables when you select them, so you can see if you have made any typos.
>
> Here's what the code chunk above looks like in VS Code -- you can easily see the typo:
> ![VSCode](assets/VS_Code.png)
>
> You can write your scripts in VS Code and then copy/paste the script into a file on the cluster to run it. 

After updating my variable names, I re-ran my code, and everything worked beautifully! 

## Conclusion 
Of course, these are just a few examples of the things you might need to debug. 

But I hope I was able to illustrate:
* The first places you should look to find errors
* The thought processes to use when approaching errors
* That the text of the errors themselves can be misleading! Try to think things through logically before getting sidetracked chasing down an erroneous error message.

The other thing to do first, especially if you are confronted with a scary error like in Example 3, is to Google it! 

Chances are, someone has had the same problem (or similar), and a kind soul has walked them through the steps to fix it. 

Just copy/paste the error into Google and it should point you in the right direction.

Happy bug hunting! 🪲
