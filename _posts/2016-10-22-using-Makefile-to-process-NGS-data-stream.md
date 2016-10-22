---
layout: post
title: using Makefile to process NGS data stream
description: "Sample post with a background image CSS override."
tags: [NGS Makefile]
image:
  background: triangular.png
---

#我们为何使用Makefile写流程 
 
1. 自动检查目标文件或源文件是否存在，如没有找到已有的源文件则退出make过程，如已找到目标文件则跳过此make步骤
2. 自动检查目标文件或源文件的变更，如果源文件经过修改则重新生成对应的目标文件
3. 易于多进程并行运行多个任务
	
	{% highlight bash %}
	make –j $processNumber
	{% endhighlight bash %}
4. 能表达复杂的数据流程  
	* 互不依赖的平行运行的多个进程数据流
	* 多个数据流汇聚成一个
	* 一个进程分出多个数据流
	* 复杂的网状数据流
5. 可用于一键部署软件，如安装perl/python/R包,或其他命令行软件

	{% highlight makefile %}
	OBJECTS = $(INSTALL_DIR)/lib/perl5/SVG \
	          $(INSTALL_DIR)/lib/perl5/x86_64-linux-gnu-thread-multi/GD.pm \
	          $(INSTALL_DIR)/lib/perl5/Parallel/ForkManager.pm \
	          $(INSTALL_DIR)/lib/perl5/Carp.pm \
	          $(INSTALL_DIR)/lib/perl5/Text/Balanced.pm \
	          $(INSTALL_DIR)/lib/perl5/Pod/Usage.pm \
	          $(INSTALL_DIR)/lib/perl5/x86_64-linux-thread-multi/Params/Validate.pm \
	          $(INSTALL_DIR)/lib/perl5/Font/TTF.pm \
	          $(INSTALL_DIR)/lib/perl5/x86_64-linux-thread-multi/Clone.pm \
	          $(INSTALL_DIR)/lib/perl5/Math/Round.pm \
	          $(INSTALL_DIR)/lib/perl5/Statistics/Basic.pm \
	          $(INSTALL_DIR)/lib/perl5/Readonly.pm \
	          $(INSTALL_DIR)/lib/perl5/x86_64-linux-thread-multi/Storable.pm \
	          $(INSTALL_DIR)/lib/perl5/Config/General.pm \
	          $(INSTALL_DIR)/lib/perl5/Math/BigInt.pm \
	          $(INSTALL_DIR)/lib/perl5/Text/Format.pm \
	          $(INSTALL_DIR)/lib/perl5/x86_64-linux-thread-multi/Time/HiRes.pm \
	          $(INSTALL_DIR)/lib/perl5/Set/IntSpan.pm \
	          $(INSTALL_DIR)/lib/perl5/Regexp/Common.pm \
	          $(INSTALL_DIR)/lib/perl5/Memoize.pm \
	          $(INSTALL_DIR)/lib/perl5/Math/VecStat.pm \
	          $(INSTALL_DIR)/lib/perl5/Math/Bezier.pm
	
	.PHONY: all
	INSTALL_DIR = $(CURDIR)/../PERL5LIB
	all:program
		$(shell export PERL5LIB=$(PERL5LIB):$(INSTALL_DIR)/lib/perl5)
	program: $(OBJECTS)
	
	define InstallPerlPM
	$(INSTALL_DIR)/lib/perl5/$1 : $2
		test -d $(INSTALL_DIR) || mkdir -p $(INSTALL_DIR);\
		tar -zxvf $3.tar.gz;\
		cd $3;\
		perl Makefile.PL INSTALL_BASE=$(INSTALL_DIR);\
		make;\
		make install;\
		cd $(CURDIR) && rm -rf $3
	endef
	
	define InstallPerlPM2
	$(INSTALL_DIR)/lib/perl5/$1 : $2
		test -d $(INSTALL_DIR) || mkdir -p $(INSTALL_DIR);\
		tar -zxvf $3.tgz;\
		cd $3;\
		perl Makefile.PL INSTALL_BASE=$(INSTALL_DIR);\
		make;\
		make install;\
		cd $(CURDIR) && rm -rf $3
	endef
	
	define InstallPerlPM3
	$(INSTALL_DIR)/lib/perl5/$1 : $2
		test -d $(INSTALL_DIR) || mkdir -p $(INSTALL_DIR);\
		tar -zxvf $3.tar.gz;\
		cd $3;\
		perl Build.PL --install_base $(INSTALL_DIR);\
		./Build;\
		./Build test;\
		./Build install --install_base $(INSTALL_DIR);\
		cd $(CURDIR) && rm -rf $3
	endef
	
	$(eval $(call InstallPerlPM,Font/TTF.pm,Font-TTF-1.05.tar.gz,Font-TTF-1.05))
	$(eval $(call InstallPerlPM,x86_64-linux-gnu-thread-multi/GD.pm,GD-2.53.tar.gz,GD-2.53))
	$(eval $(call InstallPerlPM,SVG,SVG-2.63.tar.gz,SVG-2.63))
	$(eval $(call InstallPerlPM,Test/Fatal.pm,Test-Fatal-0.014.tar.gz,Test-Fatal-0.014))
	$(eval $(call InstallPerlPM,Test/Requires.pm,Test-Requires-0.08.tar.gz,Test-Requires-0.08))
	$(eval $(call InstallPerlPM,Module/Runtime.pm,Module-Runtime-0.014.tar.gz,Module-Runtime-0.014))
	$(eval $(call InstallPerlPM,Try/Tiny.pm,Try-Tiny-0.22.tar.gz,Try-Tiny-0.22))
	$(eval $(call InstallPerlPM,Math/Bezier.pm,Math-Bezier-0.01.tar.gz,Math-Bezier-0.01))
	$(eval $(call InstallPerlPM,Math/VecStat.pm,Math-VecStat-0.08.tar.gz,Math-VecStat-0.08))
	$(eval $(call InstallPerlPM,Regexp/Common.pm,Regexp-Common-2013031301.tar.gz,Regexp-Common-2013031301))
	$(eval $(call InstallPerlPM,Set/IntSpan.pm,Set-IntSpan-1.19.tar.gz,Set-IntSpan-1.19))
	$(eval $(call InstallPerlPM,x86_64-linux-thread-multi/Time/HiRes.pm,Time-HiRes-1.9726.tar.gz,Time-HiRes-1.9726))
	$(eval $(call InstallPerlPM,Text/Format.pm,Text-Format-0.59.tar.gz,Text-Format-0.59))
	$(eval $(call InstallPerlPM,Math/BigInt.pm,Math-BigInt-1.9993.tar.gz,Math-BigInt-1.9993))
	$(eval $(call InstallPerlPM,Config/General.pm,Config-General-2.56.tar.gz,Config-General-2.56))
	$(eval $(call InstallPerlPM,x86_64-linux-thread-multi/Storable.pm,Storable-2.51.tar.gz,Storable-2.51))
	$(eval $(call InstallPerlPM,Number/Format.pm,Number-Format-1.73.tar.gz,Number-Format-1.73))
	$(eval $(call InstallPerlPM,Math/Round.pm,Math-Round-0.07.tar.gz,Math-Round-0.07))
	$(eval $(call InstallPerlPM,x86_64-linux-thread-multi/Clone.pm,Clone-0.38.tar.gz,Clone-0.38))
	$(eval $(call InstallPerlPM,Pod/Usage.pm,Pod-Usage-1.67.tar.gz,Pod-Usage-1.67))
	$(eval $(call InstallPerlPM,Text/Balanced.pm,Text-Balanced-2.03.tar.gz,Text-Balanced-2.03))
	$(eval $(call InstallPerlPM,Carp.pm,Carp-1.36.tar.gz,Carp-1.36))
	$(eval $(call InstallPerlPM,Parallel/ForkManager.pm,Parallel-ForkManager-1.13.tar.gz,Parallel-ForkManager-1.13))
	$(eval $(call InstallPerlPM,Module/Implementation.pm,Module-Implementation-0.09.tar.gz $(INSTALL_DIR)/lib/perl5/Module/Runtime.pm $(INSTALL_DIR)/lib/perl5/Try/Tiny.pm,Module-Implementation-0.09))
	$(eval $(call InstallPerlPM,Statistics/Basic.pm,Statistics-Basic-1.6611.tar.gz $(INSTALL_DIR)/lib/perl5/Number/Format.pm,Statistics-Basic-1.6611))
	
	$(eval $(call InstallPerlPM2,Memoize.pm,Memoize-1.03.tgz,Memoize-1.03))
	$(eval $(call InstallPerlPM3,Readonly.pm ,Readonly-2.00.tar.gz,Readonly-2.00))
	$(eval $(call InstallPerlPM3,x86_64-linux-thread-multi/Params/Validate.pm,Params-Validate-1.26.tar.gz $(INSTALL_DIR)/lib/perl5/Module/Implementation.pm $(INSTALL_DIR)/lib/perl5/Test/Fatal.pm $(INSTALL_DIR)/lib/perl5/Test/Requires.pm,Params-Validate-1.26))
	
	clean:
		@rm -rf $(INSTALL_DIR)
	{% endhighlight makefile %}
6. 便于模块化，清晰的流程逻辑
7. 可结合PBS/SGE，用于集群网格计算

	{% highlight makefile %}
	define FastQC
	$1_fastqc.zip \$1_fastqc.html : $1.qc.intermediate
	.INTERMEDIATE:$1.qc.intermediate
	$1.qc.intermediate:$indir/\$2
		cd $(OD) && \
		echo "cd $(OD) && fastqc -o ./ --extract -f fastq -t 8 -q $indir/\$2 "\
		> $(OD)/$1.qc.sh && \
		qsub -cwd -sync y -q all.q -l ncpu 8 $1.qc.sh
	endef
	
	$(eval $(call FastQC,fastq1_Prefix,fastq1.fastq.gz))
	$(eval $(call FastQC,fastq2_Prefix,fastq2.fastq.gz))
	
	{% endhighlight makefile %}
	
	{% highlight bash %}
	make -j 2 -f Fastqc.make
	{% endhighlight bash %}
	