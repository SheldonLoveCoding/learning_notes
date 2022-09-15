# Pytorch笔记

## 基础笔记

基础笔记可以参考李沐老师的《动手学习深度学习》https://zh-v2.d2l.ai/chapter_deep-learning-computation/parameters.html



## 梯度爆炸



## 梯度消失

在训练过程中，训练数据的predication与label的误差，梯度反传总是会让后几层的网络结构参数变化较大，训练较快，而前几层的参数更新量很小。

原因就是激活函数的导数小于1，通过链式法则求导，误差反向传播越传越小。

## Branch Normalization

训练过程中，网络参数的每一次更新都是在 `当前批次输入样本` 和 `当前网络参数` 的情况下更新的，所以本次误差反传更新的参数是适用于当前批次输入样本的（对于这一批次的样本，最终的误差会减小，模型对于当前数据是变好的），但是，对于下一批次数据，本次更新的参数反而会不适用，可能再次向反方向更新，造成震荡。（[这篇博客有附图很形象][https://blog.csdn.net/weixin_44023658/article/details/105844861]）

有一个方法缓解这一现象的就是把学习率设置的很小，避免震荡，但是效率很低。

Branch Normalization是为了解决这一问题的。他的做法是在网络的每一层都进行一步normalization。求这一branch的样本的均值与方差，然后做一个拉伸（$\gamma, \beta$）。下面的程序可以看到，对一个branch为5的数据，对每一个维度都进行了normalization。BN层的可学习的参数就是 $\gamma, \beta$

```python
voltage_num = 208
image_size = 10
N = Net(208, 64)
td_volt = torch.randn(5, 208)
print(td_volt)
'''
tensor([[ 0.4689,  0.1438,  0.6746,  ..., -0.3399,  1.4646, -0.1360],
        [-0.0510,  0.2074,  0.5683,  ...,  0.4345, -1.4404,  2.1185],
        [ 1.3306,  0.9211,  2.6028,  ...,  0.3657,  1.1169,  0.1418],
        [-1.8641,  1.1261,  0.7833,  ..., -0.4399, -0.6984,  0.7881],
        [ 0.4030,  1.0914,  1.7095,  ..., -1.3184,  0.8993,  1.0975]])
'''
print(torch.sum(td_volt, axis=1))
'''
tensor([ -4.8344, -16.4071,   3.5195,   3.5672,   9.0224])
'''
fc0 = nn.Sequential(
            nn.Linear(voltage_num, 4 * image_size * image_size),

        )
fc0.apply(init_constant)
fc1 = nn.Sequential(
            nn.Linear(voltage_num, 4 * image_size * image_size),
            nn.BatchNorm1d(4 * image_size * image_size),

        )
fc1.apply(init_constant) # 将全连接层的每一个参数都设置为1
image_vec0 = fc0(td_volt)
image_vec1 = fc1(td_volt)
print(image_vec0)
print(image_vec1)
'''
tensor([[ -4.8344,  -4.8344,  -4.8344,  ...,  -4.8344,  -4.8344,  -4.8344],
        [-16.4071, -16.4071, -16.4071,  ..., -16.4071, -16.4071, -16.4071],
        [  3.5195,   3.5195,   3.5195,  ...,   3.5195,   3.5195,   3.5195],
        [  3.5672,   3.5672,   3.5672,  ...,   3.5672,   3.5672,   3.5672],
        [  9.0224,   9.0224,   9.0224,  ...,   9.0224,   9.0224,   9.0224]],
       grad_fn=<AddmmBackward0>)
tensor([[-0.4291, -0.4291, -0.4291,  ..., -0.4291, -0.4291, -0.4291],
        [-1.7331, -1.7331, -1.7331,  ..., -1.7331, -1.7331, -1.7331],
        [ 0.5122,  0.5122,  0.5122,  ...,  0.5122,  0.5122,  0.5122],
        [ 0.5176,  0.5176,  0.5176,  ...,  0.5176,  0.5176,  0.5176],
        [ 1.1323,  1.1323,  1.1323,  ...,  1.1323,  1.1323,  1.1323]],
       grad_fn=<NativeBatchNormBackward0>)
'''
```

这是在训练过程中，会对用当前branch的均值和方差做normalization，那么测试的是，就是用训练集整体的均值和方差做normalization。这一点通过pytorch中的 `model.train()` 和 `model.eval()` 来设置。这一操作还会影响dropout（train使用，eval不使用）。使用了BN层，那么就没必要使用dropout了。因为BN层可以被认为是引入随机噪音来减小模型复杂度，因此不用dropout了。



对于全连接层，BN是作用在特征维上的：对每一个特征都进行BN；对于卷积层，BN是作用在通道维上的：对这一个batch的N个样本的同一个通道的对应像素位置进行BN。

```python
m = nn.BatchNorm2d(3) #(C)
input = torch.randn(2, 3, 3, 4) # (N, C, H, W)
output = m(input)
print(input)
print(output)
```

## Tensorboard+Pytorch操作

- 可以检查一下python环境里的setuptools的版本是不是小于60.0，如果是，恭喜。直接pip install tensorboard==2.0.2即可。如果不是，则不用白费力气了。

- 基本操作

  ```python
  from torch.utils.tensorboard import SummaryWriter
  writer = SummaryWriter('./EITNN_train_test_log')
  
  writer.add_scalar('Loss/test', test_loss, epoch)
  writer.flush()
  ```

- 然后在Pycharm的Terminal的中将目录切换到`EITNN_train_test_log`文件夹的当前级，然后运行

  ```
  tensorboard --logdir=EITNN_train_test_log
  ```

- 然后在Chrome上访问链接即可`http://localhost:6006/`

## 常用基础操作

### 判断是否在gpu上

```python
device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
print(device)
model = EITNet(voltage_num, image_size).to(device)
print(next(model.parameters()).is_cuda) #true
print(next(model.parameters()).device) #cuda:0
```

### Tensor与ndarray的转换



### 将nan替换为0

```
train_dataset = torch.where(torch.isnan(train_dataset), torch.full_like(train_dataset, 0), train_dataset)
```

### pytorch中 Tensor的boolean与int的转换

```python
mask_label = (~torch.isnan(image_label)).int()
```

### pytorch中的矩阵运算

```
 torch.mul (a, b) 是矩阵a和b对应位相乘，a和b的维度必须相等
 
```

### 将矩阵沿着某一维度重复

```python
mask_label_batch = mask_label.repeat(2, 1, 1, 1)
```

