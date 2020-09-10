---
layout: post
title:  "Kaldi interpolate"
date:   2020-09-09 14:34:33 -0700
categories: update
---

KALDI speech processing interpolation example

```bash

1. # ngram path added to the path
. ./path.sh

2. # make directory to put the interpolated arpa file
mkdir -p data/srilm_interp

3. # interpolate from 2 LMs, can try different lambda values for interpolation
ngram -lm data/local/lm_26_98/3gram-mincount/lm_unpruned.gz  -mix-lm data/local/lm_29_29/3gram-mincount/lm_unpruned.gz -lambda 0.5 -write-lm data/srilm_interp/lm.0.5.gz

4. # convert new LM to G.fst and put it in lang_test
utils/format_lm.sh data/lang data/srilm_interp/lm.0.5.gz data/local/dict/lexicon.txt data/lang_test

5. # build graph ~1hour 1pm
utils/mkgraph.sh --self-loop-scale 1.0 data/lang_test exp/tdnn_7b_chain_online exp/tdnn_7b_chain_online/graph_pp

6. # decode ~5hours

7. # get results
grep WER  exp/tdnn_7b_chain_online/decode_test/wer_* | utils/best_wer.sh

8. # see results
less exp/tdnn_7b_chain_online/decode_test/log/decode.*.log
```

Step 6: Decode

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

Results

```bash
LM1 WER = 26.98
LM2 WER = 29.29
LM3 [Interpolated model] WER = 27.12
```
