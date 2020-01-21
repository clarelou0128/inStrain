Tutorial
===================

The following tutorial goes through an example run of inStrain. You can follow along with your own data, or download the files that are used in this tutorial from the `following link
<https://www.youtube.com/watch?v=dQw4w9WgXcQ>`_.

.. seealso::
  :doc:`quickstart`
    To get started using the program
  :doc:`module_descriptions`
    For descriptions of what the modules can do
  :doc:`example_output`
    To view example output
  :doc:`preparing_input`
    For information on how to prepare data for inStrain
  :doc:`choosing_parameters`
    For detailed information on how to make sure inStrain is running correctly

Preparing .bam and .fasta files
----------------

After downloading the genome file that you would like to profile (.fasta file) and at least one set of paired reads, the first thing to do is to map the reads to the .fasta file in order to generate a .bam file. \

When this mapping is performed it is important that you map to all genomes simutaneously, so the first thing to do is to combine all of the genomes that you'd like to map into a single .fasta file::

 $  cat raw_data/S2_002_005G1_phage_Clostridioides_difficile.fasta raw_data/S2_018_020G1_bacteria_Clostridioides_difficile.fasta > allGenomes_v1.fasta

Next we must map all of the reads to this .fasta file in order to create .bam files. In this tutorial we will use the mapper Bowtie 2, and also the program `shrinksam <https://github.com/bcthomas/shrinksam>`_ to make the resulting .sam files smaller. In this case we will just make a .sam file and let inStrain handle conversion to .bam format ::

 $ mkdir bt2

 $ bowtie2-build allGenomes_v1.fasta bt2/allGenomes_v1.fasta.fa

 $ bowtie2 -p 6 -x bt2/allGenomes_v1.fasta.fa -1 raw_data/N4_005_026G1.r1.fastq.gz -2 raw_data/N4_005_026G1.r2.fastq.gz 2> N4_005_026G1_mapped.log | shrinksam > allGenomes_v1.fasta-vs-N4_005_026G1.sam

Running inStrain profile
--------------

Now that we've made a mapping file we can run inStrain to profile the microdiversity within this genome population.

.. note::
  It is possible to run make of the next steps simultaneously with this one, including profile_genes, genome_wide, and plot, just by including more parameters to this initial profile step, but in this tutorial we're going to run these steps separately

To run inStrain profile you just need to provide the .sam or .bam file, the .fasta file that the reads are mapped to, and the name of the output folder::

 $ inStrain profile allGenomes_v1.fasta-vs-N4_005_026G1.sam allGenomes_v1.fasta -o allGenomes_v1.fasta-vs-N4_005_026G1.IS

Running inStrain genome_wide
--------------

Now that we've profiled all scaffolds, its time to average the results for scaffolds that belong to the same genome. This is done using the genome_wide command ::

 $ inStrain genome_wide -i allGenomes_v1.fasta-vs-N4_005_026G1.IS -s raw_data/*.fasta

Running profile_genes
--------------

In order to get gene level information, we need to provide inStrain with a list of the genes to profile. We can call these genes using the program Prodigal::

 $ prodigal -i raw_data/S2_002_005G1_phage_Clostridioides_difficile.fasta -d S2_002_005G1_phage_Clostridioides_difficile.fasta.genes.fna

 $ prodigal -i raw_data/S2_018_020G1_bacteria_Clostridioides_difficile.fasta -d S2_018_020G1_bacteria_Clostridioides_difficile.fasta.genes.fna

 $ cat S2_002_005G1_phage_Clostridioides_difficile.fasta.genes.fna S2_018_020G1_bacteria_Clostridioides_difficile.fasta.genes.fna > allGenomes_v1.genes.fna

Once we have all the genes to profile in .fna format, we can tell inStrain to profile them

 $ inStrain profile_genes -i allGenomes_v1.fasta-vs-N4_005_026G1.IS/ -g allGenomes_v1.genes.fna

Plotting
------

To make all of the plots that you can given the current inStrain profile object, just run the plot command::

 inStrain plot -i allGenomes_v1.fasta-vs-N4_005_026G1.IS