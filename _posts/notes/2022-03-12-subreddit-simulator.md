---
title: "Building a Text Generation tool to simulate Subreddit comments"
date: 2022-03-12
permalink: /notes/2022/03/12/subreddit-simulator
tags:
    - python    
    - notebook
--- 
## Simulating r/nba comments

The goal of this project was to simulate r/nba subreddit comments to reasonable
degree of accuracy.

Eventually, I plan on using [this version of
GPT-2](https://github.com/nshepperd/gpt-2) to make a more realistic model for
each nba subreddit and post it on [threadalytics](https://threadalytics.com).
But, we need a basic understanding of how text generation works first. 
 
## Setup 
 
##### Imports 

**In [0]:**

{% highlight python %}
from tensorflow.keras.preprocessing.sequence import pad_sequences
from tensorflow.keras.layers import Embedding, LSTM, Dense, Dropout
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.models import Sequential
import tensorflow.keras.utils as ku 
from tensorflow.keras.models import load_model

import pandas as pd
import numpy as np
import string, os
import requests
from pprint import pprint
from datetime import datetime
import json
{% endhighlight %}


<p style="color: red;">
The default version of TensorFlow in Colab will soon switch to TensorFlow 2.x.<br>
We recommend you <a href="https://www.tensorflow.org/guide/migrate" target="_blank">upgrade</a> now 
or ensure your notebook will continue to use TensorFlow 1.x via the <code>%tensorflow_version 1.x</code> magic:
<a href="https://colab.research.google.com/notebooks/tensorflow_version.ipynb" target="_blank">more info</a>.</p>


 
### Load dataset

Here, we have a prescraped `json` file containing all the comments and metadata
scraped using [the Pushift API](https://github.com/pushshift/api). 

**In [0]:**

{% highlight python %}
os.listdir('.')
{% endhighlight %}




    ['.config', 'warriors.txt', 'sample_data']



**In [0]:**

{% highlight python %}
with open('./warriors.txt', 'r') as input_file:
  text = input_file.read()
print(text[:100])
{% endhighlight %}

    top_level
    t3_cogys i don't follow the nba much ... but as a resident of the bay area, i support all 

 
### Data Preprocessing

Using this list comments, we want to create one massive corpus. The corpus will
consist of each comment's `body` and some other metadata including:
* Whether it is a `reply` or a `top_level` comment. The logic in storing this
information is to have the model understand the difference between top level
comments and replies, which vary in content and size.
* `link_id` and `parent_id` - The link id will be put to the end of the body and
the parent id will be added to the start to teach the model the *nesting*
pattern in reddit threads. 

**In [0]:**

{% highlight python %}
corpus = []
s = ''
for t in text.split('\n'):
  if t=='top_level' or t=='reply':
    fixed_s = ' '.join(s.split(' ')[1:-1])
    corpus.append(fixed_s)
    s = ''
    continue
  else:
    s+=t  
corpus = corpus[-1000:]
{% endhighlight %}
 
## Data Preparation

The first step is to create a tokenizer. Since Machine Learning algorithms do
not have a notion of characters or words, we need to map each word in the corpus
to a numerical value. 

**In [0]:**

{% highlight python %}
tokenizer = Tokenizer()
# fit the tokenizer to the text (creates a mapping from word -> int)
tokenizer.fit_on_texts(corpus)
# find the total number of words
total_words = len(tokenizer.word_index) + 1
print(total_words)
{% endhighlight %}

    3786

 
Next, let's create the training data.

The first step is to take each comment and turn into an array of integers using
the tokenizer.

Then, we take *all sublists* of this array and add them to an array of inputs. 

**In [0]:**

{% highlight python %}
input_seq = []
should_print = True
for line in corpus:
    token_list = tokenizer.texts_to_sequences([line])[0]
    # token_list is a list of numbers for each sequence
    if should_print:
      print(token_list)
    for i in range(1, len(token_list)):
        n_gram_sequence = token_list[:i+1]
        input_seq.append(n_gram_sequence)
    if should_print:
      pprint(input_seq)
      should_print = False
{% endhighlight %}

    [20, 423, 710, 613, 23, 16, 455, 1, 120, 91, 1, 188, 88, 1850, 1851, 44, 24, 1, 238, 711, 1852, 11, 1, 174, 175, 66, 6, 558, 28, 1, 71, 39, 9, 228, 12, 239, 10, 1, 188, 88, 218, 52, 63, 14, 845, 3, 14, 846, 40, 3, 87, 1853, 2, 56, 3, 1, 61, 93, 166, 54, 44, 11, 424, 25, 85, 25, 16, 26, 11, 53, 30, 91, 20, 1854, 1, 1855, 7, 202, 503, 4, 26, 559, 1856, 1857, 141, 53, 30]
    [[20, 423],
     [20, 423, 710],
     [20, 423, 710, 613],
     [20, 423, 710, 613, 23],
     [20, 423, 710, 613, 23, 16],
     [20, 423, 710, 613, 23, 16, 455],
     [20, 423, 710, 613, 23, 16, 455, 1],
     [20, 423, 710, 613, 23, 16, 455, 1, 120],
     [20, 423, 710, 613, 23, 16, 455, 1, 120, 91],
     [20, 423, 710, 613, 23, 16, 455, 1, 120, 91, 1],
     [20, 423, 710, 613, 23, 16, 455, 1, 120, 91, 1, 188],
     [20, 423, 710, 613, 23, 16, 455, 1, 120, 91, 1, 188, 88],
     [20, 423, 710, 613, 23, 16, 455, 1, 120, 91, 1, 188, 88, 1850],
     [20, 423, 710, 613, 23, 16, 455, 1, 120, 91, 1, 188, 88, 1850, 1851],
     [20, 423, 710, 613, 23, 16, 455, 1, 120, 91, 1, 188, 88, 1850, 1851, 44],
     [20, 423, 710, 613, 23, 16, 455, 1, 120, 91, 1, 188, 88, 1850, 1851, 44, 24],
     [20,
      423,
      710,
      ...,
      53,
      30]]

 
Finally, we create the features and labels.

In order to make the input an `mxn` array, we have to pad the inputs with 0's.
We perform this using `pad_sequences` after finding the `max_seq_len`.

Then, we take all the the first `max_seq_len-1` words in each input sequence as
the features and the next word (i.e. the last word in the input sequence) to be
the label.

Finally, we turn this into a classification problem by converting all the labels
into categorical values. In other words, instead of each label corresponding to
a single word, each label corresponds to an array of 0's except for a single 1
indicating the next word. 

**In [0]:**

{% highlight python %}
max_seq_len = max([len(x) for x in input_seq])
input_seq = np.array(pad_sequences(input_seq, maxlen = max_seq_len, padding = 'pre'))
print('Inputs with padded 0s:')
print(input_seq)
features, labels = input_seq[:,:-1], input_seq[:,-1]
labels = ku.to_categorical(labels, num_classes = total_words)
print('First feature:')
print(features[0])
print('First label:')
print(labels[0])
{% endhighlight %}

    Inputs with padded 0s:
    [[   0    0    0 ...    0   20  423]
     [   0    0    0 ...   20  423  710]
     [   0    0    0 ...  423  710  613]
     ...
     [   0    0    0 ...    6   32 1756]
     [   0    0    0 ...   32 1756    3]
     [   0    0    0 ... 1756    3   20]]
    First feature:
    [ 0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
      0  0  0  0  0  0  0  0  0  0  0 20]
    First label:
    [0. 0. 0. ... 0. 0. 0.]

 
## Model Creation

We will use a simple model here consisting of 3 layers including:
* `Embedding` - to simplify the input layer. Word embeddings are a type of word
representation that allows words with similar meaning to have a similar
representation. Hence, similar words like 'player' and 'players' will have
similar representations. This step will improve performance by reducing the
number of dimensions of the input layer.
* `LSTM` - Long Short Term Memory layer. Nuerons in this layer has *feedback*
layers so we can *remember* information that was important.
* `Dense` - Output layer consisting of the size of the vocab 

**In [0]:**

{% highlight python %}
input_len = max_seq_len - 1
model = Sequential()

model.add(Embedding(total_words, 16, input_length=input_len))

model.add(LSTM(256))

model.add(Dense(total_words, activation='softmax'))

model.compile(loss='categorical_crossentropy', optimizer='adam')
model.summary()
{% endhighlight %}

    WARNING:tensorflow:From /usr/local/lib/python3.6/dist-packages/tensorflow_core/python/keras/initializers.py:119: calling RandomUniform.__init__ (from tensorflow.python.ops.init_ops) with dtype is deprecated and will be removed in a future version.
    Instructions for updating:
    Call initializer instance with the dtype argument instead of passing it to the constructor
    WARNING:tensorflow:From /usr/local/lib/python3.6/dist-packages/tensorflow_core/python/ops/resource_variable_ops.py:1630: calling BaseResourceVariable.__init__ (from tensorflow.python.ops.resource_variable_ops) with constraint is deprecated and will be removed in a future version.
    Instructions for updating:
    If using Keras pass *_constraint arguments to layers.
    Model: "sequential"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding (Embedding)        (None, 708, 16)           60576     
    _________________________________________________________________
    lstm (LSTM)                  (None, 256)               279552    
    _________________________________________________________________
    dense (Dense)                (None, 3786)              973002    
    =================================================================
    Total params: 1,313,130
    Trainable params: 1,313,130
    Non-trainable params: 0
    _________________________________________________________________

 
## Train the model 

**In [0]:**

{% highlight python %}
model.fit(features, labels, epochs = 5, batch_size = 64)
{% endhighlight %}

    WARNING:tensorflow:From /usr/local/lib/python3.6/dist-packages/tensorflow_core/python/ops/math_grad.py:1424: where (from tensorflow.python.ops.array_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use tf.where in 2.0, which has the same broadcast rule as np.where
    Train on 28375 samples
    Epoch 1/5
    28375/28375 [==============================] - 644s 23ms/sample - loss: 6.7666
    Epoch 2/5
    28375/28375 [==============================] - 638s 22ms/sample - loss: 6.4298
    Epoch 3/5
    28375/28375 [==============================] - 627s 22ms/sample - loss: 6.2397
    Epoch 4/5
    28375/28375 [==============================] - 629s 22ms/sample - loss: 6.0634
    Epoch 5/5
    28375/28375 [==============================] - 632s 22ms/sample - loss: 5.8841





    <tensorflow.python.keras.callbacks.History at 0x7fde9ee7dc50>



**In [0]:**

{% highlight python %}
model.save('nba.h5')
{% endhighlight %}

**In [0]:**

{% highlight python %}
model = load_model('nba.h5')
{% endhighlight %}

    WARNING:tensorflow:From /usr/local/lib/python3.6/dist-packages/tensorflow_core/python/ops/init_ops.py:97: calling GlorotUniform.__init__ (from tensorflow.python.ops.init_ops) with dtype is deprecated and will be removed in a future version.
    Instructions for updating:
    Call initializer instance with the dtype argument instead of passing it to the constructor
    WARNING:tensorflow:From /usr/local/lib/python3.6/dist-packages/tensorflow_core/python/ops/init_ops.py:97: calling Orthogonal.__init__ (from tensorflow.python.ops.init_ops) with dtype is deprecated and will be removed in a future version.
    Instructions for updating:
    Call initializer instance with the dtype argument instead of passing it to the constructor
    WARNING:tensorflow:From /usr/local/lib/python3.6/dist-packages/tensorflow_core/python/ops/init_ops.py:97: calling Zeros.__init__ (from tensorflow.python.ops.init_ops) with dtype is deprecated and will be removed in a future version.
    Instructions for updating:
    Call initializer instance with the dtype argument instead of passing it to the constructor

 
## Results 
 
## Text Generation Function

To simplify things, let's create a text generation function.

This function will take a `seed_text` and the number of words to generate.

For each word to generate, we tokenize and pad the text.

Next, we will predict the class using the token list.

Finally, we will find the word in the tokenizer that corresponds to the
generated text and add it to the seed text. 

**In [0]:**

{% highlight python %}
def generate_text(seed_text, num_next_words, model, max_seq_len, tokenizer):
    for _ in range(num_next_words):
        token_list = tokenizer.texts_to_sequences([seed_text])[0]
        token_list = pad_sequences([token_list], maxlen = max_seq_len - 1, padding = 'pre')
        predicted = model.predict_classes(token_list, verbose = 0)
        output_word = ""
        for word,index in tokenizer.word_index.items():
            if index == predicted:
                output_word = word
                break
        seed_text += " "+output_word
    print(seed_text.title() + '\n')
{% endhighlight %}

**In [0]:**

{% highlight python %}
generate_text('curry', 100, model, max_seq_len, tokenizer)
{% endhighlight %}

    Curry The Game Of The Team Is The Game And The Team And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The Season And The
    

 
Of course, my model definitely needs more work. I can fine tune it by increasing
the number of epochs and acquiring more data. This example is just a
demonstration of how to generate text in a simple manner. For better text
generation I will likely use GPT-2. 

**In [0]:**

{% highlight python %}

{% endhighlight %}
