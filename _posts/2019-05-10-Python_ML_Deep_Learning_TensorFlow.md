---
layout: post
title: "Python机器学习-深度学习入门"
date: 2019-05-10 
description: "TensorFlow基础、神经网络、卷积网络"
tag: Python 

---

### TensorFlow 基础

>```python
>import tensorflow as tf
>
># 创建模型
>model = tf.keras.Sequential([
>    tf.keras.layers.Dense(128, activation='relu'),
>    tf.keras.layers.Dropout(0.2),
>    tf.keras.layers.Dense(10, activation='softmax')
>])
>
># 编译模型
>model.compile(optimizer='adam',
>              loss='sparse_categorical_crossentropy',
>              metrics=['accuracy'])
>```

### 训练模型

>```python
># 训练
>model.fit(X_train, y_train, epochs=10, validation_split=0.2)
>
># 评估
>test_loss, test_acc = model.evaluate(X_test, y_test)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-深度学习入门](http://zhouzhiyang.cn/2019/05/Python_ML_Deep_Learning_TensorFlow/) 

