---
layout: post
title: Improvement of WGS data analysis
description: "Sample post with a background image CSS override."
tags: [WGS cloud computing]
image:
  background: triangular.png
---

## Handle gzip format
1. Using pigz to replace gzip 

{% highlight bash %}
pigz -cd -p4 R1.fastq.gz > R1.fastq && pigz -cd -p4 R2.fastq.gz > R2.fastq && cat R1.fastq R2.fastq |pigz -c -p4 >R_cat.fastq.gz
{% endhighlight %}

2. More faster way(zlib example目录下有个gzjoin 可方便合并两个gz格式文件，无需先解压，再合并压缩) 

{% highlight bash %}
gzjoin R1.fastq.gz R2.fastq.gz >R_join.fastq.gz
{% endhighlight %}

## Trace to its source([bcl2fastq](http://support.illumina.com/content/dam/illumina-support/documents/documentation/software_documentation/bcl2fastq/bcl2fastq2-v2-17-software-guide-15051736-g.pdf))

FASTQ files are saved in the compressed GNU zip format (an open source file compression program), indicated by the .gz file extension. By default, the BGZF variant of the GNU zip format is used. The BGZF variant facilitates parallel decompression of the FASTQ files by downstream applications. While BGZF is compliant with the GNU zip standard, if a downstream application cannot handle this variant, it can be turned off with the command line option --no-bgzf-compression.

## Pre-processing

The most fast tools(Both support multithread)  

1. Trimmomatic using a Palindrome mode to identify adapter for pair end reads. Then using simple mode to do this.
2. Flexbar using dynamic programming to identify adapter

## Split query & split subject, schedule multiple jobs into multiple CPUs

## Using shared memory(bwa shm)
In this way, we can save the time of loading index if we would submit bwa mapping jobs repeatedly in a single node.

## Working on a stream

You can output SAM/BAM to the standard houtput (stdout) and pipe it to a SAMtools command via standard input (stdin) without generating a temporary file. 
{% highlight bash %} 
samtools view -u -S - | samtools sort –l 0 -m 4G - $outfile_prefix.sort 
{% endhighlight %}

## Using uncompressed bam for temp bam file

In this way, we can save the time for reading and writing bam files, the utilities do not need to compress or decompress the bgzf file in substance while with more disk cost.  
{% highlight bash %} 
samtools view –u –S
samtools sort –l 0
{% endhighlight %}

## Using samtools cat to concatenate multiple sorted chromosomal bam files as the order in the sam header
{% highlight bash %} 
samtools cat –o chr1_chr2.bam chr1.sort.bam chr2.sort.bam
{% endhighlight %}

## Less writing temp file into disk(less PrintReads)

Make BQSR and call variant in a single command line.  
{% highlight python %} 
Process('java', '-Xmx24g', '-Djava.io.tmpdir=/tmp', '-jar', '/opt/bin/GenomeAnalysisTK-3.2-2.jar', '-R', '/opt/db/ucsc.hg19.fasta', '-et', 'NO_ET', '-K', '/opt/db/rbluo_cs.hku.hk.key', '-T', 'UnifiedGenotyper', '-I', bam0, '-I', bam1, '-I', bam2, '-I', bam3, '-I', bam4, '-I', bam5, '-BQSR', self.inputs.dedup_realn_bam_grp, '--dbsnp', '/opt/db/dbsnp_138.hg19.vcf', '-o', out_file_name, '-stand_call_conf', self.params.stand_call_conf, '-stand_emit_conf', self.params.stand_emit_conf, '-dcov', self.params.dcov, '-nt', '12', '-nct', '1', '-glm', 'BOTH', '-A', 'AlleleBalance', '-A', 'HomopolymerRun', '-l', 'INFO', '--max_alternate_alleles', self.params.max_alternate_alleles, '-baqGOP', '30').run() 
{% endhighlight %}

## Using multiple threads in every command
{% highlight perl %} 
$flexbarExe -n $threadNum -at 2 -u 1 -m 20 -ao 5 -f i1.8 -a $outdir/L1/clean/$prefix\_adaptor.fa --reads $read1[0] --reads2 $read2[0] -z GZ -t clean/$prefix >clean/$prefix.flexbar.log
bwa mem –M -Y  -t $thread –R ‘@RG\tID:$outfile_prefix.sam\tPL:ILLUMINA\tSM:$outfile_prefix.sam\tDS:ref=$ref,pfx=$index_prefix' -p $index_prefix $indir/$fastq1 $indir/$fastq2
{% endhighlight %}

## Using more fast tools

Sambamba to replace the smatools

## Using c/c++ rather than perl/python to write more fast NGS data Analysis utilities
1. https://github.com/attractivechaos/klib (http://attractivechaos.github.io/klib/ )
2. https://github.com/dcjones/fastq-tools 
3. https://github.com/pezmaster31/bamtools 
4. https://github.com/xiongxu/HighPerformanceNGS

## Using GPU/FPGA

## Using distributed computation models to process large data sets
1. MapReduce
2. Spark

<div xmlns:cc="http://creativecommons.org/ns#" xmlns:dct="http://purl.org/dc/terms/" about="http://subtlepatterns.com" class="notice">Background images from <span property="dct:title">Subtle Patterns</span> (<a rel="cc:attributionURL" property="cc:attributionName" href="http://subtlepatterns.com">Subtle Patterns</a>) / <a rel="license" href="http://creativecommons.org/licenses/by-sa/3.0/">CC BY-SA 3.0</a></div>