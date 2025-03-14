# CELLO_pipeline
## Introduction
Nextflow based CELLO-seq pipeline optimised for SLURM clusters. This specific code take raw fastq reads from ONT sequencing runs and outputs qc and and demultiplexed files. Note that if steps have same number, they are synchronous. Please see XXX for mroe details.

## Running step 1 
### Set-up 
1. cd to your directory with your raw CELLO-seq fastq files
2. Create a parameter file (e.g. 1_parameters.json) to specify how your pipeline should run 
* note that the cpus / time / memory you ask is per task
* Note that demultiplexing requires much more memory than the other tasks
* See CCB/1_params.json for an example for a large dataset. 
```
{
EXAMPLE
}
```

### Run nextflow 
```
module load nextflow # or Nextflow
nextflow -bg run https://github.com/franci-r/CELLO_pipeline  -main-script /CCB/step_1.nf -params-file 1_params.json -latest > step_1.log
```
- bg: background, enables run to continue even if you log out from cluster
- latest ensures version is being used 
- params-file specifies parameters file

### Output 
Open the run report file (run_report_YYYY-MM-DD_hh-ss.html)
```
tree 
```
- stdout is saved into step_1.log
- intermediate: directory with intermediate files
- output: directory with qc files and run report.  

### Working directory
Only remove work/ dir once you are happy with the outcome. This is important if you have issues with your pipeline and you want to use the -resume flag. 
```
rm -rf work/
```

## Description
Input: raw reads 
1: Concatenates them into one file
1: Removes reads >20kb
2: Yields % contamination in .txt
2: Runs dT and TSO adaptor qc (see .html)
2: Demultiplexes plate into barcodes

## Dealing with pipeline errors 
### Run report 
Each run creates a html report (run_report_YYYY-MM-DD_hh-ss.html). 
* At the end there is a table with all the processes and information:
1. Number of attempts
2. Resource usage
3. Resource allocation
4. Exit codes
5. Path to process directories

- If there are issues, you look for the error codes. If this is not enough, you can identify the problematic processes and cd ```work/problematic/process``` and ```ls -lha``` to see all files: .command.out and .command.err are the most useful. 
### work directory 
Nextflow runs each process (task) in a separate folder inside the work/ directory. If you have a failure, you can identify which task and its path either with the log file or the run_report. Do not edit the work directory as if so Nextflow will not be able to resume a failed process. If you are happy with the output, you can do whatever you want. 
```
cd pwd/work/problematic/dir
ls -lha
```
This will print output files and hidden files: 
1. .command.err # stderr of task
2. .comand.out # stdout of task
3. .command.log # graphs of resource usage (WIMM cluster)
4. .command.run # interpreted code , can sbatch it 
5. .exitcode # exitcode

### quitting a background nextflow process
Let's say you ran the pipeline, and you want to stop it:
* scancel job_ID does not work easily. As you are just cancelling a specific task, so the workflow will run it again. 
```
# identify the job ID
pgrep -fl nextflow
# get more information
ps -fp $ID
# cancel the background process
kill $ID
```

### resume a failed process
If the first 2 processes worked, but the last 3 failed. You can fix the issue and just rerun the remaining processes: 
```
nextflow -bg run https://github.com/franci-r/CELLO_pipeline.git -params-file parameters.json -resume > step_1.log
```
* resume flag makes nextflow not re-run successful processes. 

### Da fare 
1. Strategia d'errore - ignore
2. separa cpu e memoria in tipi
4. Porechop
6. Aggiungere FLARE?
10. pachettizza
