---
categories:
    - algorithm
date: 2017-12-14T11:09:39Z
description: implement BPNN by python
keywords: BP, Neural Network, python
lastmod: 2023-08-18T12:31:07Z
tags:
    - python
    - classification
title: BP神经网络算法(python实现)
---



# BPNN

    神经网络是一种运算模型，由大量的节点（或称神经元）之间相互联接构成。
    每个节点代表一种特定的输出函数，称为激励函数（activation function）。
    每两个节点间的连接都代表一个对于通过该连接信号的加权值，称之为权重，这相当于人工神经网络的记忆。
    网络的输出则依网络的连接方式，权重值和激励函数的不同而不同。
    而网络自身通常都是对自然界某种算法或者函数的逼近，也可能是对一种逻辑策略的表达。

    神经网络主要有以下几种类型: 前向型、反馈型、随机型和竞争型。
    BPNN,Back Propagation Neural Network,属于前向型。
    前向型:
        前馈神经网络是指神经元分层排列，由输入层，隐藏层和输出层构成，其中隐藏层可能会有多层。
        这种神经网络的每一层的神经元只接受前一层神经元的输入，后面的层对于前面的层没有信号反馈。
        每一层对于输入数据进行一定的转换，然后将输出结果作为下一层的输入，直到最后输出结果。

## In action

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Created by PyCharm
# @author  : mystic
# @date    : 2017/11/23 8:41
import datetime
import math
import pickle
import random


def rand(a, b):
    """
        生成[a,b)区间内的随机数
    :param a:
    :param b:
    :return:
    """
    return (b - a) * random.random() + a


def make_matrix(m, n, fill=0.):
    """
        生成m*n的矩阵,默认是零矩阵
    :param m:
    :param n:
    :param fill:
    :return:
    """
    matrix = []
    for i in range(m):
        matrix.append([fill] * n)
    return matrix


def sigmoid(x):
    """
        S型函数:Log-sigmoid和Tan-sigmoid[这里采用Log-sigmoid]
    :param x:
    :return:
    """
    return 1.0 / (1.0 + math.exp(-x))


def sigmoid_derivative(x):
    """
        S型函数Log-sigmoid的导数
    :param x:
    :return:
    """
    return x * (1 - x)


