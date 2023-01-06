# PoetAI
This is a Public Repository for PoetAI, the limerick generator!

## Members:
> Aditya Bindra\
> Carnegie Mellon University\
> Pittsburgh, PA 15213\
> bindra@cmu.edu

> Aditya Malani\
> Carnegie Mellon University\
> Pittsburgh, PA 15213\
> amalani2@andrew.cmu.edu

> Nirmalsing Patil\
> Carnegie Mellon University\
> Pittsburgh, PA 15213\
> nirmalsp@andrew.cmu.edu

> Bandish Parikh\
> Carnegie Mellon University\
> Pittsburgh, PA 15213\
> bparikh@andrew.cmu.edu

## Folder Structure

The folder structure at the time of publishing this repository is as follows:

```
├── LICENSE
├── README.md
├── PoetAI_Report.pdf
├── Data
│   ├── limericks_ballas_oedilf_clean.csv
├── Evaluation
│   └── Limericks_with_Scores1000.csv
├── Models
│   └── PoetAI_GPT2_limerick_generator.ipynb
    └── RhymeAndContext_Scorer.ipynb
    └── RhymeBERT.ipynb
├── Preprocessing
│   └── Limerick_Processing.ipynb
└── Images
    └── images
        ├── context.png
        ├── architecture.png
        ├── final_scores.png
        └── LSTM.png
```

- 'Data' folder contains the dataset that we used for our experiments\
- 'Evaluation' folder contains results of our implementation for measuring our model's performances\
- 'Models' folder is where we have our main GPT2 model as well as the Scoring and BERT notebooks\
- 'Images' contains all the images/assets for this README.md file\
- 'Preprocessing' folder contains code that we have used for pre processing the scraped data

## 1. Introduction
Poetry is an outcome of creation. It is a form of literary work that is often characterized by an interplay of thought-provoking words meant to stimulate the human mind. A limerick is a short humorous form of verse that has five lines and follows the AABBA rhyme scheme.\
\
What if we could use ‘art’-ificial intelligence to create art? While a lot of research has been done in the field of Natural Language Understanding, the area pertaining to generation and qualitative analysis of poetry still remains to be explored. Even after much research, AI has been criticized for lacking creativity and for occasionally producing texts that make no sense. The expected output in this case isn’t a simple numerical value or a class label, but rather an art piece that is meant to be creative, expressive, and appealing to humans.\
\
We derived that, in the context of an Artificial Intelligence algorithm, creativity is just the development of clearly stated mathematical objective functions that a model must be optimized on. The desired output of creativity can not be captured by conventional loss functions. Our main objective is to respond to the question, "What makes this piece of poetry/limerick a good one?" while giving objective functions to grade a specific piece of poetry/limerick. Will a large language model be able to learn the art of poetry? In this project we used 4 models - GPT2 for limerick generation, LSTM based rhyme scorer for evaluating the rhyming scheme of the limerick, pretrained sentence transformer model MiniLM-L6-V2 for evaluating context of the limericks and BERT for fixing the rhyme of a limerick with a good context score.

## 2. Literature Review
### 2.1. GPoet-2 : GPT2 based transformer with forward and reverse fine tuning
A great work that we referenced is GPoeT-2: A GPT-2 Based Poem Generator [6], where the authors propose a two-stage free-form limerick generation. The proposed two-stage generation uses the forward language model to generate a limerick’s first line with high quality and diversity, then uses the reverse language model to generate the rest of the four lines given the first line generated by forwarding LM. They also select and evaluate a few metrics that quantify the idea of “good poetry”
such as syntactical correctness, lexical diversity, and subject/topic consistency.

### 2.2. LimGen : GPT2 based limerick generator
This is another amazing work we referenced, which uses Search Metrics to enforce rhyme. This is one of the more recent works by Jianyou Wang et al.[8] where they use search metrics such as the Adaptive Multi-Templated Constraint algorithm that constrains their search to the space of realistic poems, the Multi-Templated Beam
Search algorithm which searches efficiently through the space, and the probabilistic story-line algorithm that aims to provide coherent story-lines related to a user-provided prompt word.

### 2.3. Deep-Speare : RNN based Sonnet generator with rhyming dictionary to enforce rhyming
Deep-speare[17] is a Sonnet based model used to capture language, rhyming and meter of poetry. These models under-performed in generating human level poetry but served as good reference for rhyme capture with models. Rhyme was enforced by a cosine similarity of the last words generated by the model and a loss function was employed to penalize model when it was not rhyming. A rhyming dictionary was maintained to pick words based on the context.

