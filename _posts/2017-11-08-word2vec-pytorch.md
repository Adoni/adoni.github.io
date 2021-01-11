---
layout:     post
title:      "Word2vec with Pytorch"
# subtitle:   ""
date:       2017-11-08 22:00:00
author:     "Xiaofei"
header-img: "img/in-post/2017-11-08-pytorch.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Word2vec
    - Skip-gram
    - PyTorch
---

In this post, we implement the famous word embedding model: word2vec. Here are the [paper](https://arxiv.org/pdf/1301.3781.pdf) and the original [code](https://github.com/tmikolov/word2vec) by C.

Word2vec is so classical ans widely used. However, it's implemented with pure C code and the gradient are computed manually. Nowadays, we get deep-learning libraries like Tensorflow and PyTorch, so here we show how to implement it with PyTorch.

Actually, original word2vec implemented two models, skip-gram and CBOW. Each model can be optimized with two algorithms, hierarchical softmax and negative sampling. Here we only implement Skip-gram with negative sampling.

You can find this implementation in [GitHub](https://github.com/Adoni/word2vec_pytorch).

## Word2vec, Skip-gram, Negative Sampling

It's a cliche to talk about word2vec in details so we just show the big picture. If you want to learn more details, please read their [paper](https://arxiv.org/pdf/1301.3781.pdf) and this good [tutorial](http://mccormickml.com/2016/04/19/word2vec-tutorial-the-skip-gram-model/)

The main idea of Skip-gram model is to use **center word** to predict its **context words**. Basic assumptions is that similar words will share the similar context. For example, the context of *hamburger* and *sandwich* may be similar because we can easily replace a word with the other and get meaningful sentences.

So if we use \\( word_i \\) as content word, then what's context of word \\( word_i \\)? In word2vec, context is described as a set of words shown in a *window* around the center word. So we can represent it as

$$ \{word_j | -w \le j-i \le w, j \neq i \}$$


## PyTorch Implementation

First, we need to initialize the lookup table with the following code. Note that we use two lookup tables for center word \\( u \\) and context word \\( v \\) respectly, which is coincident with original word2vec code. Also, we initialize `v_embeddings` with zeros and `u_embeddings` with a uniform distribution in \\( [-0.5/emb_{size}, 0.5/emb_{size}]\\)

```python
def __init__(self, emb_size, emb_dimension):
    """Initialize model parameters.
    Args:
        emb_size: Embedding size.
        emb_dimention: Embedding dimention, typically from 50 to 500.
    """
    super(SkipGramModel, self).__init__()
    self.emb_size = emb_size
    self.emb_dimension = emb_dimension
    self.u_embeddings = nn.Embedding(emb_size, emb_dimension, sparse=True)
    self.v_embeddings = nn.Embedding(emb_size, emb_dimension, sparse=True)
    self.init_emb()
def init_emb(self):
    initrange = 0.5 / self.emb_dimension
    self.u_embeddings.weight.data.uniform_(-initrange, initrange)
    self.v_embeddings.weight.data.uniform_(-0, 0)
```

Second, we implement the forward pard of word2vec. The sizes of input variables are as following:
* pos_u: \[batch_size\]
* pos_v: \[batch_size\]
* neg_v: \[batch_size, neg_sampling_count\]

Note that the shape of `neg_v` is different with other parameters because for each positive `u`, we will sample multiple negative `v` to construct negative pairs. We could copy each positive `u` for neg_sampling_count times. However, this method requires us to redo the following code again, which will cost lots of time.

```python
self.u_embeddings(Variable(torch.LongTensor(pos_u)))
```

Instead, we use `torch.bmm` to replace lookup and mul operation. In our test, it will accelerate the training for 3 times. You can find more about `torch.bmm` in [pytorch document](http://pytorch.org/docs/master/torch.html#torch.bmm).

```python
def forward(self, pos_u, pos_v, neg_v):
    """Forward process.
    As pytorch designed, all variables must be batch format, so all input of this method is a list of word id.

    Args:
        pos_u: list of center word ids for positive word pairs.
        pos_v: list of neibor word ids for positive word pairs.
        neg_v: list of neibor word ids for negative word pairs.
    """
    losses = []
    emb_u = self.u_embeddings(Variable(torch.LongTensor(pos_u)))
    emb_v = self.v_embeddings(Variable(torch.LongTensor(pos_v)))
    score = torch.mul(emb_u, emb_v).squeeze()
    score = torch.sum(score, dim=1)
    score = F.logsigmoid(score)
    losses.append(sum(score))
    neg_emb_v = self.v_embeddings(Variable(torch.LongTensor(neg_v)))
    neg_score = torch.bmm(neg_emb_v, emb_u.unsqueeze(2)).squeeze()
    neg_score = torch.sum(neg_score, dim=1)
    neg_score = F.logsigmoid(-1 * neg_score)
    losses.append(sum(neg_score))
    return -1 * sum(losses)
```

## Useful Suggestions

Here is some good tricks when implementing word2vec with pytorch. Without these tricks, the training speed will be so slow.

### No backpropagation

Of course, its the duty of PyTorch.

### Sparse update

This suggestion involves two different aspects.

First, set `sparse=True` when initialize lookup table:

```python
nn.Embedding(
    emb_size,
    emb_dimension,
    sparse=True
    )
```

Second, you need to carefully choose optimizer and its parameters to guarantee no global update will be excuted when training. For example, parameters like `weight_decay` and `momentum` in `torch.optim.SGD` require the global calculation on embedding matrix, which is extremely time-consuming. So in our experiments, we use the simplest version of SGD by:

```python
self.optimizer = optim.SGD(self.skip_gram_model.parameters(), lr=self.initial_lr)
```

### Use batch

To accelerate your code with GPU, please use batch. Also, all operation in PyTorch is based on batch, so learn to use it.

Some operations are different when using batch. For instance, the dot product can be calculate with
```python
score = torch.dot(emb_u, emb_v)
```
before using batch. However, it changes to
```python
score = torch.mul(emb_u, emb_v)
score = torch.sum(score, dim=1)
```
when using batch.

### Use numpy.random

One frequent operation in word2vec is to generate random number, which is used in negative sampling. To accelerate it, original word2vec use bitwise operation to simulate such generation.

In python, we could implement the same method to replace the slow `random.randint` function. However, it's kindly inelegant. Allow for the batch operation, we can use `numpy.random` package instead.

To be specific, using the following code to generate negative words:

```python
neg_v = numpy.random.choice(
        sample_table,
        size=(len(pos_word_pair),count)
    )
```
