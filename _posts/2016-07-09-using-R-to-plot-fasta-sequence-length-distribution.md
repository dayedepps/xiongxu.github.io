---
layout: post
title: using R to plot fasta sequence length distribution
description: "Sample post with a background image CSS override."
tags: [R fasta]
image:
  background: triangular.png
---

{% highlight r %}
#!/usr/bin/env Rscript
rm(list=ls())
getProgramName<-function(arguments){
	args <- commandArgs(trailingOnly = FALSE)
	sub("--file=", "", args[grep("--file=", args)])
}
program <- getProgramName()
args <- commandArgs(trailingOnly = TRUE)
if (length(args)<2) {
	stop(sprintf("Please input the information of arguments:
	args[1]		infile
	args[2]		outdir
	args[3]		outfile_prefix
	args[4]		breaks
	......
Example:
	Rscript %s /home/xuxiong/data/melon_RNA_seq/cufflinks/transcripts.fa ./ transcripts.fa 10	
Usage:
	Rscript %s infile outdir outfile breaks",program,program)
	)
}
ptm <- proc.time()

infile <- args[1]
outdir <- args[2]
outfile <- args[3]
Breaks <- as.numeric(args[4])

load_gene_fasta <-function(filename){
	fa_len <- vector(mode="integer")
	c <- file(filename,"r")
	current_len <- 0
	fa_name <- NA
	repeat {
		Line<-readLines(c,n=1)
		Line<-gsub("[\r|\n]+$","",Line,perl=TRUE)	#strip the [\r|\n]+ in the end of line
		if(grepl('^\\>',Line,perl=TRUE) || length(Line)==0) {
			if (!is.na(fa_name)) {
				current_len<-as.character(current_len)
#				cat(current_len,"\n",file=stdout())
				if(is.na(fa_len[current_len])) fa_len[current_len]<-0
				fa_len[current_len] <- fa_len[current_len]+1
			}
			if(length(Line)==0) break
			current_len <- 0
			fa_name<-Line
		}
		else{
			current_len <- current_len+nchar(Line)
		}
	}
	close(c)
	cat("Done load fasta\n",file=stderr())
	return(fa_len)
}

get_section<-function(vec,start,end=NA){
	return(
		ifelse(is.na(end),
			sum(vec[as.integer(names(vec))>=start]),
			sum(vec[as.integer(names(vec))>=start & as.integer(names(vec))<end])
		)	
	)
}

draw_length_distribution<-function(outfile_prefix,Len,breaks){
	png(paste(outfile_prefix,"len.png",sep='_'),pointsize=18,width=900,height=600)
	colors <- c('#4682B4','#87CEEB','#6B8E23','#A0522D','#FF8C00','#6A5ACD','#778899','#DAA520','#B22222','#FF6699')
	max_len<-max(as.integer(names(Len)))
	cat(max_len,"\n")
	each_range<-max(as.integer(names(Len))) %/% breaks
	ranges<-seq(0,max(as.integer(names(Len))),each_range)
	cat(ranges,"\n")
	Len<-sapply(ranges,function(X){
		ifelse(X==max(ranges),get_section(Len,X),get_section(Len,X,X+each_range))
	})
	cat(Len,"\n")
	mp <- barplot(Len,
	#	beside=TRUE,
		width = 1,
		axisnames = F,
		cex.names = 0.8,
		xlab="length(bp)",
		ylab="Counts",
		col=colors[1],
		ylim=c(0,max(Len)*1.2),
		axes=TRUE,
		plot=TRUE,
		xpd=FALSE,
		pch=15,
		args.legend = names(Len),
		main=list("Length distribution"),
	)
	sum_count<-sum(Len)
	cat(sum_count,"\n");
	text(mp,Len,
		labels = sprintf("%d\n(%.2f%%)",Len,Len/sum_count*100),
		adj=c(0.5,-0.5),
		cex=0.75,
		xpd = TRUE
	)
	text(mp, par("usr")[3],
		labels = sapply(ranges,function(X){
			ifelse(X==max(ranges),paste('>=',max(ranges),sep=''),paste('[',X,',',X+each_range,')',sep=''))
		}),
		srt = 35, 
		cex=0.8,
		adj = c(1,1),
		xpd = TRUE
	)
	box()
	pic=dev.off()
}

setwd(outdir)
Length<-load_gene_fasta(infile)
draw_length_distribution(outfile,Length,Breaks)
print(proc.time() - ptm)
{% endhighlight r %}

运行命令：

{% highlight bash %}
Rscript stat_fasta_len.R test.fa ./ test 10
{% endhighlight bash %}

{% capture images %}
    /images/test_len.png
{% endcapture %}
{% include gallery images=images caption="fasta length distribution" cols=1 %}

