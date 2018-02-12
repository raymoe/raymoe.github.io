---
title: tensorflow学习笔记
date: 2018-02-09 16:49:56
tags: [tensorflow]
---
本文是斯坦福大学[cs20](https://web.stanford.edu/class/cs20si/)的学习笔记，lecture note 和 slides 见 （https://web.stanford.edu/class/cs20si/syllabus.html）
### 变量

#### 初始化变量

变量使用之前必须初始化，如果使用之前不初始化，将会报错FailedPreconditionError: Attempting to use uninitialized value。可以将未初始化的变量打印出来
```
print(session.run(tf.report_uninitialized_variables()))
```
最简单的方法是一次性初始化所有变量
```
with tf.Session() as sess:
	sess.run(tf.global_variables_initializer())
```
初始化一部分变量，可以通过tf.variables_initializer()来初始化你想要初始化的变量列表
```
with tf.Session() as sess:
	sess.run(tf.variables_initializer([a, b]))
```

#### 变量求值
可以通过session.run
```
# W is a 784 x 10 variable of random values
V = tf.get_variable("normal_matrix", shape=(784, 10), 
                     initializer=tf.truncated_normal_initializer())

with tf.Session() as sess:
	sess.run(tf.global_variables_initializer())
	print(sess.run(V))
```
也可以通过tf.Variable.eval()
```
with tf.Session() as sess:
	sess.run(tf.global_variables_initializer())
	print(V.eval())
```

#### 变量赋值
通过tf.Variable.assign()
```
W = tf.Variable(10)
W.assign(100)
with tf.Session() as sess:
	sess.run(W.initializer)
	print(W.eval()) # >> 10
```
为什么是10而不是100，如果我们通过assign 操作来做，要使得这个op生效，需要在session中run这个操作
```
W = tf.Variable(10)

assign_op = W.assign(100)
with tf.Session() as sess:
	sess.run(assign_op)
	print(W.eval()) # >> 100
```

注意我们并没有初始化W这个变量，因为assign()替我们做了，实际上，initializer操作是一个将变量的初始值赋值给变量自身的assign()操作
在源代码里面
```
# in the source code
self._initializer_op = state_ops.assign(self._variable, self._initial_value,                                                                      validate_shape=validate_shape).op
```

一个有意思的例子
```
# create a variable whose original value is 2
a = tf.get_variable('scalar', initializer=tf.constant(2)) 
a_times_two = a.assign(a * 2)
with tf.Session() as sess:
	sess.run(tf.global_variables_initializer()) 
	sess.run(a_times_two) # >> 4
	sess.run(a_times_two) # >> 8
	sess.run(a_times_two) # >> 16
```

对于简单的变量自增运算和变量自减，TensorFlow提供了tf.Variable.assign_add() 和 tf.Variable.assign_sub() 方法，和tf.Variable.assign()方法不同，tf.Variable.assign_add() 和 tf.Variable.assign_sub()并不初始化变量，因为这两个操作并不依赖变量的初始值。

```
W = tf.Variable(10)

with tf.Session() as sess:
	sess.run(W.initializer)
	print(sess.run(W.assign_add(10))) # >> 20
	print(sess.run(W.assign_sub(2)))  # >> 18
```

因为TensorFlow的sessions单独保存变量，每个session都有自己定义在图中变量的当前值
```
W = tf.Variable(10)
sess1 = tf.Session()
sess2 = tf.Session()
sess1.run(W.initializer)
sess2.run(W.initializer)
print(sess1.run(W.assign_add(10)))		# >> 20
print(sess2.run(W.assign_sub(2)))		# >> 8
print(sess1.run(W.assign_add(100)))		# >> 120
print(sess2.run(W.assign_sub(50)))		# >> -42
sess1.close()
sess2.close()
```


当你有一个变量依赖另外一个变量，假设你有定义U=W*2
```
# W is a random 700 x 10 tensor
W = tf.Variable(tf.truncated_normal([700, 10]))
U = tf.Variable(W * 2)
```
在这种情况下你应该使用initialized_value() 来保证在初始化U时W自己的值已经被初始化
```
U = tf.Variable(W.initialized_value() * 2)
```

### InteractiveSession

InteractiveSession 和 Session的区别在于InteractiveSession使他自己成为一个默认的session,以使得我们在执行run和eval方法时不需要在前面显式启动session
```
sess = tf.InteractiveSession()
a = tf.constant(5.0)
b = tf.constant(6.0)
c = a * b
print(c.eval()) # we can use 'c.eval()' without explicitly stating a session
sess.close()
```

### Control Dependencies

控制独立操作的执行次序
```
# your graph g have 5 ops: a, b, c, d, e
with g.control_dependencies([a, b, c]):
  # `d` and `e` will only run after `a`, `b`, and `c` have executed.
  d = ...
  e = …
```

### Importing Data
#### 老式方法：placeholders 和 feed_dict
记住TensorFlow程序通常有两个阶段
```
阶段1：组装图
阶段2：通过一个session来执行图中的操作和变量求值
```

```
a = tf.placeholder(tf.float32, shape=[3]) # a is placeholder for a vector of 3 elements
b = tf.constant([5, 5, 5], tf.float32)
c = a + b # use the placeholder as you would any tensor
with tf.Session() as sess:
	print(sess.run(c)) 
with tf.Session() as sess:
	# compute the value of c given the value of a is [1, 2, 3]
	print(sess.run(c, {a: [1, 2, 3]})) 		# [6. 7. 8.]
```

可以给一个不是placeholders的tensors feed 数据。Any tensors that are feedable can be fed
To check if a tensor is feedable or not, use:
```
tf.Graph.is_feedable(tensor)
```

```
a = tf.add(2, 5)
b = tf.multiply(a, 3)

with tf.Session() as sess:
	print(sess.run(b)) 						# >> 21
	# compute the value of b given the value of a is 15
	print(sess.run(b, feed_dict={a: 15})) 			# >> 45
```
#### 新式方法 tf.data
关于placeholders和feed_dicts的一个好处就是他们把数据处理放在了TensorFlow之外，这样就很容易在Python中随机生成随机数据。缺点是这种机制可能会减慢你的程序。用户通常最终会在一个线程中处理数据，导致这成为性能瓶颈，从而降低执行速度。

TensorFlow的tf.data模块比占位符更快，比队列更容易使用，不会崩溃。

通过tf.data,不像feed_dict将数据存储在非TensorFlow对象中，而是将数据存在tf.data.Dataset对象中，我们可以这样创建一个dataset
```
tf.data.Dataset.from_tensor_slices((features, labels))
```
打印output_types 和 output_shapes
```
print(dataset.output_types)			# >> (tf.float32, tf.float32)
print(dataset.output_shapes)		       # >> (TensorShape([]), TensorShape([]))
```


可以通过使用TensorFlow的文件格式解析器来创建tf.data.Dataset,有点类似于老式的DataReader


1. tf.data.TextLineDataset(filenames): 适用于数据以行来分割
2. tf.data.FixedLengthRecordDataset(filenames): 适用于数据以固定长度来分割
3. tf.data.TFRecordDataset(filenames)：适用于数据tfrecord 格式来存储



当我们将数据转为Dataset对象之后，我们可以通过iterator来迭代样本数据
```
iterator = dataset.make_one_shot_iterator()
X, Y = iterator.get_next()          
```

通过tf.data.Dataset,可以通过一条命令batch,shuffle,repeat 数据, 还可以通过map映射将一个数据集生成一个新的数据集
```
dataset = dataset.shuffle(1000)
dataset = dataset.repeat(100)
dataset = dataset.batch(128)
dataset = dataset.map(lambda x: tf.one_hot(x, 10)) 
```

### 优化器

当TensorFlow执行optimizer时，他会执行图里面这个操作所依赖的那一部分。
![tensorflow_20180210000319](..\images\tensorflow_20180210000319.png)





通过这张图，可以看到GradientDescentOptimizer 这个节点依赖三个节点，weights, bias, 和 gradients 。GradientDescentOptimizer  表示我们的更新方案是梯度下降法。TensorFlow 会自动为我们做微分，然后更新w和b的值以使得损失函数最小。



对于我们不想训练的参数，可以设置Trainable 为 False

```

tf.Variable(
    initial_value=None,
    trainable=True,
    collections=None,
    validate_shape=True,
    caching_device=None,
    name=None,
    variable_def=None,
    dtype=None,
    expected_shape=None,
    import_scope=None,
    constraint=None
)

tf.get_variable(
    name,
    shape=None,
    dtype=None,
    initializer=None,
    regularizer=None,
    trainable=True,
    collections=None,
    caching_device=None,
    partitioner=None,
    validate_shape=True,
    use_resource=None,
    custom_getter=None,
    constraint=None
)

```



还可以让优化器对特定的变量求梯度，还可以修改优化器算出的梯度

```
# create an optimizer.
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.1)

# compute the gradients for a list of variables.
grads_and_vars = optimizer.compute_gradients(loss, <list of variables>)

# grads_and_vars is a list of tuples (gradient, variable).  Do whatever you
# need to the 'gradient' part, for example, subtract each of them by 1.
subtracted_grads_and_vars = [(gv[0] - 1.0, gv[1]) for gv in grads_and_vars]

# ask the optimizer to apply the subtracted gradients.
optimizer.apply_gradients(subtracted_grads_and_vars)
```



** 还可以通过 tf.stop_gradient来阻止某些tensors在计算梯度求导时候的贡献 **

```
stop_gradient( input, name=None )
```

