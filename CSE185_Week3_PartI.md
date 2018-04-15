# Week 4: Where do all these reference sequences come from, anyway? (part 1)
Skills covered: contig assembly, scaffolding, assembly evaluation, plotting

Today, you will assemble next generation sequencing reads into a complete genome. On Thursday,
you will compare your genome to that of closely related species, and evaluate it against the published
reference genome to see how you did. 

Today’s sequencing reads are in two libraries corresponding to DNA fragments of the bacteria
*Staphylococcus aureus.*

The first library (“frag”) is paired-end with the read directions toward each other (“innie”). The
insert size of the first library is 180 bp (the length of DNA between the Illumina adapters), and
the read length is 101 bp. We will use this library today to assemble contigs.

The second library (“short_jump”), which you will use in the scaffolding step is paired-end with
the reads oriented away from one another (“outie”). Sometimes this library set up is called
mate-pair instead of paired-end. The short_jump insert size is 3500 bp, and the read length is
37 bp. We will use this library on Thursday to scaffold our assembled contigs. 

## 1. Inspect the data and run fastqc

First, to get set up clone your github repository for this week to your home directory in a directory called `week3`.

```
git clone https://github.com/cse185-sp18/cse185-week2-<username>.git week3
cp ../public/week3/*.fastq week3/
```

The data for this week's lab is in the `public/week3` and consists of four fastq files (`frag_1.fastq`, `frag_2.fastq`, `short_jump_1.fastq`, `short_jump_2.fastq`). Copy these to your `week3` directory (copying fastq files is not generally a good practice, but some of the tools we'll run today require the files to be in a directory where you have write permissions.)

Use `wc -l` to determine the number of reads in each raw data file. Record it in your notebook. Then run `fastqc` on all four of the fastq files. As in previous labs, use `scp` to retrieve the html files and save screenshots of the per base read quality for each file.

Wow! That data is pretty bad. We will use a k-mer based approach to error correction to clean up the
small insert library (`frag_1.fastq` & `frag_2.fastq`). You’ll explore the k-mer distribution in the next
section. (We will deal with the short jump library on Thursday). 

But before moving on, let's look at the state of our git repository, which is a good idea to do periodically. Run:
```
git status
```

You'll likely see that your lab notebook has some modifications that aren't yet committed. But you'll also see all of the fastqc output files (`.zip` and `.html`) listed under "Untracked files". We don't actually plan on tracking these files with git, and it is a bit annoying to have those keep appearing when we type `git status`. We can tell git a list of files to ignore by adding a file called `.gitignore`. Open a new file:
```
emacs .gitignore
```
And add the following lines:
```
*.html
*.zip
```
Type `ctrl-x ctrl-s` to save and `ctrl-x ctrl-c` to exit. Now add the gitignore file, commit, and push your changes:
```
git add .gitignore
git commit -a -m"Adding gitignore file to the repository"
git push
```

<blockquote>
**UNIX TIP**: Save your work frequently! It is a good idea to type "ctrl-x ctrl-s" (if using Emacs) often (I do it subconsciously literally every minute or so) to make sure you don't lose your work. If you are using Emacs and by chance forgot to save your work before exiting, you can often find the unsaved changes in a temporary file ending with a "~" in the same directory.
</blockquote>

<blockquote>
**UNIX/GIT TIP**: If you make changes to your lab notebook or report directly from the web browser, and want to update your local repository on ieng6 with those changes, you can use "git pull" to see the updates.
</blockquote>

<blockquote>
**UNIX/GIT TIP**: We will often be dealing with a lot of files, and not all of these make sense to track with git. In general, git is great for tracking code or small text (or markdown) files. You can also store figure files (e.g. pngs of screenshots). On the other hand, don't add any data files, like fastqs or BAMs. 
</blockquote>

## 2. Use jellyfish to make k-mer histograms for the "frag" library

We will be using the `frag_1` and `frag_2` fastq files to assemble our genome, and we will be using a de
Bruijn graph strategy, which breaks the reads up into k-mers to facilitate assembly of correctly
connected contigs. 

The size and type of these k-mers are important parameters in de novo assembly, so in this section
you will spend some time looking at how k-mers are distributed in our data. For this purpose it’s OK to only analyze the forward fragment (fragment 1). 

*Jellyfish* is a kmer counting program that will count the frequency of all possible k-mers of a given
length in our data. The `jellyfish count` command takes the following options:

