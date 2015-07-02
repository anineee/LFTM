# LF-LDA and LF-DMM latent feature topic models

Implementations of the LF-LDA and LF-DMM latent feature topic models, as described in my TACL paper:

Dat Quoc Nguyen, Richard Billingsley, Lan Du and Mark Johnson. 2015. [Improving Topic Models with Latent Feature Word Representations](https://tacl2013.cs.columbia.edu/ojs/index.php/tacl/article/view/582). <i>Transactions of the Association for Computational Linguistics</i>, vol. 3, pp. 299-313. [[.bib]](http://web.science.mq.edu.au/~dqnguyen/papers/TACL.bib)

<b>NOTE (30/6/2015):</b> When optimizing source codes for sharing, I found a common bug in both my implementations for the DMM and LF-DMM topic models. It changed the document clustering and document classification results for DMM and LF-DMM. However, <i>this does not affect the conclusions of the TACL paper</i>. In fact, in clustering and classification tasks, LF-DMM obtains 5+% absolute improvements on short text datasets against DMM, instead of 3+% improvements as presented in the TACL paper. See more details at [DMM-LFDMM-ResultErratum.pdf](http://web.science.mq.edu.au/~dqnguyen/papers/ResultErratum.pdf)

The source codes for LDA and DMM are available at  [http://jldadmm.sourceforge.net/](http://jldadmm.sourceforge.net/)

Due to restrictions of Twitter, I  cannot offer a public download for the preprocessed Twitter dataset used in the TACL paper (the corresponding Tweet IDs can be found at [Sanders Analytics](http://www.sananalytics.com/lab/twitter-sentiment/)). The other preprocessed experimental datasets can be found [HERE](http://web.science.mq.edu.au/~dqnguyen/papers/TACL-datasets.zip). 


## Usage

This section is to describe the usage of the implementations  in command line or terminal, employing the pre-compiled `LFTM.jar` file. Here, we supposed that Java 1.7+ is set to run in command line or terminal (e.g. adding Java to the environment variable `path` in Windows OS).

Users can find the pre-compiled `LFTM.jar` file and source codes in the `jar` and `src` folders, respectively. The users can recompile the source codes, simply running `ant` (also supposed that `ant` is already installed). In addition, uses can find input examples in the `test` folder.

#### Input corpus format

Similar to the `corpus.txt` file in the `test` folder, each line in the input corpus is a document (i.e. a sequence words/tokens separated by white space characters). The users should preprocess the input corpus before training the topic models, for example: down-casing, removing non-alphabetic characters and stop-words, removing words shorter than 3 characters and words appearing less than a certain times in the input corpus.  

<b>Note:</b> the users have to remove words/terms which are not found the word vector list.

#### Input vector format

Similar to the `wordVectors.txt` file in the `test` folder, each line in the input vector file starts with a word/term which is followed by a vector representation.

Some sets of pre-trained word vectors can be found at:

[Word2Vec: https://code.google.com/p/word2vec/](https://code.google.com/p/word2vec/)

[Glove: http://nlp.stanford.edu/projects/glove/](http://nlp.stanford.edu/projects/glove/)

To speed up the training process, the users may consider to use the pre-trained 50-dimensional vectors in Glove. When adapting to different domains, for example in the biomedical domain, the users can employ Word2Vec or Glove for learning 50-dimensional word vectors on the MEDLINE corpus.

### Training LF-LDA and LF-DMM

`$ java [-Xmx2G] -jar jar/LFTM.jar –model <LFLDA_or_LFDMM> -corpus <Input_corpus_file_path> -vectors <Input_vector_file_path> [-ntopics <int>] [-alpha <double>] [-beta <double>] [-lambda <double>] [-initers <int>] [-niters <int>] [-twords <int>] [-name <String>] [-sstep <int>]`

where parameters in [ ] are optional.

* `-model`: Specify the topic model.

* `-corpus`: Specify the path to the input training corpus file.

* `-vectors`: Specify the path to the file containing word vectors.

* `-ntopics <int>`: Specify the number of topics. The default value is 20.

* `-alpha <double>`: Specify the hyper-parameter alpha. The default value is 0.1.

* `-beta <double>`: Specify the hyper-parameter beta. The default value is 0.01.

* `-lambda <double>`: Specify the mixture weight lambda (0.0 < lambda <= 1.0). The default value is 0.6.

* `-initers <int>`: Specify the number of initial sampling iterations to separate the counts for latent feature and Dirichlet multinomial components. The default value is 2000.

* `-niters <int>`: Specify the number of EM-style sampling iterations for the latent feature topic model. The default value is 200.

* `-twords <int>`: Specify the number of the most probable topical words. The default value is 20.

* `-name <String>`: Specify a name to the topic modeling experiment. The default value is “model”.

* `-sstep <int>`: Specify a step to save the sampling output (`-sstep` value < `-niters` value). The default value is 0 (i.e. only saving the output from the last sample).

<b>Examples:</b>

`$ java -jar jar/LFTM.jar -model LFLDA -corpus test/corpus.txt -vectors test/wordVectors.txt -ntopics 4 -alpha 0.1 -beta 0.01 -lambda 0.6 -initers 2000 -niters 20 -name testLFLDA`

The output files are saved in the same folder as the input training corpus file, in this case in the `test` folder. We have `testLFLDA.theta`, `testLFLDA.phi`, `testLFLDA.topWords`, `testLFLDA.topicAssignments` and `testLFLDA.paras` output files referring to the document-to-topic distributions, topic-to-word distributions, top topical words, topic assignments and model parameters, respectively. Similarly, we perform:

`$ java -jar jar/LFTM.jar -model LFDMM -corpus test/corpus.txt -vectors test/wordVectors.txt -ntopics 4 -alpha 0.1 -beta 0.01 -lambda 0.6 -initers 2000 -niters 20 -name testLFDMM`

### Document clustering evaluation

 After using a topic model to calculate the topic probabilities of a document, we assign every document the topic with the highest probability given the document. To get the  clustering scores of Purity and normalized mutual information, we perform:

`$ java –jar jar/LFTM.jar –model Eval –label <Golden_label_file_path> -dir <Directory_path> -prob <_suffix>`

* `–label`: Specify the path to the ground truth label file. Each line in this label file contains the golden label of the corresponding document in the input training corpus. See the `corpus.LABEL` and `corpus.txt` files in the `test` folder.

* `-dir`: Specify the path to the directory containing document-to-topic distribution files.

* `-prob`: Specify a document-to-topic distribution file or a group of document-to-topic distribution files.

<b>Examples:</b>

`$ java -jar jar/LFTM.jar -model Eval -label test/corpus.LABEL -dir test -prob testLFLDA.theta`

`$ java -jar jar/LFTM.jar -model Eval -label test/corpus.LABEL -dir test -prob testLFDMM.theta`

The above commands will produce the clustering scores for the `testLFLDA.theta` and `testLFDMM.theta` files, separately.

Given the following command:

`$ java -jar jar/LFTM.jar -model Eval -label test/corpus.LABEL -dir test -prob theta`

This command will produce the clustering scores for all the document-to-topic distribution files with their names ending by  `theta`. In this case, the distribution files are `testLFLDA.theta` and `testLFDMM.theta`. It also provides the mean and standard deviation of the clustering scores.

## Acknowledgments

The LF-LDA and LF-DMM implementations used utilities including the LBFGS implementation from [MALLET toolkit](http://mallet.cs.umass.edu/), the random number generator in [Java version of MersenneTwister](http://cs.gmu.edu/~sean/research/), the `Parallel.java` utility from [Mines Java Toolkit](http://dhale.github.io/jtk/api/edu/mines/jtk/util/Parallel.html) and the [Java command line arguments parser](http://args4j.kohsuke.org/sample.html).  I would like to thank the authors of the mentioned utilities for sharing the codes.