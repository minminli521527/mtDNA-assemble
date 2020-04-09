#### mtDNA-assemble


  #### 1.) canu assemble
$ conda create -n canu canu -y
$ conda activate canu
###### 1.1) 纠错
###### -p 指定输出前缀；-d 指定输出结果目录；genomeSize设置一个预估的基因组大小，便于让Canu估计测序深度，单位是g，m，k；maxThreads 设置最大线程数；minReadLength 表示只使用大于阈值的序列；minOverlapLength 设置Overlap的最小长度，提高minReadLength可以提高运行速度，增加minOverlapLength可以降低假阳性的overlap；另外需要指定输入数据的类型，是原始测序的数据，还是经过处理的：-pacbio-raw 直接测序得到的原始pacbio数据；-pacbio-corrected 经过纠正的pacbio数据；-nanopore-raw 原始的nanopore数据；-nanopore-corrected 结果纠正的nanopore数据；corOutCoverage: 用于控制多少数据用于纠错。比如说拟南芥是120M基因组，100X测序后得到了12G数据，如果只打算使用最长的6G数据进行纠错，那么参数就要设置为50(120m x 50)。设置一个大于测序深度的数值，例如120，表示使用所有数据。
$ canu -correct -p mtDNA -d ./correct maxThreads=4 genomeSize=450k minReadLength=2000 minOverlapLength=500 corOutCoverage=120 corMinCoverage=2 -pacbio-raw ../data/mtDNA.fastq.gz
###### Corrected reads saved in 'mtDNA.correctedReads.fasta.gz'.
###### 1.2) 修整
$ canu -trim -p mtDNA -d ./trim maxThreads=8 genomeSize=450k minReadLength=2000 minOverlapLength=500 -pacbio-corrected ./correct/mtDNA.correctedReads.fasta.gz
###### Trimmed reads saved in 'mtDNA.trimmedReads.fasta.gz'.
###### 1.3) 组装
###### 这里需要调整纠错后的错误率， correctedErrorRate: 两个read交叠部分的差异程度的忍受程度，降低此值可以减少运行时间，如果覆盖率高的话，建议降低这个值，它会影响utgOvlErrorRate。这一步可以尝试多个参数，因为速度比较块。
###### error rate 0.035
$ canu -assemble -p mtDNA -d ./assemble_0.035 maxThreads=20  genomeSize=450k correctedErrorRate=0.035 -pacbio-corrected ../mapping/mtDNA.trimmedReads_minimap2.fastq
###### $ canu -assemble -p mtDNA -d ./assemble_0.035 maxThreads=20  genomeSize=450k errorRate=0.035 -pacbio-corrected ../mapping/mtDNA.trimmedReads_minimap2.fastq
###### 最后输出文件下的mtDNA.contigs.fasta就是结果文件
###### error rate 0.050
$ canu -assemble -p mtDNA -d ./assemble_0.050 maxThreads=20  genomeSize=450k correctedErrorRate=0.050 -pacbio-corrected ../mapping/mtDNA.trimmedReads_minimap2.fastq
###### 最后输出文件下的mtDNA.contigs.fasta就是结果文件



  #### 2.) Falcon assemble
$ conda create -n pb-assembly pb-assembly
$ conda activate pb-assembly
###### 2.1) 创建input_fofn
###### FOFN指的是包含文件名的文件, 每一行里面都要有fasta文件的全路径:
###### 2.2) 创建配置文件
###### 配置文件fc_run.cfg最好是下载模板，进行修改，否则容易出错，配置文件控制着Falcon组装的各个阶段所用的参数，然而一开始我们并不知道哪一个参数才是最优的，通常都需要不断的调整才行。当然由于目前已经有比较多的物种使用了Falcon进行组装，所以可以从他们的配置文件中进行借鉴(https://pb-falcon.readthedocs.io/en/latest/parameters.html)
$ wget https://pb-falcon.readthedocs.io/en/latest/_downloads/fc_run_ecoli_local.cfg
###### 该文件的大部分内容都不需要修改，除了如下几个参数：input_fofn: 这里的input.fofn就是上一步创建的文件。建议把该文件放在cfg文件的同级目录下，这样子就不需要改配置文件该文件的路径了。genome_size,seed_coverage,length_cutoff,length-cutoff_pr 这三个参数控制纠错所用数据量和组装所用数据量. 如果要让程序在运行的时候自动确定用于纠错的数据量，就将length_cutoff设置成"-1"，同时设置基因组估计大小genome_size和用于纠错的深度seed_coverage。jobqueue: 这里用的是单主机而不是集群，所以其实随便取一个名字就行，但是对于SGE则要选择能够提交的队列名。xxx_concurrent_jobs: 同时运行的任务数。显然是越多越快，有些配置文件都写了192，但是对于大部分人而言是没有那么多资源资源的，盲目写多只会导致服务器宕机。
###### 2.3) 运行
###### Falcon的运行非常简单，就是准备好配置文件传给fc_run.py，然后让fc_run.py调度所有需要的软件完成基因组组装即可。
$ fc_run.py fc_run_local.cfg
###### 生成的最终主要结果文件为 2-asm-falcon/p_ctg.fa
###### 0-rawreads/该目录存放对raw subreads进行overlpping分析与校正的结果；0-rawreads/cns-runs/cns_*/*/*.fasta存放校正后的序列信息；1-preads_ovl/该目录存放对校正后reads进行overlapping的结果；2-asm-falcon/该目录是最终结果目录，主要的结果文件是p_ctg.fa和a_ctg.fa




  #### 3.) MECAT2 assemble  
