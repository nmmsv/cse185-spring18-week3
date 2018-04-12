# Week 4: Where do all these reference sequences come from, anyway? (part 2)
Skills covered: contig assembly, scaffolding, assembly evaluation

## 7. Compare contig assembly results with the rest of class
Use the whole class data (shared on Slack) to make two scatter plots: k-mer size vs maximum contig length, and k-mer size vs contig N50 or longest k-mer. You can use anything you like to make the plots, and for k-mers where there are two different minimum abundance thresholds, just plot them as two separate points. In each plot, indicate which data points are yours.

If your k-mer size is not in the data already, add it before you make the plot. If you weren’t able to get
your QUAST results, that’s OK, this data will be in the output from the next step, and you can make
the full plot later.

Did k-mer size influence the success of the assembly? Answer in your lab report discussion.

Take a look at your quast results page. If you don’t have one, use the example results linked to in the spreadsheet. Click on the small link that says “Icarus: contig browser” near the top of the results, and use the browser to answer the IClicker question. 

## 8. Use short jump library to scaffold minia contigs. 
These short contigs are certainly not a complete genome. Even if they cover all of the sequence in our genome (they might not; there could be missing regions), we do not know the correct order to put them in to reconstitute the real genome sequence.

Fortunately, we can use a second set of DNA sequencing data to help us figure out the correct way to join the contigs together. Our contigs were initially made by stitching together 101 bp reads from DNA fragments that were 180 bp long. Our second DNA library (short jump) consists of paired 37 bp reads from DNA fragments that were 3500 bp long. You will map these short jumps to your set of contigs.

If the forward member of a pair binds to one contig, and its reverse partner binds to a different contig, we know that those positions must be ~3500 bp apart in the actual genome. By combining all of the mapping data from these pairs, scaffolding software can figure out the optimal, correct way to link the contigs together.

You will use the program `SSPACE` for scaffolding. First, it will use the `bowtie` aligner (within the program) to map the shortjump reads to your `minia` contigs. Then, it will use those mapping results to connect the contigs into bigger scaffolds. Today you will generate a lot of new files, so make a subdirectory in your `week4` folder called “Thursday.” (no need to commit these files to github). Copy the `contig.fa` file from minia to this subdirectory. Also, move the `shortjump_1` and `shortjump_2` files to this directory. 

`SSPACE` requires a library file, telling the program the names of our `short_jump` files and some other information about them. Use `emacs` to create a new file with the general format below. You will have to look at the SSPACE user manual to figure out exactly what each column means (https://github.com/nsoranzo/sspace_basic/blob/master/F132-01%20SSPACE_Basic_User_Manual_v2.0.pdf). 

```
emacs sspace_library
```

*Example library file:*
```
Lib1 file1.fastq file2.fastq insertsize sizeerror orientation 
```

You will have to specify the aligner (use `bowtie`), the insert size, the tolerable error for the insert size, and the orientation. Once you’ve made the library file, run `SSPACE` with the command below, which you will have to edit according to your exiting file names and your output name preference. A description of what each flag is doing follows the command.

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

Make a *new library file*, in the same format as before, but this time include the original reads as well. If
quality trimming improved your assembly, use the quality trimmed short jump file, and decide if you
want to do quality trimming on the original files. You will have to `mv` the original files (`frag_1.fastq`,
`frag_2.fastq`) to your Thursday directory. 

## 11. Evaluate your assembly
Even though we were unable to link every stretch of DNA together into a finished genome, we were still able to produce lots of sequences, which could be used for gene finding, comparison to related  bacteria, or as a guide for designing additional sequencing projects to fill in the gaps and link the remaining contigs. 

For today, since we are using raw data from a genome that is actually already solved, we can align our contigs to that reference to evaluate our de novo assembly performance. Remember, if we were trying to solve the genome of a new or unknown organism, we wouldn’t be able to do this, but we could try using a close relative.

The NCBI id of the Staphylococcus aureus strain used here is NC_010079. Download the fasta file from NCBI to your desktop, and use secure copy to transfer the `*gapfilled.final.fa` file (which contains your final, scaffolded, de novo assembly) to the desktop. 

Go to QUAST again (http://quast.bioinf.spbau.ru). 

First, make extra certain that you will get an email report. On the right hand side, type in your email and click ‘get personal page’. Close the QUAST tab, then wait for the email to arrive. Click on the “Personal Page” link, and perform your analysis there in order to get the results emailed. 

Use quast to align your scaffolds to the actual reference sequence of the bacterial strain used to generate the original data.

* Use “Add Files” to upload your assembly. 
* Check the “scaffolds” and “find genes” boxes, and the “Prokaryotic” button. 
* Under genome, check “another genome” and fill in the name. Under “Reference” upload the fasta file you downloaded from the NCBI. 
* Type in a title for this analysis under caption, then click evaluate. 

While you are waiting for the results, set up one additional analysis. For this, we will use the genome of the Staphylococcus aureus type strain, which is annotated (it has all the genes marked and identified) in NCBI. Search for Staphylococcus aureus at NCBI, and select the single “genome” result that shows up. 

At the very top of the page are download links. Click ‘genome’ under ‘download sequences in fasta format’, and also click ‘GFF’ under ‘download genome annotation’ to download these files. Run quast again, but this time align your scaffolds to the ‘type strain’ official reference sequence, and include the GFF file under genes. Once you click evaluate, it may take a while to run.

You will use both reports to answer specific questions for your lab report, but you don’t have to wait for the results before you leave. You do not have to paste a screen shot of the entire QUAST list of numbers, but do take a screen shot of the icaraus: contig browser (link at top left of report) showing your entire assembly aligned to each reference. (show the entire length, not the zoomed in portion).

## For your lab report:
*Abstract*: remember our goal this week isn’t to answer a scientific question, but to demonstrate a de
novo assembly pipeline. Be sure to state what you did and why, and state your overall result.

*Intro*: Include some background information about why de novo assembly is used and the types of
situations where it can have a significant impact. Briefly summarize the data you are using in this
study (what organism is it from, why are we using it, how the library is set up). 

*Methods*: Summarize each step of the analysis, and be sure to include non default parameters and
awk scripts. 

*Results:*
You will have a lot of figures (the fastqc output, histogram plots, and the Icarus snapshots). Also
include a table that shows the number of contigs, max contig length, and N50 for each step of your
analysis. Be sure to include text that explains each result (ie say things like “Minia was used to
assemble overlapping reads from the small insert library into contigs, and the results are shown in
table 1.”

In addition to presenting the results of each step of the analysis, you should use the final QUAST
reports to answer the following questions. 

1. Explain why there are two results, and what the ‘broken’ result is (you will have to use the QUAST
manual).
2. What fraction of the genome was covered by our scaffolds? (report for in both alignments)
3. What is the mismatch rate? (report for in both alignments)
4. How many misassemblies are there? (report for in both alignments)
5. What are the orange/red scaffolds in the Icarus browser?

*Discussion*:
Evaluate the success of your assembly. Would you consider it a finished genome? Does the number
of genes predicted by QUAST for your assembly match the actual number of genes in the “type”
reference (use NCBI to find the number of actual genes? Do your scaffolds contain complete gene
sequences?
What could you do to improve your assembly, and properly connect those final ~1000 scaffolds,
without relying on a reference sequence?
What was the effect of k-mer length during assembly?
