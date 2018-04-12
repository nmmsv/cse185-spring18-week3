# Week 4: Where do all these reference sequences come from, anyway? (part 1)
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

You will use the program SSPACE for scaffolding. First, it will use the `bowtie` aligner (within the program) to map the shortjump reads to your `minia` contigs. Then, it will use those mapping results to connect the contigs into bigger scaffolds. Today you will generate a lot of new files, so make a subdirectory in your `week4` folder called “Thursday.” (no need to commit these files to github). Copy the `contig.fa` file from minia to this subdirectory. Also, move the `shortjump_1` and `shortjump_2` files to this directory. 

SSPACE requires a library file, telling the program the names of our `short_jump` files and some other information about them. Use `emacs` to create a new file with the general format below. You will have tolook at the SSPACE user manual to figure out exactly what each column means. 

```
emacs sspace_library
```

*Example library file:*
```
Lib1 aligner file1.fastq file2.fastq insertsize sizeerror orientation 
```

You will have to specify the aligner (use `bowtie`), the insert size, the tolerable error for the insert size, and the orientation. Once you’ve made the library file, run `SSPACE` with the command below, which you will have to edit according to your exiting file names and your output name preference. A description of what each flag is doing follows the command.

```
SSPACE_Standard_v3.0.pl -l libraryfile -s minia_out.contigs.fa -z 100 -v 1 -p 1 -b output_scaffold_prefix
```

* `-l` specifies the library file
* `-s` specifies the list of contigs that `minia` produced
* `-z` sets the minimum required size for contigs (anything shorter than this will be discarded)
* `-p` tells the program to make a dot file, which can be used to visualize the scaffold
* `-v` sets verbose output
* `-b` sets the base file name for the output directory and the files in it.

After the command runs, `cd` into the directory it created, and open the summary file. Record how many scaffolds there are “After scaffolding”, the max scaffold size, and the N50. (This is also where you can get those numbers for your original contig file, under “Inserted contig file”).