## 3. Model Description
### 3.1 Main Limerick Generator (GPT2 based)
We used a GPT2 model with the following -
Tokenizer: We used standard GPT2 pretrained tokenizer (medium) with the maximum length capped
at 64.
Optimizer: We used the Adam optimization scheme with a max learning rate of 1e-4.
Scheduler: We used Linear Scheduler with warmup to increase the learning from a small learning
rate to a constant with a warmup step of 10000.
We further propose a filter-and-fix pipeline with the aim of identifying limericks with good context but that don’t adhere to the AABBA rhyme in an attempt to try and ’fix’ their rhyming using a RhymeBERT model.

### 3.2 Rhyme Scorer
The baseline model fell short in enforcing the rhyme scheme as required by the limerick design (AABBA). Below is an example generated by the model:

>I just went for a swim in the bay,\
>with my swimwear so light I should say\
>that i just couldn’t cope,\
>with my cinchons as dope\
>I’d say I should have stayed home today.\

On looking at more examples generated by the baseline model, we observed that in its current form, the model is not doing a perfect job of generating AABBA rhyme scheme always. As a way to quantify this, we use a discriminator (Siamese LSTM) to measure rhyming. Essentially, discriminator is a classifier that takes in a pair of words as inputs and produces a binary output 1/-1 based on whether the two words are rhyming or non-rhyming respectively. For our use case, we have used a Siamese LSTM based architecture as described in the figure.

<p style="text-align: center;">
![Rhyme Scorer](/Images/LSTM.png "Rhyme scorer")
Figure 1: Rhyme Scorer (LSTM)</p>

### 3.3 Context Scorer
For context scoring, we use the all-MiniLM-L6-V2 model which has been fine-tuned on a 1B sentence pairs dataset. It uses a contrastive learning objective: given a sentence from the pair, the model tries to predict which out of a set of randomly sampled other sentences, was actually paired with it in the dataset. It maps sentences paragraphs to a 384-dimensional dense vector space which we further use to find semantic similarity using Cosine Similarity.

<p style="text-align: center;">
![Context Scorer](/Images/context.png "Context scorer")
Figure 2: Context Scorer</p>

### 3.4 RhymeBERT
For our use case, we fine-tune the pre-trained BERT model on the masked language model. We make use of this MLM task of BERT to enhance/fix the rhyming of the generated limericks. We utilized the limerick training dataset and masked the last words of all five sentences in the limericks. BERT’s
bidirectional learning from the text by masking (hiding) a word in a sentence and forcing BERT to bi-directionally use the words on either side of the covered word to predict the masked word helps in learning the rhyming pattern. Below is an example of the training samples fed for fine-tuning:

>Some folks get a kick out of [MASK]\
>With jokes that fool people they’re [MASK]\
>Of course, they are [MASK]\
>Since the people they [MASK]\
>With their jokes are not usually [MASK]\

The objective during the inference phase would be that the limericks that meet the contextual threshold but lack rhyming are fed to this fine-tuned model

Below is the complete architecture we use for this project:

<p style="text-align: center;">
![Architecture](/Images/architecture.png "Architecture")
Figure 3: Complete Architecture</p>

## 4. Results
We trained it for 160,000 iterations on approximately 148,000 data points and the final training loss was 2.338. Following are some examples of limericks generated by the model. The number of such good limericks with perfect rhyme-scheme (AABBA) is approximately 200 out of every 1000. This
is a substantial improvement over the previous work carried out.

>An old witch made her mark on the street.\
>her face was the symbol of heat\
>but the touch made her frown,\
>saying oh dear oh the clown\
>she was cursed and had witchlike feet.\
>
>Some folks get a kick out of fooling,\
>with jokes that fool people theyre doing\
>of course they are sweet,\
>since the people they meet\
>with their jokes are not usually amusing\
>
>Here’s my plan lets combine our forces,\
>with a force thats both mighty and courseless\
>I’m as sharp as a tack,\
>and I won’t leave our slack\
>I’m your captain your marshal your forceless\

Here's the output from the scoring stage:

<p style="text-align: center;">
![scores](/Images/final_scores.png "scores")
Figure 4: Scores</p>