---
layout: single
classes: wide
title: "TensorFlow 기본"
date: 2017-08-17
description: "텐서플로우는 데이터 플로우 그래프(Data flow graph)를 사용하여 수치 연산을 하는 오픈소스 소프트웨어 라이브러리로 이를 사용하여 다양한 딥러닝 모델을 학습할 수 있습니다."
categories:
  - Deep learning
tags: [deep learning, tensorflow]
comments: true
share: true
---

텐서플로우는 데이터 플로우 그래프(Data flow graph)를 사용하여 수치 연산을 하는 오픈소스 소프트웨어 라이브러리로 이를 사용하여 다양한 딥러닝 모델을 학습할 수 있습니다.

## 개요
---

### Tensorflow란
- 텐서플로우는 데이터 플로우 그래프(Data flow graph)를 사용하여 수치 연산을 하는 오픈소스 소프트웨어 라이브러리
- 최신 버전: r1.4
- API 지원 언어: Python, C++, Java, GO


## 구성
---

![Data flow graph](https://camo.githubusercontent.com/4ee55154486232ec9edd8f1a3bad4c4a146f6cfe/68747470733a2f2f7777772e74656e736f72666c6f772e6f72672f696d616765732f74656e736f72735f666c6f77696e672e676966)

#### Graph
- node: op(opertion)을 나타냄
- edge: node 사이를 이동하는 다차원 데이터 배열(Tensor)을 나타냄

#### Operation (op)
- Tensor들을 입력으로 받아서 연산을 수행한 후, 다시 Tensor들을 출력

#### Tensor
- 데이터를 표현하기 위한 타입을 가지는 다차원 배열
- Rank, Shape, Data Type 으로 구성
  - https://www.tensorflow.org/programmers_guide/dims_types

> 예제 코드

```python
# 텐서플로우의 기본적인 구성을 익힙니다.

# tf.constant: 상수
hello = tf.constant('Hello, TensorFlow!')

a = tf.constant(10)
b = tf.constant(32)

c = tf.add(a, b)  # c = a + b

print hello
print c
```

> 실행 결과

```python
Tensor("Const:0", shape=(), dtype=string)
Tensor("Add:0", shape=(), dtype=int32)
```

#### Session
- graph를 계산하기 위해서는 session 내에서 실행해야 함
- session은 graph의 op를 디바이스(CPU, GPU)에 할당하고 실행함

> 예제 코드

```python
# 그래프를 실행할 세션을 구성합니다.
sess = tf.Session()

# sess.run: 설정한 Tensor를 실행
print sess.run(hello)
print sess.run([a, b, c])

sess.close()
```

> 실행 결과

```python
Hello, TensorFlow!
[10, 32, 42]
```

#### Variable
- tesnor를 담기 위한 인메모리 버퍼
- 명시적으로 초기화해야 함
- 학습 중에 디스크에 저장 가능하며, 저장된 값을 다시 사용 가능

#### Placeholder
- operation의 입력으로 직접 값을 주기 위한 Tensor

> 예제 코드

```python
# 플레이스홀더와 변수의 개념을 익혀봅니다

# 입력 데이터, shape
x_data = [[1, 2, 3], [4, 5, 6]]

# tf.placeholder: 계산을 실행할 때 입력값을 받는 변수로 사용
# None 은 크기가 정해지지 않았음을 의미
X = tf.placeholder(tf.float32, [None, 3])
print "X: ", X

# tf.Variable: 그래프를 계산하면서 최적화 할 변수들
# tf.random_normal: 각 변수들의 초기값을 정규분포 랜덤 값으로 초기화
W = tf.Variable(tf.random_normal([3, 2]))
b = tf.Variable(tf.random_normal([2, 1]))
print "W: ", W
print "b: ", b

# tf.matmul 처럼 mat* 로 되어 있는 함수로 행렬 계산을 수행
expr = tf.matmul(X, W) + b
print "expr: ", expr
```

> 실행 결과

```python
X:  Tensor("Placeholder:0", shape=(?, 3), dtype=float32)
W:  <tf.Variable 'Variable:0' shape=(3, 2) dtype=float32_ref>
b:  <tf.Variable 'Variable_1:0' shape=(2, 1) dtype=float32_ref>
expr:  Tensor("add:0", shape=(2, 2), dtype=float32)
```

#### Feed
- graph에 정의한 placeholder로 데이터를 입력하는 것

#### Fetches
- operation의 출력값을 가져오는 것

> 예제 코드

```python
# Fetch & Feed 개념

# 세션 생성
sess = tf.Session()

# Variable 값 초기화
sess.run(tf.global_variables_initializer())

# expr 실행시에는 X라는 변수에 실제 입력값이 필요.
result = sess.run(expr, feed_dict={X: x_data})
print result

sess.close()
```

> 실행 결과

```python
[[  7.62927246   2.90463758]
 [ 19.67732811   7.28190136]]
```

---

## 예제

- x와 y의 상관관계를 분석하는 기초적인 선형 회귀 모델을 만들고 실행

#### TensorBoard
- 시각화 툴
- 실행
  - tensorboard --logdir=./tb
- 접속
  - <a href="http://localhost:6006/">TensorBoard</a>

#### Saver
- 학습한 모델을 저장하고 재사용

> 예제 코드

```python
# 학습 데이터
x_data = [1, 2, 3]
y_data = [1, 2, 3]

# hyperparameter
max_epochs = 100
learning_rate = 0.1

# 세션 생성
tf.reset_default_graph()
sess = tf.Session()

# Graph 정의

# name: 텐서보드나 디버깅 등에서 값의 변화를 추적하거나 보기 쉽게 하기 위해
x = tf.placeholder(tf.float32, name="x")
y = tf.placeholder(tf.float32, name="y")
print "x: ", x
print "y: ", y

w = tf.Variable(tf.random_uniform([1], -1.0, 1.0), name="weight")
b = tf.Variable(tf.random_uniform([1], -1.0, 1.0), name="bias")
print "w: ", w
print "b: ", b

# y = w * x + b
# w 와 x 가 행렬이 아니므로 단순 곱셈 기호를 사용
hypothesis = w * x + b
print "hypothesis: ", hypothesis

# 손실 함수를 작성합니다.
# mean(h - y)^2 : 예측값과 실제값의 거리를 비용(손실, loss) 함수로 정의
cost = tf.reduce_mean(tf.square(hypothesis - y), name="cost")

# 경사 하강법 최적화를 수행하여 비용을 최소화
optimizer = tf.train.GradientDescentOptimizer(learning_rate=learning_rate)
# optimizer = tf.train.AdamOptimizer(learning_rate=learning_rate)

train_op = optimizer.minimize(cost)
```

> 실행 결과

```python
x:  Tensor("x:0", dtype=float32)
y:  Tensor("y:0", dtype=float32)
w:  <tf.Variable 'weight:0' shape=(1,) dtype=float32_ref>
b:  <tf.Variable 'bias:0' shape=(1,) dtype=float32_ref>
hypothesis:  Tensor("add:0", dtype=float32)
```

> 예제 코드

```python
# saver 생성
saver = tf.train.Saver(tf.global_variables())

sess.run(tf.global_variables_initializer())

ckpt = tf.train.get_checkpoint_state("./model")
if ckpt and tf.train.checkpoint_exists(ckpt.model_checkpoint_path):
    print "restore model: %s" % ckpt.model_checkpoint_path
    # 이전 학습 모델 복구
    saver.restore(sess, ckpt.model_checkpoint_path)
```

> 실행 결과

```python
restore model: ./model/my-model
INFO:tensorflow:Restoring parameters from ./model/my-model
```

> 예제 코드

```python
# tensorboard 관련
tf.summary.scalar("cost", cost)

tf.summary.histogram("hypothesis", hypothesis)
tf.summary.histogram("weights", w)
tf.summary.histogram("bias", b)

tb_path = "./tb/GD_%d_%s_%s" % (max_epochs, learning_rate, datetime.strftime(datetime.now(), "%s"))
merged = tf.summary.merge_all()
writer = tf.summary.FileWriter(tb_path, sess.graph)
# writer = tf.summary.FileWriter('./tb/Adam_%d_%f' % (max_epochs, learning_rate), sess.graph)
```


> 실행 결과

```python
<tf.Tensor 'cost_1:0' shape=() dtype=string>
<tf.Tensor 'hypothesis:0' shape=() dtype=string>
<tf.Tensor 'weights:0' shape=() dtype=string>
<tf.Tensor 'bias_1:0' shape=() dtype=string>
```

> 예제 코드

```python
print "\n====== Training ======"
for step in range(max_epochs):
    # train_op 와 cost 그래프를 계산
    _, weight, bias, cost_val, summary = sess.run([train_op, w, b, cost, merged], feed_dict={x: x_data, y: y_data})

    if step % 10 == 0:
        print "[%d] loss: %f, w: %s, b: %s" % (step + 1, cost_val, weight, bias)

    # tensorboard에 summary 값 추가
    writer.add_summary(summary, step)

    # 중간 학습 모델 저장
#     saver.save(sess, "./model/my-model", global_step=step)

# 최종 학습 모델 저장
saver.save(sess, "./model/my-model")

# 최적화가 완료된 모델에 테스트 값을 넣고 결과가 잘 나오는지 확인
print "\n====== Test ======"
print "x = 5.0, y = ", sess.run(hypothesis, feed_dict={x: 5.0})
print "x = 2.5, y = ", sess.run(hypothesis, feed_dict={x: 2.5})
```

> 실행 결과

```python
====== Training ======
[1] loss: 0.000065, w: [ 0.99082839], b: [ 0.02084933]
[11] loss: 0.000040, w: [ 0.99280936], b: [ 0.01634615]
[21] loss: 0.000025, w: [ 0.99436241], b: [ 0.01281556]
[31] loss: 0.000015, w: [ 0.99558008], b: [ 0.01004757]
[41] loss: 0.000009, w: [ 0.99653471], b: [ 0.0078774]
[51] loss: 0.000006, w: [ 0.99728316], b: [ 0.00617599]
[61] loss: 0.000004, w: [ 0.99786997], b: [ 0.00484211]
[71] loss: 0.000002, w: [ 0.99833], b: [ 0.00379628]
[81] loss: 0.000001, w: [ 0.99869066], b: [ 0.00297631]
[91] loss: 0.000001, w: [ 0.99897349], b: [ 0.00233349]

'./model/my-model'

====== Test ======
x = 5.0, y =  [ 4.99775124]
x = 2.5, y =  [ 2.49981284]
```


#### 참고 자료
- https://www.tensorflow.org/programmers_guide/
- https://www.tensorflow.org/versions/master/get_started/get_started
- https://tensorflow.blog/1-%ED%85%90%EC%84%9C%ED%94%8C%EB%A1%9C%EC%9A%B0-%EA%B8%B0%EB%B3%B8%EB%8B%A4%EC%A7%80%EA%B8%B0-first-contact-with-tensorflow/
- 코드 참조
  - https://github.com/golbin/TensorFlow-Tutorials
