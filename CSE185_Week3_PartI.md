# Week 4: Where do all these reference sequences come from, anyway? (part 1)
Skills covered: contig assembly, scaffolding, assembly evaluation, plotting, tool installation

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

The data for this week's lab is in the `public/week3` and consists of four fastq files (`frag_1.fastq`, `frag_2.fastq`, `short_jump_1.fastq`, `short_jump_2.fastq`). Use `wc -l` to determine the number of reads in each raw data file. Record it in your notebook. Then run `fastqc` on all four of the fastq files. As in previous labs, use `scp` to retrieve the html files and save screenshots of the per base read quality for each file.

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
**UNIX TIP**: Save your work frequently! It is a good idea to type "ctrl-x ctrl-s" (if using Emacs) often (I do it subconsciously) to make sure you don't lose your work. If you are using Emacs and by chance forgot to save your work before exiting, you can often find the unsaved changes in a temporary file ending with a "~" in the same directory.
</blockquote>

<blockquote>
**UNIX/GIT TIP**: If you make changes to your lab notebook or report directly from the web browser, and want to update your local repository on `ieng6` with those changes, you can use "git pull" to see the updates.
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
jellyfish count -m 31 -s 10000000 -o 31 -C ../../public/week3/frag_1.fastq 
```

This might take a while. You can get started on the recommended reading while you're waiting.

Once the command is finished, run the command below to make a histogram file. 
```shell
jellyfish histo 31 > 31.histo
```

Cat the histo files and take a look. On the left is a list of the bins (the number of times a k-mer occurs
or its ‘depth’), and on the right is the count for the number of k-mers in the data that fit into that
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
plt.bar(r31.iloc[0:100,]["kmercount"], r31.iloc[0:100,]["number"]);
plt.show()
plt.savefig("frag_1_31.pdf")
```

Exit Ipython by typing `ctrl-D` and answering `y` to the prompt asking if you want to exit. When you exit, you should see a pdf file in your working directory. Use `scp` to open it on your desktop; save it for your lab report, and answer the IClicker question.

Reopen the actual `.histo` file with cat, and find the bin (“number of times that k-mer appears”) that
corresponds to small valley in the figure. Record this “initial valley point” in your lab notebook. 
