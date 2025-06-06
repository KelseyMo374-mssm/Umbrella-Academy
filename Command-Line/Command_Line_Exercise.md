# "Getting Started with The Command Line" Exercise

This exercise introduces common bioinformatics command line activities on a High-Performance Computing (HPC) cluster.

These activities include:
- 🗺️Navigating the command line
- 📝Creating and editing simple scripts
- ☑️Executing array jobs (using the LSF job scheduler) on Minerva, Mount Sinai's HPC.
- 📊Aggregating quality control reports using MultiQC

By the end of this exercise, you will be able to:
- Run [fastp](https://github.com/OpenGene/fastp), a "tool designed to provide ultrafast all-in-one preprocessing and quality control for FastQ data," on four RNA-seq .fastq files
- Aggregate the QC documents using [MultiQC](https://github.com/MultiQC/MultiQC), a "tool to create a single report with interactive plots for multiple bioinformatics analyses across many samples."

## Step 1: Locate the Data
Using `cd` (change directory), navigate to the `master_data` folder on Minerva containing the raw .fastq files for the exercise:

``` Shell
cd /sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy
```
*If you are following along on GitHub, these files can be found in the `master_data` [folder](master_data) within this directory.*

These .fastqs are from S. cerevisiae (yeast) samples and are 101bp paired-end strand-specific RNA-seq files, sub-sampled to 50,000 reads (from [NF-core test data repository](https://github.com/nf-core/test-datasets/tree/rnaseq)).

<!-- HTML_START -->
<details>
<summary>Sample details</summary>

**Sample info:**

| run_accession | experiment_alias | read_count | sample_title |
|---------------|------------------|------------|--------------|
| SRR6357070 | GSM2879618 | 47629288 | Wild-type total RNA-Seq biological replicate 1 |
| SRR6357071 | GSM2879619 | 68628914 | Wild-type total RNA-Seq biological replicate 2 |
| SRR6357072 | GSM2879620 | 54771596 | Wild-type total RNA-Seq biological replicate 3 |
| SRR6357073 | GSM2879621 | 56006930 | Rap1-AID degron no induction total RNA-Seq biological replicate 1 |

**Sample citation:**

> Andrew C K Wu, Harshil Patel, Minghao Chia, Fabien Moretto, David Frith, Ambrosius P Snijders, Folkert J van Werven. Repression of Divergent Noncoding Transcription by a Sequence-Specific Transcription Factor. Mol Cell. 2018 Dec 20;72(6):942-954.e7. doi: 10.1016/j.molcel.2018.10.018.
> [Pubmed](https://pubmed.ncbi.nlm.nih.gov/30576656/) [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE110004)

</details>
<!-- HTML_END -->

Each subfolder contains the .fastq files for each of the four samples we are using for this exercise. 

>💡**Tip**: As you move throughout your directories, creating and modifying files, it can become confusing.
> - If you get lost or disoriented, just type `pwd` (for "print working directory"). This will output the full path of your current directory.
> - You can also type `ls` to list all the files in your current directory. Try it out now!

Let's look inside one of the subfolders:
``` Shell
cd SRR6357070
ls
```
You'll notice there are two .fastq files for each sample. These files come from paired-end, strand-specific RNA-seq, so read 1 is the forward read and read 2 is the reverse read. 

Let's go back to the main data folder. This command moves us one folder up in the file hierarchy:
``` Shell
cd ..
```

## Step 2: Create a Sample List
Next, we create a sample list pointing to the names and locations of the raw .fastq files. This will tell your script what files to look for and where they are stored.

Run `printf '%s\n' "$PWD"/* >FastP_practice_samplelist_<MY_NAME>.txt`.

Replace `<MY_NAME>` with, you guessed it, your name. For example, mine is:
``` Shell
printf '%s\n' "$PWD"/* >FastP_practice_samplelist_Kelsey.txt
```
For the rest of the exercise, we'll just refer to this file as `"samplelist.txt"`

<!-- HTML_START -->
<details>
  <summary><b>Breaking down the command</b></summary>

- **`printf '%s\n'`**
  - `printf`: used to format and print text.
  - Format string `'%s\n'`: specifies that it should print a string (`%s`) followed by a new line (`\n`).
- **`"$PWD"/*`**
  - `$PWD`: path to the current working directory
  - `*`: the asterisk is a wildcard character that matches any file or directory within the current directory.
    - We are in the `/sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy` folder; this is the working directory that will be printed
    - This will be followed by the contents of the directory, which are the subfolders containing the .fastq files for each sample.
- **`>samplelist.txt`**
  - Output redirection operator `>`: sends the output of a command to a file instead of printing it in the terminal.
  - You decide the file's name and extension. Here we are creating a .txt file. 
</details>

Putting it all together, this command will:
  - *Expand the wildcard:*
      The `*` wildcard will be expanded to match all files and directories in the current working directory.
      
  - *Print the paths:*
      For each file or directory matched, the `printf` command will print its full path, followed by a new line.
    
  - *Save to a text file:*
      Instead of displaying the output of the command in the terminal, `>` will redirect the output to a new file, `samplelist.txt`.
    
>⚠️**Important**: Be careful when using `>`  
> - If a file with the same name does not exist, defining the name and the file extension in the command will create a new file.
> - If you use the name of a file that already exists, **this command will overwrite the contents of the existing file.** 


## Step 3: Modify your Sample List
The sample list you just created will look like this:

|  | 
|---------------|
| /sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy/README.txt |
| /sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy/SRR6357070 |
| /sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy/SRR6357071 |
| /sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy/SRR6357072 |
| /sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy/SRR6357073 |

Note that the first line contains a `README.txt` file. This is a file in the `/sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy` folder that includes some sample details.

It is good practice to have README files in your master data folders so you and current/future lab members know what the files are, where the data comes from, what species it is, the read length, etc. 

But we will get an error if we try to run an array job using this file as our sample list as-is, because the array job is written to look for subdirectories containing .fastq files. It will see the `README.txt` file and throw an error when it finds out it isn't a subdirectory like it expected.  

There are many ways to modify the file to remove the line; this is one way to do it quickly using the command line:

`Nano` is one of many Linux text editors you can use to modify files on the command line. Use `nano` followed by the name of your sample list to open the file interactively.

>💡**Tip:** This is a great time to try out "Tab to complete"!
> - Since your file will be named something like `FastP_practice_samplelist_Kelsey.txt`, start typing `nano Fa` and then hit Tab.
> - It should auto-complete the file name until `FastP_practice_samplelist_`
> - Then enter the first letter of your name and hit Tab again, entering letters and hitting Tab until you have completed your file's name.  

Once the file is open, use the arrow keys to navigate to the line we wish to remove, then delete the line with `ctrl + k`.

To save and quit: `ctrl + o` ("Write Out" the file, saving the output), `Enter` (when prompted to provide the file name to be written, we hit Enter to keep the same name), and `ctrl + x` (exit `nano`).

## Step 4: Create the Script
Navigate to the `Scripts` directory and create your own subdirectory using `mkdir` (make directory):
``` Shell
cd /sc/arion/projects/NGSCRC/Scripts/Umbrella_Academy

mkdir Kelsey
```
We need the sample list in the same folder as the script, but it's currently still in the `master_data` folder where it was made. We will use `mv` to *move* your sample list to the directory you just created. 

Because we have navigated to the `Scripts` directory, we need to append the location of the sample list to this command to locate it.
The command will look like `${path to sample list}/FastP_practice_samplelist_<MY_NAME>.txt <MY_NAME>`:
``` Shell
mv /sc/arion/projects/NGSCRC/master_data/test/Umbrella_Academy/FastP_practice_samplelist_Kelsey.txt Kelsey
```
>💡**Tip:** The basic syntax for most UNIX commands is essentially:
> - "What are you doing?" (*action*)
> - "What are you doing it to?" (*input*)
> - "What is the desired result?" (*output*)
>  
> ➡️ Applying this here:
> - "What are you doing?" *moving a file*
> - "What are you doing it to?" *the sample list*
> - "What is the desired result?" *move the file to this specific directory*

Now, make a copy of the `FastP_practice_annotated.sh` script using `cp`. 

In one step, you can also append your name to the end of the file and move it into the subdirectory that you just created. The command will look like `cp FastP_practice_annotated.sh <MY_NAME>/FastP_practice_<MY_NAME>.sh`:
``` Shell
cp FastP_practice_annotated.sh Kelsey/FastP_practice_Kelsey.sh
```
Do you see the UNIX syntax in action here?

*If you are following along on GitHub, follow [this link](FastP_practice_annotated.sh) to access the `FastP_practice_annotated.sh` script.*

## Step 5: Modify the Script
Now you have your own sample list and your own shell script that you can modify to run [fastp](https://github.com/OpenGene/fastp) yourself! 

Navigate to your folder and open the script for editing using `nano`:
``` Shell
cd Kelsey
nano FastP_practice_Kelsey.sh
```
Revise according to the instructions contained in the file.

Don't forget to save and exit (`ctrl + o`, `Enter`, and `ctrl + x`).

>💡**Tip:** If you ever want to create your own blank file, you can type `nano` and then a new file name + file extension.

## Step 6: Submit the Job
When the file is ready to be executed, we will use LSF syntax to submit the job. 

[LSF (Load Sharing Facility)](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=overview-lsf-introduction) is the Minerva job scheduling platform, which determines your job's priority, available resources, elapsed time, etc. 

Such schedulers are essential for HPC environments, where massive amounts of computing power must be shared across many individuals. LSF is the one used on Minerva, but there are others (e.g. Slurm).

>💡**Tip:** Because array jobs run the same script over multiple samples, it is good practice to test your script on a single sample first. Debug your script until it works on one sample, and then scale up.

We will first test the LSF batch job on a single sample:
``` Shell
bsub -J MyArrayJob[1] < FastP_practice_Kelsey.sh
```
Let's break down the command:
- `bsub:` The syntax to submit a job using LSF is `bsub`.
- `-J MyArrayJob[1]:` We specify the job type (`-J`) as an array job (`MyArrayJob`) and specify the samples included in the array.  
  Typical array syntax would have you specify the span of samples in the array (e.g. `[1-2]` over samples 1 and 2). We *could* tell it to run it from sample 1 to sample 1 (`[1-1]`), but it understands that when we specify `[1]`, we want it to run for a single sample.
- `< FastP_practice_Kelsey.sh:` This is how we tell it to submit (`<`) your script (`FastP_practice_Kelsey.sh`) to the LSF batch job we just described. 

Because we have written the script to run as an array job, we must still submit it as an array, even when testing on a single sample. 

To check on the progress of the job (whether it is pending or running, how long it has been running, if it has finished), we can use `bjobs`. To continually check on the progress, we use `watch bjobs`. 

## Step 7: Debug (as Needed)
Once the job has finished, we can check to see whether it was successful. 

View the log and error files that have been generated in your `Scripts` folder. If we just want to look at a file and not edit it, we can use `less` to open it, and `ctrl + z` to close it.

**Were your jobs successful? How do you know?**

You can also check the actual output in the `Work` directory. We haven't been to this folder yet, but we created a new output folder in the `Work` directory in the array job script. 

The command to access it will look like `cd /sc/arion/projects/NGSCRC/Work/Umbrella_Academy/trimmed_reads_practice/<MY_NAME>`. 

Is there output in the folder?

> 🐞 **Did your job fail?** Here are a few troubleshooting ideas:
> - Check that you updated all the sections of the script that needed to be updated. Make sure you remembered to change the `${MY_NAME}` variable to your name.
> - Check that the variables you are calling point to real files. For example, if you named your sample list `FastP_practice_samplelist_JohnnyCoolGuy.txt` but defined the `${MY_NAME}` variable in your script as `John`, your script won't find your sample list.
> - Make sure your sample list contains only 4 rows, one for each sample, and points to the correct sample directories for the raw .fastq files.
> - Make sure you typed your `bsub` command correctly, including the name of your script.
> - Make sure you are launching the job from the `Scripts` folder that contains your script. 

## Step 8: Run the Final Array Job
Once you are satisfied that your job works for one sample, you can submit your array job for the remaining samples:
``` Shell
bsub -J MyArrayJob[2-4] < FastP_practice_Kelsey.sh
```
> 💡**Tip:** Note that you don't have to restart the array from 1 (e.g. `MyArrayJob[1-4]`) since we've already successfully finished the first sample.
> If you do run from `[1-4]`, note that it will overwrite your output for sample 1 from your previous job. 

Now that we submitted an actual array job, you can now monitor multiple jobs at once using `bjobs` or `watch bjobs`. Are some running and some still pending? 

Once they have finished running, check the output and error files for each sample and check the output in the `Work` folder to make sure they completed successfully.

## Step 9: Request an Interactive Shell
[fastp](https://github.com/OpenGene/fastp) generates summary statistics and quality control (QC) reports. You can review these individually, but this quickly becomes tedious over many samples, and it is difficult to appreciate trends and outliers.

[MultiQC](https://github.com/MultiQC/MultiQC) is a powerful tool to aggregate results from bioinformatics analyses and generate an interactive and user-friendly .html report. 

To run MultiQC, we will practice using an **interactive shell**, where we request compute resources and then interact directly with the shell by typing commands and getting output from those commands. 

**Why are we doing this now?**

- When we typed basic commands like `cd`, `mkdir`, `mv`, etc., we were also in an interactive shell. These commands are simple and use very little memory and computing resources, so we were able to use the login shell. That is, we didn't have to specifically request time and resources, we just used the shared resources available to everyone on the login node.
- When we ran our array job using a shell script, we were using a non-interactive shell. We requested the resources to run our job within the body of the script, the shell ran the commands in the script and then exited when the script finished, without us interacting with the shell.

>⚠️**Important:** If you are not using a shell script and are doing anything other than very simple tasks, **you must request an interactive shell!** The HPC is a shared computing resource, and running time- and memory-intensive jobs on the login node slows things down for everyone else. Requesting an interactive node with the time and memory needed for your activity ensures HPC resources are allocated appropriately.

So how do we request an interactive shell? Like this!

``` Shell
bsub -Is -n 1 -R "rusage[mem=10000]" -P acc_NGSCRC -W 60 /bin/bash
```
Let's break it down:
- `bsub`: You know this one by now 🙂
- `-Is`: This is the command to request an interactive shell
- `-n 1`: This will not take much memory and doesn't need to be parallelized, so we are requesting 1 compute core
- `-R "rusage[mem=10000]`: This is the memory request. In LSF, memory is requested in MB, so we are requesting 10,000 MB or 10GB
- `-P acc_NGSCRC`: This is our project allocation account; each bsub job needs to include the associated project.
- `-W 60`: This is the wall time -- here we are requesting 60 minutes
- `/bin/bash`: This specifies that you want to run an interactive Bash shell on the compute node once LSF allocates the resources. This means you will be working in a Bash environment, just like we are when we use the login node, so we can use familiar Bash syntax like `cd`, `mv`, etc.

## Step 10: Aggregate the QC Reports Using MultiQC
Now that we have our interactive shell, we can run MultiQC on the fastp QC files. 

MultiQC runs using Python, so we need to load the Python module (like we loaded the fastp module in our shell script):
``` Shell
module load python
```
This loads the most recent version of Python available on the cluster; if we wanted to load a specific version (some programs are only compatible with specific versions of other programs), we could type `module avail python` to see all the versions of Python that are available on Minerva. 

If you type `module list`, it will list the modules we currently have loaded. You will see that several other modules were loaded when we loaded Python. 

To keep all of our MultiQC reports separate, let's navigate into the folder in the Work directory containing our fastp output and print the working directory path:
``` Shell
cd /sc/arion/projects/NGSCRC/Work/Umbrella_Academy/trimmed_reads_practice/Kelsey
pwd
```
Copy that path by highlighting it in the terminal and right-clicking on it. We'll run MultiQC from within our personal folders so the .html reports will be output here.

MultiQC is available on Minerva, so we can execute it using the following Python code. Right click to paste the path of your fastp output so you don't need to type it all out.
``` Shell
python -m multiqc /sc/arion/projects/NGSCRC/Work/Umbrella_Academy/trimmed_reads_practice/Kelsey
```
If you look in the folder now using `ls`, you will see a report, `multiqc_report.html`, and a subfolder, `multiqc_data`. The subfolder contains data needed to generate the report, but we are only interested in the `multiqc_report.html` file. 

## Step 11: Download the MultiQC Report Locally
If you try to view the `multiqc_report.html` file using the UNIX commands we've learned (like `less` or `nano`), you'll notice that it's not very easy to understand. This is because it's written in .html, which is only really interpretable by browsers like Chrome and Safari.

So how can we view the report? 

There's no way to open it on a browser while it's still stored on the HPC, so we need to find a way to download it to our personal computers. 

This requires a **FTP** or **"file transfer protocol"** to transfer the .html report from Minerva to our laptop. 

>💡**Tip:** Use one of the below FTP programs to move the `multiqc_report.html` from Minerva to your laptop:  
>- 🪟**Windows FTP:** [WinSCP](https://winscp.net/eng/index.php)  
>- 🍏**Mac FTP:** [Cyberduck](https://cyberduck.io/)

Now you can open the `multiqc_report.html` file in your browser and take a look!

## Step 12: Celebrate your Success
### 🎉Congratulations! 
You successfully ran [fastp](https://github.com/OpenGene/fastp) on the HPC using an array job, and generated a [MultiQC](https://github.com/MultiQC/MultiQC) report! Pat yourself on the back! 👋

### Learning Recap
After completing this exercise, you learned how to:
- 🗺️Navigate the command line
  - Move to different folders by changing your working directory
  - Make new directories
  - View the contents of directories
- 📝Create and manipulate files
  - Copy and create new files
  - View and edit the contents of files
  - Move files from one folder to another
- ☑️Launch a job using LSF
  -  Understand how to structure an array job script
  -  Submit an array job
  -  Monitor the status of your jobs
- 📊Run an interactive job and view .html files
  -  Request an interactive shell
  -  Load modules using the command line
  -  Run a job using the command line
  -  Download files from the HPC to your local computer


### Resources
- Clean example script: [FastP_practice_clean.sh](FastP_practice_clean.sh).
  - This gives you an example of what an actual script might look like.
  - This clean script is ultimately very simple because we ran [fastp](https://github.com/OpenGene/fastp) using only the default settings, meaning we only specified input and output files in our command.
  - For more complex scripts, I highly suggest including the definition of what each variable in the program command does, and why.
    - This is helpful to jog your memory when you return to a script after some time, so you can remember the rationale behind selecting the specific parameters you chose.
    - It's also helpful when writing the methods sections of your papers and proposals, and justifying your choices to reviewers.

- Command line cheat sheet: [command_line_cheat_sheet.md](command_line_cheat_sheet.md).
  - This is a cheat sheet with all the commands we learned today while completing this exercise.
  - Keep this handy to refer to when you're navigating the command line yourself!
