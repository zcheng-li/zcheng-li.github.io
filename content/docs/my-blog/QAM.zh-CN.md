---
title: "动态 QAM 调制解调 python 实现"
blogType: "tutorial"
weight: 1
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

## 引入

题目基于 University of Glasgow 课程 ENG3014 Communications Systems 3 的大作业, 希望我们用课程学到的知识写一个简单的信息收发模拟程序:

{{% hint info %}}
This should include at minimum:
* A message of your choice, comprising more than 2048 bits, converted from a “STRING” to a “BINARY”.
* Using your modulation format of choice, modulate a carrier to encode this information.
* The channel SNR to be considered will be 6dB, 18dB and 24dB with respect to AGWN.
* An output filter limiting the transmitter bandwidth to less than 20% of the carrier frequency.
* Demodulation of the signal and recovery of the signal. Please analyse the Bit Error Rate of the received signal over multiple retransmission of the signal over the simulated channel.
* Discuss in your report choices you make, such as modulation format, Baud Rate, Filter type and bandwidth.

Up to 30% of the marks allocated will be included for creative additions (10% each), these could include:
* Simulate fading conditions associated with an urban environment. Implementation of Hamming coding or other error correction technique.
* Transmission of video, audio or other time varying signal over the channel.
* Dynamic modulation, based on measured system performance.
* Automatic resending of errored signals.
* Or anything else you think would be a cool addition!

{{% /hint %}}

