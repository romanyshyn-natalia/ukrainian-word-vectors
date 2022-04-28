# An effective method for learning Ukrainian vectors 
Artificial Intelligence course project

### Task description
The goal of the current research is to find a method for extracting the high-quality word representation for Ukrainian, having small amount of data and computational resources.

## Installation
To install from our github repository, you can do:
```bash
git clone https://github.com/romanyshyn-natalia/ukrainian-word-vectors.git
cd ukrainian-word-vectors
```

## Requirements
The following command installs all necessary packages:
```bash
pip install -r requirements.txt
```
## Dataset
We train our word vectors using UberText corpus.  The lang-uk team collected and arranged a large number of texts from Ukrainian periodicals.

size | # tokens | # words 
--- | --- | ---
20.1 Gb | 1745M | 2665029


## Training

`run_the_grid.py` is the script that allows a grid training of fasttext on a given corpus on different servers (nodes). It uses google spreadsheet (through gspread) to perform task distribution and to write down the results. This repo also contains the previous version of the script (`gensim_fasttext`), that was based on gensim. However, on our data, vectors trained on gensim shown weaker performance on our intrinsic tests, so we left it in the repo for the reference.

Script has two commands, `setup` and `train`. First allows you to create folders, download and unpack vectors, checkout and compile Facebook's fasttext binaries and store the config. You'll need to create your own spreadsheet (you can copy it from [here](https://docs.google.com/spreadsheets/d/150DjEZKCuJEcsCJWahWmhPkfHzn9pA-N3UIYYx7XM04/edit?usp=sharing)) to manage tasks distribution. Spreadsheet contains two worksheets, first has the hyperparams you'd like to try in the grid and stats, second is generated from the hyperparams (first two columns) and is being updated by worker nodes (columns 3-5). The last column is updated manually after the upload of the vectors. You'll also need to create a json file for gspread with access credentials and store in the `api_keys` folder. You would probably like to replace the hostname with `--hostname` and adjust number of threads (`--threads`, defaulted to the number of CPU cores -2).

You should also specify the id of your google spreadsheet with `--spreadshet_id` (yes, we are aware of a typo there) and specify your api key file location with `--api_key_location`. Please refer to the original documentation of the gspread for a [perfect step-by-step guide](https://docs.gspread.org/en/latest/oauth2.html#enable-api-access-for-a-project).

```bash
python ./run_the_grid.py -v setup
```

## Parameters
```bash

~/fasttext-vectors-uk$ python run_the_grid.py setup --help
usage: run_the_grid.py setup [-h] [--overwrite_config] [--overwrite_corpus] [--overwrite_fasttext]
                             [--corpus_location CORPUS_LOCATION] [--corpus_url CORPUS_URL]
                             [--fasttext_location FASTTEXT_LOCATION] [--vectors_location VECTORS_LOCATION]
                             [--api_key_location API_KEY_LOCATION] [--spreadshet_id SPREADSHET_ID] [--threads THREADS]
                             [--logfile LOGFILE] [--hostname HOSTNAME]

optional arguments:
  -h, --help            show this help message and exit
  --overwrite_config    Overwrite config if it is already exists
  --overwrite_corpus    Overwrite corpus if it is already exists
  --overwrite_fasttext  Download and rebuild fasttext, if it is already exists
  --corpus_location CORPUS_LOCATION
                        Download corpus to specific folder
  --corpus_url CORPUS_URL
                        Download corpus to specific folder
  --fasttext_location FASTTEXT_LOCATION
                        Download and build fasttext to specific folder
  --vectors_location VECTORS_LOCATION
                        Store vectors to given folder after the training
  --api_key_location API_KEY_LOCATION
                        Location of json file with service account credentials for google drive and spreadsheet
  --spreadshet_id SPREADSHET_ID
                        Google Spreadsheet id (the one from the url) with the spreadsheet of tasks and results
  --threads THREADS     Number of threads to use
  --logfile LOGFILE     JSONLines file to write training details
  --hostname HOSTNAME   Identifier of the worker, defaulted to the hostname
```
You might also modify generated `config.json` manually or re-run the `setup` command with the `--overwrite_config` key. 

Once you are done with the setup on your first node, you might run the training immediately on that machine.
To do so run 
```bash
python ./run_the_grid.py -v train
```

It'll run handful of preflight checks, connect to the spreadsheet, pick the next available task, set it to the `Processing` state and start training.

## Evaluation
For intrinsic evaluation we use test set with word analogies for Ukrainian, prepared by Tetiana Kodliuk. It consists of 23397 questions of 12 topics:
* Adjective-adverb
* Comparative
* Country-capital
* Country-regional
* Currency
* Family
* Opposite
* Past
* Singular-plural
* Superlative
* Verb
* total

`intrinsic_evaluation.py` is the script for evaluating trained models on word analogies. You can find the file
`test_vocabulary.txt` in `word_analogies` folder.
```bash
~/ukrainian-word-vectors$ python intrinsic_evaluation.py --help
usage: intrinsic_evaluation.py [-h] [--questions QUESTIONS] [--first_n FIRST_N] [--verbosity {0,1,2,3}] models_path results

Perform intrinsic evaluation of word vectors using gensim. You can evaluate more than one model

positional arguments:
  models_path           Path to models for validation
  results               File to store results too (csv)

optional arguments:
  -h, --help            show this help message and exit
  --questions QUESTIONS
                        Path to the file with word_analogies questions
  --first_n FIRST_N     Number of variants to look into
  --verbosity {0,1,2,3}
                        Level of verbosity
```

Also, we trained NER model with Spacy on our vectors to see the performance on downstream task. 
See, the `Extrinsic_NER_Spacy.ipynb`

## Results
* Top accuracy on intrinsic evaluation:

 Model configuration | Accuracy 
--- | ---
algo-skipgram.epochs-15.subwords-2.5.wordngram-3.neg_sampling-15 | 0, 6158

* Extrinsic evaluation best model:

 Model configuration | F1 score 
--- | ---
algo-cbow.epoch-15.subword-5.6.wordngram-3.negative_sampling-15 | 0,8304


### Related works
[1] Novotný et al (2021). “One Size Does Not Fit All: Finding the Optimal Subword Sizes for FastText Models across Languages”. arXiv:2102.02585 [cs.CL]

[2] Avetisyan et al (2019). “Word Embeddings for the Armenian Language: Intrinsic and Extrinsic Evaluation”. arXiv:1906.03134 [cs.CL] 

[3] Grave et al (2018). “Learning Word Vectors for 157 Languages”. In  Proceedings of the Eleventh International Conference on Language Resources and Evaluation, pp. 3483-3487. 


## Contributors
* [Nataliia Romanyshyn](https://github.com/romanyshyn-natalia?tab=repositories)
* [Dmytro Chaplinsky](https://github.com/dchaplinsky)
