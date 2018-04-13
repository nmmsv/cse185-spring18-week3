# Week 4: Where do all these reference sequences come from, anyway? (part 2)
Skills covered: contig assembly, scaffolding, assembly evaluation

## 7. Compare contig assembly results with the rest of class
Use the whole class data (shared on Slack) to make two scatter plots: k-mer size vs maximum contig length, and k-mer size vs contig N50. You can use anything you like to make the plots. In each plot, indicate which data points are yours.

If your k-mer size is not in the data already, add it before you make the plot. If you weren’t able to get
your QUAST results, that’s OK, this data will be in the output from the next step, and you can make
the full plot later.

Did k-mer size influence the success of the assembly? Answer in your lab report discussion.

## 8. Use short jump library to scaffold minia contigs. 
These short contigs are certainly not a complete genome. Even if they cover all of the sequence in our genome (they might not; there could be missing regions), we do not know the correct order to put them in to reconstitute the real genome sequence.

Fortunately, we can use a second set of DNA sequencing data to help us figure out the correct way to join the contigs together. Our contigs were initially made by stitching together 101 bp reads from DNA fragments that were 180 bp long. Our second DNA library (short jump) consists of paired 37 bp reads from DNA fragments that were 3500 bp long. You will map these short jumps to your set of contigs.

If the forward member of a pair binds to one contig, and its reverse partner binds to a different contig, we know that those positions must be ~3500 bp apart in the actual genome. By combining all of the mapping data from these pairs, scaffolding software can figure out the optimal, correct way to link the contigs together.

You will use the program `SSPACE` for scaffolding. First, it will use the `bowtie` aligner (within the program) to map the shortjump reads to your `minia` contigs. Then, it will use those mapping results to connect the contigs into bigger scaffolds. Today you will generate a lot of new files, so make a subdirectory in your `week3` folder called “Thursday.” (no need to commit these files to github). Copy the `contig.fa` file from `minia` to this subdirectory. Also, move the `short_jump_1` and `short_jump_2` files to this directory. 