:-(

归纳一下差不多如下:
1. 将 String 的输入转化为由 0 和 1 组成的二进制编码.
2. 用 Hamming code 编码.
3. 加入噪音.
4. 选一个喜欢的办法调制.
5. 再解调.
6. 计算误码率等其它指标.

我的项目做了下面的步骤:

![QAM 流程图](/img/QAM/QAM_flow.png "图1: 实现流程图")

不要慌, 我们一步步看👀.

---

## 库
```python
import numpy as np
from numpy import log2, arange, sqrt, flip, concatenate, zeros, array, floor, sqrt, array, bitwise_xor
from numpy.random import randint
import math
from math import ceil

from matplotlib import pyplot as plt
from scipy import signal 

from hamming import encode, decode
from bitarray import bitarray

```

---

## String 与 Binary 的转换

```python
def string2bits(s=''):
    out = ''.join([bin(ord(x))[2:].zfill(8) for x in s])
    return [int(x) for x in list(out)]
```

好多奇奇怪怪的函数😩.

* `ord(x)`: 返回代表字符 `x` 的 unicode 编码 (`int`), 比方说: `‘a’` 的 unicode 编码是 97, `‘?’` 的是 63.

* `bin(ord(x))`: 返回x的unicode编码的二进制字符串 (`string`), 注意:`ord()`的输出是十进制的呀. 比如: 

    `'a'` -> 97 -> `"0b1100001"`

    `'?'` -> 63 -> `"0b111111"`


注意到这里的字符串前面带了 `0b`, 因此我们需要:

* `bin(ord(x))[2:]`: 去掉 `0b`

    `'a'` -> 97 -> `"0b1100001"` -> `"1100001"`

    `'?'` -> 63 -> `"0b111111"` -> `"111111"`

又注意到每个字符串长度不一样, 这可能会让我们在解码的时候出现问题, 因此我们需要:

* `bin(ord(x))[2:].zfill(8)`: 补齐字符串到8位

    `'a'` -> 97 -> `"0b1100001"` -> `"1100001"` -> `"01100001"`

    `'?'` -> 63 -> `"0b111111"`-> `"111111"` -> `"00111111"`

* `out = ''.join([bin(ord(x))[2:].zfill(8) for x in s])`: 将所有字符转换后的 8 位二进制拼接成一个长字符串.

    `'a?'` -> `"0110000100111111"`

但是因为整形(`int`)要更好处理一些, 我们最后再换一换:

* `return [int(x) for x in list(out)]`: 遍历每一个字符 (`char`), 将其换成整形 (`int`):

    `'a?'` -> `"0110000100111111"` -> `[0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 1, 1, 1, 1, 1, 1]`

当当~~

既然都把这么乱七八糟的转换搞懂了, 那就不妨再看看怎么把它变回去. (Binary -> String)

```python
def bits2string(b=None):
    return ''.join([chr(int(x, 2)) for x in b])

```
道理也大差不差, `[... for x in b]` 用来遍历Binary数组中的每一个元素, `int(x, 2)` 将二进制的输入变成 10 进制, `chr()` 再把它变成字符, 最后再用 `''.join()` 合并在一起.

---

## AWGN 噪音

咱先跳过 Hamming Code, 看看怎么加 AWGN 噪音:

```python
def awgn(signal, snr, seed=False):
    ''' 
    Additive White Gaussian Noise (AWGN) function.
    '''
    
    if seed:
        np.random.seed(1)
        
    # Calculate the signal power as the average of the squared magnitudes of the signal
    sigpower = sum([math.pow(abs(signal[A_I]), 2) for A_I in range(len(signal))])
    sigpower = sigpower / len(signal)
    
    # Calculate the noise power based on the SNR
    # SNR (in linear scale) = Signal Power / Noise Power
    noisepower = sigpower / (math.pow(10, snr/10))
    
    # Generate Gaussian noise with mean 0 and standard deviation based on noise power
    noise = np.random.normal(0, np.sqrt(noisepower), len(signal))
    
    return np.array([noise])
    
```

其中

```python
    sigpower = sum([math.pow(abs(signal[A_I]), 2) for A_I in range(len(signal))])
    sigpower = sigpower / len(signal)

```
遍历 `signal` (即之前提到的 `[0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 1, 1, 1, 1, 1, 1]`)中的每一个元素, 将其平方相加, 然后再除以总长, 计算**信号功率**.

那么为什么要计算信号功率呢? 由信噪比公式:

{{< katex display=true >}}

\mathrm{SNR}_{\text{dB}} = 10 \log_{10} \left( \frac{P_{\text{signal}}}{P_{\text{noise}}} \right)

{{< /katex >}}

其中: 

* {{< katex >}} \mathrm{SNR}_{\text{dB}} {{< /katex >}} : 信噪比（分贝表示）
* {{< katex >}} P_{\text{signal}} {{< /katex >}} : 信号功率
* {{< katex >}} P_{\text{noise}}： {{< /katex >}} : 噪声功率

因此, 通过给定的信噪比 SNR, 我们可以通过信号功率反推出应该加入的**噪声功率**. 也就是这一步:

```python
    noisepower = sigpower / (math.pow(10, snr/10))
```

然后再通过这一噪声功率计算出 AWGN (也就是这个函数的输出), 最后和原本的信号做个加法就行啦~ 

`[0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 1, 1, 1, 1, 1, 1]` 

SNR=24:

```
array([[-0.03305892,  0.93648725,  1.07229167,  0.03032389, -0.08373429,
        -0.04651428, -0.02556458,  1.04447924, -0.04254978, -0.01583561,
         1.01729117,  0.98901276,  1.01125261,  0.97203918,  1.07735198,
         1.03366213]])
```

SNR=6:

```
array([[ 0.15621611,  0.5966272 ,  0.40825385, -0.03236429,  0.80902723,
         0.67793688,  0.05471882,  1.1841027 , -0.48631011, -0.56585869,
         1.15182751,  0.44881134,  0.74662291,  0.95341178,  0.9046891 ,
         0.71363364]])
```

可以明显发现信噪比 SNR 越大(如 SNR=24), 噪音对信号的影响越小, 反之, 信噪比 SNR 越小(如 SNR=6), 信号就变得更加乱七八糟.

---

## QAM 调制

(话说是不是应该先讲一讲 QAM 是什么?)

### QAM 是什么?

根据维基百科:

{{% hint info %}}
Quadrature amplitude modulation (QAM) is the name of a family of digital modulation methods and a related family of analog modulation methods widely used in modern telecommunications to transmit information. It conveys two analog message signals, or two digital bit streams, by changing (modulating) the amplitudes of two carrier waves, using the amplitude-shift keying (ASK) digital modulation scheme or amplitude modulation (AM) analog modulation scheme. The two carrier waves are of the same frequency and are out of phase with each other by 90°, a condition known as orthogonality or quadrature. The transmitted signal is created by adding the two carrier waves together. At the receiver, the two waves can be coherently separated (demodulated) because of their orthogonality. 
{{% /hint %}}

(啊, 好长...)

一句话, QAM 的传输效率更高.

比如 QAM16 一次可以传递 4 bit, 而 AM 一次只能传 1 bit.

![各种调制方法通道容量比较](/img/QAM/QAM_capacity_compare.png "图2: 各种调制方法通道容量比较")

对于本文中的 QAM 的实现, 简单来说, 就是三步: 1. 先把输入数组均分成两份, 2. 再通过幅移键控 (ASK) 编码, 3. 然后乘上各自的载波就OK了. 可以看看示意图:

![QAM 调制](/img/QAM/QAM_flow_mod.png "图3: QAM调制示意图")

### QAM 原理
至于它为什么能够成功编码解码, 用公式要更好理解一点.

首先将输入均分为两半, 然后分别计算得到各自的 ASK 编码: {{< katex >}} A_I(t) {{< /katex >}}和{{< katex >}} A_Q(t) {{< /katex >}}. 

{{< katex display=true >}}

I(t) = A_I(t) cos(2\pi f t) \\

Q(t) = A_Q(t) sin(2\pi f t)

{{< /katex >}}

其中, {{< katex >}} f {{< /katex >}} 是载波的频率. 然后将{{< katex >}} I(t) {{< /katex >}}和{{< katex >}} Q(t) {{< /katex >}}叠加, 得到: 

{{< katex display=true >}}

I(t) + Q(t) = A_I(t) cos(2\pi f t) + A_Q(t) sin(2\pi f t)

{{< /katex >}}

在解码的时候, 先分别乘上各自的载波:

{{< katex display=true >}}

(I(t) + Q(t)) cos(2\pi f t) = \frac{1}{2} (A_I(t) + A_I(t)cos(4\pi f t) + A_Q(t)sin(4\pi f t))

{{< /katex >}}

{{< katex display=true >}}

(I(t) + Q(t)) sin(2\pi f t) = \frac{1}{2} (A_Q(t) + A_I(t)cos(4\pi f t) - A_Q(t)sin(4\pi f t)) 

{{< /katex >}}

在过滤掉高频分量后:

{{< katex display=true >}}

Filter((I(t) + Q(t)) cos(2\pi f t)) \approx \frac{1}{2} A_I(t)

{{< /katex >}}


{{< katex display=true >}}

Filter((I(t) + Q(t)) sin(2\pi f t)) \approx \frac{1}{2} A_Q(t)

{{< /katex >}}

就可以成功得到{{< katex >}} A_I(t) {{< /katex >}}和{{< katex >}} A_Q(t) {{< /katex >}}啦 🎉.

### QAM 代码实现

为了让图更好看, 我们改一下初始输入字符串和信噪比:

```python
# Signal-to-Noise Ratio (SNR) in dB for the simulation
snr = 24
# Input string to be transmitted. 
input_string = "Communications Systems Communication Channel Simulation and Report. In this project, you will complete a simulation of a wireless communication system. This builds on the tasks in Lab 1 and the lectures cover all the background theory for each part of the task."
# input_string = "a?"

```

为了实现动态QAM, 我们希望对于不同的信噪比选择不同的QAM模型(QAM4, QAM16 或是 QAM64). 此处, 对应于上述 `snr = 24`, 其本会动态选择 QAM64, 但出于展示目的, 我们将其**手动调整为 QAM16**.

```python
# Dynamically adjust the modulation order based on the Signal-to-Noise Ratio (SNR)
if snr >= 28: 
    M = 64  # Use 64-QAM for high SNR, allowing more symbols (6 bits per symbol) for higher data rates
elif snr >= 20: 
    M = 16  # Use 16-QAM for moderate SNR, balancing data rate and noise resilience
else: 
    M = 4  # Use 4-QAM (QPSK) for low SNR, prioritizing robustness over data rate

# if you do NOT want to use dynamic modulation you can use the following instruction.
# Default modulation order for QAM (16-QAM) indicating 16 different symbols
M = 16 

M

```

一些其它的衍生量, 为了方便后面的计算.

```python
sqrt_M = int(sqrt(M))
size_symbol = int(log2(M))

n_symbol = int(len(input_string) * (8 / size_symbol)) # number of QAM symbols to create

size_symbol, n_symbol

```
对于 QAM16 和 指定的发送信息, `size_symbol = 4`, `n_symbol = 522`. 前者表示其”一口气“最多可以传4bit, 也就是一个 symbol, 后者表示对于给定的信息, 它需要传 522 symbol.

然后是两个载波:

```python
# Define carrier signal parameters
f_s = 10000 # Sampling frequency in MHz
rep_symbol = 500 # Number of samples used to represent a single QAM symbol
max_t = rep_symbol * n_symbol / f_s # Total time duration of the signal
print(max_t)
t = np.linspace(0, max_t, int(f_s * max_t)) # Generate the time vector for the signal

f_c = 100 # Carrier frequency in MHz
A_c = 1 # Amplitude of the carrier

# Generate the in-phase (I) and quadrature (Q) carrier signals
carrier_I = A_c*np.cos(2*np.pi*f_c*t)
carrier_Q = A_c*np.sin(2*np.pi*f_c*t)

plt.plot(t, carrier_I)
plt.xlim(left=0, right=1)

```

输出:

`26.3`

![QAM 载波](/img/QAM/QAM_carrier.png "图4: QAM 载波")

代码中我们可以发现一个变量 `rep_symbol`, 它在下列的代码中被用来“重复”输入数组, 已将其扩展为时间序列, 以此方便后面画图时更准确的采样. 而`f_s` 正是在话这个图是的采样频率.

```python
# Reshape the noisy input into blocks corresponding to QAM symbols
input_QAM = input_b_noise.reshape((-1,size_symbol))
print(input_QAM.shape)

# Initialize arrays for amplitude components of in-phase (I) and quadrature (Q)
A_I = zeros((input_QAM.shape[0],))
A_Q = zeros((input_QAM.shape[0],))

# Amplitude modulation: Calculate the amplitudes for I and Q components
for n in range(int(size_symbol / 2)):
    A_I = A_I + input_QAM[:,n] * 2 ** (int(size_symbol / 2) - 1 - n)
    A_Q = A_Q + input_QAM[:,n + int(size_symbol / 2)] * 2 ** (int(size_symbol / 2) - 1 - n)

# QAM
# print(A_I)
# print(A_Q)

# Generate repeated in-phase amplitude signal to match the time resolution
s_I = A_I.reshape(-1, 1).repeat(rep_symbol, 1).reshape(-1)
# Modulate the in-phase component with the cosine carrier
s_I_mod = s_I * carrier_I
# print(s_I)

# Generate repeated quadrature amplitude signal to match the time resolution
s_Q = A_Q.reshape(-1, 1).repeat(rep_symbol, 1).reshape(-1)
# Modulate the quadrature component with the sine carrier
s_Q_mod = s_Q * carrier_Q

```


然后把调制后的信号画出来:

```python
plt.figure()
plt.plot(t, s_I_mod)
plt.plot(t, s_I, linewidth=3)
plt.legend(["modulated I", "I"])
plt.grid()
plt.xlim(left=0, right=1)

plt.figure()
plt.plot(t, s_Q_mod)
plt.plot(t, s_Q, linewidth=3)
plt.legend(["modulated Q", "Q"])
plt.grid()
plt.xlim(left=0, right=1)

s_mod = s_I_mod + s_Q_mod

plt.figure()
plt.plot(t, s_mod, linewidth=3)
plt.xlim(left=0, right=1)

# plt.legend(["modulated s", "s"])
plt.grid()

```
输出:

![QAM 调制后的 I 分量信号](/img/QAM/QAM_I_mod.png "图5: QAM 调制后的 I 分量信号")

![QAM 调制后的 Q 分量信号](/img/QAM/QAM_Q_mod.png "图6: QAM 调制后的 Q 分量信号")

![QAM 调制后的总信号](/img/QAM/QAM_I+Q_mod.png "图7: QAM 调制后的总信号")

### QAM 星座图

为了绘制星座图, 我们有如下变量和函数:

```python
# Generate a normalized QAM constellation by representing points in binary format
# For example, QAM16: 00, 01, 10, 11
norm_constallation = [bin(x)[2:].zfill(size_symbol // 2) for x in arange(sqrt_M)]
norm_constallation

```

```python
def plot_constellation(S, color="blue", label=''):
        """
        Plots the QAM constellation diagram, showing signal points and the grid of possible symbols.
        """
        # Separate the real and imaginary parts of the symbols for plotting
        sx,sy = np.real(S), np.imag(S)
        
        # Map to constellation coord
        k = 2
        b = -(sqrt_M - 1)
        sx = [k * x + b for x in sx]
        sy = [k * y + b for y in sy]
        
        # Plot the symbols as scatter points
        if label:
            plt.scatter(sx,sy,s=30, c=color, label=label)
        else:
            plt.scatter(sx,sy,s=30, c=color)
        
        # Overlay the QAM grid points and annotate them with their binary representations
        for i, x in enumerate(range(-sqrt_M + 1, sqrt_M, 2)):
            for j, y in enumerate(range(-sqrt_M + 1, sqrt_M, 2)):
                plt.scatter(x, y, s=60, c="r")
                plt.annotate(f"{norm_constallation[i]}{norm_constallation[j]}", xy=(x-0.25, y-0.5))
                
        axis_limit = sqrt_M
        plt.axis([-axis_limit,axis_limit,-axis_limit,axis_limit])
        plt.axhline(0, color='black')
        plt.axvline(0, color='black')
        plt.grid(True)

```

```python
# draw constellation
S = A_I + 1j * A_Q
print(S[:10])

plot_constellation(S)

```

输出:

`[1.88111148+2.08807572j 3.01975263+0.01366766j 2.05328757+1.12644641j
 1.83682459+3.0908774j  2.00044805+2.97572787j 2.9585253 +0.84307519j
 1.92937708+3.06050133j 1.07733372+1.0006585j  2.98315354+0.84013525j
 2.08884394+1.86715759j]`

![QAM16 星座图(发送前)](/img/QAM/QAM16_cons_b.png "图8: QAM16 星座图(发送前)")

---

## 路径损失模拟 Hata Model

```python
# Hata Model for small or medium-sized city.

#f_c: in MHz
h_b = 30 # height of BS antenna in metres
h_m = 2 # height of mobile antenna in matres
distance = np.array([1, 2, 3, 4, 5]);  #in km

C_H = 0.8 + (1.1 * np.log10(f_c) - 0.7) * h_m - 1.56 * np.log10(f_c)
L_U = 69.55 + 26.16 * np.log10(f_c) - 13.82 * np.log10(h_b) - C_H + (44.9-6.55*np.log10(h_b))*np.log10(distance)

plt.plot(distance, L_U)
plt.xlabel('Distance from transmitter(in km)')
plt.ylabel('Path loss (in dB)')
plt.grid()

```

output:

![Hata model](/img/QAM/Hata_model.png "图9: Hata model")

可以发现, 距离越远, 损失越多.

---

## QAM 解调

为了方便解调, 我们先写一个滤波器出来:

```python
def butter_lowpass_filter(data, cutoff, fs, order):
    '''
    Applies a Butterworth lowpass filter to the given data. 
    '''
    
    # Calculate the Nyquist frequency (half the sampling frequency)
    nyq = 0.5 * fs
    
    # Normalize the cutoff frequency to the Nyquist frequency
    normal_cutoff = cutoff / nyq
    
    # b, a are the filter coefficients
    b, a = signal.butter(order, normal_cutoff, btype='low', analog=False)
    
    # Apply the filter to the data using forward-backward filtering
    y = signal.filtfilt(b, a, data)
    return y

```

QAM 解调以及对应的波形:

```python
# Demodulate the in-phase (I) component
# Multiply the received signal by the in-phase carrier and scale by 2 to isolate the I component
s_I_demod = s_mod * carrier_I * 2
# Apply the Butterworth lowpass filter to remove high-frequency components
s_I_demod = butter_lowpass_filter(s_I_demod, 50, f_s, 2)

# Demodulate the quadrature (Q) component
# Multiply the received signal by the quadrature carrier and scale by 2 to isolate the Q component
s_Q_demod = s_mod * carrier_Q * 2
# Apply the Butterworth lowpass filter to remove high-frequency components
s_Q_demod = butter_lowpass_filter(s_Q_demod, 50, f_s, 2)

# Plot the modulated and demodulated in-phase components
plt.figure()
plt.plot(t, s_I_mod)
plt.plot(t, s_I_demod)
plt.xlim(left=0, right=1)
plt.legend(["modulated I", "demodulated I"])
plt.grid()

# Plot the modulated and demodulated quadrature components
plt.figure()
plt.plot(t, s_Q_mod)
plt.plot(t, s_Q_demod)
plt.xlim(left=0, right=1)
plt.legend(["modulated Q", "demodulated Q"])
plt.grid()

```
output:

![QAM 解调后的 I 分量信号](/img/QAM/QAM_I_demod.png "图10: QAM 解调后的 I 分量信号")

![QAM 解调后的 Q 分量信号](/img/QAM/QAM_Q_demod.png "图11: QAM 解调后的 Q 分量信号")

对应的星座图:

```python
# Reshape the demodulated signal into blocks of size `rep_symbol` and calculate the average for each block
A_I_demod = np.average(s_I_demod.reshape((-1, rep_symbol)), 1)
A_Q_demod = np.average(s_Q_demod.reshape((-1, rep_symbol)), 1)

print(s_I_demod.reshape((rep_symbol, -1)).shape)
print(A_I_demod.shape)

# To draw constrllation

# Combined in-phase and quadrature components to form complex symbols
S_demod = A_I_demod + 1j * A_Q_demod

# Plot the constellation
plot_constellation(S, label="Transmitted")
plot_constellation(S_demod, color="green", label="Received")
plt.legend()

```
output:

```
(500, 526)
(526,)
```

![QAM16 星座图(解调后)](/img/QAM/QAM16_cons_a.png "图12: QAM16 星座图(解调后)")


```python
# Round the demodulated signal's amplitudes to the nearest integers
A_I_demod = [int(round(x)) for x in A_I_demod]
A_Q_demod = [int(round(x)) for x in A_Q_demod]

# Map the rounded amplitudes to their corresponding binary strings in the normalized constellation
output_b = [norm_constallation[min(len(norm_constallation) - 1, int(x))] + norm_constallation[min(len(norm_constallation) - 1, int(y))] for x, y in zip(A_I_demod, A_Q_demod)]

# Concatenate all binary strings into a single bitstream
output_b = ''.join(output_b)

# Decode the bitstream using a Hamming decode function
output_b = list(decode(bitarray(output_b)))
output_b_int_list = output_b

# Convert each binary value in the list to a string for easier manipulation
output_b = [str(x) for x in output_b]

# Group the binary strings into bytes (8 bits each) for interpretation
output_b = [''.join(output_b[8*i : 8*i + 8]) for i in range(len(output_b) // 8)]
print(output_b[:10])

# Convert the grouped binary values back into the original string
output_string = bits2string(output_b)
print(output_string)

```
output:

```
['01000011', '01101111', '01101101', '01101101', '01110101', '01101110', '01101001', '01100011', '01100001', '01110100']
Communications Systems Communication Channel Simulation and Report. In this project, you will complete a simulation of a wireless communication system. This builds on the tasks in Lab 1 and the lectures cover all the background theory for each part of the task.
```

## 误码率

```python 
print(output_b_int_list[:24])
print(input_b_int_list[:24])

# Initialize the Bit Error Rate (BER) counter
BER = 0
# Initialize a list to store the indices of errors
error_list = []

# Iterate through pairs of input and output bits (or values) for comparison
for i, (input, output) in enumerate(zip(input_b_int_list, output_b_int_list)):
    if input != output: 
        BER += 1
        error_list += [i]

# Calculate the BER by dividing the total errors by the length of the output
BER /= len(output_b_int_list)
print(f"BER is {BER}")
print(f"find error at {error_list}")

```

output:
```
[0, 1, 0, 0, 0, 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1]
[0, 1, 0, 0, 0, 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1]
BER is 0.0
find error at []

```

十分完美!







