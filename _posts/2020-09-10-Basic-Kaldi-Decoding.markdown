---

layout: post
title:  "Basic Kaldi Decoding"
date:   2020-09-10 14:34:33 -0700
categories: update
---


Basic decoding steps.


The example is about decoding. Didn't train model. For trained models check kaldi website.

```bash
0. # create text file
# text must be put inside train_all folder [4.4M lines of text]
# i would recommend to find text sample closer to the decoding audio
# important to add foo to the begining of each line. adding foo only works for decoding
# for training the procedure will be different
awk '{print "foo", $0}' < output_31600.txt > text

1. # prepare_dictionary
local/fisher_prepare_dict.sh

2. # preapare lang
utils/prepare_lang.sh data/local/dict "<unk>" data/local/lang data/lang

3. # train 20m
local/fisher_train_lms.sh

4. # This script formats ARPA LM into G.fst. ~15 min
local/fisher_create_test_lang.sh

5. # build graph ~1hour 1pm
utils/mkgraph.sh --self-loop-scale 1.0 data/lang_test exp/tdnn_7b_chain_online exp/tdnn_7b_chain_online/graph_pp

6. # decode see below

7. # get results
grep WER  exp/tdnn_7b_chain_online/decode_test/wer_* | utils/best_wer.sh

8. # see results
less exp/tdnn_7b_chain_online/decode_test/log/decode.*.log

```

Decode

```bash
. ./path.sh
. ./cmd.sh
# allow down sampling
echo "--allow-downsample=true" >> conf/mfcc_hires.conf
# create features
steps/make_mfcc.sh --mfcc-config conf/mfcc_hires.conf data/test
# decode, 5 cpus used but can increase number of cpus
steps/online/nnet3/decode.sh --cmd utils/run.pl --nj 5 --acwt 1.0 --post-decode-acwt 10.0 exp/tdnn_7b_chain_online/graph_pp data/test exp/tdnn_7b_chain_online/decode_test
```

Results 1

```bash
%WER 26.72 [ 55196 / 206597, 6730 ins, 18273 del, 30193 sub ]
```





Change scoring by using local/wer_hyp_filter

Goal: remove  \<unk\> from decoded test before scoring

Create a "local/wer_hyp_filter" / "wer_hyp_filter.sh" would not work

```bash
 sed 's/<unk>//g;'
```

Run score script:

```bash
local/score.sh --cmd utils/run.pl data/test exp/tdnn_7b_chain_online/graph_pp exp/tdnn_7b_chain_online/decode_test
```

Results 2:

```bash
%WER 26.54 [ 54823 / 206597, 6371 ins, 19471 del, 28981 sub ]
```

As expected scoring improved very slightly

Note: my scoring script score.sh points to score_kaldi.sh

```bash
score.sh -> ../steps/score_kaldi.sh
```