`SSPACE` requires a library file, telling the program the names of our `short_jump` files and some other information about them. Use `emacs` to create a new file with the general format below. You will have to look at the SSPACE user manual to figure out exactly what each column means (https://github.com/nsoranzo/sspace_basic/blob/master/F132-01%20SSPACE_Basic_User_Manual_v2.0.pdf). 

```
emacs sspace_library
```

*Example library file:*
```
Lib1 short_jump_1.fastq short_jump_2.fastq insertsize insertsizeerror orientation
```

You will have to specify the the insert size, the tolerable error for the insert size, and the orientation. Once you’ve made the library file, run `SSPACE` with the command below, which you will have to edit according to your exiting file names and your output name preference. A description of what each flag is doing follows the command.

```
SSPACE_Basic_v2.0.pl -l libraryfile -s minia_out.contigs.fa -z 100 -v 1 -p 1 -b output_scaffold_prefix
```

* `-l` specifies the library file
* `-s` specifies the list of contigs that `minia` produced
* `-z` sets the minimum required size for contigs (anything shorter than this will be discarded)
* `-p` tells the program to make a dot file, which can be used to visualize the scaffold
* `-v` sets verbose output
* `-b` sets the base file name for the output directory and the files in it.

After the command runs, open the summary file it created. Record how many scaffolds there are “After scaffolding”, the max scaffold size, and the N50. (This is also where you can get those numbers for your original contig file, under “Inserted contig file”).

## 9. Clean up original shortjump read files, see if scaffolding improves

In next generation sequence analysis, we often have to make a choice between using as much data as possible, or using only the cleanest, highest quality data. The scaffolding we just performed used our raw short jump files, which we know have a lot of low quality base calls. In this section, you’ll clean up the short jump reads with quality trimming, then try the scaffolding again. 

Use sickle to trim both short jump files. Since the read length is small to begin with (only 37 basepairs), set a minimum length for the output files, so that you don’t have a bunch of tiny 3- or 4-mers after filtering, which would be impossible to correctly align. (You want at least 20, but you can choose higher) Type `sickle pe` into terminal to bring up the list of options and figure out how to do this. 

```
sickle pe -options -f file1.fastq -r file2.fastq -o trimmedfile1.fastq -p trimmedfile2.fastq
```

Run `SSPACE` again, this time using the trimmed data in the library file. Be sure to choose a new output prefix so that you don’t lose your original results. 

Examine the summary file and again record the number of scaffolds, the max scaffold length, and the N50. Which scaffolding was better? The one with the raw data, or the one with the trimmed data? 

## 10. Try to close the gaps in the scaffolds. 
Now that we’ve linked the contigs together into longer stretches of sequence, we can try to improve things even further, by aligning our original reads to the scaffolds, and using them to fill in some of the gaps. We’ll try this with both our original (101 bp, 180 insert) and short jump reads.

To close gaps, we'll use a tool called `GapCloser`. You can use the following commands (with each option explained below):
```
GapCloser \
    -a scaffolds.fasta \
    -o output.fa \
    -l maxreadlen \
    -b configfile
```

where:

* `-a` is a fasta file with your scaffolds created by `SSPACE`
* `-o` is the name of the output fasta file. This will contain your original scaffolds with gaps filled in
* `-l` The maximum read length in your experiments
* `-b` Configuration file describing the read libraries to input.

Read about how to build the configuration file here: http://soap.genomics.org.cn/soapdenovo.html. The "example.config" file given there is very similar to what we'll need for our own GapCloser run. It contains two libraries, one with short insert size and "forward-reverse" orientation, and the other with long insert size and "reverse-forward" orientation. A good strategy is to first copy that example configuration file and edit the parameters to reflect our own data. Ask for help if you have questions about how to create the config file. Once you have the configuration file set up you can run the `GapCloser` command.

Below, we'll use QUAST to evaluate our assembly after gap closing. 

## 11. Evaluate your assembly
Even though we were unable to link every stretch of DNA together into a finished genome, we were still able to produce lots of sequences, which could be used for gene finding, comparison to related  bacteria, or as a guide for designing additional sequencing projects to fill in the gaps and link the remaining contigs. 

For today, since we are using raw data from a genome that is actually already solved, we can align our contigs to that reference to evaluate our de novo assembly performance. Remember, if we were trying to solve the genome of a new or unknown organism, we wouldn’t be able to do this, but we could try using a close relative.

The NCBI id of the Staphylococcus aureus strain used here is NC_010079. Download the fasta file and gene annotation in GFF form from NCBI to your desktop (see https://www.ncbi.nlm.nih.gov/nuccore/NC_010079.1?report=fasta). You can download these files by selecting "Send to" at the top. Choose "Complete sequence", "File", and "FASTA" to get the reference genome. Choose "Complete sequence", "File", and "GFF3" to get the gene annotation. Use secure copy to transfer the gap filled scaffolds file (which contains your final, scaffolded, de novo assembly) to the desktop. 

Go to QUAST again (http://quast.bioinf.spbau.ru). 

First, make extra certain that you will get an email report. On the right hand side, type in your email and click ‘get personal page’. Close the QUAST tab, then wait for the email to arrive. Click on the “Personal Page” link, and perform your analysis there in order to get the results emailed. 

Use quast to align your scaffolds to the actual reference sequence of the bacterial strain used to generate the original data.

* Use “Add Files” to upload your assembly. 
* Check the "scaffolds", “find genes” boxes, and the “Prokaryotic” button. 
* Under genome, check “another genome” and fill in the name. Under “Reference” upload the fasta file you downloaded from the NCBI. 
* Additionally add the GFF3 file under "Genes"
* Type in a title for this analysis under caption, then click evaluate. 

While that is running, start a separate analysis without choosing the "scaffolds" button so we'll be able to visualize the results using the "Icarus" browser.

Once the analyses are finished, spend some time exploring the results to see how your assembly compares to the NCBI reference.

How do the N50 and max contig values after gap closing as reported by Quast compare to those metrics before gap closing? Is this what you expected?

Use the Quast report to gather information for your lab report described below.

## For your lab report:
*Abstract*: remember our goal this week isn’t to answer a scientific question, but to demonstrate a de
novo assembly pipeline. Be sure to state what you did and why, and state your overall result.

*Intro*: Include some background information about why de novo assembly is used and the types of
situations where it can have a significant impact. Briefly summarize the data you are using in this
study (what organism is it from, why are we using it, how the library is set up). 

*Methods*: Summarize each step of the analysis, and be sure to include non default parameters and
awk scripts. 

*Results:*
You will have a lot of figures (the fastqc output, histogram plots). Also
include a table that shows the number of contigs, max contig length, and N50 for each step of your
analysis. Be sure to include text that explains each result (ie say things like “Minia was used to
assemble overlapping reads from the small insert library into contigs, and the results are shown in
table 1.”

In addition to presenting the results of each step of the analysis, you should use the final QUAST
reports to answer the following questions. 

1. Explain why there are two results, and what the ‘broken’ result is (you will have to use the QUAST
manual).
2. What fraction of the genome was covered by our scaffolds? 
3. What is the mismatch rate? 
4. How many misassemblies are there? 
5. What are the orange/red scaffolds in the Icarus browser?

*Discussion*:
Evaluate the success of your assembly. Would you consider it a finished genome? Does the number
of genes predicted by QUAST for your assembly match the actual number of genes in the “type”
reference (use NCBI to find the number of actual genes? Do your scaffolds contain complete gene
sequences?
What could you do to improve your assembly, and properly connect those final ~1000 scaffolds,
without relying on a reference sequence?
What was the effect of k-mer length during assembly?