class BPNeuralNetwork:
    def __init__(self):
        self.input_node = 0
        self.hidden_node = 0
        self.output_node = 0
        self.input_cells = []
        self.hidden_cells = []
        self.output_cells = []
        self.input_weights = []
        self.output_weights = []
        self.input_correction = []
        self.output_correction = []

    def setup(self, ni, nh, no):
        # 输入层、隐藏层、输出层的节点(数)
        self.input_node = ni + 1  # 增加一个偏差节点
        self.hidden_node = nh
        self.output_node = no
        # init cells 激活神经网络的所有节点
        self.input_cells = [1.0] * self.input_node
        self.hidden_cells = [1.0] * self.hidden_node
        self.output_cells = [1.0] * self.output_node
        # init weights 建立输入层到隐含层权重和隐含层到输出层的权重
        self.input_weights = make_matrix(self.input_node, self.hidden_node)
        self.output_weights = make_matrix(self.hidden_node, self.output_node)
        # random activate
        for i in range(self.input_node):
            for h in range(self.hidden_node):
                self.input_weights[i][h] = rand(-0.2, 0.2)
        for h in range(self.hidden_node):
            for o in range(self.output_node):
                self.output_weights[h][o] = rand(-2.0, 2.0)
        # init correction matrix 建立动量因子
        self.input_correction = make_matrix(self.input_node, self.hidden_node)
        self.output_correction = make_matrix(self.hidden_node, self.output_node)

    def predict(self, inputs):
        # activate input layer
        for i in range(self.input_node - 1):
            self.input_cells[i] = inputs[i]
        # activate hidden layer
        for j in range(self.hidden_node):
            total = 0.0
            for i in range(self.input_node):
                total += self.input_cells[i] * self.input_weights[i][j]
            self.hidden_cells[j] = sigmoid(total)
        # activate output layer
        for k in range(self.output_node):
            total = 0.0
            for j in range(self.hidden_node):
                total += self.hidden_cells[j] * self.output_weights[j][k]
            self.output_cells[k] = sigmoid(total)
        return self.output_cells[:]

    def back_propagation(self, case, label, learn, correct):
        """
            反向传播
        :param case: 样本
        :param label: 期望样本输出值
        :param learn: 学习速率
        :param correct: 动量因子
        :return:
        """
        # feed forward
        self.predict(case)
        # get output layer error
        output_deltas = [0.0] * self.output_node
        for o in range(self.output_node):
            error = label[o] - self.output_cells[o]
            output_deltas[o] = sigmoid_derivative(self.output_cells[o]) * error
        # get hidden layer error
        hidden_deltas = [0.0] * self.hidden_node
        for h in range(self.hidden_node):
            error = 0.0
            for o in range(self.output_node):
                error += output_deltas[o] * self.output_weights[h][o]
            hidden_deltas[h] = sigmoid_derivative(self.hidden_cells[h]) * error
        # update output weights
        for h in range(self.hidden_node):
            for o in range(self.output_node):
                change = output_deltas[o] * self.hidden_cells[h]
                self.output_weights[h][o] += learn * change + correct * self.output_correction[h][o]
                self.output_correction[h][o] = change
        # update input weights
        for i in range(self.input_node):
            for h in range(self.hidden_node):
                change = hidden_deltas[h] * self.input_cells[i]
                self.input_weights[i][h] += learn * change + correct * self.input_correction[i][h]
                self.input_correction[i][h] = change
        # get global error
        error = 0.0
        for o in range(len(label)):
            error += 0.5 * (label[o] - self.output_cells[o]) ** 2
        return error

    def train(self, cases, labels, limit=10000, learn=0.05, correct=0.1):
        for j in range(limit):
            error = 0.0
            for i in range(len(cases)):
                label = labels[i]
                case = cases[i]
                error += self.back_propagation(case, label, learn, correct)
        # 返回训练好的权重、动量因子等信息，便于BP网络的保存
        return dict(input_node=self.input_node, hidden_node=self.hidden_node, output_node=self.output_node,
                    input_cells=self.input_cells, hidden_cells=self.hidden_cells, output_cells=self.output_cells,
                    input_weights=self.input_weights, output_weights=self.output_weights,
                    input_correction=self.input_correction, output_correction=self.output_correction)

    def test(self):
        cases = [
            [0, 0.321, 0, 0.54, 0.337, 0.43, 0.64, 0, 0.618, 0.25, 0.36, 0.321, 0, 0.54, 0.337, 0.43, 0.64, 0, 0.618,
             0.25, 0.374],
            [0, 0.43, 0.39, 0.43, 0, 0.43, 0.55, 0.61, 0.21, 1, 0, 0.43, 0.39, 0.43, 0, 0.43, 0.55, 0.61, 0.21, 1,
             0.21],
            [0, 1, 0.66, 0, 0.13, 0.54, 0.32, 0.33, 0.25, 0.34, 0.52, 1, 0.66, 0, 0.13, 0.54, 0.32, 0.33, 0.25, 0.34,
             0.86],
            [0.81, 0.31, 0.23, 0.12, 0.32, 0.15, 0.56, 0.12, 0.33, 0.33, 0.42, 0.31, 0.23, 0.12, 0.32, 0.15, 0.56, 0.12,
             0.33, 0.33, 0.321],
            [0.61, 0, 0, 0.52, 0.55, 0.56, 0.25, 1, 1, 0, 0.76, 0, 0, 0.52, 0.55, 0.56, 0.25, 1,
             1, 0, 0.62],
            [0.37, 0, 1, 0.832, 0.643, 0.931, 0.821, 0.21, 0.235, 0.841, 0.213, 0, 1, 0.832, 0.643, 0.931, 0.821, 0.21,
             0.235, 0.841, 0.87],
            [0.42, 0.41, 0.32, 0.451, 0.324, 1, 0, 0.543, 0.328, 0.642, 0.872, 0.41, 0.32, 0.451, 0.324, 1, 0, 0.543,
             0.328, 0.642, 0.76],
            [0, 0.56, 0.43, 0.872, 0.432, 0.683, 0.5, 1, 0.52, 0.9, 0.42, 0.56, 0.43, 0.872, 0.432, 0.683, 0.5, 1,
             0.52, 0.9, 0.911],
            [0, 0.54, 0.62, 1, 0.24, 0.317, 0.58, 0.82, 0.432, 0.12, 0.9, 0.54, 0.62, 1, 0.24, 0.317, 0.58, 0.82,
             0.432, 0.12, 0.62],
            [1, 1, 0, 0.231, 0.321, 0.43, 0.42, 0.21, 0.56, 0.21, 0.661, 1, 0, 0.231, 0.321, 0.43, 0.42, 0.21,
             0.56, 0.21, 0.668]
        ]
        labels = [[0.257], [0.473], [0.261], [0.561], [0.201], [0.681], [0.697], [0.733], [0.375], [0.583]]
        self.setup(21, 4, 1)
        begin = datetime.datetime.now()
        save_net = self.train(cases, labels, 1000000, 0.1, 0.1)
        end = datetime.datetime.now()
        print('spend:', (end - begin))
        # 保存网络
        # with open('resource/bp_net.txt', 'wb') as fw:
        #     pickle.dump(save_net, fw, 0)
        for case in cases:
            print(self.predict(case))
            # print(self.predict(
            #     [1, 1, 1, 0.75, 0.833, 0.688, 0.858, 0.63, 0.859, 0, 0.322, 0.875,
            # 1, 0, 1, 1, 0.5, 0.834, 0.376, 0.233,1]))


if __name__ == '__main__':
    nn = BPNeuralNetwork()
    # nn.test()
    # 加载网络
    trained_net = None
    with open('resource/bp_net.txt', 'rb') as fr:
        trained_net = pickle.load(fr)
    nn.input_node = trained_net['input_node']
    nn.hidden_node = trained_net['hidden_node']
    nn.output_node = trained_net['output_node']
    nn.input_cells = trained_net['input_cells']
    nn.hidden_cells = trained_net['hidden_cells']
    nn.output_cells = trained_net['output_cells']
    nn.input_weights = trained_net['input_weights']
    nn.output_weights = trained_net['output_weights']
    nn.input_correction = trained_net['input_correction']
    nn.output_correction = trained_net['output_correction']
    predict_value = nn.predict(
        [1, 1, 0, 0.231, 0.321, 0.43, 0.42, 0.21, 0.56, 0.21, 0.661, 1, 0, 0.231, 0.321, 0.43, 0.42, 0.21,
         0.56, 0.21, 0.668])
    print(predict_value)

```

## Something worth noting

[Github Source Code](https://github.com/pplmx/MachineLearning.git)