* `-m` specifies the length
* `-C` tells it to ignore directionality (it treats each read the same as its reverse complement)
* `-s` is an initial estimate for the size of the hash table jellyfish uses, set > genome size
* `-o` specifies the name of the output file. choose a name with the k-mer length in it

Run the command below on the `frag_1` data. Use a k-mer sizes of 31.

```shell
jellyfish count -m 31 -s 10000000 -o 31 -C frag_1.fastq 
```

This might take a while. You can get started on the reading assignment while you're waiting for this and other commands below.

Once the command is finished, run the command below to make a histogram file. 
```shell
jellyfish histo 31 > 31.histo
```

Cat the histo files and take a look. On the left is a list of the bins (the number of times a k-mer occurs
or its "depth"), and on the right is the count for the number of k-mers in the data that fit into that
category. 

## 3. Plot the k-mer histograms in python

The k-mer distribution is easier to understand if we actually visualize it. For this, we will use the
programming language python to make plots of each histogram. (Note, you're welcome to use another language like R to plot).

Start IPython by typing:
```
ipython
```

First we'll load `pandas`, a popular library for manipulating tables of data in python, and `matplotlib` for creating plots:

```python
import matplotlib.pyplot as plt
import pandas as pd
```

Now load the data and create a histogram. Note there are many resources online to learn more about plotting with python. These commands will get you started, but you're welcome to try and figure out how to make the default plots prettier.

```python
plt.figure()
r31 = pd.read_csv("31.histo", sep=" ", names=["kmercount", "number"])
plt.bar(r31.iloc[1:100,]["kmercount"], r31.iloc[1:100,]["number"]);
plt.show()
plt.savefig("frag_1_31.pdf")
```

Exit Ipython by typing `ctrl-D` and answering `y` to the prompt asking if you want to exit. When you exit, you should see a pdf file in your working directory. Use `scp` to open it on your desktop; save it for your lab report, and answer the IClicker question.

Reopen the actual `.histo` file with `cat`, and find the bin (“number of times that k-mer appears”) that
corresponds to small valley in the figure. Record this “initial valley point” in your lab notebook and answer the IClicker question.

## 4.  K-mer based error correction

K-mer based error correction takes advantage of the fact that our data has a relatively high depth of
coverage. For each of the low frequency kmers (likely errors, therefore untrustworthy), the software
will try to find similar high-frequency k-mers that are only one or two mutations away, and ‘correct’ the
read with the untrustworthy k-mer so that we can still use it in assembly. 

We are going to use error correction modules from SOAPdenovo2 (SOAP = “Short Oligonucleotide
Alignment Program). SOAP has been around since the dawn of next-generation sequencing (the
original paper, published in 2008, has been cited 1372 times), and this error correction module is from
2012. 

Run error correction on the `frag_1.fastq` and `frag_2.fastq` files together. Here, we will use the maximum k-mer size allowed by the software, which is 27. The two commands below use a file that contains a list of the files to correct. Use emacs or your favorite text editor to make this list file, with the files (`frag_1.fastq` and `frag_2.fastq`) each listed on separate lines. 

```
emacs filelist
```

After you type in the file names, type `ctrl-x ctrl-s` to save and `ctrl-x ctrl-c` to exit. 

First run the `KmerFreqHA` command, which is part of the `SOAPdenovo2` package. Check the
commands usage page to figure out how to set the `-L` option. Replace the prefix with something
memorable, like "corrected":

```shell
KmerFreq_HA -k 27 -L NNN -i 10000000 -p prefix -l yourlist
```

This command will take about 5-6 minutes to run. It is generating the frequency of every k-mer in our
data, then creating a hash-table to make them easier to access. When the command completes, run
the corrector command shown below which actually corrects our reads.

*Edit the corrector command* so that `-l` corresponds to the “valley point” in your k=31 kmer histogram.
You can double check with us if you are unsure. 

```shell
Corrector_HA -k 27 -Q 33 -o 3 -l N prefix.freq.gz yourlist
```

This command will also take 5-6 minutes to run. When it is complete, inspect your working directory.
There should be several new output files, the ones we are interested in end in `*cor.pair_N.fq`. These
are the corrected fastq files, and we will use them for our assembly. 

First, use jellyfish to see how the distribution of k-mers is different in the corrected data:

```shell
jellyfish count -m 31 -s 10000000 -o 31corrected -C frag_1.fastq.cor.pair_1.fq
jellyfish histo 31corrected > 31corrected.histo 
```

Use Python again to make a PDF of the histogram, and include both pdfs (before and after correction) in
your lab report. 

## 5. Collect data on corrected reads, calculate genome size

Use `wc -l` on the corrected fastq files to see how many reads are left. 

Open the corrected histogram and find where the new “valley point” is. Record this in your lab
notebook. Also, record the multiplicity (or bin) of the first “peak” after that valley. Answer the IClicker question before moving on. 

In addition to replacing basecalls that are likely erroneous, `Corrector_HA` also trimmed data with
particularly low quality scores. We will need to know the average read length of our corrected files, so
use the `awk` command below on the corrected data to calculate it for each corrected file, record the
results.

```shell
awk 'NR%4==2{sum+=length($0)}END{print sum/(NR/4)}' input.fastq
```

We also need to know the total number of bases in each corrected file. Modify the `awk` command
above so that it will output the total number of bases in all the reads. Record the results for each file in
your notebook. 

Use the peak bin you identified, and the numbers you just calculated, to estimate the genome size of
our bacteria with the following formulas:

N = (M*L)/(L-K+1)
Genome_size = T/N

(N: Depth of coverage, M: Kmer peak, K: Kmer-size, L: avg readlength T: Total bases)

Recall that we calculated the kmer distribution using only the first pairs, so only use numbers for the first fastq file. Record the calculated genome size.

The actual length of the Staph aureus genome is around 3 million bp. How close were you? If you didn't that (which we didn't), hypothesize why you might have over or under estimated the genome size. 

