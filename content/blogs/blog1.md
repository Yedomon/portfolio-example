---
categories:
- ""
- ""
date: "2021-06-17"
description: ""
draft: false
image: pic10.jpg
keywords: ""
slug: ipsum
title: How to extract all N positions from a genome assembly file?
---

## Brainstorming

 I will use a toy fasta for illustration. Here is it:


```bash

$ cat toy.fasta


>chr01
AAANNNNNGCT
>chr02
AAAANNNNNGCT


```

My toy has just two chromosmes. The first one has N from 4 to 8 and the second one from 5 to 9 ( **1-based coordinate**)


My initial task for my whole genome project is to know the position of those gaps in each sequence. It is important for me to know the gaps' positions precisely in my Hi-C based assembly that I have received from [Dovetails genomics](https://dovetailgenomics.com/) company.

Knowing those gaps will help me to design appropriate markers, test them using PCR method and fill the gaps accordingly. So how can I get the exact position of my gaps?


## Solution


After googling, I found on biostar this [solution](https://www.biostars.org/p/152592/). Prakki Rama suggests this [approach](https://www.biostars.org/p/133742/#377084) as a one line perl-based code.

That is awesome! I love one line solution! Let's test it!

Using the toy data, I proceeded like this:



```bash

perl -ne 'chomp;if( />(.*)/){$head = $1; $i=0; next};@a=split("",$_); foreach(@a){$i++; if($_ eq "N" && $s ==0 ){$z=$i-1; print "$head\t$z"; $s =1}elsif($s==1 && $_ ne "N"){$j=$i-1;print "\t$j\n";$s=0}}' toy.fasta


```

Here is the result:

```bash

chr01   3       8
chr02   4       9

```

**The first number indicated a zero-based coordinate while the second number indicates a 1-based coordinate.**

So in practical case, using a 1-based coordinate the start position will be 3 +1 bp = 4  for the chr01 and 4 + 1 bp = 5 for chr02.


My plan is to use samtools for cutting my sequence. Samttols uses 1-base coordinate. So my appropriate result will be:


```bash

chr01   4       8
chr02   5       9

```

That's it! Big thank to Prakki Rama.


## Bonus

To get the full N sequence, it is possible to get it by doing:


```bash

$ bedtools getfasta -fi toy.fasta -b toy.bed


```


I got this result



```bash

>chr01:3-8
NNNNN
>chr02:4-9
NNNNN


```

Note: Zero-based and one-based coordinate need to be considered for downstream analysis. 
To get more info about zero and one-based coordinate, please refer to bedtools page [here](https://bedtools.readthedocs.io/en/latest/content/overview.html). 

 - **BED starts are zero-based and BED ends are one-based**.
 - **GFF starts and ends are one-based.**
 - **VCF coordinates are one-based**
