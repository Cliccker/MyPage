---
title: Tensorflow读书笔记
author: Hank
categories: 学习
img: "https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20200825155153.jpg"
mathjax: true
tags:
  - python
  - TensorFlow
summary: 系统的学习TensorFlow
abbrlink: 53334
date: 2020-08-24 18:52:19
---
## Hello World！
```python
import tensorflow as tf
h = tf.constant("Ciao!")
w = tf.constant("World")
hw = h + w
with tf.compat.v1.Session() as sess:
    ans = sess.run(hw)
print(ans)
>b'Ciao!World'
```
这一段代码展示了**计算图**的主要思想，即首先定义计算需要的元素，再采用一个外部机制去触发这个计算。也就是说`hw = h + w`并**没有**执行操作，真正执行操作的是`ans = sess.run(hw)`

## SoftmaxMnist

```python
import tensorflow as tf

from tensorflow.examples.tutorials.mnist import input_data

data_dir = 'data'
num_steps = 1000
minibatch_size = 100

data = input_data.read_data_sets(data_dir, one_hot=True)  # 自动加载数据集

x = tf.placeholder(tf.float32, [None, 784])  # None表示当前不指定每次使用图片的数量
W = tf.Variable(tf.zeros([784, 10]))  # 权重

y_true = tf.placeholder(tf.float32, [None, 10])
y_pred = tf.matmul(x, W)

cross_entropy = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(logits=y_pred, labels=y_true))
# 使用交叉熵作为loss
gd_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
# 定义如何训练
correct_mask = tf.equal(tf.argmax(y_pred, 1), tf.argmax(y_true, 1))
accuracy = tf.reduce_mean(tf.cast(correct_mask, tf.float32))

with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    for _ in range(num_steps):
        batch_xs, batch_ys = data.train.next_batch(minibatch_size)
        sess.run(gd_step, feed_dict={x: batch_xs, y_true: batch_ys})

    ans = sess.run(accuracy, feed_dict={x: data.test.images, y_true: data.test.labels})

print("Accuracy: {:.4}%".format(ans*100))
```

由于这本书是两年前的，很多模块都有了变化，后面我找了另一本更新的TensorFlow教程来看。