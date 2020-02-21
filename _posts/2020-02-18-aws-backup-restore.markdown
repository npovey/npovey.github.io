---
layout: post
title:  "Backup and Restore a Directory from/to AWS S3 bucket"
date:   2020-2-18 14:34:33 -0700
categories: update
---

  ####Directory Back up and Restore for the cloud

[TOC]

Youtube video for the project:  <https://youtu.be/4Q544dIqClA>

GitHub link: <https://github.com/npovey/aws/tree/master/backup_AWS_S3>

The project consists of two python programs. Backup.py and Restore.py One backs up and the other one restores the given directory. Must use python3 (not python).

 #### Assumptions

The user already set up it's keys and can connect to AWS and Azure from his/her computer. The program has backet hard coded "npovey2". Directories allways backed up to "npovey2"

#### Backup

1. Navigate to the directory with program3

```python
(base) nps-MacBook-Air-2:Desktop np$ ls
2020			Programs		aws.mov			npovey.github.io
AWS			all			backup_AWS_S3		resume
(base) nps-MacBook-Air-2:Desktop np$
```

2. Run the Backup.py file

```python
(base) nps-MacBook-Air-2:Desktop np$ python3 backup_AWS_S3/Backup.py resume

```

3. Output

```python
(base) nps-MacBook-Air-2:Desktop np$ python3 backup_AWS_S3/Backup.py resume
Created: resume/cover_letter_2020_npovey.pdf
Created: resume/cover_letter_2019.pages
Created: resume/resume_2020.pages
Created: resume/.DS_Store
Created: resume/abc.txt
Created: resume/resume_2020_povey.pdf
(base) nps-MacBook-Air-2:Desktop np$
```

4. Output

```python
(base) nps-MacBook-Air-2:Desktop np$ python3 backup_AWS_S3/Backup.py resume
Already Backed Up: resume/cover_letter_2020_npovey.pdf
Already Backed Up: resume/cover_letter_2019.pages
Already Backed Up: resume/resume_2020.pages
Already Backed Up: resume/.DS_Store
Already Backed Up: resume/abc.txt
Already Backed Up: resume/resume_2020_povey.pdf
(base) nps-MacBook-Air-2:Desktop np$
```



#### Restore

1. Navigate to the directory with program3 folder

```python
(base) nps-MacBook-Air-2:Desktop np$ ls
2020			Programs		aws.mov			npovey.github.io
AWS			all			backup_AWS_S3
(base) nps-MacBook-Air-2:Desktop np$
```

2. Run Restore.py

```python
(base) nps-MacBook-Air-2:Desktop np$ python3 backup_AWS_S3/Restore.py resume
```

3. Output

```python
(base) nps-MacBook-Air-2:Desktop np$ python3 backup_AWS_S3/Restore.py resume
Created: resume/.DS_Store
Created: resume/abc.txt
Created: resume/cover_letter_2019.pages
Created: resume/cover_letter_2020_npovey.pdf
Created: resume/resume_2020.pages
Created: resume/resume_2020_povey.pdf
(base) nps-MacBook-Air-2:Desktop np$
```

#### Dependencies

```python
nps-MacBook-Air-2:Desktop np$ python3 --version
Python 3.7.6

nps-MacBook-Air-2:Desktop np$ uname -a
Darwin nps-MacBook-Air-2.local 18.7.0 Darwin Kernel Version 18.7.0: Tue Aug 20 16:57:14 PDT 2019; root:xnu-4903.271.2~2/RELEASE_X86_64 x86_64
# use boto3 to connect to AWS
boto3

```



#### Testing on local computer MacOS

![before](/images_2020_17/before.png)

Ran the Backup.py from outside the program3 folder

```python
nps-MacBook-Air-2:Desktop np$ python3 program3/Backup.py text2
Created: text2/Backup.py
Created: text2/.DS_Store
Created: text2/makefile
Created: text2/README.md
Created: text2/texts/virus.txt
Created: text2/texts/Untitled.txt
Created: text2/hi/xc.txt
Created: text2/hi/bc.txt
Created: text2/hi/a.txt
nps-MacBook-Air-2:Desktop np$
```

Notice that npovey2 folder appeared.

![after](/images_2020_17/after.png)

Ran the same command the second time

```python
nps-MacBook-Air-2:Desktop np$ python3 program3/Backup.py text2
Already Backed Up: text2/Backup.py
Already Backed Up: text2/.DS_Store
Already Backed Up: text2/makefile
Already Backed Up: text2/README.md
Already Backed Up: text2/texts/virus.txt
Already Backed Up: text2/texts/Untitled.txt
Already Backed Up: text2/hi/xc.txt
Already Backed Up: text2/hi/bc.txt
Already Backed Up: text2/hi/a.txt
nps-MacBook-Air-2:Desktop np$

```

Edit Backup.py and try again

![local_folder](/images_2020_17/local_folder.png)

```python
nps-MacBook-Air-2:Desktop np$ python3 program3/Backup.py text2
Rewrote: text2/Backup.py
Rewrote: text2/.DS_Store
Already Backed Up: text2/makefile
Already Backed Up: text2/README.md
Already Backed Up: text2/texts/virus.txt
Already Backed Up: text2/texts/Untitled.txt
Already Backed Up: text2/hi/xc.txt
Already Backed Up: text2/hi/bc.txt
Already Backed Up: text2/hi/a.txt
nps-MacBook-Air-2:Desktop np$
```
