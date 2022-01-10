---
layout: post
title:  "AWS & Kaldi Installation"
date:   2022-01-02 00:07:21 -0800
categories: jekyll update
comments: true
---


* [Step 1: Choose an instance on AWS](#step-1-choose-an-instance-on-aws)
* [Step 2: Download Kaldi from GitHub](#step-2-download-kaldi-from-github)
* [Step 3: Read the Kaldi installation instructions and follow Option 1](#step-3-read-the-kaldi-installation-instructions-and-follow-option-1)
* [Step 3: Go to tools/  and follow INSTALL instructions there.](#step-3-go-to-tools--and-follow-install-instructions-there)
* [Step 4: go to src/ and follow INSTALL instructions there.](#step-4-go-to-src-and-follow-install-instructions-there)

YouTube Tutorial Install Kaldi AWS Ubuntu:
[https://youtu.be/NuB2BTYNosE](https://youtu.be/NuB2BTYNosE)
### Step 1: Choose an instance on AWS

![png](/Export_aws/AWS_Kaldi_Installation/aws.png)


 Select the `g4dn.2xlarge` instance as it has 8 vCPUs and 1 GPU, and its 75 cents per hour, so make sure to pause the instance when you’re not using it. Need 10 days of training. For Step 3, select a specific subnet. This is optional since it defaults to a random region. For Step 4, disable the “Delete on Termination” setting and change the Root’s size to 600 GB. From personal experience, 400GB isn’t going to be enough. Then, you can skip to step 7, review the instance settings, and launch the instance. I’m creating a key pair but you can use an existing one.

### Step 2: Download Kaldi from GitHub

```bash
git clone https://github.com/kaldi-asr/kaldi.git kaldi --origin upstream
cd kaldi
git pull
```

### Step 3: Read the Kaldi installation instructions and follow Option 1

[https://github.com/kaldi-asr/kaldi/blob/master/INSTALL](https://github.com/kaldi-asr/kaldi/blob/master/INSTALL)

```bash
$ nano INSTALL

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

### Step 3: Go to tools/  and follow INSTALL instructions there.

[https://github.com/kaldi-asr/kaldi/blob/master/tools/INSTALL](https://github.com/kaldi-asr/kaldi/blob/master/tools/INSTALL)

Check for dependencies using one of the provided lines and install anything that isn’t downloaded yet.

```bash
CXX=g++-4.8 extras/check_dependencies.sh
```

Installed the following packages but you might need more, keep checking for dependencies using the line above.

```bash
sudo apt-get install g++ sox subversion
sudo apt-get install g++
extras/install_mkl.sh
make -j 8
```

When that’s done running, I saw the warning that IRSTLM is not installed, so I’m running the provided line to have it installed.

```bash
extras/install_irstlm.sh
```

If you recall the original installation steps we found, you’ll remember that when we’re done in the tools folder we should head to the source folder.

### Step 4: go to src/ and follow INSTALL instructions there.

[ubuntu@ip-172-31-6-113 src]$ nano INSTALL

```bash
cd src
[ubuntu@ip-172-31-6-113 src]$ nano INSTALL
./configure --shared
make depend -j 8
make -j 8
```

**You’re Done!**

---

Transcript for tutorial on YouTube:

Today we’re going to set up an AWS instance and install Kaldi on it.  First, open your AWS account and find the EC2 instances page. Click launch instance, scroll down to the “**Deep Learning AMI (Ubuntu 18.04)”** and select it. For Step 2, select the g4dn.2xlarge instance as it has 8 vCPUs and 1 GPU, and its 75 cents per hour, so make sure to pause the instance when you’re not using it. For Step 3, select a specific subnet. This is optional since it defaults to a random region. For Step 4, disable the “Delete on Termination” setting and change the Root’s size to 600 GB. From personal experience, 400GB isn’t going to be enough. Then, you can skip to step 7, review the instance settings, and launch the instance. I’m creating a key pair but you can use an existing one.

Now its time to open your terminal and login into your AWS instance. The address you’ll be ssh-ing into is ubuntu “at” your instance’s public IP address. If you’re using a new key .pem file, make sure to change its privacy settings by using the command “chmod 400” with your file name. Once you successfully ssh into your instance, it is time to install Kaldi.

The initial downloading steps can be found on the official Kaldi webpage. Just copy and paste the first two commands on this page into your terminal and you should be set. Next, open and read the INSTALL file in the Kaldi directory, and follow Option 1. This means you should change to the tools directory and follow the directions in the INSTALL file there. Check for dependencies using one of the provided lines and install anything that isn’t downloaded yet. There’s a bug for one dependency where even if you download it, the check dependencies command will still say its uninstalled. You can just ignore this warning if you have already installed it. Now we're going to run make with the last option in the installation instruction, since it uses multiple CPUs. When that’s done running, I saw the warning that IRSTLM is not installed, so I’m running the provided line to have it installed.

If you recall the original installation steps we found, you’ll remember that when we’re done in the tools folder we should head to the source folder. Here we can open the installation file and follow the three simple commands.

And that was it! You’re all done installing Kaldi. Thanks for watching, and hopefully his was helpful.

history.txt

```bash
    1  LS
    2  ls
    3  tree -l
    4  tree -L
    5  get-apt install tree
    6  apt-get install tree
    7  ls
    8  tree -
    9  tree -L 2
   10  exit
   11  ls
   12  tree -L 2
   13  sudo apt  install tree
   14  tree -L 2
   15  git clone https://github.com/kaldi-asr/kaldi.git kaldi --origin upstream
   16  cd kaldi
   17  ls
   18  nano INSTALL
   19  cd tools/
   20  nano INSTALL
   21  CXX=g++-4.8 extras/check_dependencies.sh
   22  sudo apt-get install g++ sox subversion
   23  CXX=g++-4.8 extras/check_dependencies.sh
   24  sudo apt-get install g++
   25  CXX=g++-4.8 extras/check_dependencies.sh
   26  extras/install_mkl.sh
   27  CXX=g++-4.8 extras/check_dependencies.sh
   28  nano INSTALL
   29  make -j 8
   30  extras/install_irstlm.sh
   31  ls
   32  cd ..
   33  cd src
   34  ls
   35  cd ..
   36  nano INSTALL
   37  cd src/
   38  ls
   39  nano INSTALL
   40  ./configure --shared
   41  nano INSTALL
   42  make depend -j 8
   43  nano INSTALL
   44 make -j 8
   45  cd ..
   46  tree -L 2
   47  tree -L 1
   48  tree -L 2
   49  df -hT
   50  lsblk
   51  ls
   52  cd egs
   53  ls
   54  cd librispeech/
   55  ls
   56  cd s5
   57  ls
   58  sudo apt install flac
   59  ls
   60  nano cmd.sh
   61  ls
   62  cp run.sh run_edited.sh
   63  ls
   64  nano run_edited.sh
   65  nvidia-smi
   66  nvidia-smi -c 3
   67  sudo nvidia-smi -c 3
   68  nano run_edited.sh
   69  nano local/chain/run_tdnn.sh
   70  screen -S libri
   71  top
   72  history > history.txt
```

{% include disqus.html %}
