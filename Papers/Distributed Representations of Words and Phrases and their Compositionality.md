# Distributed Representations of Words and Phrases and their Compositionality
*A [paper](https://papers.nips.cc/paper/5021-distributed-representations-of-words-and-phrases-and-their-compositionality.pdf) by Tomas Mikolov, Ilya Sutskever, Kai Chen, Greg Corrado, and Jeffrey Dean*

This paper can be divided into 3 parts.
- First, it introduces improved methods for efficiently building a skip-gram model to precisely represent word vectors.
- Next, it introduces a simple method for learning phrases in a text
- Finally, it explains the additive property of word vectors.

*Some ideas below are not explained in the paper, but is needed to undestand concepts introduced*

## 0. Introduction
### NLP(Natural Language Processing)
Natural language is the language we use in everyday life.
NLP is the process of interpreting natural language so that computers can understand the syntatic and semantic meaning of words.

### Statistical Language Model
A **language model** is the probability distribution of word sequences.
It assigns a probability P(w_1, w_2, ..., w_m), which is the probability that m word sequences will appear given m words.
This model is based on the idea that natural language is context-dependent.

P(w_1, w_2, ..., 2_n) = PRODUCT [P(w_n|w_1, ..., w_n-1)]

= P(w_1) x P(w_2|w_1) x P(w_3|w_1, w_2) x ... x P(w_n|w_1, ..., w_n-1)

### BoW(Bag of Words)
BoW does not consider the order of words in a context, but only the frequency.
Creating a bag of words can be thought as a 2-step process

1. Assign a distinct index to each word
2. For each index, create a vecotr that records the frequency of the word

Remember that the index itself is meaningless, as BoW only considers the frequency of each word.

### Word Embedding
Word embedding is the method of representing words as vecotrs. There are mainly 2 methods in creating vector representations of words, while word embedding is a dense represtation

#### Sparse Representation
A one-hot vector is a sparse vector.
A word can be one-hot encoded by setting the index of the word to 1, while all other values are 0.

The problem of sparse representation appears when the size of vocabulary increases, as it results in waste of memory.
Moreover, sparse representation of words cannot contain the meaning of words.

#### Dense Representation
The dimension of a dense representated vector space does not depend on the size of the vocabulary.
Rather, the user can set a certain dimension value so that each vector element takes a real number.

#### Distributed Representation
Distributed representation is based on the idea that words that appear toegether must have a similar meaning.
For example, the word 'puppy' can be used frequently with 'cute'.
Distributed representation will place these words closely together because it assumes they are semantically related.

### Word2Vec
Word2Vec enables one-hot encoded word vectors to contain the meaning of that word and also the relatiionship with other words.

Words that are one-hot encoded become sparse vectors in that excluding the index of the word itself, all other indices are 0.
Such sparse representations cannot show relationships between words.
Through word embedding, we can create a distributed representation of words in a multi-dimension space that contains the meaning of a word.

There are 2 methods in creating a distributed representation of words; **CBOW** and **Skip-gram**

#### CBoW(Continuous Bag of Words)
CBoW predicts the center word based on its context words. 

#### Skip-gram
Skip-gram predicts its context words based on a center word, opposed to CBoW.
The skip-gram model is an efficient method for learning high-quality distributed vecotr representations that capture a large number of precise syntatic and semantic word relationships.


## 1. Improving the Skip-gram model
The skip-gram model uses the softmax function to update each word in the text.
This makes it computationaly impractical, as the cost is proportional to W(size of the vocabulary).

This chapter introduces 3 improved methods in building a skip-gram model.

### Softmax & Hierarchical Softmax
#### Softmax

#### Hierarchical Softmax
The hierarchical softmax is a computationally efficient approximation of the full softmax.
It uses a binary tree representation of the output layer.

![image.png](https://raw.githubusercontent.com/alstjgg/alstjgg.github.io/master/Distributed%20Representations%20of%20Words%20and%20Phrases%20and%20their%20Compositionality/hierarchical%20softmax%20binary%20tree.PNG)

The tree(which works as the outer layer of the network) has W words as leaves, and each node represents the relative transition probabilities of its child nodes.
A random walk, or decision process, can be defined to assign a probability to each word by the computing the production of every inner node on the path.

That is, *P(w_O|w_I)* can be defined as below.

![image.png](https://raw.githubusercontent.com/alstjgg/alstjgg.github.io/master/Distributed%20Representations%20of%20Words%20and%20Phrases%20and%20their%20Compositionality/hierarchical%20softmax.PNG)

- *n(w, j)* is the j-th node on the path from the root to w.
- *L(w)* is the length of the path.
- *ch(n)* of inner node n is an arbitrary fixed child of n. This means choosing an arbitrary left or right child node.
- *||x||* is 1 if x is true and -1 otherwise.

There are some advantages in using hierarchical softmax.

1. **Computational cost**

The cost of computing is proportional to L(w_O), which is no greater than log W.
Only parameters which the current training example depends on are updated, thus the number of parameters is logarithmic to the size of W.

2. **Word representation**

Unlike full softmax, hierarchical softmax hhas one representation for each word and inner node.

3. **Performance**

Using hierarchical softmax results in both better training time and model accuracy.

This paper uses a binary Huffman tree.

![image.png](http://building-babylon.net/wp-content/uploads/2017/07/Screen-Shot-2017-07-27-at-15.32.50.png)

The main objective of the Huffman tree is to minimize the expected path lengths of words by placing frequently used words on higher levels of nodes.

### Negative Sampling
Negative Sampling is a simplified version of NCE(Noise Contrastive Estimation).

Negative sampling takes the NLP problem as a binary classification problem; it only needs to decide which word choice is right and chich word wrong. Therefore it uses logistic regression as a decision maker.
It can be said that computing the probability that a word is not the answer is most important. This process can be done on the whole vocabulary. However, this would be too costworthy.

Thus negative sampling only considers k constrastive words to take into computation.

Instead of summing over the probabilities of every incorrect word, NCE picks k contrastive words(negative samples).
Although this is not the precise normalization, the approximation works pretty well.
The objective function can be defined as below.

![image.png](https://raw.githubusercontent.com/alstjgg/alstjgg.github.io/master/Distributed%20Representations%20of%20Words%20and%20Phrases%20and%20their%20Compositionality/negative%20sampling%20objective%20function.PNG)

- *P(w)* is the noise distribution that selects which k words to select for computation. The unigram distritbution raise to the 3/4rd power ouperforms other distributions. *raising to the power of 3/4 reduces the difference in frequency*

### Subsampling of Frequent Words
In a given dataset, some words such as 'the' are frequently used and thus frequently updated.
However, their importance is much less than rare words; they provide less information.
Also, the vector representation of frequent words do not change significantly with each update.

Thus, if we discard words that frequently appear in the set, we can efficiently build the model.
In other words, subsampling can be used to counter the imbalance between rare and frequent words by discarding each word in the training set with a probability computed with the formula below.

![image.png](https://raw.githubusercontent.com/alstjgg/alstjgg.github.io/master/Distributed%20Representations%20of%20Words%20and%20Phrases%20and%20their%20Compositionality/subsampling%20discarding%20formula.PNG)

- *P(w)* is the probability that the word w is discarded
- *f(w)* is the number of observations of the word w
- The formula subsamples words whose frequency if greater then the threshold *t*, while preserving the ranking of the frequencies.

### Empirical Results
Evaluation of models are done by the **analogical reasoning task**, which consists of 2 categories.
1. Syntactic Analogies (*quick:quickly::slow:slowly*)
2. Semantic Analogies (*Germany:Berlin::France:Paris*)

Evaluation is done on **Hierarchical Softmax, Noise Contrastive Estimation, Negative Sampling**, and **subsampling of training words**.

![image.png](https://raw.githubusercontent.com/alstjgg/alstjgg.github.io/master/Distributed%20Representations%20of%20Words%20and%20Phrases%20and%20their%20Compositionality/result.PNG)

## 2. Learning Phrases
Some words are frequently used together although their meanings are not related, such as New York Times.
This limitation can be solved by learning vector representations for phrases.
Words that frequently appear together, but infrequently in other contexts, are replaced by unique tokens.

## 3. Additive Compositionality
Because the skip-gram odel exhibits a linear structure, it is possible to perform analogical reasoning using simple vecotr arithmetics.

For example, the result of (Madrid) - (Spain) + (France) is (Paris)

Word vectors are in a linear relationship with the inputs to the softmax nonlinearity.
They represent the distribution of context in which the word appears.

The sum of 2 words, therefore, can be seen as the product of 2 context distributions, as they are logarithmically related.

Words that are assigned high probability by both words vecotrs will have higher probabilty.
