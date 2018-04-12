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
