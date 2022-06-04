# Megatron-MSA
Protein Multiple Sequence Alignments (MSA) Pretraining with Megatron-LM.

## Setup
### Docker Environment
We provide a docker image for environment setup.
```bash
docker pull yijiaxiao/protein:pretrain_ssh
```
### Conda Environment
We also provide the conda environment configuration file ([env.yaml](./env.yaml)) for environment setup.


## Pretrain Process
### Tokenizers
Besides the 20 standard amino acids tokens, there are 3 special tokens: `[MASK]` (masked language model), `[-]` (gap token), and `[|]` (spilt token, used in data flow).
The vocabulary is provided in text format [msa_vocab.txt](./msa_tools/msa_vocab.txt).

### Data preprocessing
Each MSA sample consists multiple protein sequences (or one alignment, for orphan protein multiple sequence alignment).

MSA sample:
```
protein seq0
protein seq1
protein seq2
protein seq3
...
protein seqn
```

We squeeze the alignments into one sequence to fit into the data format required by Megatron-LM (shown below).
```
{"text": "seq1 | seq2 seq3 ... seqn"}
```

Note that there is one `|` token in the concatenated sequence, which is used to indicate the boundary of protein sequences. The dataloader will locate the `|` token and get the length of the MSA sample.

The name of the `text` field of the json can be changed by using the `--json-key` flag in [`preprocess_data.py`](./tools/preprocess_data.py) The other metadata are optional and are not used in training.

The loose json is then processed into a binary format for training. To convert the json into mmap, cached index file, or the lazy loader format use `preprocess_data.py`. Set the `--dataset-impl` flag to `mmap`, `cached`, or `lazy`, respectively (default is `mmap`).

```bash
python ../tools/preprocess_data.py \
       --input <PATH_TO_JSON> \
       --output-prefix <PREFIX> \
       --vocab msa_vocab.txt \
       --dataset-impl mmap \
       --tokenizer-type BertWordPieceLowerCase \
```

We provide a Python script [preprocess.py](./msa_tools/preprocess.py) that integrates data preprocessing (first to json, then build to binary file).

Scripts and guidance are available in [msa_tools](./msa_tools/).


### Training
We've provided an example script for pretraining MSA transformer model: [train.sh](./train.sh).

**Note**
If you encounter `timeout` problem when training with multiple computing nodes, please try setting `'timeout'` parameter of `torch.distributed.init_process_group()` to a longer interval.

## Models Availability
We provide two pretrained protein models.

- Protein-MSA (1B)
    - Protein-MSA (1B) contains around billion parameters (950 million). The model is pretrained on 1.5 million multiple sequence alignments samples.

- Protein-MSA (0.7B)
    - Protein-MSA (0.7B) contains 700 million parameters. The model is pretrained on 40 million multiple sequence alignments samples.


# Reference

Our work is based on the following papers.
For protein individual sequence pretraining, please visit [ProteinLM](https://github.com/THUDM/ProteinLM).


[__Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism__](https://arxiv.org/abs/1909.08053v4)
```
@article{DBLP:journals/corr/abs-1909-08053,
  author    = {Mohammad Shoeybi and
               Mostofa Patwary and
               Raul Puri and
               Patrick LeGresley and
               Jared Casper and
               Bryan Catanzaro},
  title     = {Megatron-LM: Training Multi-Billion Parameter Language Models Using
               Model Parallelism},
  journal   = {CoRR},
  volume    = {abs/1909.08053},
  year      = {2019},
  url       = {http://arxiv.org/abs/1909.08053},
  archivePrefix = {arXiv},
  eprint    = {1909.08053},
  timestamp = {Tue, 24 Sep 2019 11:33:51 +0200},
  biburl    = {https://dblp.org/rec/journals/corr/abs-1909-08053.bib},
  bibsource = {dblp computer science bibliography, https://dblp.org}
}
```

[__MSA Transformer__](https://www.biorxiv.org/content/10.1101/2021.02.12.430858v1)
```
@article {Rao2021.02.12.430858,
	author = {Rao, Roshan and Liu, Jason and Verkuil, Robert and Meier, Joshua and Canny, John F. and Abbeel, Pieter and Sercu, Tom and Rives, Alexander},
	title = {MSA Transformer},
	elocation-id = {2021.02.12.430858},
	year = {2021},
	doi = {10.1101/2021.02.12.430858},
	publisher = {Cold Spring Harbor Laboratory},
	abstract = {Unsupervised protein language models trained across millions of diverse sequences learn structure and function of proteins. Protein language models studied to date have been trained to perform inference from individual sequences. The longstanding approach in computational biology has been to make inferences from a family of evolutionarily related sequences by fitting a model to each family independently. In this work we combine the two paradigms. We introduce a protein language model which takes as input a set of sequences in the form of a multiple sequence alignment. The model interleaves row and column attention across the input sequences and is trained with a variant of the masked language modeling objective across many protein families. The performance of the model surpasses current state-of-the-art unsupervised structure learning methods by a wide margin, with far greater parameter efficiency than prior state-of-the-art protein language models.Competing Interest StatementThe authors have declared no competing interest.},
	URL = {https://www.biorxiv.org/content/early/2021/02/13/2021.02.12.430858},
	eprint = {https://www.biorxiv.org/content/early/2021/02/13/2021.02.12.430858.full.pdf},
	journal = {bioRxiv}
}

```
