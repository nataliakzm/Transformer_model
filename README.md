# Self-attention, Transformers, Pretraining

This is an investigation into Transformer self-attention building blocks, and the effects of pretraining. So, a Transformer model has been trained to attempt to answer simple questions of the form "*Where was person [x] born?*" – without providing any input text from which to draw the answer.

You can see that these models are able to learn some facts about where people were born through pretraining, and access that information during finne-tuning to answer the questions.

# Pretrained Transformer models and knowledge access

Training a Transformer to perform a task involves accessing knowledge about the world – knowledge which isn't provided via the task's training data (at least if you want to generalize outside the training set). It's more or less fails entirely at the task. Here, Transformer is pretrained on Wikipedia text that contains world knowledge, and finetuned on the same knowledge-intensive task that enables the model to access some of the knowledge learned at pretraining time. So, we can see that this enables models to perform considerably above chance on a held out development set.

This code is a fork of **Andrej Karpathy's** [**minGPT**](https://github.com/karpathy/minGPT). It's nicer than most research code since it's relatively simple and transparent. The "**GPT**" in minGPT refers to the Transformer language model of OpenAI, originally described in [this paper](https://s3-us-west-2.amazonaws.com/openai-assets/research-covers/language-unsupervised/language_understanding_paper.pdf).

## What should I do first?

### Review the minGPT demo code:

In the *code/mingpt-demo/*  folder, there's a Jupyter notebook (*play char.ipynb*) that trains and samples from a Transformer language model. Some of the code written below was inspired by what you see in this notebook.

### Read through NameDataset in code/dataset.py, the dataset for reading name-birth place pairs.

The task on pretrained models is attempting to access the birth place of a notable person, as written in their Wikipedia page. Let's think of this as a particularly simple form of question answering:

  *Q: Where was [person] born?
  
  A: [place]*
  
In *dataset.py*, you'll see the the class `NameDataset`, which reads a TSV file of name/place pairs and produces examples of the above form that is feed to a Transformer model later.

To see `NameDataset` on the training set birth places train.tsv run:

 ```bash
 cd src/submission
 python dataset.py namedata
 ```

### Finetuning (without pretraining):

- The Transformer is finetuned on the name/birth place dataset, via examples from the `NameDataset` class. The hyperparameters for the Trainer are specified in the _code/helper.py_ code.

### Making first predictions (without pretraining)

- Train on the names dataset
  `./run.sh vanilla_finetune_without_pretrain`

- Evaluate on the dev set, writing out predictions
  `./run.sh vanilla_eval_dev_without_pretrain`

- Evaluate on the test set, writing out predictions
  `./run.sh vanilla_eval_test_without_pretrain`

### A span corruption function for pretraining:

Class `CharCorruptionDataset` in the file *code/dataset.py* is implemented within the `getitem ()` function. Span corruption is explored in the [T5 paper](https://arxiv.org/pdf/1910.10683.pdf). It randomly selects spans of text in a document and replaces them with unique tokens (noising).

Models take this noised text, and are required to output a pattern of each unique sentinel followed by the tokens that were replaced by that sentinel in the input.

In this example, you can see a simplified option which only masks out a single sequence of characters. To sample a few examples from the `CharCorruptionDataset` on the pretraining dataset wiki.txt:
`cd src/submission`
  `python dataset.py charcorruption`

### Pretraining, finetunng, and making final predictions. Takes around 2 hours for training.

Afterward the model was pretrained on the span corruption task: specifically on wiki.txt (which took approximately two hours).
-  Pretrain the model
  `./run.sh vanilla_pretrain`

- Finetune the model
 ` ./run.sh vanilla_finetune_with_pretrain`
 
- Evaluate on the dev set; write to disk
  `./run.sh vanilla_eval_dev_with_pretrain`
  
-  Evaluate on the test set; write to disk
  `./run.sh vanilla_eval_test_with_pretrain`

## The self-attention module

Below are bash commands that your code should support in order to pretrain the model, finetune it, and make predictions on the dev and test sets. Note that the pretraining process will take approximately 2 hours.

-  Pretrain the model
  `./run.sh synthesizer_pretrain`
-  Finetune the model
  `./run.sh synthesizer_finetune_with_pretrain`
-  Evaluate on the dev set; write to disk
  `./run.sh synthesizer_eval_dev_with_pretrain`
-  Evaluate on the test set; write to disk
  `./run.sh synthesizer_eval_test_with_pretrain`