###### http://blog.sciencenet.cn/blog-3406804-1203984.html
###### 直接下载二进制版本
$ wget https://github.com/xiaochuanle/MECAT2/releases/download/20192026/mecat2_20190226_linuax_amd64.tar.gz
$ tar xzvf mecat2_20190226_linuax_amd64.tar.gz
###### 之后将主程序添加至环境变量里，例如我的 MECAT2 存放路径在 /home/my/software/MECAT2/
$  export PATH=/home/my/software/MECAT2/Linux-amd64/bin:$PATH
###### 查看帮助
$  mecat.pl
###### 3.1) 数据格式
###### 目前MECAT2还不支持gz压缩文件，输入fastq或fasta
###### 3.2) 配置文件：https://www.jianshu.com/p/176fc8105000
$ mecat.pl config ecoli_config_file.txt
###### 使用vim修改ecoli_config_file.txt文件为config_file.txt
###### 3.3)  原始数据纠错
$ mecat.pl correct config_file.txt
###### 3.4)  对纠错后的reads进行组装
$ mecat.pl assemble config_file.txt
###### 3.5)  结果解读
###### 纠错后的reads： 1-consensus/cns_reads.fasta.
###### 最长30/20X纠错后用于trimming的reads: 1-consensus/cns_final.fasta.
###### trimmed reads: 2-trim_bases/trimReads.fasta
###### 组装的contigs: 4-fsa/contigs.fasta



  #### 4.) NextDenovo assemble
###### NextDenovo软件安装
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
###### 4.1)  在input.fofn中记录文件的实际位置
###### 4.2) 复制和修改配置文件：https://www.jianshu.com/p/fa26792435eb  http://blog.sciencenet.cn/blog-3406804-1204832.html
$ cp ~/software/NextDenovo/doc/run.cfg ./
$ vi run.cfg
###### 4.3) 运行NextDenovo
$ nextDenovo run.cfg
###### 对于NextDenovo最终的组装序列，可见：03.ctg_graph/01.ctg_graph.sh.work/ctg_graph00/nextgraph.assembly.contig.fasta



  #### 5.) wtdbg2 assemble
$ conda activate canu
###### 5.1) canu纠错
$ canu -correct -p mtDNA -d ./correct maxThreads=4 genomeSize=450k minReadLength=2000 minOverlapLength=500 corOutCoverage=120 corMinCoverage=2 -pacbio-raw ../data/mtDNA.fastq.gz
###### Corrected reads saved in 'mtDNA.correctedReads.fasta.gz'.
###### 5.2) canu修整
$ canu -trim -p mtDNA -d ./trim maxThreads=8 genomeSize=450k minReadLength=2000 minOverlapLength=500 -pacbio-corrected ./correct/mtDNA.correctedReads.fasta.gz
###### Trimmed reads saved in 'mtDNA.trimmedReads.fasta.gz'.
###### 5.3) 进行基因组装
###### 用canu trim校正之后的序列
$ wtdbg2 -t 16 -i mtDNA.trimmedReads.fasta.gz -o prefix -L 5000
###### 5.4) 得到一致性序列
$ wtpoa-cns -t 2 -i prefix.ctg.lay.gz -o prefix.ctg.lay.fa
###### 5.5) 利用三代reads的比对结果对基因组序列进行打磨修正
$ minimap2 -t 2 -x map-pb -a prefix.ctg.lay.fa mtDNA.trimmedReads.fasta.gz | samtools view -Sb - > prefix.ctg.lay.map.bam
$ samtools sort -o prefix.ctg.lay.map.sorted prefix.ctg.lay.map.bam
$ samtools view prefix.ctg.lay.map.srt.bam | wtpoa-cns -t 2 -d prefix.ctg.lay.fa -i - -o prefix.ctg.lay.2nd.fa
###### 5.6) 最终组装结果是prefix.ctg.lay.2nd.fa
$ less prefix.ctg.lay.2nd.fa | grep ">"



  #### 6.) quickmerge assemble
###### 6.1) 一步法：运行一个py脚本
$ merge_wrapper.py prefix.ctg.lay.2nd.fa  nextgraph.assembly.contig.fasta
###### ......
###### 结果是merged_out.fasta
###### 6.2) 分步运行（有时候：第3步-l报错，多试几次）
######  -l|minmatch 设置单个匹配的最小长度(默认20)。-p|prefix 设置输出文件的前缀(默认为out)
$ nucmer -l 20 -p wtdbg2_nextDenovo prefix.ctg.lay.2nd.fa  nextgraph.assembly.contig.fasta
######  -i float 设置最小对齐标识[0,100]，默认为0。-r  允许query overlaps（多对多）。-q 允许reference overlaps（多对多）
$ delta-filter -i 5 -r -q wtdbg2_nextDenovo.delta > wtdbg2_nextDenovo.rq.delta
###### 一般-l选择引用(-r)参考序列组装的N50作为初始值，quast计算。-ml一般大于5000。
$ quickmerge -d wtdbg2_nextDenovo.rq.delta -q ./prefix.ctg.lay.2nd.fa -r ./nextgraph.assembly.contig.fasta -hco 5.0 -c 1.5 -l 34171 -ml 6000 -p wtdbg2_nextDenovo
###### 结果是merged_wtdbg2_nextDenovo.fasta
