# BrainWebs
Neural nets for developers (for when you need one, but don't care about how it works)

## DISCLAIMER!!!

Look, you should probably *actually* learn some of this stuff. This is just a
temporary patch in your knowledge, so that you'll know some of the basics to get
by when you need to do some machine learning for a project, but don't expect to
really get great results or do anything really interesting until you understand
the theory behind how things work.

## Ok, here's the fun part

Neural networks have become kind of trendy recently, and Google gave us a nice
library called Tensorflow for working with them. Well, Tensorflow is also kind
of hard to get the grasp of, so then they made another abstracted layer on top
of Tensorflow to make creating neural networks as easy as humanly possible, so
that's how TFLearn happened. This is what we're going to be learning about
today.

We're going to build a classifier to determine how likely a semifinalist is to
get into TJ, using data from the [FCAG](http://www.fcag.org) website.

## A couple of notes

Tensorflow is symbolic: this means nothing gets evaulated until you compile the
chain of functions. This makes Tensorflow really efficient, because it can
optimize stuff based on what you're trying to do.

## Here's the code

```python
# Import all of the things we want

import pandas as pd # pandas for reading in data
import numpy as np  # numpy because numpy is wonderful
import tflearn      # for the neural net™

data = pd.read_excel('./201718.xlsx') # data at fcag.org/tjstatistics.shtml

def preprocess(data):
    del data['M/S GPA']
    del data['Semifinalist']
    del data['ID']
    del data['Application Year']
    del data['CombineScore']
    del data['Math and Verbal']

    data.AAP = data['AAP'].fillna(value='No')
    data['Final Decision'] = data['Final Decision'].fillna(value='N')
    
    data = data.dropna()
    data.reindex()
    
    np_data = data.as_matrix()
    
    np_data[np_data == 'Yes'] = 1
    np_data[np_data == 'No'] = 0
    np_data[np_data == 'F'] = 1
    np_data[np_data == 'M'] = 0
    np_data[np_data == 'Y'] = 2
    np_data[np_data == 'N'] = 0
    np_data[np_data == 'W'] = 1
    
    return np_data

data = preprocess(data)
trX = data[...,:6] # input data
trY = data[..., 6] # labels

optimizer = tflearn.optimizers.Adam(learning_rate=0.01)

net = tflearn.input_data(shape=[None, 6])
net = tflearn.fully_connected(net, 32, activation='relu')
net = tflearn.fully_connected(net, 32, activation='relu')
net = tflearn.fully_connected(net, 3, activation='softmax')
net = tflearn.regression(net, to_one_hot=True, n_classes=3, optimizer=optimizer, shuffle_batches=True)

model = tflearn.DNN(net)
model.fit(trX, trY, n_epoch=50, batch_size=300, show_metric=True, validation_set=.2)

model.predict([[0, 1, 1, 4, 47, 42]])
```
