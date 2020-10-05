# BESIII分析新手入门---讲稿

[TOC]

## 1. 有用资源的位置

***授人以鱼不如授人以渔***

"再三思考，决定把这一部分放在第一个，以体现这一部分的重要性。对于一个合格的研究生来说，知道如何做科研，显然不是最重要的，我个人觉得**自学能力**或者**调研能力**才是最重要的。我不觉得这短短的3天会议结束之后，大家都成为专家，但我们会努力让每一个人都**学会如何在面对问题的时候用正确的方法论去解决问题**。而这一章正是发挥着这样的作用。" ----------***宋昀轩***

### 1.1 关于物理分析的资源

- [**BESIII官网**](http://bes3.ihep.ac.cn/)这个网站囊括了我们需要用到之时的60%
  - **Collaboration**这个部分会介绍了合作组的一些介绍以及大事件，对于了解合作组很有用。
  - **Research** 是平时科研中用到的最多的地方，涵盖了BESII的硬件、软件、和物理，重点掌握。
    - **Hardware** 探测器的各种子系统的简单介绍，如果希望对探测器详细的了解，可以阅读[**Design and Construction of the BESIII Detector**](https://arxiv.org/pdf/0911.4960.pdf)
    - **Physics** 这里列出了5个工作组的网站，每个工作组的网站里，资源也是相当丰富，此处省略**万字，现场演示。

- [**HyperNews**](http://hnbes3.ihep.ac.cn/HyperNews/get/software.html) 
  
  - 这些有BESIII上所有工作（包括推进中和发表的）的所有细节。科研中遇到困惑，前人大概率也会遇到，看看别人的memo，豁然开朗。
- 订阅感兴趣的HyperNews，跟踪工作。
  
- [**Document Database**](https://docbes3.ihep.ac.cn/cgi-bin/DocDB/DocumentDatabase) BESIII的文档库

  

### 1.2 关于软件的资源

- [**ROOT**](https://root.cern.ch/) 某一个函数不懂的时候，在[**ROOT Reference Documentation**](https://root.cern/doc/v618/)查找.

- [**Offline Software Group**](https://docbes3.ihep.ac.cn/~offlinesoftware/index.php/Main_Page) 这个网站太重要，此处省略20万字，现场演示。

- [**BOSS 函数大全**](http://bes3.to.infn.it/BESIII_Doxygen_Documentation.html) 分析包里有任何关于函数的问题，都可以在这个网站里找到答案。

  

### 1.3 关于计算的资源

计算中心有两个关于计算资源的版本，[老版本](http://afsapply.ihep.ac.cn/quick/)和[新版本](http://afsapply.ihep.ac.cn/cchelp/zh/)，都有很丰富的资源可以使用。磨刀不误砍柴工，建议仔细研读。

[helpdesk](http://helpdesk.ihep.ac.cn/ticket/) 有任何计算中心可以帮助的事情，都可以发helpdesk票单求助。比如besfs被封了。



## 2.基本操作

### 2.1 BOSS简介

BOSS的全程为**BESIII Offline Software System**,

- BOSS-7.0.3
- 框架：[Gaudi-v23-r9](https://gaudi-framework.readthedocs.io/en/latest/)
- 外部库：[ROOT](https://root.cern.ch/), [GEANT4](http://geant4.web.cern.ch/), CLHEP..........
- 开发语言：C++
- 配置管理工具：Configuration Management Tool (**CMT**)

### 2.2 BOSS环境以及账户配置

- 首先申请账号，一通操作之后，会得到账号和密码。登录的时候，要明确自己需要登录的节点

  - **BOSS版本的选择由使用的data种类决定**，具体参考[Production](https://docbes3.ihep.ac.cn/~offlinesoftware/index.php/Production)，这里介绍了每个BOSS版本下重建了什么数据。

  - **登录什么节点由使用的BOSS版本决定**，检查对应版本BOSS的$BesArea下的TestRelease下编译的x86文件夹的名字，如果出现**slc6**则登录`lxslc6`，如果出现**slc5**则登录`lxslc7` 。如

    ~~~bash
    ##登录slc6
    /afs/ihep.ac.cn/bes3/offline/Boss/7.0.3/TestRelease/TestRelease-00-00-86/x86_64-slc6-gcc46-opt
    ~~~

  - 由于Scientific Linux 5早已经不维护了，于是我们登录SL7，在SL7上使用容器，容器里配置SL5的环境。

    ~~~bash
    ##这句话加到.bashrc里
    export PATH=/afs/ihep.ac.cn/soft/common/sysgroup/container/bin/:${PATH}
    
    ##启动容器
    hep_container shell SL5 
    ~~~
    

- 拿到账号密码的同时，还会得到4个属于自己的目录。

~~~bash
#home主目录    500M    有备份，无法提交作业
/afs/ihep.ac.cn/users/s/songyx  

#算法存放目录   5G      有备份，误删之后联系计算中心恢复代码
/workfs/bes/songyx

#数据盘        50G     在此提交作业，做模拟，重建和分析，超出额度之后会锁盘
/besfs/user/songyx

#暂存盘        500G     超出额度之后无法写入
/scratchfs/bes/songyx
~~~



- 配置环境（以703为例）

~~~bash
##在主目录新建一个配置环境的文件夹取名cmthome里面放置各种版本的配置文件
mkdir cmthome

##进入这个目录
cd cmthome 

##拷贝
cp -r /afs/ihep.ac.cn/bes3/offline/Boss/cmthome/cmthome-7.0.3 ./

##修改requirements
set WorkArea "/workfs/bes/songyx/7.0.5"
path_remove CMTPATH  "${WorkArea}"
path_prepend CMTPATH "${WorkArea}"

##cmthome-7.0.3中执行
source setupCMT.sh
cmt config 
source setup.sh

##检查
echo $CMTPATH
~~~



- 编译必须的算法TestRelease

~~~bash
##进入到工作目录
cd $WorkArea

##将$BesArea下的TestRelease文件夹考到工作目录
cp -r $BesArea/TestRelease  ./

##进入TestReleast/*/cmt 编译
cd TestReleast/*/cmt
cmt config
source setup.sh
cmt make
~~~



- 日常登录的时候

~~~bash
##在home下新建一个703.sh 复制粘贴以下内容，包括
source ~/cmthome/cmthome-7.0.3/setupCMT.sh
source ~/cmthome/cmthome-7.0.3/setupCVS.sh
source ~/cmthome/cmthome-7.0.3/setup.sh
source /workfs/bes/songyx/7.0.3/TestRelease/TestRelease-00-00-80/cmt/setup.sh
alias work='cd /workfs/bes/songyx/7.0.3'
source ~/.bashrc
~~~



- BOSS下拷贝一个包到工作目录，并设置环境

~~~bash
cd $WorkArea
##举例 拷贝其他分析包到本地
cp /afs/ihep.ac.cn/bes3/offline/Boss/7.0.3/Analysis/Physics/PsiPrime ./ 
cd cmt/
cmt broadcast cmt config
cmt broadcast make
source setup.sh
echo $XXX

##注意，如果是一个新的分析包，需要讲它添加到TestRelases/*/cmt/requirements里
~~~



### 2.3 BOSS 算法包

- /src               				源文件(*.cxx)
- /<package-name>   头文件(*.h)
- /cmt                             requirement 文件和其他
- /share                          jobOption 文件
- /run                             准备好的一些例子



#### 2.3.1 **CMT**

- 集结了已有的物理分析需要的工具，算法
- 根据CMT约定的格式、内容等，开发个人算法
- 提供了标准的工具编译你的算法包

~~~bash
[7.0.3 02:38:27 songyx@lxslc607 ~]$ echo $CMTPATH
/besfs/users/songyx/workArea_703:/afs/ihep.ac.cn/bes3/offline/Boss/7.0.3:/afs/ihep.ac.cn/bes3/offline/ExternalLib/SLC6/ExternalLib/gaudi/GAUDI_v23r9:/afs/ihep.ac.cn/bes3/offline/ExternalLib/SLC6/ExternalLib/LCGCMT/LCGCMT_65a
~~~

#### 2.3.2 编译算法包

~~~bash
##第一次编译 或 有新算法包调用加入
rm -rf x86_64_slc6-gcc46-opt
cd cmt
cmt clean
make clean

##日常编译算法包
cmt broadcast cmt config 
cmt broadcast cmt make

##看到  "#CMT--->all ok" 就说明编译成功
~~~



#### 2.3.3 TestRelease的介绍

TestRelease 可以理解成每个BOSS软件版本的注册表，需要在TestRelease/cmt/requirements里写入你的算法包，才能在每次sourceBOSS软件之后，在环境变量中找到你的算法包。

~~~bash
#=========  for Analysis   ============
#use Analysis              Analysis-*
#use AnalysisDemoAlg       AnalysisDemoAlg-*             Analysis
#use BKlamsVTX             BKlamsVTX-00-*                Analysis
#use GammaEtaAlg           GammaEtaAlg-00-*              Analysis/Physics
#use KlamsTestAlg          KlamsTestAlg-00-*             Analysis/Physics
use RhopiAlg              RhopiAlg-00-*                 Analysis/Physics
use PipiJpsiAlg           PipiJpsiAlg-*              Analysis/Physics/PsiPrime
use AbsCor                AbsCor-*                   Analysis/PhotonCor
#use Telesis               Telesis-00-*                  Analysis
#use VertexFit             VertexFit-*                   Analysis
~~~

编译通过之后，新开一个shell窗口，检查算法是否成功

~~~bash
$ echo $RHOPIALGROOT
/afs/ihep.ac.cn/bes3/offline/Boss/7.0.3/Analysis/Physics/RhopiAlg/RhopiAlg-00-00-23
~~~



### 2.4 提交BOSS作业

- 将这一行添加到`.bashrc`里

~~~bash
##在HTCondor基础上封装的BOSS专用的作业提交命令
export PATH=/afs/ihep.ac.cn/soft/common/sysgroup/hep_job/bin:${PATH}
~~~

- 提交模拟重建分析作业

~~~bash
cd $WorkArea
cd TestRelease/TestRelease-00-00-*/run
boss.condor jobOptions_sim.txt
boss.condor jobOptions_rec.txt
boss.condor jobOptions_ana_rhopi.txt
~~~

- 提交一般性作业，参考[HTCondor](http://afsapply.ihep.ac.cn/cchelp/zh/local-cluster/jobs/HTCondor/)



## 3 数据结构

### 3.1 TTree & TLorentzVector

介绍一种基于***TTree***的数据结构，以及在TTree里直接存储***TLorentzVector***的方式。

**数据的存储**

~~~c++
TFile *File = new TFile("File","recreate");
TTree *Tree = new TTree("Tree");

Int_t runid;
Double_t decaylength,pid_p[5];
TLorentzVector *pProton = new TLorentzVector();

Tree->Branch("runid", &runid, "runid/I");//Int_t
Tree->Branch("decaylength", &decaylength, "decaylength/D");//Double_t
Tree->Branch("pid_p",pid_p,"pid_p[5]/D");//array
Tree->Branch("pProton",&pProton);//TLorentzVector

for(i=0;i<Nevent;i++)
{
  ...........
	pProton->SetPxPyPzE(px,py,pz,e);
	runid=_runid;
	decaylength = _decaylength;
  pid_p[0]=prob_e;pid_p[1]=prob_mu;pid_p[2]=prob_k;.......
  .............
  Tree->Fill();
}

File->Write();
File->Close();
~~~



### 3.2 TClonesArray

TClonesArray 功能类似于vector ,可以用来存由TObject继承的类，如***TLorentzVector***。



~~~c++
TFile *File = new TFile("File","recreate");
TTree *Tree = new TTree("Tree");

Int_t runid;
Double_t decaylength,pid_p[5];
TLorentzVector *pProton  = new TLorentzVector();
TClonesArray   *Gamma    = new TClonesArray("TLorentzVector");

Tree->Branch("runid", &runid, "runid/I");//Int_t
Tree->Branch("decaylength", &decaylength, "decaylength/D");//Double_t
Tree->Branch("pid_p",pid_p,"pid_p[5]/D");//array
Tree->Branch("pProton",&pProton);//TLorentzVector
Tree->Branch("Gamma", "TClonesArray", &Gamma, 256000, 0);

//Initialize TClonesArray
Gamma->BypassStreamer();

for(i=0;i<Nevent;i++)
{
  ...........
	pProton->SetPxPyPzE(px,py,pz,e);
	runid=_runid;
	decaylength = _decaylength;
  pid_p[0]=prob_e;pid_p[1]=prob_mu;pid_p[2]=prob_k;
  Gamma->Clear();
  for(int i = 0; i < nGamma; i++)
  {
  	TLorentzVector p_Gamma(0, 0, 0, 0);
  	p_Gamma.SetPxPyPzE(ptrk.px(), ptrk.py(), ptrk.pz(), ptrk.e());
    new ((*Gamma)[i]) TLorentzVector(p_Gamma);
  }
  
  Tree->Fill();
}

File->Write();
File->Close();
~~~



