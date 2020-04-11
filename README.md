# mtDNA-assemble



* ## 1) canu assemble
* ### 1.1) 	canu software installation
###### use conda
	$ conda create -n canu canu -y
	$ conda activate canu
* ### 1.2) assemble
	* #### 1.2.1) correction
###### Parameter analysis：-p, specify the output prefix; -d specify the output result directory; genomeSize sets an estimated genome size, which is convenient for Canu to estimate the sequencing depth, the unit is g, m, k; maxThreads sets the maximum number of threads; minReadLength means only use the threshold value MinOverlapLength Set the minimum length of Overlap, increase minReadLength can increase the running speed, increase minOverlapLength can reduce the false positive overlap; In addition, you need to specify the type of input data, whether it is original sequencing data or processed (-pacbio-raw, Direct pacbio data obtained by direct sequencing; -pacbio-corrected corrected pacbio data; -nanopore-raw original nanopore data; -nanopore-corrected result corrected nanopore data); corOutCoverage: used to control how much data is used for error correction, for example, Arabidopsis thaliana is a 120M genome, and 12G data is obtained after 100X sequencing, if only the longest 6G data is to be used for error correction, then the parameter should be set to 50 (120m x 50), set a value greater than the sequencing depth , for example 120, means to use all data.
	$ canu -correct -p mtDNA -d ./correct maxThreads=4 genomeSize=450k minReadLength=2000 minOverlapLength=500 corOutCoverage=120 corMinCoverage=2 -pacbio-raw ../data/mtDNA.fastq.gz
###### Corrected reads saved in 'mtDNA.correctedReads.fasta.gz'.
	* #### 1.2.2) trim
###### Trimmed reads saved in 'mtDNA.trimmedReads.fasta.gz'.
	$ canu -trim -p mtDNA -d ./trim maxThreads=8 genomeSize=450k minReadLength=2000 minOverlapLength=500 -pacbio-corrected ./correct/mtDNA.correctedReads.fasta.gz
	* #### 1.2.3) assemble
###### The error rate after error correction needs to be adjusted here. correctedErrorRate: the degree of tolerance of the difference between the overlapping parts of the two reads. Lowering this value can reduce the running time. If the coverage is high, it is recommended to reduce this value, it will affect utgOvlErrorRate. Multiple parameters can be tried in this step because of the speed comparison block.
###### error rate 0.035
	$ canu -assemble -p mtDNA -d ./assemble_0.035 maxThreads=20  genomeSize=450k correctedErrorRate=0.035 -pacbio-corrected ../mapping/mtDNA.trimmedReads_minimap2.fastq
###### Different canu versions have different commands. If an error is reported, you may try the following command, and the mtDNA.contigs.fasta under the final output file is the result file.
	$ canu -assemble -p mtDNA -d ./assemble_0.035 maxThreads=20  genomeSize=450k errorRate=0.035 -pacbio-corrected ../mapping/mtDNA.trimmedReads_minimap2.fastq
###### error rate 0.050, and the mtDNA.contigs.fasta under the final output file is the result file.
	$ canu -assemble -p mtDNA -d ./assemble_0.050 maxThreads=20  genomeSize=450k correctedErrorRate=0.050 -pacbio-corrected ../mapping/mtDNA.trimmedReads_minimap2.fastq



* ## 2) Falcon assemble
* ### 2.1) Falcon software installation
###### use conda
	$ conda create -n pb-assembly pb-assembly
	$ conda activate pb-assembly
* ### 2.2) Prepare data
* #### 2.2.1) create input_fofn
###### input_fofn refers to the file containing the sequencing files name, each line must have the full path of the fasta file, and the file has been uploaded to this repository.
* #### 2.2.2) create the configuration file
###### The file fc_run.cfg has been uploaded to this repository.
###### And, it is best to download the template of the configuration file file fc_run.cfg and then modify it, otherwise it is easy to make mistakes. The configuration file controls the parameters used in various stages of Falcon assembly. However, at the beginning, we did not know which parameter is the optimal one. Adjustment. Of course, since there are already many species using Falcon for assembly, they can learn from their configuration files (https://pb-falcon.readthedocs.io/en/latest/parameters.html).
	$ wget https://pb-falcon.readthedocs.io/en/latest/_downloads/fc_run_ecoli_local.cfg