## 6. Assemble reads with minia

Now we are ready to start assembling our reads. Unfortunately, while `SOAPdenovo2` is still a widely
used tool for denovo assembly, it is too much of a memory hog to run on our ieng6 servers. So we will
use a lightweight program called minia instead. 

`minia` needs a list of our files as input, so use emacs to make a list like before, with one line listing each
of the *corrected* files.

The minia command takes the format shown below (all on one line). Use the data we collected about
the corrected reads in section 5 to figure out how to specify each of this parameters. Choose any
output prefix you like. 

*The most important parameter is the kmer size*. From our jellyfish histogram of the corrected data, we
know that a kmer of 31 produces a defined ‘true-reads’ peak, and we know where the valley is. 

However, other kmers may results in better assemblies. Find at least two partners to share data with,
and between you, try the minia assembly with kmer values between 27 and 43, as well as 31. One person in the group must use
kmer size 31.

For the other k-mer sizes, use the two jelly fish commands to make a histogram and find out if the
‘valley point’ has changed. For the minia command, you only want to use k-mers with abundance
higher than this valley point. 

```
minia \
  -in correctedlist \
  -kmer-size kmer_size \
  -abundance-min min_abundance \
  -out outprefix
```

## 7. Analyze your contigs

Examine your (own) assembly. The most important file minia created is the `*.contigs.fa` file, which is a
list of each assembled contigs in fasta format. Use head to look at the first few contigs, then use the commands below to gather some basic statistics on how `minia` did. How many contigs are there? (most fasta files have 2 lines per sequence). Record the answer in your notebook. 

```
wc -l minia_out.contigs.fa
```

What is the longest contig? The shortest? This command will get you one of them, edit it to get you
the other, and record both in your notebook. 

```
cat minia_out.contigs.fa | awk 'NR%2==0{print length}' | sort -n | head 
```

<blockquote>
**UNIX**: datamash is a really useful tool for computing simple operations on columns of data. Below is an example of how to do the command 
</blockquote>

```
cat minia_out.contigs.fa | awk 'NR%2==0{print length}' | datamash min 1 max 1
```

As your results come in, add them to the spread sheet on the board. For your lab report, you will make a plot of kmer size vs assembly success (We’ll summarize and post the data you need next time). You will be performing additional analysis on either your assembly or the best assembly from your group. 

Use a web tool, called QUAST (QUality ASsesment Tool for genome assemblies) to get some more
metrics on your contigs. First, use `scp` to transfer the `*contgs.fa` file to your desktop. Then go to  http://quast.bioinf.spbau.ru and follow these steps:

* Upload your contig file, and type 100 in the “skip contigs shorter than” box. 
* Check find genes (prokaryotic).
* Leave the genome as unknown. 
* Make the caption something useful, like minia_greaterthan_99
* Type in your email so that you can access the report at any time. 
* Then click Evaluate. 

Once your report has finished, record the N50 value for your kmer size in the spreadsheet.

**That's it for today. Next time we will try to link our contigs into longer pieces of genome with a technique called
scaffolding.**

**Acknowledgements: Adapted from a lab originally written by Dr. Katie Petrie**
