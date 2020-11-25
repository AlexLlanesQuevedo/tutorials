# Analysis of Introgression with SNP Data

A tutorial on the analysis of hybridization and introgression with SNP data by Milan Malinsky (millanek@gmail.com) and Michael Matschiner

## Introduction

Admixture between populations and hybridisation between species are common and a bifurcating tree is often insufficient to capture their evolutionary history ([example](https://www.sciencedirect.com/science/article/pii/S0959437X16302052)). Patterson’s D, also known as ABBA-BABA, and the related estimate of admixture fraction <i>f</i>, referred to as the f4-ratio are commonly used to assess evidence of gene flow between populations or closely related species in genomic datasets. They are based on examining patterns of allele sharing across populations or closely related species. Although they were developed in within a population genetic framework the methods can be successfully applied for learning about hybridisation and introgression within groups of closely related species, as long as common population genetic assumptions hold – namely that (a) the species share a substantial amount of genetic variation due to common ancestry and incomplete lineage sorting; (b) recurrent and back mutations at the same sites are negligible; and (c) substitution rates are uniform across species. 

Patterson's D and related statistics have also been used to identify introgressed loci by sliding window scans along the genome, or by calculating these statistics for particular short genomic regions. Because the D statistic itself has large variance when applied to small genomic windows and because it is a poor estimator of the amount of introgression, additional statistics which are related to the f4-ratio have been designed specifically to investigate signatures of introgression in genomic windows along chromosomes. These statistics include <i>f</i><sub>d</sub> (Martin et al., 2015), its extension <i>f</i><sub>dM</sub> (Malinsky et al., 2015), and the distance fraction <i>d</i><sub>f</sub> (Pfeifer & Kapan, 2019).

## Table of contents

* [Outline](#outline)
* [Requirements](#requirements)
* [1. Inferring the species-tree and gene-flow from a simulated dataset](#simulation)
* [1.1 Simulating phylogenomic data with msprime](#simulation)
* [1.2 Reconstructing phylogenies from simulated data](#ReconstructingFromSimulation)
* [1.3 Testing for gene-flow in simulated data ](#TestingInSimulations)
* [1.3.2 Do we find geneflow in data simulated without geneflow?](#TestingWithoutGeneflow)
* [1.3.2 Do we find geneflow in data simulated with geneflow?](#TestingWithGeneflow)
* [Ancestry painting](#painting)

<!--- * [Dataset](#dataset)-->

<a name="outline"></a>
## Outline

In this tutorial, we are first going to use simulated data to demonstrate that, under gene-flow, some inferred species relationships might not correspond to any real biologial relationships. Then we are going to use [Dsuite](https://doi.org/10.1111/1755-0998.13265), a software package that implements Patterson’s D and related statistics in a way that is straightforward to use and computationally efficient. This will allow us to identify admixed taxa. While exploring Dsuite, we are also going to learn or revise concepts related to application, calculation, and interpretation of the D and of related statistics. Next we apply sliding-window statistics to identify particular introgressed loci in a real dataset of Malawi cichlid fishes. Finally, we look at the same data that was used for species-tree inference with SVDQuartets in tutorial [Species-Tree Inference with SNP Data](../species_tree_inference_with_snp_data/README.md) to ..... 

Students who are interested can also apply ancestry painting to investigate a putatitve case of hybrid species.  

<!--- The original ABBA-BABA test has been extended in various ways, including the <i>D</i><sub>FOIL</sub>-statistic that allows inferring the direction of introgression from sets of five species ([Pease and Hahn 2015](https://academic.oup.com/sysbio/article/64/4/651/1650669)) and the <i>f</i><sub>D</sub>-statistic that is better suited for the identification of gene flow pertaining to certain regions of the genome ([Martin et al. 2014](https://academic.oup.com/mbe/article/32/1/244/2925550)). If a putative hybrid individual as well as the presumed parental species have already been identified, patterns of introgression can be investigated with ancestry painting, a method that focuses on sites that are fixed between the parental species and the alleles observed at these sites in the putative hybrid.) -->


<!--- 
<a name="dataset"></a>
## Dataset

The SNP data used in this tutorial is the unfiltered dataset used for species-tree inference with SVDQuartets in tutorial [Species-Tree Inference with SNP Data](../species_tree_inference_with_snp_data/README.md). More detailed information about the origin of this dataset is given in the Dataset section of this other tutorial. In brief, the dataset includes SNP data for the 28 samples of 14 cichlid species listed in the table below, and this data has already been filtered based on read quality and depth. Only SNPs mapping to chromosome 5 of the tilapia genome assembly ([Conte et al. 2017](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-017-3723-5)) are included in the dataset.

<center>

| Sample ID | Species ID | Species name                  | Tribe         |
|-----------|------------|-------------------------------|---------------|
| IZA1      | astbur     | *Astatotilapia burtoni*       | Haplochromini |
| IZC5      | astbur     | *Astatotilapia burtoni*       | Haplochromini |
| AUE7      | altfas     | *Altolamprologus fasciatus*   | Lamprologini  |
| AXD5      | altfas     | *Altolamprologus fasciatus*   | Lamprologini  |
| JBD5      | telvit     | *Telmatochromis vittatus*     | Lamprologini  |
| JBD6      | telvit     | *Telmatochromis vittatus*     | Lamprologini  |
| JUH9      | neobri     | *Neolamprologus brichardi*    | Lamprologini  |
| JUI1      | neobri     | *Neolamprologus brichardi*    | Lamprologini  |
| LJC9      | neocan     | *Neolamprologus cancellatus*  | Lamprologini  |
| LJD1      | neocan     | *Neolamprologus cancellatus*  | Lamprologini  |
| KHA7      | neochi     | *Neolamprologus chitamwebwai* | Lamprologini  |
| KHA9      | neochi     | *Neolamprologus chitamwebwai* | Lamprologini  |
| IVE8      | neocra     | *Neolamprologus crassus*      | Lamprologini  |
| IVF1      | neocra     | *Neolamprologus crassus*      | Lamprologini  |
| JWH1      | neogra     | *Neolamprologus gracilis*     | Lamprologini  |
| JWH2      | neogra     | *Neolamprologus gracilis*     | Lamprologini  |
| JWG8      | neohel     | *Neolamprologus helianthus*   | Lamprologini  |
| JWG9      | neohel     | *Neolamprologus helianthus*   | Lamprologini  |
| JWH3      | neomar     | *Neolamprologus marunguensis* | Lamprologini  |
| JWH4      | neomar     | *Neolamprologus marunguensis* | Lamprologini  |
| JWH5      | neooli     | *Neolamprologus olivaceous*   | Lamprologini  |
| JWH6      | neooli     | *Neolamprologus olivaceous*   | Lamprologini  |
| ISA6      | neopul     | *Neolamprologus pulcher*      | Lamprologini  |
| ISB3      | neopul     | *Neolamprologus pulcher*      | Lamprologini  |
| ISA8      | neosav     | *Neolamprologus savoryi*      | Lamprologini  |
| IYA4      | neosav     | *Neolamprologus savoryi*      | Lamprologini  |
| KFD2      | neowal     | *Neolamprologus walteri*      | Lamprologini  |
| KFD4      | neowal     | *Neolamprologus walteri*      | Lamprologini  |

</center>
-->

<a name="requirements"></a>
## Requirements

* **Dsuite:** The [Dsuite](https://github.com/millanek/Dsuite) program allows the fast calculation of the *D*-statistic from SNP data in VCF format. The program is particularly useful because it automatically calculates the *D*-statistic for all possible species trios, optionally also in a way that the trios are compatible with a user-provided species tree. Instructions for download and installation on Mac OS X and Linux are provided on [https://github.com/millanek/Dsuite](https://github.com/millanek/Dsuite). Installation on Windows is not supported, but Windows users can use the provided output files to learn how to plot and analyze the Dsuite output.

* **FigTree:** The program [FigTree](http://tree.bio.ed.ac.uk/software/figtree/) should already be installed if you followed the tutorials [Bayesian Phylogenetic Inference](../bayesian_phylogeny_inference/README.md), [Phylogenetic Divergence-Time Estimation](../divergence_time_estimation/README.md) or other tutorials. If not, you can download it for Mac OS X, Linux, and Windows from [https://github.com/rambaut/figtree/releases](https://github.com/rambaut/figtree/releases).

<a name="simulation"></a>
## 1. Inferring the species-tree and gene-flow from a simulated dataset

### 1.1 Simulating phylogenomic data with msprime
One difficulty with applying and comparing different methods in evolutionary genomics and phylogenomics is that we rarely know what the right answer is. If methods give us conflicting answers, or any answers, how do we know if we can trust them? One approach that is often helpful is the use of simulated data. Knowing the truth allows us to see if the methods we are using make sense. 

One of fastest software packages around for simulating phylogenomic data is the coalescent-based [msprime](https://msprime.readthedocs.io/en/stable/) ([manuscript](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004842)). The msprime manuscript and the software itself are presented in population genetic framework. However, we can use it to produce phylogenomic data. This is because, from the point of view of looking purely at genetic data, there is no fundamental distinction between a set of allopatric populations of a single species and a set of different species. The genealogical processes that play out across different population are indeed the same as the processes that determine the genetic relationships along-the-genome of any species that may arise. We will return to this theme of a continuum between population genetics and phylogenomics later.

We have simulated SNP data for 20 species in the VCF format, two individuals from each species. The species started diverging 1 million years ago, with effective population sizes on each branch set to 50,000. Both the recombination and mutation rates were set to 1e-8 and 20Mb of data were simulated. Because these simulations take some time to run, we have the ready simulated data available for you. First a simulation without gene-flow ([VCF](data/chr1_no_geneflow.vcf.gz), true tree: [image](img/simulated_tree_no_geneflow.pdf), [newick](data/simulated_tree_no_geneflow.nwk), [json](data/simulated_tree_no_geneflow.nwk.mass_migrations.json)), and second, a simulation where five gene-flow events have been added to a tree ([VCF](data/with_gene-flow.vcf.gz), true tree with gene-flow: [image](img/simulated_tree_with_geneflow.pdf), [newick](data/simulated_tree_with_geneflow.nwk), [json](data/simulated_tree_with_geneflow.nwk.mass_migrations.json)). Details for how to generate such simulated datasets are provided below.      

<details>
  <summary>Generating simulated phylogenomic data with msprime</summary>
  
  It is in principle possible to simulate data from an arbitrary phylogeny with msprime, but specifying the phylogenetic tree directly in the program is complicated. Therefore, a number of 'helper' wrapper programs have been developed that can make this task much easier for us. For this exercise, we use Hannes Svardal's [pypopgen3](https://github.com/feilchenfeldt/pypopgen3). After installing pypogen3 and its dependencies, using the instructions on the webpage, we simulated the data using the following code:
  
  ```python3
  from pypopgen3.modules import treetools
  from pypopgen3.modules import simulate
  
  # generating a 'random' tree with 20 species, 2 diploid individuals per species, and no gene-flow events:
  t = treetools.get_random_geneflow_species_tree(n_species=20,n_geneflow_events=0,sample_sizes=4,random_tree_process='ete3')
  # preparing the msprime command input:
  (samples, population_configurations, sorted_events, sample_to_pop, sample_names, id_to_name) = simulate.msprime_input_from_split_tree(t)
  # running the simulation - this takes about 2 hours
  tree_sequence = simulate.simulate_to_vcf("self_chr1_no_geneflow.vcf.gz", samples, population_configurations, sorted_events, 1e-8, 1e-8, 20000000, sample_names=sample_names, chrom_id="chr1")
  # write out the tree:
  t.write(format=1, outfile="self_simulated_tree_no_geneflow.nwk")
  
  # generating a 'random' tree with 20 species, 2 diploid individuals per species, and five randomly placed gene-flow events:
  t_gf = treetools.get_random_geneflow_species_tree(n_species=20,n_geneflow_events=5,sample_sizes=4,random_tree_process='ete3')
  # preparing the msprime command input:
  (samples, population_configurations, sorted_events, sample_to_pop, sample_names, id_to_name) = simulate.msprime_input_from_split_tree(t_gf)
  # running the simulation - this takes about 2 hours
  tree_sequence = simulate.simulate_to_vcf("self_chr1_with_geneflow.vcf.gz", samples, population_configurations, sorted_events, 1e-8, 1e-8, 20000000, sample_names=sample_names, chrom_id="chr1")
  # write out the tree:
  t_gf.write(format=1, outfile="self_simulated_tree_with_geneflow.nwk")
  ```
  
  Note that if you run this code yourselves, the data and the trees that you get out will be somewhat different from the ones we prepared for you, because the trees are randomly generated and the coalescent simulation run by msprime is also a stochastic process. Therefore, every simulation will be different. If you want to simulate more data using the trees we provided to you, you would replace the  ```treetools.get_random_geneflow_species_tree```  commands with the following code to read the provided trees:


 ```python3
    # reads the newick tree without gene-flow (the empty json file also needs to be in the folder) 
    t = treetools.HsTree("simulated_tree_no_geneflow.nwk")
  
    # reads the newick tree and json file with gene-flow evens
    t_gf = treetools.HsTree("simulated_tree_with_geneflow.nwk")
```
  
</details>

<a name="ReconstructingFromSimulation"></a>
### 1.2 Reconstructing phylogenies from simulated data

Now we apply the phylogentic (or phylogenomic) approaches that we have learned to the simulated SNP data to see if we can recover the phylogentic trees that were used as input to the simulations. As in the tutorial on [Species-Tree Inference with SNP Data](../species_tree_inference_with_snp_data/README.md), we are going to use algorithms implemented in [PAUP\*](http://phylosolutions.com/paup-test/). 

Our msprime simulation did not use any specific [substitution model](https://en.wikipedia.org/wiki/Models_of_DNA_evolution) for mutations, but simply designated alleles as `0` for ancestral and `1` for derived. The alleles are indicated in the fourth (REF) and fifth (ALT) column of the VCF as per the [VCF file format](https://samtools.github.io/hts-specs/VCFv4.2.pdf). To use PAUP\* we first need to convert the the VCF into the Nexus format, and this needs the  `0` and `1` alleles to be replaced by actual DNA bases. We can use the [vcf2phylip.py](src/vcf2phylip.py) python script and achieve these steps as follows, first for the dataset simulated without gene-flow:

```bash
# unzip the VCF and process it with AWK to replace each ancestral allele (fourth column) with an A and each derived allele (fifth column) with a T
gunzip -c chr1_no_geneflow.vcf.gz | awk 'BEGIN{OFS=FS="\t"}{ if(NR > 6) { $4="A"; $5="T"; print $0} else {print}};' | gzip -c > chr1_no_geneflow_nt.vcf.gz
# convert the VCF to the Nexus format:
python2 vcf2phylip.py -i chr1_no_geneflow_nt.vcf.gz -p --nexus
```
* Next, open the Nexus file `chr1_no_geneflow_nt.min4.nexus` in PAUP\*, again making sure that the option "Execute" is set in the opening dialog, as shown in the screenshot.<p align="center"><img src="img/PAUP_open.png" alt="PAUP\*" width="600"></p>

* Then designate the outgroup (Data->Define_outgroup) as you learned in the tutorial on [Species-Tree Inference with SNP Data](../species_tree_inference_with_snp_data/README.md).

* Then use the Neighbor Joining algorithm (Analysis->Neighbor-Joining/UPGMA) with default parameters to build a quick phylogeny. 

As you can see by comparison of the tree you just reconstructed (also below) against [the input tree](img/simulated_tree_no_geneflow.pdf), a simple Neigbor Joining algorithm easily reconstructs the tree topology perfectly, and even the branch lengths are almost perfect. 

<details>
<summary>Click here to see the reconstructed NJ tree without gene-flow</summary>

<p align="center"><img src="img/chr1_no_geneflow_bases.min4_NJ.jpg" alt="PAUP\*" width="600"></p>

</details>

Now we repeat the same tree-raconstruction procedure for the simulation with gene-flow, starting with file format conversion: 

```bash
# unzip the VCF and process it with AWK to replace each ancestral allele (fourth column) with an A and each derived allele (fifth column) with a T
gunzip -c with_geneflow.vcf.gz | awk 'BEGIN{OFS=FS="\t"}{ if(NR > 27) { $4="A"; $5="T"; print $0} else {print}};' | gzip -c > with_geneflow_nt.vcf.gz
# convert the VCF to the Nexus format:
python2 vcf2phylip.py -i with_geneflow_nt.vcf.gz -p --nexus
```

Then load the file `with_geneflow_nt.min4.nexus` into PAUP\*, again making sure the option "Execute" is set, then designate the outgroup, and finally run the Neighbor Joining tree reconstruction. You should get a tree like the one below:

<p align="center"><img src="img/with_geneflow_bases.min4_NJ.jpg" alt="PAUP\*" width="600"></p>

An examination of this reconstructed tree reveals that in this case we did not recover the topology of the [true tree](img/simulated_tree_with_geneflow.pdf) used as input to the simulation. Unlike in the true tree, in the reconstructed tree species  `S14` is "pulled outside" the group formed by  `S13,S15,S16`. This is most likely because of the gene-flow that `S14` received from `S00`. This is a typical pattern: when one species from within a group receives introgression from another group, it tends to be "pulled out" like this in phylogenetic reconstruction. One argument that could be made here is that the Neighbor Joining algorithm is outdated, and that perhaps newer, more sophisticated, methods would recover the correct tree. You can now try to apply SVDQuartets in PAUP\*, and also try any of the other phylogenomic methods you know to see if any of these will succeed.

<details>
<summary>Click here to see the reconstructed SVDQuartets tree with gene-flow</summary>

As you can see, the topology is in fact different from the Neighbor Joining, but also is not correct (`S13,S14` should not be sister taxa, also `S10` and `S11` are swapped) . 

<p align="center"><img src="img/with_geneflow_bases.min4_SVDQ.jpg" alt="PAUP\*" width="600"></p>

</details>

<a name="TestingInSimulations"></a>
### 1.3 Testing for gene-flow in simulated data 

Under incomplete lineage sorting alone, two sister species are expected to share about the same proportion of derived alleles with a third closely related species. Thus, if species "P1" and "P2" are sisters and "P3" is a closely related species, then the number of derived alleles shared by P1 and P3 but not P2 and the number of derived alleles that is shared by P2 and P3 but not P1 should be approximately similar. In contrast, if hybridization leads to introgression between species P3 and one out the two species P1 and P2, then P3 should share more derived alleles with that species than it does with the other one, leading to asymmetry in the sharing of derived alleles. These expectations are the basis for the so-called "ABBA-BABA test" (first described in the Supporting Material of [Green et al. 2010](http://science.sciencemag.org/content/328/5979/710.full)) that quantifies support for introgression by the *D*-statistic. Below is an illustration of the basic principle. 

<p align="center"><img src="img/DstatIllustration.png" alt="Dstat\*" width="600"></p>

In short, if there is gene-flow between P2 &lt;-&gt; P3, there is going to be an excess of the of the ABBA pattern, leading to positive D statistics. In contrast, gene-flow between P1 &lt;-&gt; P3 would lead to a an excess of the BABA pattern and a negative D statistic. However, whether a species is assigned in the P1 or P2 position is arbitrary, so we can always assign them so that P2 and P3 share more derived alleles and the D statistic is then bounded between 0 and 1. There is also a related and somewhat more complicated measure,  the f4-ratio, which strives to estimate the admixture proportion in percentage. We will not go into the maths here - if you are interested, have a look at the [Dsuite paper](https://doi.org/10.1111/1755-0998.13265). 


The Dsuite software has several advantages: it brings several related statistics together into one software package, has a straightforward workflow to calculate the D statistics and the f4-ratio for all combinations of trios in the dataset, and the standard VCF format, thus generally avoiding the need for format conversions or data duplication. It is computationally more efficient than other software in the core tasks, making it more practical for analysing large genome-wide data sets with tens or even hundreds of populations or species. Finally, Dsuite implements the calculation of the <i>f</i><sub>dM</sub> and <i>f</i>-branch statistics for the first time in publicly available software.

To calculate the D statistics and the f4-ratio for all combinations of trios of species, all we need is the file that specifies what individuals belong to which population/species - we prepared it for you: [species_sets.txt](data/species_sets.txt). Such a file could be simply prepared manually, but, in this case we can save ourselves the work and automate the process using a combination of `bcftools` and `awk`:   

```bash
bcftools query -l chr1_no_geneflow.vcf.gz | awk '{ if (NR <= 40) {sp=substr($1,1,3); print $1"\t"sp} else {print $1"\tOutgroup";}}' > species_sets.txt
```
Something similar to the above can be useful in many cases, depending on how the individuals are named in your VCF file. 

* Then, to familiarize yourself with Dsuite, simply start the program with the command `Dsuite`. When you hit enter, you should see a help text that informs you about three different commands named "Dtrios", "DtriosCombine", and "Dinvestigate", with short descriptions of what these commands do. Of the three commands, we are going to focus on Dtrios, which is the one that calculates the *D*-statistic for all possible species trios.

* To learn more about the command, type `Dsuite Dtrios` and hit enter. The help text should then inform you about how to run this command. There are numerous options, but the defaults are approprite for a vast majority of use-cases.  All we are going to do is to provide a run name using the `-n` option, the correct tree using the `-t` option, and use the `-c` option to indicate that this is the entire dataset and, therefore, we don't need intermediate files for "DtriosCombine". 

<a name="TestingWithoutGeneflow"></a>
#### 1.3.1 Do we find geneflow in data simulated without geneflow?

We run Dsuite for the dataset without gene-flow as follows:

```bash
Dsuite Dtrios -c -n no_geneflow -t simulated_tree_no_geneflow.nwk chr1_no_geneflow.vcf.gz species_sets.txt 
```
The run takes about 50 minutes. Therefore, we already put the output files for you in the [data](data/) folder.  Let's have a look at the first few lines of  [`species_sets_no_geneflow_BBAA.txt`](data/species_sets_no_geneflow_BBAA.txt):

```bash
head species_sets_no_geneflow_BBAA.txt
P1    P2    P3    Dstatistic    Z-score    p-value    f4-ratio    BBAA    ABBA    BABA
S01    S02    S00    0.00645161    0.228337    0.409692    7.6296e-05    40635    936    924
S01    S00    S03    0.0299321    1.08142    0.139754    0.000543936    28841    1724.75    1624.5
S01    S00    S04    0.0072971    0.308069    0.379015    0.00013279    28889.2    1691    1666.5
S01    S00    S05    0.00312175    0.133303    0.446977    5.6877e-05    28861.2    1687    1676.5
S01    S00    S06    0.00312175    0.135082    0.446273    5.69626e-05    28876.5    1687    1676.5
S01    S00    S07    0.00490269    0.210116    0.416789    8.92693e-05    28871.5    1691    1674.5
S00    S01    S08    0.0613992    1.7814    0.0374237    0.000460113    45178.5    806    712.75
S00    S01    S09    0.0704225    2.17799    0.0147034    0.000530632    45323    817    709.5
S00    S01    S10    0.07982    2.25263    0.0121413    0.000589494    45131.8    810    690.25
```

Each row shows the results for the analysis of one trio. For example in the first row, species *S01* was used as P1, *S02* was considered as P2, and *S00* was placed in the position of P3. Then we see the D statistic, associated Zscore and p-value, the f4-ratio estimating admixture proportion and then the counts of BBAA sites (where  `S01` and  `S02` share the derived allele) and then the counts of ABBA and BABA sites. As you can see, ABBA is always more than BABA and the D statistic is always positive because Dsuite orients P1 and P2 in this way. Since these results are for coalescent simulations without gene-flow, the ABBA and BABA sites arise purely through incomplete lineage sorting and the difference between them is purely random - therefore, even though the D statistic can be quite high (e.g. up to 8% on the last line), this is not a result of gene flow. 

**Question 1:** Can you tell why the BBAA, ABBA, and BABA numbers are not integer numbers but have decimal positions? [(see answer)](#q1)
**Question 2:** How many different trios are listed in the file? Are these all possible (unordered) trios? [(see answer)](#q2)

In [`species_sets_no_geneflow_BBAA.txt`](data/species_sets_no_geneflow_BBAA.txt), trios are arranged so that P1 and P2 always share the most derived alleles (BBAA is always the highest number). There are two other output files: one with the `_tree.txt` suffix: [`species_sets_no_geneflow_tree.txt`](data/species_sets_no_geneflow_tree.txt) where trios are arranged according to the tree we gave Dsuite, and a file with the `_Dmin.txt` suffix [`species_sets_no_geneflow_Dmin.txt`](data/species_sets_no_geneflow_Dmin.txt) where trios are arranged so that the D statistic is minimised - providing a kind of "lower bound" on gene-flow statistics. This can be useful in cases where the true relationships between species are not clear, as illustrated for example in [this paper](https://doi.org/10.1038/s41559-018-0717-x) (Fig. 2a). 

Let's first see how the other outputs differ from the  `_tree.txt` file, which has the correct trio arrangments. :

```bash
diff species_sets_no_geneflow_BBAA.txt species_sets_no_geneflow_no_geneflow_tree.txt
922c922
< S08    S10    S09    0.0170473    1.22568    0.110159    0.00133751    6829.88    6764    6537.25
---
> S09    S10    S08    0.0218914    1.73239    0.0416019    0.00172128    6764    6829.88    6537.25
```
There is one difference in the `_BBAA.txt` ile. Because of incomplete lineage sorting being a stochastic (random) process, we see that  `S08` and  `S10` share more derived alleles than `S09` and  `S10`, which are sister species in the input tree. If you look again at the input tree, you will see that  the branching order between `S08`,  `S09` and  `S10` is very rapid, it is almost a polytomy.

A comparison of the `_tree.txt` file against the `_Dmin.txt`, which minimises the Dstatistic, reveals nine differences. However, the correct trio arrangements in all these cases are very clear.

```bash
diff species_sets_no_geneflow_Dmin.txt species_sets_no_geneflow_no_geneflow_tree.txt
```

Next, let's look at the results in more detail, for example in [R](https://www.r-project.org). We load the `_BBAA.txt` file and first look at the distribution of D values:  

```R
D_BBAA_noGF <- read.table("species_sets_no_geneflow_BBAA.txt",as.is=T,header=T)
plot(D_BBAA_noGF$Dstatistic, ylab="D",xlab="trio number")
```

<p align="center"><img src="img/no_geneflow_Dvals.png" alt="DstatNoGF\*" width="600"></p>

There are some very high D statistics. In fact, the D statistics for 9 trios are &gt;0.7, which is extremely high. So how is this possible in a dataset simulated with no geneflow?

```R
D_BBAA_noGF[which(D_BBAA_noGF$Dstatistic > 0.7),]
     P1  P2  P3 Dstatistic Z.score    p.value    f4.ratio   BBAA ABBA BABA
171 S18 S19 S00   1.000000 0.00000        NaN 7.42372e-06 179922  1.5    0
324 S18 S19 S01   1.000000 0.00000        NaN 7.42743e-06 179814  1.5    0
460 S18 S19 S02   1.000000 0.00000        NaN 7.39892e-06 179800  1.5    0
580 S18 S19 S03   1.000000 0.00000        NaN 7.43931e-06 179778  1.5    0
685 S18 S19 S04   1.000000 0.00000        NaN 7.44097e-06 179765  1.5    0
690 S06 S05 S11   0.833333 2.91667 0.00176897 4.94462e-05 138120 11.0    1
776 S18 S19 S05   1.000000 0.00000        NaN 7.43290e-06 179768  1.5    0
854 S18 S19 S06   1.000000 0.00000        NaN 7.44536e-06 179777  1.5    0
920 S18 S19 S07   1.000000 0.00000        NaN 7.42259e-06 179771  1.5    0
```
These nine cases arise because there is amost no incomplete lineage sorting among these trios almost all sites are `BBAA` - e.g. 179922 sites for the first trio, while the count for `ABBA` is only 1.5 and for `BABA` it is 0 . The D statistic is calculated as D = (ABBA-BABA)/(ABBA+BABA), which for the first trio would be D = (1.5-0)/(1.5+0)=1. So, the lesson here is that the D statistic is very sensitive to random fluctuations when there is a small number of ABBA and BABA sites. One certainly cannot take the D value seriously unless it is supported by a statistical test suggesting that the D is significanly different from 0. In the most extreme cases above, the p-value could not even be calculated, becuase there were so few sites. Those definitely do not represent geneflow. But in one case we see a p value of 0.0018. Well, that looks significant, if one considers for example the traditional 0.05 cutoff. So, again, how is this possible in a dataset simulated with no geneflow?

```R
plot(D_BBAA_noGF$p.value, ylab="p value",xlab="trio number",ylim=c(0,0.05))
```
<p align="center"><img src="img/no_geneflow_Pvals.png" alt="DstatNoGF-Pvals\*" width="600"></p>

In fact, there are many p values that are &lt;0.05. For those who have a good understanding of statistics this will be not be suprising. This is because [p values are uniformly distributed](https://neuroneurotic.net/2018/10/29/p-values-are-uniformly-distributed-when-the-null-hypothesis-is-true/) when the null hypopthesis is true. Therefore, we expect 5% of the (or 1 in 20) p-values, they will be &lt;0.05. If we did a 1140 tests, we can expect 57 of them to be &lt;0.05. Therefore, any time we conduct a large amount of statistical tests, we should apply a multiple testing correction - commonly used is the Benjamini-Hochberg (BH) correction which controls for the [false discovery rate](https://en.wikipedia.org/wiki/False_discovery_rate).

```R
plot(p.adjust(D_BBAA_noGF$p.value,method="BH"), ylab="p value",xlab="trio number",ylim=c(0,0.05))
```
<p align="center"><img src="img/no_geneflow_correctedPvals.png" alt="DstatNoGF-PvalsCorrected\*" width="600"></p>

However, even after applying the BH correction there are three p-values which look significant. These are all false discoveries. Here comes an important scientific lesson - that even if we apply statistical tests correctly, seeing a p-value below 0.05 is not a proof that the null hypothesis is false. All hypothesis testing has [false positives and false negatives](https://en.wikipedia.org/wiki/Type_I_and_type_II_errors). It may be helpful to focus less on statistical testing and aim for a more nuanced understanding of the dataset, as argued for example in this [Nature commentary](https://www.nature.com/articles/d41586-019-00857-9). 

Therefore, we should also plot the f4-ratio, which estimates the proportion of genome affected by geneflow. It turns out that this is perhaps the most reliable - all the f4-ratio values are tiny, as they should be for a dataset without geneflow.  

```R
plot(D_BBAA_noGF$f4.ratio, ylab="f4-ratio",xlab="trio number", ylim=c(0,1))
```
<p align="center"><img src="img/no_geneflow_f4-ratios.png" alt="DstatF4ratio\*" width="600"></p>

Finally, we use visualisation heatmap in which the species in positions P2 and P3 are sorted on the horizontal and vertical axes, and the color of the corresponding heatmap cell indicates the most significant *D*-statistic found between these two species, across all possible species in P1. To prepare this plot, we need to prepare a file that lists the order in which the P2 and P3 species should be plotted along the heatmap axes. The file should look like  [`plot_order.txt`](data/plot_order.txt). You could prepare this file manually, or below is a programmatic way: 

```bash
cut -f 2 species_sets.txt | uniq | head -n 20 > plot_order.txt
```

Then make the plots using the scripts [`plot_d.rb`](src/plot_d.rb) and [`plot_f4ratio.rb`](src/plot_f4ratio.rb). 

```bash
ruby plot_d.rb species_sets_no_geneflow_BBAA.txt plot_order.txt 0.7 species_sets_no_geneflow_BBAA_D.svg
ruby plot_f4ratio.rb species_sets_no_geneflow_BBAA.txt plot_order.txt 0.7 species_sets_no_geneflow_BBAA_f4ratio.svg
```

<p align="center"><img src="img/species_sets_no_geneflow_BBAA_Dandf4ratio.png" alt="DstatNoGF-PvalsCorrected\*" width="600"></p>



<a name="TestingWithGeneflow"></a>
#### 1.3.2 Do we find geneflow in data simulated with geneflow?

How do the results for the simulation with geneflow differ from the above? Here we are going to run a similar set of analyses and make comparisons. We run Dsuite for the dataset with gene-flow as follows:

```bash
Dsuite Dtrios -c -n with_geneflow -t simulated_tree_with_geneflow.nwk with_geneflow.vcf.gz species_sets.txt 
```

Again, this takes about 50 minutes, so the output files  [`species_sets_with_geneflow_BBAA.txt`](data/species_sets_with_geneflow_BBAA.txt),  [`species_sets_with_geneflow_tree.txt`](data/species_sets_with_geneflow_tree.txt) and [`species_sets_with_geneflow_Dmin.txt`](data/species_sets_with_geneflow_Dmin.txt) are already provided.

Now we find 39 differences between the  `_tree.txt` file and the `_BBAA.txt`, reflecting that, under geneflow, sister species often do not share the most derived alleles. Between  `_tree.txt` file and the `_Dmin.txt` there are 124 differences.

```bash
diff species_sets_with_geneflow_BBAA.txt species_sets_with_geneflow_tree.txt | grep -c '<'
diff species_sets_with_geneflow_Dmin.txt species_sets_with_geneflow_tree.txt | grep -c '<'
```

We can visualise the overlap between these different files using a Venn diagram. For example in R:

```R
library("VennDiagram")
D_BBAA <- read.table("species_sets_with_geneflow_BBAA.txt",as.is=T,header=T)
D_Dmin <- read.table("species_sets_with_geneflow_Dmin.txt",as.is=T,header=T)
D_tree <- read.table("species_sets_with_geneflow_tree.txt",as.is=T,header=T)

D_BBAA$P123 <- paste(D_BBAA$P1,D_BBAA$P2,D_BBAA$P3,sep="_"); D_Dmin$P123 <- paste(D_Dmin$P1, D_Dmin$P2, D_Dmin$P3,sep="_"); D_tree$P123 <- paste(D_tree$P1, D_tree$P2, D_tree$P3,sep="_")

draw.triple.venn(length(D_BBAA$P123),length(D_Dmin$P123),length(D_tree$P123),length(which(D_BBAA$P123 == D_Dmin$P123)),length(which(D_tree$P123 == D_Dmin$P123)),length(which(D_tree$P123 == D_BBAA$P123)),length(which(D_tree$P123 == D_BBAA$P123 & D_Dmin$P123 == D_BBAA$P123 & D_Dmin$P123 == D_tree$P123)),category = c("BBAA", "Dmin", "tree"), lty = "blank", fill = c("skyblue", "pink1", "darkolivegreen1"))
```

<p align="center"><img src="img/trippleVenn.png" alt="trippleVenn\*" width="600"></p>

Then we explore the results in the `_BBAA.txt` in R, analogously to how we did it above for the no-geneflow case:

```R
plot(D_BBAA$Dstatistic, ylab="D",xlab="trio number") # The D statistic
plot(p.adjust(D_BBAA$p.value,method="BH"), ylab="corected p value",xlab="trio number",ylim=c(0,0.05))
plot(D_BBAA$f4.ratio, ylab="f4-ratio",xlab="trio number", ylim=c(0,0.2))
```

<details>
<summary>Click here to see the resulting plots:</summary>

<p align="center"><img src="img/geneflow_Dvals.png" alt="Dvals\*" width="600"></p>
<p align="center"><img src="img/geneflow_correctedPvals.png" alt="pvals\*" width="600"></p>
<p align="center"><img src="img/geneflow_f4-ratios.png" alt="f4\*" width="600"></p>

</details>

The statistics are markedly different for the no-geneflow scenario. The number of D statistics &gt;0.7 here is 85 (compared with 9 above) and very many have significant p values - even after FDR correction, we have p<0.05 for whopping 671 trios. Finally, the f4-ratios are also elevated, up to almost 15% in some cases.    

<p>
<img style="float: left;" src="img/simulated_tree_with_geneflow.png" width="200"> 

To remind ourselves, the simulated tree and geneflow events are shown on the left. The 15% f4-ratios estimates correspond reasonably well with the strength of the five geneflow events that we simulated (in the region of 8% to 18%).   
</p>




## Identifying introgression with *D*-statistics

When more than four species are included in the dataset, obtaining *D*-statistics has long been a bit tedious because no program was available to calculate them automatically for all possible combinations of species in a VCF input file. This gap the available methodology, however, has recently been filled with the program Dsuite by Milan Malinsky ([Malinsky 2019](https://www.biorxiv.org/content/10.1101/634477v1)), which will be used in this tutorial. To calculate *D*-statistics not just for one specific quartet but comprehensively for sets of species in a VCF file, Dsuite keeps the outgroup (which is specified by the user) fixed, but tests all possible ways in which three species can be selected from the ingroup species and placed into the positions P1, P2, and P3. In addition to the *D*-statistic, Dsuite allows the calculation of a p-value based on jackknifing for the null hypothesis that the *D*-statistic is 0, which means that the absence of introgression can be rejected if the p-value is below the significance level.

In this part of the tutorial, we are going to calculate *D*-statistics with Dsuite to test for introgression among sets of species from our dataset. We will apply these statistics in particular to determine potential introgression into *Neolamprologus cancellatus*, the species that had been excluded from species-tree analyses in tutorials [Species-Tree Inference with SNP Data](../species_tree_inference_with_snp_data/README.md) and [Divergence-Time Estimation with SNP Data](../divergence_time_estimation_with_snp_data/README.md) due to its presumed hybrid nature. As mentioned in tutorial [Species-Tree Inference with SNP Data](../species_tree_inference_with_snp_data/README.md), *Neolamprologus cancellatus* ("neocan") is not accepted as a valid species by some authors who speculate that it is a "natural hybrid between *Telmatochromis vittatus* and another species" ([Konings 2015](http://www.cichlidpress.com/books/details/tc3.html)), based on field observations.

* Make sure that you have the file [`NC_031969.f5.sub1.vcf.gz`](data/NC_031969.f5.sub1.vcf.gz) in your current analysis directory, either by copying it from the analysis directory for tutorial [Species-Tree Inference with SNP Data](../species_tree_inference_with_snp_data/README.md), or by downloading it with the link. Make sure not to uncompress it yet.
	
* To assign samples to species for the calculation of the *D*-statistic, we'll need to write a table with two tab-delimited columns, where the first column contains the sample IDs and the second column contains the corresponding species ID. We are going to use the only haplochromine species in the dataset, *Astatotilapia burtoni* as the outgroup (its outgroup position was confirmed in tutorial [Divergence-Time Estimation with SNP Data](../divergence_time_estimation_with_snp_data/README.md)), which needs to be specified for Dsuite with the keyword "Outgroup". Thus, write the following text to a new file named `samples.txt`:

		IZA1	Outgroup
		IZC5	Outgroup
		AUE7	altfas
		AXD5	altfas
		JBD5	telvit
		JBD6	telvit
		JUH9	neobri
		JUI1	neobri
		LJC9	neocan
		LJD1	neocan
		KHA7	neochi
		KHA9	neochi
		IVE8	neocra
		IVF1	neocra
		JWH1	neogra
		JWH2	neogra
		JWG8	neohel
		JWG9	neohel
		JWH3	neomar
		JWH4	neomar
		JWH5	neooli
		JWH6	neooli
		ISA6	neopul
		ISB3	neopul
		ISA8	neosav
		IYA4	neosav
		KFD2	neowal
		KFD4	neowal

	(note that the file with the same name written in tutorial [Divergence-Time Estimation with SNP Data](../divergence_time_estimation_with_snp_data/README.md) should not be re-used here as its content differs).



* As shown at the top of the Dsuite help text, the program can be run with `Dsuite Dtrios [OPTIONS] INPUT_FILE.vcf SETS.txt`. In our case, the input file is [`NC_031969.f5.sub1.vcf.gz`](data/NC_031969.f5.sub1.vcf.gz) and the sets files is the table with sample and species IDs written earlier to [`samples.txt`](res/samples.txt). Thus, start the Dsuite analysis of our dataset with

		Dsuite Dtrios NC_031969.f5.sub1.vcf.gz samples.txt
		
	The program should finish within a few minutes.

* Now have a look at the files in the current directory, using the `ls` command. You should see that Dsuite has written a number of files named after the sample-table file:

		samples__BBAA.txt
		samples__Dmin.txt
		samples__combine.txt
		samples__combine_stderr.txt
		
	1. In a first set of calculations, all possible rearrangements are tested, and the lowest *D*-statistic, called *D*<sub>min</sub> is reported for each trio. *D*<sub>min</sub> is therefore a conservative estimate of the *D*-statistic in a given trio. This output is written to a file with the ending `__Dmin.txt`.
	2. The trio is arranged so that P1 and P2 are the two species of the trio that share the largest number of derived sites. In other words, the rearrangement is done so that *C*<sub>BBAA</sub> > *C*<sub>ABBA</sub> and *C*<sub>BBAA</sub> > *C*<sub>BABA</sub>. In addition, P1 and P2 are rotated so that the number of derived sites shared between P2 and P3 is larger than that between P1 and P3. In sum this means that *C*<sub>BBAA</sub> > *C*<sub>ABBA</sub> > *C*<sub>BABA</sub>. The idea behind this type of rearrangement is that the two species that share the largest number of derived sites are most likely the true two sister species in the trio, and thus rightfully placed in the positions P1 and P2. This assumption is expected to hold under certain conditions (e.g. clock-like evolution, absence of homoplasies, absence of introgression, and panmictic ancestral populations), but how reliable it is for real datasets is sometimes difficult to say. *D*-statistics based on this type of rearrangement are written to a file with ending `__BBAA.txt`.
	3. To tell Dsuite directly which species of a trio should be considered sister species (and thus, which should be assigned to P1 and P2), a tree file that contains all species of the dataset can be provided by the user with option `-t` or `--tree`. The output will then be written to an additional file with ending `__tree.txt`. We will use this option later in the tutorial.

* So, let's see how the *D*-statistics were calculated based on the first two rules for trio rearrangement. Have a look at file [`samples__Dmin.txt`](res/samples__Dmin.txt), e.g. with the `less` command:

		less samples__Dmin.txt
		
	You should see the first part of the file as shown below:
	
		P1      P2      P3      Dstatistic      p-value
		altfas  neocan  neobri  0.493787        0
		neobri  neochi  altfas  0.00885755      0.3836
		neocra  neobri  altfas  0.0372849       0.013765
		neogra  neobri  altfas  0.0184394       0.258714
		neohel  neobri  altfas  0.0448571       0.113967
		neomar  neobri  altfas  0.0679957       0.0195241
		neooli  neobri  altfas  0.0662179       0.0133793
		neopul  neobri  altfas  0.0470166       0.10957
		neosav  neobri  altfas  0.0338796       0.103889
		neowal  neobri  altfas  0.0172581       0.266909
		neobri  telvit  altfas  0.0203511       0.199163
		altfas  neocan  neochi  0.481745        0
		altfas  neocan  neocra  0.491942        0
		altfas  neocan  neogra  0.497877        0
		altfas  neocan  neohel  0.501518        0
		altfas  neocan  neomar  0.492416        0
		altfas  neocan  neooli  0.504355        0
		...

	One thing to notice here is that the species in the first three rows are no longer sorted alphabetically, so obviously they have been rearranged. As the header line indicates, the fourth column now shows the *D*-statistic for the trio in this constellation, and the fifth column shows the *p*-value based on jackknifing for the null hypothesis of *D* = 0.
	
* Let's pick one species trio and see how it appears in the three different files [`samples__combine.txt`](res/samples__combine.txt), [`samples__Dmin.txt`](res/samples__Dmin.txt), and [`samples__BBAA.txt`](res/samples__BBAA.txt). We'll select the first species trio, including "altfas", "neocan", and "neobri". To see only the line for this trio from the three files, you could use this command:

		cat samples__combine.txt | grep altfas | grep neocan | grep neobri
		cat samples__Dmin.txt | grep altfas | grep neocan | grep neobri
		cat samples__BBAA.txt | grep altfas | grep neocan | grep neobri

	This should produce the following three output lines:
	
		altfas	neobri	neocan	1378.2	13479.6	4066.95
		altfas	neocan	neobri	0.493787	0
		altfas	neocan	neobri	0.493787	0


	You'll see that compared to the alphabetical arrangement in [`samples__combine.txt`](res/samples__combine.txt), P1 has remained the same ("altfas") in the other two files, but P2 and P3 have been swapped ("neocan" and "neobri"). This swap also implied that the counts for ABBA, BABA, and BBAA patterns were swapped accordingly, so that after the swap and with P1="altfas", P2="neocan", P3="neobri", the counts were as follows: *C*<sub>ABBA</sub> = 4066.95, *C*<sub>BABA</sub> = 1378.2, and *C*<sub>BBAA</sub> = 13479.6. Thus, "neocan" and "neobri" share 4066.95 derived sites, "altfas" and "neobri" share 1378.2 derived sites, and "altfas" and "neocan" share 13479.6 derived sites. With these counts, *D* = (4066.95 - 1378.2) / (4066.95 + 1378.2) = 0.493787, as correctly reported by Dsuite in both files [`samples__Dmin.txt`](res/samples__Dmin.txt) and [`samples__BBAA.txt`](res/samples__BBAA.txt). **Question 3:** With *C*<sub>ABBA</sub> = 4066.95, *C*<sub>BABA</sub> = 1378.2, and *C*<sub>BBAA</sub> = 13479.6, the rule *C*<sub>BBAA</sub> > *C*<sub>ABBA</sub> > *C*<sub>BABA</sub> is fulfilled for this trio rearrangement, and thus it is not surprising that this particular rearrangement is reported in the output corresponding to this rule, the file ending in `__BBAA.txt`. But the fact that the same rearrangement is also included in the file ending in `__Dmin.txt` implies that the *D*-statistic for this rearrangement is smaller than it would be in all alternative rearrangements. Can you confirm this? [(see answer)](#q3)

* Repeat the same as in the last step, but this time using a species trio for which the rearrangements differ between the files [`samples__Dmin.txt`](res/samples__Dmin.txt) and [`samples__BBAA.txt`](res/samples__BBAA.txt). The trio "neobri", "neocra", and "neogra" is one such example. Use these commands to see the line for this trio in all three files:

		cat samples__combine.txt | grep neobri | grep neocra | grep neogra
		cat samples__Dmin.txt | grep neobri | grep neocra | grep neogra
		cat samples__BBAA.txt | grep neobri | grep neocra | grep neogra

	This should produce these output lines:
	
		neobri	neocra	neogra	3788.23	3552.38	2992.93
		neogra	neocra	neobri	0.0321294	0.145298
		neocra	neobri	neogra	0.0854723	8.58201e-07

	The results in file [`samples__BBAA.txt`](res/samples__BBAA.txt) indicate that when P1="neocra", P2="neobri", and P3="neogra", then *C*<sub>BBAA</sub> = 3788.23, *C*<sub>ABBA</sub> = 3552.38, and *C*<sub>BABA</sub> = 2992.93, and therefore *D* = (3552.38 - 2992.93) / (3552.38 + 2992.93) = 0.0854723.
	
	However, the results in file [`samples__Dmin.txt`](res/samples__Dmin) show that this time, another rearrangement (and therefore a rearrangement in which *C*<sub>BBAA</sub> is not larger than the two other counts) produces a lower *D*-statistic: With P1="neogra", P2="neocra", and P3="neobri", then *C*<sub>BBAA</sub> = 2992.93, *C*<sub>ABBA</sub> = 3788.23, and *C*<sub>BABA</sub> = 3552.38, and therefore *D* = (3788.23 - 3552.38) / (3788.23 + 3552.38) = 0.0321294. This illustrates that the *D*<sub>min</sub> value, reporting the lowest possible *D*-statistic for a given trio, sometimes selects an arrangement of the trio in which P1 and P2 actually share less derived sites with each other than both of them share with P3. This is in conflict with the original assumption of the ABBA-BABA test that P1 and P2 are more closely related to each other than to P3. When interpreting the results of Dsuite analyses, it should therefore be kept in mind that the *D*<sub>min</sub> values reported in the file ending in `__Dmin.txt` are in fact conservative estimates of the *D*-statistic. The values reported in the file ending in `__BBAA.txt`, which are based on the rearrangement that ensures *C*<sub>BBAA</sub> > *C*<sub>ABBA</sub> > *C*<sub>BABA</sub>, may often be better measurements of the *D*-statistic, however, the best option may be to also run the analysis with the `--tree` option, providing an input tree that directly tells Dsuite how to rearrange all trios.
	
To use the `--tree` option of Dsuite, we will obviously need a tree file. As a basis for that, we can use the time-calibrated species tree generated in tutorial [Divergence-Time Estimation with SNP Data](../divergence_time_estimation_with_snp_data/README.md), in file [`snapp.tre`](data/snapp.tre). However, we need to prepare the tree file before we can use it with Dsuite. First, Dsuite requires tree files in Newick format, but the tree file [`snapp.tre`](data/snapp.tre) is written in Nexus format. Second, as the tree generated in tutorial [Divergence-Time Estimation with SNP Data](../divergence_time_estimation_with_snp_data/README.md) did not include *Neolamprologus cancellatus*, we will need to add this species to the tree manually.

* To convert the tree file from Nexus to Newick format, simply open [`snapp.tre`](data/snapp.tre) in FigTree, and select "Export Trees..." from FigTree's "File" menu. In the dialog window, select "Newick" from the drop-down menu next to "Tree file format:", as shown in the next screenshot.<p align="center"><img src="img/figtree1.png" alt="FigTree" width="600"></p> Name the new file `snapp.as_newick.tre`.

	The tree file [`snapp.as_newick.tre`](res/snapp.as_newick.tre) should now have the following content, in which branch lengths are encoded by numbers following colon symbols:

		((altfas:2.594278046475669,((((((neobri:0.4315861683285048,(neooli:0.3619356529967183,neopul:0.3619356529967182):0.06965051533178657):0.12487340944557368,(neogra:0.5002021127811682,neohel:0.5002021127811681):0.05625746499291029):0.046536377715870936,(neocra:0.3843764668581544,neomar:0.3843764668581544):0.21861948863179498):0.13696107245777023,neosav:0.7399570279477196):0.3048349276159825,telvit:1.044791955563702):0.06307730474269935,(neochi:0.1368491602607037,neowal:0.1368491602607037):0.9710201000456977):1.4864087861692676):3.570689300019641,astbur:6.16496734649531);

* To add *Neolamprologus cancellatus* ("neocan") to the tree, we need to make a decision about where to place it. Perhaps the best way to do this would be a separate phylogenetic analysis, but as we will see later, there is no single true position of *Neolamprologus cancellatus* ("neocan") in the species tree anyway. So for now it will be sufficient to use the counts of shared derived sites from file [`samples__BBAA.txt`](res/samples__BBAA.txt) as an indicator for where best to place the species. Thus, we need to find the largest count of derived sites that *neocan* shares with any other species. To do this, we can use the following commands:

		cat samples__combine.txt | grep neocan | sort -n -k 4 -r | head -n 1
		cat samples__combine.txt | grep neocan | sort -n -k 5 -r | head -n 1
		cat samples__combine.txt | grep neocan | sort -n -k 6 -r | head -n 1

	This should produce the following lines:
	
		altfas	neocan	neooli	14834.4	1408.33	4274.5
		altfas	neobri	neocan	1378.2	13479.6	4066.95
		neocan	neochi	neowal	801.883	754.94	12863.7

	The largest number of all of these lines is 14834.4. As this number is found in the fourth column, it specifies the count of BBAA sites, *C*<sub>BBAA</sub>, and therefore the number of derived sites shared by the first two species on this line, "altfas" and "neocan". This means that "neocan" appears to be more closely related to "altfas" than to other species in the dataset.
	
* To insert "neocan" into the tree string as the sister species of "altfas", replace "altfas" with "(altfas:1.0,neocan:1.0)", including the parentheses (the branch length of 1.0 is arbitrary but will not be used by Dsuite anyway). This could be done using a text editor, or on the command line with the following command, saving the modified tree string in a new file named `snapp.complete.tre`:

		cat snapp.as_newick.tre | sed "s/altfas/(altfas:1.0,neocan:1.0)/g" > snapp.complete.tre

	File `snapp.complete.tre` should then contain the following tree string:
	
		(((altfas:1.0,neocan:1.0):2.594278046475669,((((((neobri:0.4315861683285048,(neooli:0.3619356529967183,neopul:0.3619356529967182):0.06965051533178657):0.12487340944557368,(neogra:0.5002021127811682,neohel:0.5002021127811681):0.05625746499291029):0.046536377715870936,(neocra:0.3843764668581544,neomar:0.3843764668581544):0.21861948863179498):0.13696107245777023,neosav:0.7399570279477196):0.3048349276159825,telvit:1.044791955563702):0.06307730474269935,(neochi:0.1368491602607037,neowal:0.1368491602607037):0.9710201000456977):1.4864087861692676):3.570689300019641,astbur:6.16496734649531);

* Run Dsuite again, this time adding the `-t` (or `--tree`) option to specify the newly prepared tree file [`snapp.complete.tre`](res/snapp.complete.tre):

		Dsuite Dtrios -t snapp.complete.tre NC_031969.f5.sub1.vcf.gz samples.txt

	Dsuite should again finish the analysis within a few minutes. The output should be identical to the previously written output except that a file named `samples__tree.txt` should now also be written. **Question 4:** Do the results differ when trios are rearranged either to maximize the number of derived sites shared between P1 and P2 (thus, the results in file [`samples__BBAA.txt`](res/samples__BBAA.txt)) or according to the provided tree (in file [`samples__tree.txt`](res/samples__tree.txt))?

* To further explore differences among the output files based on different rules for trio rearrangement, find the highest *D*-statistic reported in each of the files [`samples__Dmin.txt`](res/samples__Dmin.txt)), [`samples__BBAA.txt`](res/samples__BBAA.txt)), and [`samples__tree.txt `](res/samples__tree.txt)). One way to do this are the three following commands:

		cat samples__Dmin.txt | tail -n +2 | sort -n -k 4 -r | cut -f 4 | head -n 1
		cat samples__BBAA.txt | tail -n +2 | sort -n -k 4 -r | cut -f 4 | head -n 1
		cat samples__tree.txt | tail -n +2 | sort -n -k 4 -r | cut -f 4 | head -n 1
		
	You'll see that the highest *D*-statistic reported in file [`samples__Dmin.txt`](res/samples__Dmin.txt)) (0.504355) is smaller than those in the the other two files (0.778765 in both cases). This is not surprising, given that, as explained above, the *D*<sub>min</sub> values are conservative measure of introgression in a given trio.

* Find the trio responsible for these extreme *D*-statistics, using the following commands:

		cat samples__Dmin.txt | tail -n +2 | sort -n -k 4 -r | head -n 1
		cat samples__BBAA.txt | tail -n +2 | sort -n -k 4 -r | head -n 1
		cat samples__tree.txt | tail -n +2 | sort -n -k 4 -r | head -n 1

	You'll see that the high *D*-statistic of 0.778765 is reported for the trio in which P1="altfas", P2="neocan", and P3="telvit".
	
* Look up the counts of shared sites for this trio from file [`samples__combine.txt`](res/samples__combine.txt), using the following command:

		cat samples__combine.txt | grep altfas | grep neocan | grep telvit
		
	You should then see that *C*<sub>BBAA</sub> = 12909.6 (in the fourth column), *C*<sub>BABA</sub> = 893.302 (in the fifth column), and *C*<sub>ABBA</sub> = 7182.28 (in the sixth column). Thus, "altfas" and "neocan" share 12909.6 derived sites, "neocan" and "telvit" share 7182.28 derived sites, but "altfas" and "telvit" share only 893.302 derived sites.
	
	The *D*-statistic is therefore
	
	*D* = (7182.28 - 893.302) / (7182.28 + 893.302) = 0.778765
	
	as reported in files [`samples__BBAA.txt`](res/samples__BBAA.txt)) and [`samples__tree.txt `](res/samples__tree.txt)). In file [`samples__Dmin.txt`](res/samples__Dmin.txt)), however, *D* was calculated for this trio as 
	
	*D* = (12909.6 - 7182.28) / (12909.6 + 7182.28) = 0.285055
	
	after rearranging the species so that P1="telvit", P2="altfas", and P3="neocan", and thus *C*<sub>BBAA</sub> = 893.302, *C*<sub>ABBA</sub> = 12909.6 and *C*<sub>BABA</sub> = 7182.28. This *D*-statistic is lower than those reported for other trios in file [`samples__Dmin.txt`](res/samples__Dmin.txt)), therefore, the command `cat samples__Dmin.txt | tail -n +2 | sort -n -k 4 -r | head -n 1` reported a different trio that included "neooli" instead of "telvit". Either way, the *p*-values for the *D*-statistics are highly significant for many trios and in particular for those that include *Neolamprologus cancellatus* ("neocan"), indicating introgression involving this species is highly supported. Given that the highest *D*-statistics are reported for the trio in which P1="altfas", P2="neocan", and P3="telvit", we may hypothesize that *Neolamprologus cancellatus* "neocan" has received a large amount of introgression from *Telmatochromis vittatus*, in agreement with the speculation by [Konings (2015)](http://www.cichlidpress.com/books/details/tc3.html) that *Neolamprologus cancellatus* is a "natural hybrid between *Telmatochromis vittatus* and another species".

* To get a better overview of introgression patterns supported by *D*-statistics, we'll plot these in the form of a heatmap in which the species in positions P2 and P3 are sorted on the horizontal and vertical axes, and the color of the corresponding heatmap cell indicates the most significant *D*-statistic found with these two species, across all possible species in P1. To prepare this plot, we will, however, first need to prepare a file that lists the order in which the P2 and P3 species should be listed along the heatmap axes. It makes sense to sort species for this according to the tree file that we used ([`snapp.complete.tre`](res/snapp.complete.tre)); thus, write the following list to a new file named `species_order.txt`:

		altfas
		neocan
		neochi
		neowal
		telvit
		neosav
		neocra
		neomar
		neogra
		neohel
		neobri
		neooli
		neopul
		
	(note that even though the outgroup "astbur" is included in the tree file, we here exclude it because it was never placed in the positions of P2 or P3 in the Dsuite analyses).

* To plot the heatmap, use the Ruby script [`plot_d.rb`](src/plot_d.rb), specifying one of Dsuite's output files, file [`species_order.txt`](res/species_order.txt), the maximum *D* value to plot, and the name of an output file as command-line options. Start the script with the [`samples__Dmin.txt`](res/samples__Dmin.txt) file as input and a maximum *D* of 0.7, and name the output `samples__Dmin.svg`:

		ruby plot_d.rb samples__Dmin.txt species_order.txt 0.7 samples__Dmin.svg
		
* The heatmap plot in file `samples__Dmin.svg` is written scalable vector graphic (SVG) format. Open this file with a program capable of reading files in SVG format, for example with a browser such as Firefox or with Adobe Illustrator. The heatmap plot should look as shown below:<p align="center"><img src="img/samples__Dmin.png" alt="Heatmap Dmin" width="600"></p> As you can see from the color legend in the bottom right corner, the colors of this heatmap show two things at the same time, the *D*-statistic as well as its *p*-value. So red colors indicate higher *D*-statistics, and generally more saturated colors indicate greater significance. Thus, the strongest signals for introgression are shown with saturated red, as in the bottom right of the color legend. While none of the cells in this plot actually have that color, almost all cells in rows or columns for "neocan" have a red-purple color corresponding to a highly significant *D*-statistic around 0.3.

	In case that you're wondering why the heatmap looks symmetric, that is because no matter whether the maximum *D*-statistic for two species is obtained with the first species in position P2 and the second in position P3 or vice versa, the same value is plotted. The reason for this is that even though different *D*-statistics may result in the two cases, those differences should not be taken as evidence for a directionality of introgression from P3 to P2; it could just as well have come from P2 and into P3.
		
* Before further interpreting the patterns of introgression shown in the heatmap, we'll also produce heatmap plots for the other two of Dsuite's output files, [`samples__BBAA.txt`](res/samples__BBAA.txt) and [`samples__tree.txt`](res/samples__tree.txt):

		ruby plot_d.rb samples__BBAA.txt species_order.txt 0.7 samples__BBAA.svg
		ruby plot_d.rb samples__tree.txt species_order.txt 0.7 samples__tree.svg

	The two heatmaps should look as shown below (`samples__BBAA.svg` at top, `samples__tree.svg` below):<p align="center"><img src="img/samples__BBAA.png" alt="Heatmap BBAA" width="600"></p><p align="center"><img src="img/samples__tree.png" alt="Heatmap tree" width="600"></p> As you can see, both of the above heatmaps are overall a bit more saturated than the first heatmap based on *D*<sub>min</sub>, in agreement with the interpretation of *D*<sub>min</sub> as a conservative estimate of introgression. The strongest difference, however, is in the patterns shown for *Telmatochromis vittatus* ("telvit"). The most saturated red colors in the latter two plots are those for the cells connecting "neocan" and "telvit", which in contrast is only light blue in the first plot based on *D*<sub>min</sub>. These plots therefore support the hypothesis that introgression occurred between *Neolamprologus cancellatus* ("neocan") and *Telmatochromis vittatus* ("telvit"). However, the other red-purple cells in the rows and columns for "neocan" and "telvit" may only be side-effects of this introgression. In the last heatmap from file `samples__tree.svg`, for example, the red-purple cells could be caused by sites that are shared between *Neolamprologus cancellatus* ("neocan") and other species only because these other species are closely related to *Telmatochromis vittatus* ("telvit"), the putative donor of introgression.
	
	In addition, there seems to be some support for introgression among the species of the genus *Neolamprologus*, such as between *N. crassus* and *N. brichardi* or between *N. brichardi* and *N. pulcher*.

So far we only calculated *D*-statistics across the entire chromosome 5 (the only chromosome included in the VCF), but we haven't yet tested whether the *D*-statistic is homogeneous or variable throughout the chromosome. This information could be valuable to determine how recent introgression has occurred because younger introgression events are expected to produce more large-scale variation in the *D*-statistic than older introgression events. The information could also help to identify individual introgressed loci.

* To quantify the *D*-statistic in sliding windows across the genome, Dsuite's Dinvestigate command can be used, and we will apply it to the trio with the strongest signals of introgression, composed of the three species *Altolamprologus fasciatus* ("altfas"), *Neolamprologus cancellatus* ("neocan"), and *Telmatochromis vittatus* ("telvit"). Have a look at the available options for this command, simply by typing the following:

		Dsuite Dinvestigate

	You'll see that besides the VCF input file [`NC_031969.f5.sub1.vcf.gz`](data/NC_031969.f5.sub1.vcf.gz) and the samples table [`samples.txt`](res/samples.txt), two more pieces of information are required. One is a file containing only the species names from one or more trios, and the other is a window size (plus the step size by which the window is incremented) that should be used for the sliding window-analyses.
	
* Prepare a file specifying the trio "altfas", "neocan", and "telvit", and name this file `test_trios.txt`. This could be done on the command line with the following command:

		echo -e "altfas\tneocan\ttelvit" > test_trios.txt
		
* We'll use a window size of 2,500 SNPs, incremented by 500 SNPs per iteration. To start sliding-window analyses with these settings, execute the following command:

		Dsuite Dinvestigate -w 2500,500 NC_031969.f5.sub1.vcf.gz samples.txt test_trios.txt 
		
	This analysis should again take only a few minutes. Dsuite should then have written a file named `altfas_neocan_telvit_localFstats__2500_500.txt`.
	
* Have a look at file [`altfas_neocan_telvit_localFstats__2500_500.txt`](res/altfas_neocan_telvit_localFstats__2500_500.txt), for example using the `less` command:

		less altfas_neocan_telvit_localFstats__2500_500.txt
		
	The rows in this file correspond to individual chromosomal windows, and the  six columns included in the file contain information on the chromosome ID and the start and end positions of each window. This is followed by three numbers on each line, which represent the *D*-statistic calculated for the window in the fourth column, and two further statistics aiming to quantify the proportions of the window affected by introgression. The two statistics are the *f*<sub>d</sub> statistic of [Martin et al. (2015)](https://academic.oup.com/mbe/article/32/1/244/2925550) and the related *f*<sub>dM</sub> statistic introduced by [Malinsky et al. (2015)](https://science.sciencemag.org/content/350/6267/1493).
	
* Find out into how many windows chromosome 5 was divided, using the following command:

		cat altfas_neocan_telvit_localFstats__2500_500.txt | tail -n +2 | wc -l
		
	This should show that 509 windows were used in Dsuite's Dinvestigate analysis. If this number would be much higher or lower, the information density could be adjusted by specifying a larger or smaller window size with `-w`.
	
* To visualize the variation of the *D* and *f*<sub>d</sub> statistics across chromosome 5, we'll generate plots with the R environment. If you're familiar with R, you could use the software R Studio or another GUI program for R, but you could also run R on the command line as shown below. To start R interactively on the command line, simply type `R`. Then, write the following commands to read file [`altfas_neocan_telvit_localFstats__2500_500.txt`](res/altfas_neocan_telvit_localFstats__2500_500.txt) and plot the *D* and *f*<sub>d</sub> statistics against the chromosomal position of the center of the alignment:

		table <- read.table("altfas_neocan_telvit_localFstats__2500_500.txt", header=T)
		windowCenter=(table$windowStart+table$windowEnd)/2
		pdf("altfas_neocan_telvit_localFstats__2500_500.pdf", height=7, width=7)
		plot(windowCenter, table$D, type="l", xlab="Position", ylab="D (gray) / fD (black)", ylim=c(0,1), main="Chromosome 5 (altfas, neocan, telvit)", col="gray")
		lines(windowCenter, table$f_d)
		dev.off()

* To exit the interactive R environment and return to the command line, type this command:

		quit(save="no")

	The above commands should have written the plot to a file named [`altfas_neocan_telvit_localFstats__2500_500.pdf`](res/altfas_neocan_telvit_localFstats__2500_500.pdf) in the analysis directory. The plot should show that the *D*-statistic (in gray) is almost in all windows substantially higher than the *f*<sub>d</sub>-statistic (in black), and that the *f*<sub>d</sub>-statistic, which estimates admixture proportion is mostly close 0.5:<p align="center"><img src="img/trio1.png" alt="Dinvestigate" width="600"></p> There also appears to be a dip in both statistics at around 5 Mbp; however, whether this is an artifact resulting from e.g. missing data or misassembly in the reference genome, or whether it shows a biological signal of reduced introgression is difficult to tell without further analyses.

* Repeat the same for a second trio that appeared to show signals of introgression, composed of *Neolamprologus olivaceous* ("neooli"), *N. pulcher* ("neopul), and *N. brichardi* ("neobri"). To do so, use these commands on the command line:

		echo -e "neooli\tneopul\tneobri" > test_trios.txt
		Dsuite Dinvestigate -w 2500,500 NC_031969.f5.sub1.vcf.gz samples.txt test_trios.txt

* To produce the same type of plot as before now for the trio of "neooli", "neopul", and "neobri", use again the R environment and enter the following commands:

		table <- read.table("neooli_neopul_neobri_localFstats__2500_500.txt", header=T)
		windowCenter=(table$windowStart+table$windowEnd)/2
		pdf("neooli_neopul_neobri_localFstats__2500_500.pdf", height=7, width=7)
		plot(windowCenter, table$D, type="l", xlab="Position", ylab="D (gray) / fD (black)", ylim=c(0,1), main="Chromosome 5 (neooli, neopul, neobri)", col="gray")
		lines(windowCenter, table$f_d)
		dev.off()
		
	(again, quit the R environment with `quit(save="no")`). This should have produced the plot shown below, saved to file [`neooli_neopul_neobri_localFstats__2500_500.pdf`](res/neooli_neopul_neobri_localFstats__2500_500.pdf):<p align="center"><img src="img/trio2.png" alt="Dinvestigate" width="600"></p>

	As you can see, the *D*-statistic (in black) now shows more variation and even becomes negative in several windows, indicating that in these windows, *neooli* shares more similarity with *neobri* than *neopul*. Overall, both statistics are lower than for the first trio. Notably, the dip at around 5 Mbp that was apparent in both statistics for the first trio is again visible. As it appears improbable that the same region has more introgression in both species trios, this may indicate that the dip is in fact caused by technical artifacts rather than biological processes. To further investigate whether any of the peaks or troughs in this plot result from actual changes in the introgression proportion, it might be necessary to compare the *D* and *f*<sub>d</sub>-statistics to measures of between-species differentiation (*F*<sub>ST</sub>) and divergence (*d*<sub>XY</sub>) (see, e.g. [The Heliconius Genome Consortium 2012](https://www.nature.com/articles/nature11041)). The reliability of these patterns could also be tested by repeating the analysis after mapping to an alternative, if possible more closely related, reference genome. Such in-depth analyses of introgressed loci based on genome scans are covered, for example, in the [Physalia Speciation Genomics workshop](https://www.physalia-courses.org/courses-workshops/course37/curriculum-37/) run by Mark Ravinet and Joana Meier.


<a name="painting"></a>
## Ancestry painting

A very simple alternative way of investigating patterns of ancestry in potentially introgressed or hybrid species is to "paint" their chromosomes according to the genotypes carried at sites that are fixed between the presumed parental species. This type of plot, termed "ancestry painting" was used for example by [Fu et al. (2015; Fig. 2)](https://www.nature.com/articles/nature14558) to find blocks of Neanderthal ancestry in an ancient human genome, by [Der Sarkassian et al. (2015; Fig. 4)](https://www.cell.com/current-biology/abstract/S0960-9822(15)01003-9) to investigate the ancestry of Przewalski's horses, by [Runemark et al. (2018; Suppl. Fig. 4)](https://www.nature.com/articles/s41559-017-0437-7) to assess hybridization in sparrows, and by [Barth et al. (2019; Fig. 2)](https://www.biorxiv.org/content/10.1101/635631v1) to identify hybrids in tropical eels.

* If you haven't seen any of the above-named studies, you might want to have a look at the ancestry-painting plots in some of them. You may note that the ancestry painting in [Fu et al. (2015; Fig. 2)](https://www.nature.com/articles/nature14558) is slightly different from the other two studies because no discrimination is made between heterozygous and homozygous Neanderthal alleles. Each sample in Fig. 2 of [Fu et al. (2015)](https://www.nature.com/articles/nature14558) is represented by a single row of cells that are white or colored depending on whether or not the Neanderthal allele is present at a site. In contrast, each sample in the ancestry paintings of [Der Sarkassian et al. (2015; Fig. 4)](https://www.cell.com/current-biology/abstract/S0960-9822(15)01003-9), [Runemark et al. (2018; Suppl. Fig. 4)](https://www.nature.com/articles/s41559-017-0437-7), and [Barth et al. (2019; Fig. 2)](https://www.biorxiv.org/content/10.1101/635631v1) is drawn with two rows of cells. However, as the analyses in both studies were done with unphased data, these two rows do not represent the two haplotypes per sample. Instead, the two cells per site were simply both colored in the same way for homozygous sites or differently for heterozygous sites without regard to haplotype structure.

	Here, we are going to use ancestry painting to investigate ancestry in *Neolamprologus cancellatus* ("neocan"), assuming that it is a hybrid between the parental species *Altolamprologus fasciatus* ("altfas") and *Telmatochromis vittatus* ("telvit"). As in [Der Sarkassian et al. (2015; Fig. 4)](https://www.cell.com/current-biology/abstract/S0960-9822(15)01003-9), [Runemark et al. (2018; Suppl. Fig. 4)](https://www.nature.com/articles/s41559-017-0437-7), and [Barth et al. (2019; Fig. 2)](https://www.biorxiv.org/content/10.1101/635631v1), we are going to draw two rows per sample to indicate whether genotypes are homozygous or heterozygous.

* To generate an ancestry painting, we will need to run two Ruby scripts. The first of these, [`get_fixed_site_gts.rb`](src/get_fixed_site_gts.rb) determines the alleles of the putative hybrid species at sites that are fixed differently in the two putative parental species. The second script, [`plot_fixed_site_gts.rb`](src/plot_fixed_site_gts.rb) then uses the output of the first script to draw an ancestry painting. As the first script requires an uncompressed VCF file as input, first uncompress the VCF file for the SNP dataset with the following command:

		gunzip -c NC_031969.f5.sub1.vcf.gz > NC_031969.f5.sub1.vcf

* Then, run the Ruby script [`get_fixed_site_gts.rb`](src/get_fixed_site_gts.rb) to determine the alleles at sites that are fixed differently in the two parents. This script expects six arguments; these are
	* the name of the uncompressed VCF input file, `NC_031969.f5.sub1.vcf`,
	* the name of an output file, which will be a tab-delimited table,
	* a string of comma-separated IDs of samples for the first putative parent species,
	* a string of comma-separated IDs of samples for the putative hybrid species,
	* another string of comma-separated IDs of samples for the second putative parent species,
	* a threshold value for the required completeness of parental genotype information so that sites with too much missing data are discarded.

	We'll use `NC_031969.f5.sub1.vcf` as the input and name the output file `pops1.fixed.txt`. Assuming that the parental species are *Altolamprologus fasciatus* ("altfas") and *Telmatochromis vittatus* ("telvit") and the hybrid species is *Neolamprologus cancellatus* ("neocan"), we'll specify the sample IDs for these species with the strings "AUE7,AXD5", "JBD5,JBD6", and "LJC9,LJD1". Finally, we'll filter for sites without missing data by specifying "1.0" as the sixth argument. Thus, run the script [`get_fixed_site_gts.rb`](src/get_fixed_site_gts.rb) with the following command:

		ruby get_fixed_site_gts.rb NC_031969.f5.sub1.vcf pops1.fixed.txt AUE7,AXD5 LJC9,LJD1 JBD5,JBD6 1.0

* The second script, [`plot_fixed_site_gts.rb`](src/plot_fixed_site_gts.rb), expects four arguments, which are
	* the name of the file written by script [`get_fixed_site_gts.rb`](src/get_fixed_site_gts.rb),
	* the name of an output file which will be a plot in SVG format,
	* a threshold value for the required completeness, which now applies not only to the parental species but also to the putative hybrid species,
	* the minimum chromosomal distance in bp between SNPs included in the plot. This last argument aims to avoid that the ancestry painting is overly dominated by high-divergence regions.

	We'll use the file [`pops1.fixed.txt`](res/pops1.fixed.txt) as input, name the output file `pops1.fixed.svg`, require again that no missing data remains in the output, and we'll thin the remaining distances so that those plotted have a minimum distance of 1,000 bp to each other. Thus, use the following command to draw the ancestry painting:

		ruby plot_fixed_site_gts.rb pops1.fixed.txt pops1.fixed.svg 1.0 1000
		
	The screen output of this script will include some warnings about unexpected genotypes, these can be safely ignored as the script automatically excludes those sites. At the very end, the output should indicate that 6,069 sites with the required completeness were found, these are the sites included in the ancestry painting. Th output also reports, for all analyzed specimens, the heterozygosity at those 6,069 sites. For first-generation hybrids, this heterozygosity is expected to be close to 1.

* Open the file [`pops1.fixed.svg`](res/pops1.fixed.svg) with a program capable of reading files in SVG format, for example with a browser such as Firefox or with Adobe Illustrator. You should see a plot like the one shown below. <p align="center"><img src="img/pops1.fixed.png" alt="Ancestry Painting" width="600"></p>

	In this ancestry painting, the two samples of the two parental species are each drawn in solid colors because all included sites were required to be completely fixed and completely without missing data. The samples of *Neolamprologus cancellatus*, "LJC9" and "LJD1" are drawn in between, with two rows per sample that are colored according to genotypes observed at the 6,069 sites. Keep in mind that even though the pattern may appear to show phased haplotypes, this is not the case; instead the bottom row for a sample is arbitrarily colored in red and the top row is colored in blue when the genotype is heterozygous.
	
	**Question 5:** Do you notice any surprising difference to the ancestry plots of [Der Sarkassian et al. (2015; Fig. 4)](https://www.cell.com/current-biology/abstract/S0960-9822(15)01003-9) and [Runemark et al. (2018; Suppl. Fig. 4)](https://www.nature.com/articles/s41559-017-0437-7)? [(see answer)](#q5)

	**Question 6:** How can this difference be explained? [(see answer)](#q6)


<br><hr>

<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

## Answers

<a name="q1"></a>

* **Question 1:** In fact, integer counts of ABBA and BABA sites would only be expected if each species would be represented only by a single haploid sequence. With diploid sequences and multiple samples per species, allele frequences are taken into account to weigh the counts of ABBA and BABA sites as described by equation 1 of Malinsky [(2019](https://www.biorxiv.org/content/10.1101/634477v1)).


<a name="q2"></a>

* **Question 2:** Because each trio is listed on a single row, the number of trios in file [`samples__combine.txt`](res/samples__combine.txt) is identical to the number of lines in this file. This number can easily be counted using, e.g. the following command:

		cat samples__combine.txt | wc -l
		
	You should see that the file includes 286 lines and therefore results for 286 trios. Given that the dataset includes (except the outgroup species) 13 species and 3 of these are required to form a trio, the number of possible trios is 
	
	[13 choose 3](https://en.wikipedia.org/wiki/Binomial_coefficient) = 13! / (3! &times; 10!) = 6227020800 / (6 &times; 3628800) = 286.
	
	(where ! denotes the [factorial function](https://en.wikipedia.org/wiki/Factorial))
	
	Thus, all possible trios seem to be included in file [`samples__combine.txt`](res/samples__combine.txt).
	

<a name="q3"></a>

* **Question 3:** *D*-statistics that could be obtained with alternative rearrangements of the trio "altfas", "neocan", and "neobri" are the following:
	
	*D* = (13479.6 - 1378.2) / (13479.6 + 1378.2) = 0.814481
	*D* = (13479.6 - 4066.95) / (13479.6 + 4066.95) = 0.536438
	
	Without violating the convention that *C*<sub>ABBA</sub> > *C*<sub>BABA</sub>, ensuring that *D* is positive, no other *D* statistics can be obtained for this trio. Thus, the reported *D*-statistic of 0.493787, even though it is extremely high, is in fact the lowest *D*-statistic possible for this trio.
	

<a name="q4"></a>

* **Question 4:** Comparing the two files [`samples__BBAA.txt`](res/samples__BBAA.txt) and [`samples__tree.txt`](res/samples__tree.txt), for example using the command `diff samples__BBAA.txt samples__tree.txt` shows some differences. For example, the trio "neobri", "neocan", and "telvit" is arranged so that "neocan" and "telvit" are in the positions of P1 and P2 in [`samples__BBAA.txt`](res/samples__BBAA.txt), but "neobri" and "telvit" are in the same positions in [`samples__tree.txt`](res/samples__tree.txt). Such disagreement can arise when sister species do not share the largest number of derived sites with each other, for example as a result of introgression, ancestral population structure, or variation in substitution rates.


<a name="q5"></a>

* **Question 5:** One remarkable difference compared to the ancestry painting of [Der Sarkassian et al. (2015; Fig. 4)](https://www.cell.com/current-biology/abstract/S0960-9822(15)01003-9) and [Runemark et al. (2018; Suppl. Fig. 4)](https://www.nature.com/articles/s41559-017-0437-7) is that almost no homozygous genotypes are observed in the two samples of *Neolamprologus cancellatus*: the bottom rows are drawn almost entirely in red for the two putative hybrid individuals and the top rows are almost entirely in blue. The same pattern, however, can be found in [Barth et al. (2019; Fig. 2)](https://www.biorxiv.org/content/10.1101/635631v1).


<a name="q6"></a>

* **Question 6:** The fact that both *Neolamprologus cancellatus* samples are heterozygous for basically all sites that are differentially fixed in the two parental species can only be explained if both of these samples are in fact first-generation hybrids. If introgression would instead be more ancient and backcrossing (or recombination within the hybrid population) had occurred, we would expect that only certain regions of the chromosome are heterozygous while others should be homozygous for the alleles of one or the other of the two parental species. However, unlike in cases where the genomes have been sequenced of parent-offspring trios, we do not know who the actual parent individuals were. We can guess that the parent individuals were members of the species *Altolamprologus fasciatus* and *Telmatochromis vittatus*, but whether the parental individuals were part of the same population as the sampled individuals or a more distantly related population within these species remains uncertain. 