###### Most of the content of this file does not need to be modified, except for the following parameters. "input_fofn": the file input.fofn here is the file created in the previous step, it is recommended to put this file in the same directory of the configuration file, so that there is no need to change the path of the file in the configuration file. "genome_size", "seed_coverage", "length_cutoff", "length-cutoff_pr": these parameters control the amount of data used for error correction and the amount of data used for assembly, if you want the program to automatically determine the amount of data used for error correction when running, set "length_cutoff" to -1 ", set genome estimated size genome_size and depth seed_coverage for error correction at the same time. "jobqueue": here is a single host instead of a cluster, so in fact, just pick a name, but for SGE, you must choose the name of the queue that can be submitted. "xxx_concurrent_jobs": the number of jobs running at the same time is obviously more and faster, some configuration files have written 192, but for most people, there are not so many resources, blind writing more will only lead to server downtime.
* ### 2.3) assemble
###### Running Falcon is very simple, just prepare the configuration file and pass it to fc_run.py, and then let fc_run.py schedule all the required software to complete the genome assembly.
	$ fc_run.py fc_run_local.cfg
###### The final main result file generated is 2-asm-falcon/p_ctg.fa
###### 0-rawreads/: this directory stores the results of overlpping analysis and correction of raw subreads; 0-rawreads/cns-runs/cns_...*.fasta: stores the sequence information after correction; 1-preads_ovl/: this directory stores the overlaying of reads after correction The result; 2-asm-falcon/: this directory is the final result directory, the main result files are p_ctg.fa and a_ctg.fa.


* ## 3) MECAT2 assemble  
###### Please refer to the URL for the process：http://blog.sciencenet.cn/blog-3406804-1203984.html
* ### 3.1) MECAT2 software installation
###### download the binary version directly
	$ wget https://github.com/xiaochuanle/MECAT2/releases/download/20192026/mecat2_20190226_linuax_amd64.tar.gz
	$ tar xzvf mecat2_20190226_linuax_amd64.tar.gz
###### 之后将主程序添加至环境变量里，例如我的 MECAT2 存放路径在 /home/my/software/MECAT2/
	$  export PATH=/home/my/software/MECAT2/Linux-amd64/bin:$PATH
###### 查看帮助
	$  mecat.pl
* ### 3.2) 数据格式
###### 目前MECAT2还不支持gz压缩文件，输入fastq或fasta
* ### 3.3) 配置文件：https://www.jianshu.com/p/176fc8105000
	$ mecat.pl config ecoli_config_file.txt
###### 使用vim修改ecoli_config_file.txt文件为config_file.txt
* ### 3.4)  原始数据纠错
	$ mecat.pl correct config_file.txt
* ### 3.5)  对纠错后的reads进行组装
	$ mecat.pl assemble config_file.txt
* ### 3.6)  结果解读
###### 纠错后的reads： 1-consensus/cns_reads.fasta.
###### 最长30/20X纠错后用于trimming的reads: 1-consensus/cns_final.fasta.
###### trimmed reads: 2-trim_bases/trimReads.fasta
###### 组装的contigs: 4-fsa/contigs.fasta



* ## 4) NextDenovo assemble
* ### 4.1) NextDenovo software installation
###### 下载已编译好的二进制版，可直接使用，无需安装。
	$  wget https://github.com/Nextomics/NextDenovo/releases/download/v2.1-beta.0/NextDenovo.tgz
	$  tar -vxzf NextDenovo.tgz
###### 添加可执行权限
	$  cd NextDenovo
	$  chmod -R 755 *
###### 添加至环境变量
	$  export PATH=~/software/NextDenovo/:$PATH
	$  export PATH=~/software/NextDenovo/bin/:$PATH
###### 这时候没什么问题的话就可直接使用了
###### 软件运行需要python2.7，提示缺少模块时安装即可；但不支持python3
	$  nextDenovo -h
* ### 4.2)  在input.fofn中记录文件的实际位置
* ### 4.3) 复制和修改配置文件：https://www.jianshu.com/p/fa26792435eb  http://blog.sciencenet.cn/blog-3406804-1204832.html
	$ cp ~/software/NextDenovo/doc/run.cfg ./
	$ vi run.cfg
* ### 4.4) 运行NextDenovo
	$ nextDenovo run.cfg
###### 对于NextDenovo最终的组装序列，可见：03.ctg_graph/01.ctg_graph.sh.work/ctg_graph00/nextgraph.assembly.contig.fasta



* ## 5) wtdbg2 assemble
	$ conda activate canu
* ### 5.1) canu纠错
	$ canu -correct -p mtDNA -d ./correct maxThreads=4 genomeSize=450k minReadLength=2000 minOverlapLength=500 corOutCoverage=120 corMinCoverage=2 -pacbio-raw ../data/mtDNA.fastq.gz
###### Corrected reads saved in 'mtDNA.correctedReads.fasta.gz'.
* ### 5.2) canu修整
	$ canu -trim -p mtDNA -d ./trim maxThreads=8 genomeSize=450k minReadLength=2000 minOverlapLength=500 -pacbio-corrected ./correct/mtDNA.correctedReads.fasta.gz
