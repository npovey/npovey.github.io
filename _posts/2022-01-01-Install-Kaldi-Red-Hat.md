---
layout: post
title:  "Install Kaldi: Red Hat"
date:   2022-01-01 00:07:21 -0800
categories: jekyll update
comments: true
---

* [Step1: Install git on AWS Instance](#step1-install-git-on-aws-instance)
* [Step 2: Download Kaldi from GitHub](#step-2-download-kaldi-from-github)
* [Step 3: Read the Kaldi installation instructions and follow Option 1](#step-3-read-the-kaldi-installation-instructions-and-follow-option-1)
* [Step 4: go to tools/  and follow INSTALL instructions there.](#step-4-go-to-tools--and-follow-install-instructions-there)
* [Step 5: go to src/ and follow INSTALL instructions there.](#step-5-go-to-src-and-follow-install-instructions-there)

YouTube
[https://youtu.be/YR_kcw6FOd0](https://youtu.be/YR_kcw6FOd0)
How to Install Kaldi Red Hat| Tutorial

Note: I’ll be using an AWS EC2 instance for the installation but you can do this directly on a linux machine.

### Step1: Install git on AWS Instance

(Keep in mind that the yum install is only for red hat, for debian/ubuntu it would be apt-get install.)

[https://cloudaffaire.com/how-to-install-git-in-aws-ec2-instance/](https://cloudaffaire.com/how-to-install-git-in-aws-ec2-instance/)

```bash
sudo yum update -y
sudo yum install git -y
```

### Step 2: Download Kaldi from GitHub

 [https://kaldi-asr.org/doc/install.html](https://kaldi-asr.org/doc/install.html)

```bash
git clone https://github.com/kaldi-asr/kaldi.git kaldi --origin upstream
cd kaldi
git pull
```

### Step 3: Read the Kaldi installation instructions and follow Option 1

[https://github.com/kaldi-asr/kaldi/blob/master/INSTALL](https://github.com/kaldi-asr/kaldi/blob/master/INSTALL)

```bash
[ec2-user@ip-172-31-6-113 kaldi]$ nano INSTALL

This is the official Kaldi INSTALL. Look also at INSTALL.md for the git mirror installation.
[Option 1 in the following does not apply to native Windows install, see windows/INSTALL or following Option 2]

Option 1 (bash + makefile):

  Steps:
    (1)
    go to tools/  and follow INSTALL instructions there.

    (2)
    go to src/ and follow INSTALL instructions there.

Option 2 (cmake):

    Go to cmake/ and follow INSTALL.md instructions there.
    Note, it may not be well tested and some features are missing currently.
```

### Step 4: go to tools/  and follow INSTALL instructions there.

[https://github.com/kaldi-asr/kaldi/blob/master/tools/INSTALL](https://github.com/kaldi-asr/kaldi/blob/master/tools/INSTALL)

```bash
cd tools
[ec2-user@ip-172-31-6-113 tools]$ nano INSTALL
```

Check for dependencies and install missing items

```bash
[ec2-user@ip-172-31-6-113 tools]$ ./extras/check_dependencies.sh
[ec2-user@ip-172-31-6-113 tools]$ sudo yum install gcc-c++ automake autoconf patch sox gcc-gfortran libtool subversion
```

Check for dependencies and install missing items

```bash
[ec2-user@ip-172-31-6-113 tools]$ ./extras/check_dependencies.sh
[ec2-user@ip-172-31-6-113 tools]$ sudo yum install zlib-devel
```

Check for dependencies and install missing items

```bash
[ec2-user@ip-172-31-6-113 tools]$ ./extras/check_dependencies.sh
./extras/check_dependencies.sh: Intel MKL does not seem to be installed.
 ... Run extras/install_mkl.sh to install it. Some distros (e.g., Ubuntu 20.04) provide
 ... a version of MKL via the package manager, but verify that it is up-to-date.
 ... You can also use other matrix algebra libraries. For information, see:
 ...   http://kaldi-asr.org/doc/matrixwrap.html
[ec2-user@ip-172-31-6-113 tools]$
```

Run `make`

```bash
[ec2-user@ip-172-31-6-113 tools]$ ./extras/install_mkl.sh
[ec2-user@ip-172-31-6-113 tools]$ make
```

Install IRSTLM: can be useful for  interpolations when building LMs

```bash
***() Installation of IRSTLM finished successfully
***() Please source the tools/env.sh in your path.sh to enable it
[ec2-user@ip-172-31-6-113 tools]$
```

### Step 5: go to src/ and follow INSTALL instructions there.

[ec2-user@ip-172-31-6-113 src]$ nano INSTALL

```bash
cd src
[ec2-user@ip-172-31-6-113 src]$ nano INSTALL
./configure --shared
make depend -j 8
make -j 8
```

**You’re Done!**

history.txt

```bash
1  sudo yum update -y
2  sudo yum install git -y
3  git clone https://github.com/kaldi-asr/kaldi.git kaldi --origin upstream
4  cd kaldi
5  git pull
6  nano INSTALL
7  cd tools/
8  nano INSTALL
9  ./extras/check_dependencies.sh
10  sudo yum install gcc-c++ automake autoconf patch sox gcc-gfortran libtool subversion
11  ./extras/check_dependencies.sh
12  sudo yum install zlib-devel
13  ./extras/check_dependencies.sh
14  ./extras/install_mkl.sh
15  ./extras/check_dependencies.sh
16  make -j 8
17  ./extras/install_irstlm.sh
18  cd src/
19  ./configure --shared
20  make depend -j 8
21  make -j 8
```

Kaldi directory:

```bash
ubuntu@ip-172-31-6-144:~/kaldi$ tree -L 2
.
├── CMakeLists.txt
├── COPYING
├── INSTALL
├── README.md
├── cmake
│   ├── FindBLAS.cmake
│   ├── FindCUB.cmake
│   ├── FindICU.cmake
│   ├── FindLAPACK.cmake
│   ├── FindNvToolExt.cmake
│   ├── INSTALL.md
│   ├── Utils.cmake
│   ├── VersionHelper.cmake
│   ├── gen_cmake_skeleton.py
│   ├── kaldi-config.cmake.in
│   └── third_party
├── docker
│   ├── README.md
│   ├── debian10-cpu
│   ├── debian9.8-cpu
│   ├── ubuntu16.04-gpu
│   └── ubuntu18.04-cuda10.0
├── egs
│   ├── README.txt
│   ├── aidatatang_200zh
│   ├── aishell
│   ├── aishell2
│   ├── ami
│   ├── an4
│   ├── apiai_decode
│   ├── aspire
│   ├── aurora4
│   ├── babel
│   ├── babel_multilang
│   ├── bentham
│   ├── bn_music_speech
│   ├── callhome_diarization
│   ├── callhome_egyptian
│   ├── casia_hwdb
│   ├── chime1
│   ├── chime2
│   ├── chime3
│   ├── chime4
│   ├── chime5
│   ├── chime6
│   ├── cifar
│   ├── cmu_cslu_kids
│   ├── cnceleb
│   ├── commonvoice
│   ├── csj
│   ├── dihard_2018
│   ├── fame
│   ├── farsdat
│   ├── fisher_callhome_spanish
│   ├── fisher_english
│   ├── fisher_swbd
│   ├── formosa
│   ├── gale_arabic
│   ├── gale_mandarin
│   ├── gigaspeech
│   ├── gop_speechocean762
│   ├── gp
│   ├── heroico
│   ├── hi_mia
│   ├── hkust
│   ├── hub4_english
│   ├── hub4_spanish
│   ├── iam
│   ├── iban
│   ├── icsi
│   ├── ifnenit
│   ├── libri_css
│   ├── librispeech
│   ├── lre
│   ├── lre07
│   ├── madcat_ar
│   ├── madcat_zh
│   ├── malach
│   ├── mandarin_bn_bc
│   ├── material
│   ├── mgb2_arabic
│   ├── mgb5
│   ├── mini_librispeech
│   ├── mobvoi
│   ├── mobvoihotwords
│   ├── multi_cn
│   ├── multi_en
│   ├── nsc
│   ├── opensat20
│   ├── ptb
│   ├── reverb
│   ├── rimes
│   ├── rm
│   ├── sad_rats
│   ├── sitw
│   ├── snips
│   ├── spanish_dimex100
│   ├── sprakbanken
│   ├── sprakbanken_swe
│   ├── sre08
│   ├── sre10
│   ├── sre16
│   ├── svhn
│   ├── swahili
│   ├── swbd
│   ├── tedlium
│   ├── thchs30
│   ├── tidigits
│   ├── timit
│   ├── tunisian_msa
│   ├── uw3
│   ├── voxceleb
│   ├── voxforge
│   ├── vystadial_cz
│   ├── vystadial_en
│   ├── wenetspeech
│   ├── wsj
│   ├── yesno
│   ├── yomdle_fa
│   ├── yomdle_korean
│   ├── yomdle_russian
│   ├── yomdle_tamil
│   ├── yomdle_zh
│   └── zeroth_korean
├── misc
│   ├── README.txt
│   ├── htk_conversion
│   ├── htk_decode_example
│   ├── htk_graph_creation_example
│   ├── logo
│   ├── maintenance
│   └── papers
├── scripts
│   ├── rnnlm
│   └── wakeword
├── src
│   ├── Doxyfile
│   ├── INSTALL
│   ├── Makefile
│   ├── NOTES
│   ├── TODO
│   ├── base
│   ├── bin
│   ├── chain
│   ├── chainbin
│   ├── configure
│   ├── cudadecoder
│   ├── cudadecoderbin
│   ├── cudafeat
│   ├── cudafeatbin
│   ├── cudamatrix
│   ├── decoder
│   ├── doc
│   ├── feat
│   ├── featbin
│   ├── fgmmbin
│   ├── fstbin
│   ├── fstext
│   ├── gmm
│   ├── gmmbin
│   ├── gst-plugin
│   ├── hmm
│   ├── itf
│   ├── ivector
│   ├── ivectorbin
│   ├── kaldi.mk
│   ├── kws
│   ├── kwsbin
│   ├── lat
│   ├── latbin
│   ├── lib
│   ├── lm
│   ├── lmbin
│   ├── makefiles
│   ├── matrix
│   ├── nnet
│   ├── nnet2
│   ├── nnet2bin
│   ├── nnet3
│   ├── nnet3bin
│   ├── nnetbin
│   ├── online
│   ├── online2
│   ├── online2bin
│   ├── onlinebin
│   ├── probe
│   ├── rnnlm
│   ├── rnnlmbin
│   ├── sgmm2
│   ├── sgmm2bin
│   ├── tfrnnlm
│   ├── tfrnnlmbin
│   ├── transform
│   ├── tree
│   └── util
├── tools
│   ├── ATLAS_headers
│   ├── CLAPACK
│   ├── INSTALL
│   ├── Makefile
│   ├── config
│   ├── cub -> cub-1.8.0
│   ├── cub-1.8.0
│   ├── cub-1.8.0.tar.gz
│   ├── env.sh
│   ├── extras
│   ├── install_pfile_utils.sh -> extras/install_pfile_utils.sh
│   ├── install_portaudio.sh -> extras/install_portaudio.sh
│   ├── install_speex.sh -> extras/install_speex.sh
│   ├── install_srilm.sh -> extras/install_srilm.sh
│   ├── irstlm
│   ├── openfst -> openfst-1.7.2
│   ├── openfst-1.7.2
│   ├── openfst-1.7.2.tar.gz
│   ├── sctk -> sctk-20159b5
│   ├── sctk-20159b5
│   ├── sctk-20159b5.tar.gz
│   ├── sph2pipe -> sph2pipe_v2.5
│   ├── sph2pipe-2.5.tar.gz
│   └── sph2pipe_v2.5
└── windows
    ├── INSTALL.atlas
    ├── INSTALL.md
    ├── INSTALL.mkl
    ├── NewGuidCmd.exe
    ├── NewGuidCmd.exe.config
    ├── cuda_7.0.props
    ├── generate_solution.pl
    ├── get_version.pl
    ├── kaldiwin_atlas.props
    ├── kaldiwin_mkl.props
    ├── kaldiwin_openblas.props
    ├── kaldiwin_win32.props
    ├── openfstwin_debug.props
    ├── openfstwin_debug_win32.props
    ├── openfstwin_release.props
    ├── openfstwin_release_win32.props
    ├── portaudio.props
    ├── portaudio_debug.props
    ├── portaudio_release.props
    └── variables.props.dev

186 directories, 55 files
ubuntu@ip-172-31-6-144:~/kaldi$
```


{% include disqus.html %}
