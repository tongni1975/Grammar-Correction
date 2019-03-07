# Seq2seq Neural Grammar Correction

The goal of this project is to experiment with elmo and bert embedding along with transformer framework and to see if there's an improvement for grammar correction. 

## Requirements

Three datasets
1. CoNLL-2013 and CoNLL-2014 Shared Task for grammar correction. They have original sentence and corrected sentence with position of error in the sentence and error type. CoNLL-2013 has 5 types of errors while CoNLL-2014 has 28 types of errors. 
2. Lang8
3. AESW Dataset 

### Step 1: Preprocess the data
```
python parser/lang8_parser.py \
       -i lang-8-20111007-L1-v2.dat \
       -o data/src \
       -l2 English
```

### Step 2: Split datasets
```
awk -F $'\t' '{print $1}' data/src/lang8.txt > data/src/lang8.src 
awk -F $'\t' '{print $2}' data/src/lang8.txt > data/src/lang8.trg
cd data/src
python ../../prepare_csv.py \
       -i lang8.src \
       -train lang8.train.src \
       -train_r 0.6 \
       -test lang8.test.src \
       -test_r 0.2 \
       -val lang8.val.src \
       -val_r 0.2
python ../../prepare_csv.py \
       -i lang8.trg \
       -train lang8.train.trg \
       -train_r 0.6 \
       -test lang8.test.trg \
       -test_r 0.2 \
       -val lang8.val.trg \
       -val_r 0.2
cd -
```

### Step 3: Download pretrained word embeddings
```
wget -P data/embs/ https://s3-us-west-2.amazonaws.com/allennlp/models/elmo/2x4096_512_2048cnn_2xhighway/elmo_2x4096_512_2048cnn_2xhighway_options.json
wget -P data/embs/ https://s3-us-west-2.amazonaws.com/allennlp/models/elmo/2x4096_512_2048cnn_2xhighway/elmo_2x4096_512_2048cnn_2xhighway_weights.hdf5
```

### Step 4: Virtualenv

[Transformer] http://www.realworldnlpbook.com/blog/building-seq2seq-machine-translation-models-using-allennlp.html
[ELMo] https://github.com/allenai/allennlp/blob/master/tutorials/how_to/elmo.md 
* transformer\_env

        pip install allennlp
        pip install torch numpy matplotlib spacy torchtext seaborn 
        python -m spacy download en 

[Batched seq2seq] https://github.com/howardyclo/pytorch-seq2seq-example/blob/master/seq2seq.ipynb
* batched\_seq2seq\_env

        pip install -r batched_seq2seq/requirements.txt
        python -m spacy download en_core_web_lg
    
[BERT] https://github.com/huggingface/pytorch-pretrained-BERT
* bert
        
        pip install pytorch-pretrained-bert
        
## Transformer Quickstart

### Step 1: Train the model
```
(transformer_env)
CUDA_VISIBLE_DEVICES=0,1,2,3 python transformer/trainsformer_train.py
```

### Step 2: Evaluate the model
```
(transformer_env)
python evaluation/gleu.py \
       -s data/eval/lang8.eval.src \
       -r data/eval/lang8.eval.trg \
       --hyp data/eval/lang8.eval.pred
``` 

## Batched Seq2seq Quickstart

### Step 1: Train and validate the model
```
cd batched_seq2seq
(batched_seq2seq_env)
python seq2seq_train.py \
       -train_src ./data/lang8_english_src_500k.txt \
       -train_tgt ./data/lang8_english_tgt_500k.txt \
       -val_src ./data/lang8_english_src_val_100k.txt \
       -val_tgt ./data/lang8_english_src_val_100k.txt \
       -emb_type glove
```

### Step 2: Test the model
```
(batched_seq2seq_env)
python seq2seq_pred.py \
       -test_src ./data/lang8_english_src_test_100k.txt
```

### Step 3: Evaluate the model
```
(batched_seq2seq_env)
python ../evaluation/gleu.py \
       -s ./data/lang8_english_src_test_100k.txt \
       -r ./data/lang8_english_tgt_test_100k.txt \
       --hyp ./data/pred.txt

```
## BERT Embedding

### Train word embeddings
```
(bert_venv)
python emb/bert.py --input_file data/test/lang8_small.txt \
        --output_file data/embeddings/lang8_small.bert \
        --bert_mode bert-base-uncased \
        --do_lower_case \
        --batch_size 16
```