###### Trimmed reads saved in 'mtDNA.trimmedReads.fasta.gz'.
* ### 5.3) 进行基因组装
###### 用canu trim校正之后的序列
	$ wtdbg2 -t 16 -i mtDNA.trimmedReads.fasta.gz -o prefix -L 5000
* ### 5.4) 得到一致性序列
	$ wtpoa-cns -t 2 -i prefix.ctg.lay.gz -o prefix.ctg.lay.fa
* ### 5.5) 利用三代reads的比对结果对基因组序列进行打磨修正
	$ minimap2 -t 2 -x map-pb -a prefix.ctg.lay.fa mtDNA.trimmedReads.fasta.gz | samtools view -Sb - > prefix.ctg.lay.map.bam
	$ samtools sort -o prefix.ctg.lay.map.sorted prefix.ctg.lay.map.bam
	$ samtools view prefix.ctg.lay.map.srt.bam | wtpoa-cns -t 2 -d prefix.ctg.lay.fa -i - -o prefix.ctg.lay.2nd.fa
* ### 5.6) 最终组装结果是prefix.ctg.lay.2nd.fa
	$ less prefix.ctg.lay.2nd.fa | grep ">"


* ## 6) quickmerge assemble
* ### 6.1) 一步法：运行一个py脚本
	$ merge_wrapper.py prefix.ctg.lay.2nd.fa  nextgraph.assembly.contig.fasta
###### ......
###### 结果是merged_out.fasta
* ### 6.2) 分步运行（有时候：第3步-l报错，多试几次）
######  -l|minmatch 设置单个匹配的最小长度(默认20)。-p|prefix 设置输出文件的前缀(默认为out)
	$ nucmer -l 20 -p wtdbg2_nextDenovo prefix.ctg.lay.2nd.fa  nextgraph.assembly.contig.fasta
######  -i float 设置最小对齐标识[0,100]，默认为0。-r  允许query overlaps（多对多）。-q 允许reference overlaps（多对多）
	$ delta-filter -i 5 -r -q wtdbg2_nextDenovo.delta > wtdbg2_nextDenovo.rq.delta
###### 一般-l选择引用(-r)参考序列组装的N50作为初始值，quast计算。-ml一般大于5000。
	$ quickmerge -d wtdbg2_nextDenovo.rq.delta -q ./prefix.ctg.lay.2nd.fa -r ./nextgraph.assembly.contig.fasta -hco 5.0 -c 1.5 -l 34171 -ml 6000 -p wtdbg2_nextDenovo
###### 结果是merged_wtdbg2_nextDenovo.fasta



* ## 7) BLAST比对数据库，初步获取线粒体序列
	$ conda create blast blast -y
	$ conda activate blast
###### 参考：http://blog.sciencenet.cn/blog-3406804-1199850.html
###### 需要用到本地版的NCBI核酸数据库（下文简称NT库），通过后续进行本地BLAST，在组装结果中挑选出比对到线粒体的contigs序列。
###### 将所有得到的contigs/scaffolds序列与NT库中收录的核酸序列做BLAST比对，定位目标序列。
* ### 7.1) blastn 核酸比对，指定 NT 库路径，默认对每条序列输出一条最佳 hits 
	$ blastn -db /database/nt/nt -query ./configs.fasta -out blast -num_threads 4 -num_descriptions 1 -num_alignments 1 -dust no
* ### 7.2) 对blast结果格式作个转化，perl脚本获取链接：https://pan.baidu.com/s/1-HkUh_C9JgYH9q-J2R7ZDA
	$ perl blast_trans.pl spades_blast spades_blast.txt
* ### 7.3) 根据注释描述，提取其中命中到“线粒体”的序列比对结果
	$ grep 'mitocho' blast.txt > blast.select.txt
* ### 7.4) 查看“blast.select.txt”，该文件中只保留了能够比对至数据库中已知线粒体序列的结果，即可大致确定哪些contigs序列是来自线粒体的。



* ## 8) contigs的定位和定向
###### 需要结合手动过程
###### 经过初步拼接后，获得了几个组装结果fasta文件。这些fasta文件中通常存在多条contigs/scaffolds序列（仅凭软件自动组装得到一整条序列，几乎不太可能），下一步就需要确定这些contigs/scaffolds序列在基因组中的相对位置和方向（定位和定向），以继续往完整的环状线粒体基因组序列搭建。
###### 这一步也需借助参考基因组来完成。将组装得到的那些contigs/scaffolds序列与参考基因组对齐，确定位置和方向关系。
###### BLAST比对结果中，给出了这些contigs/scaffolds序列最佳命中的参考线粒体基因组序列名称。可以从中找一条最相似的参考基因组，通过ID在NCBI或EMBL等数据库中下载它们，辅助我们确定这些contigs/scaffolds序列在基因组中的相对位置和方向（定位和定向）。此外，参考基因组还能帮助我们确定自己线粒体基因组的最终长度范围。
###### 能够实现该功能的工具有很多，可视化的工具如geneious，命令行工具如MUMmer，等等。
* ### 8.1) 例如通过MUMmer共线性分析定位 contigs/scaffolds的顺序
###### 参考基因组序列要单一物种的，不要混合物种的
	$ conda create mummer mummer -y
	$ conda activate mummer
	$ mkdir mummer && cd mummer
	$ nucmer --mum -p mitochondria ref_NC_030753.1.fasta mtDNA.contigs.fasta
	$ delta-filter -m mitochondria.delta > mitochondria.filter
	$ show-coords -T -r -l mitochondria.filter > mitochondria.1coords
	$ mummerplot --postscript -p mitochondria mitochondria.delta
	$ ps2pdf mitochondria.ps mitochondria.pdf
###### 对于共线性分析结果，可以直接查看文本结果文件“mitochondria.1coords”中的内容，记录了组装scaffolds序列和参考基因组序列的共线性匹配详细信息。或者更直观的，查看最后生成的共线性结果图，紫色/红色表示正向，蓝色表示反向。
* ### 8.2) 依次提取序列--按照顺序组合得到tig_2.fasta
	$ samtools faidx mtDNA.contigs.fasta
	$ samtools faidx mtDNA.contigs.fasta tig00000011 > tig00000011.fa
	$ ......
###### 提取反向互补序列
	$ seqkit seq -t dna tig00000045.fa -r -p > tig00000045-RC.fa
###### 组合不同的序列
	$ cat tig000000011.fa tig00000045-RC.fa tig00000044.fa tig00000009-RC.fa tig00000027-RC.fa tig00000030-RC.fa tig00000058-RC.fa tig00000032.fa tig00000029.fa tig00000046.fa tig00000018.fa tig00000068.fa tig00000040.fa tig00000050.fa tig00000062-RC.fa tig00000044-RC.fa tig00000024-RC.fa tig00000020.fa tig00000054.fa tig00000061-RC.fa tig00000019.fa tig00000009.fa tig00000044-RC.fa tig00000011.fa tig00000060.fa tig00000038-RC.fa tig00000021-RC.fa tig00000013-RC.fa tig00000043-RC.fa tig00000012.fa tig00000046-RC.fa tig00000035-RC.fa tig00000014-RC.fa > tig_2.fasta
* ### 8.3) 对tig_2.fasta重复步骤1，再次进行共线性分析



* ## 9) PBJelly2用于利用Pacbio数据进行基因组补洞和scaffold连接
###### 如果上一步并未完全将线粒体基因组环起来，中间还存在gap，那么这一部分的内容将会是有用的。
* ### 9.1) PBJelly2软件安装
###### 按照步骤：https://sr-c.github.io/2019/07/02/PBJelly-and-blasr-installation/，安装PBJelly2，同时借鉴步骤：http://cache.baiducontent.com/c?m=9f65cb4a8c8507ed4fece763105392230e54f73266808c4b2487cf1cd4735b36163bbca63023644280906b6677ed1a0dbaab6b66725e60e1948ad8128ae5cc6338895734&p=c363c64ad4d914f306bd9b78084d&newp=8f73c64ad48811a05ee8c6365f4492695d0fc20e38d3d701298ffe0cc4241a1a1a3aecbf2d211301d7c47f6006a54359e9fb30703d0034f1f689df08d2ecce7e64&user=baidu&fm=sc&query=pbjelly2&qid=e4e870de000cfaa0&p1=9，安装PBJelly2软件。
###### 特别注意：需要在python2.7环境运行，否则报错.py，##行
###### 特别注意：conda install networkx==1.11
* ### 9.2) 运行
###### 首先创建配置文件 Protocol.xml
###### 然后依次运行下6步：
	$ Jelly.py setup Protocol.xml
	$ Jelly.py mapping Protocol.xml
	$ Jelly.py support Protocol.xml
	$ Jelly.py extraction Protocol.xml
	$ Jelly.py assembly Protocol.xml -x "--nproc=24"
	$ Jelly.py output Protocol.xml
###### --nproc 参数设置运行线程数。
###### 输出结果文件为 jelly.out.fasta 。
###### 使用 PBJelly2 进行 scaffold 连接
	$ grep -Ho N jelly.out.fasta | uniq -c
