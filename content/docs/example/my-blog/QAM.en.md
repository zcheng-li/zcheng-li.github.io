---
title: "Dynamic QAM Modulation & Demodulation in Python"
weight: 1
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

## Introduction

This project is based on the major coursework of **ENG3014 Communications Systems 3** at the University of Glasgow.  
The task is to use what we have learned in the course to implement a simple communication system simulation:

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

In short, the task can be summarised as follows:

1. Convert a string input into a binary sequence consisting of 0s and 1s.
2. Encode it using Hamming code.
3. Add noise.
4. Modulate the signal using a modulation scheme of your choice.
5. Demodulate it at the receiver.
6. Compute the bit error rate (BER) and other performance metrics.

My project follows the pipeline shown below:

![QAM Flowchart](/img/QAM/QAM_flow.png "Figure 1: Implementation flowchart")

Don’t panic — we’ll go through everything step by step 👀.

---

## Libraries

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

⸻

## Converting Between String and Binary

```python
def string2bits(s=''):
    out = ''.join([bin(ord(x))[2:].zfill(8) for x in s])
    return [int(x) for x in list(out)]
```

That’s a lot of strange-looking functions 😩, so let’s break them down.

* `ord(x)`: returns the Unicode code point (int) of character x.

For example: 'a' → 97, '?' → 63.

* `bin(ord(x))`: converts that decimal integer into a binary string (str).

'a' → 97 → "0b1100001"
'?' → 63 → "0b111111"

Notice the "0b" prefix. We therefore need:

* `bin(ord(x))[2:]`: to strip off "0b".

'a' → "1100001"

'?' → "111111"

However, the resulting bit strings do not have a fixed length, which would cause problems during decoding.

So we pad them to 8 bits:

* `bin(ord(x))[2:].zfill(8)`

'a' → "01100001"
'?' → "00111111"

Next:

* `''.join([bin(ord(x))[2:].zfill(8) for x in s])`

concatenates all 8-bit binary representations into one long string.

'a?' → "0110000100111111"

Finally, integers are easier to work with than characters, so:

* `return [int(x) for x in list(out)]`

'a?' → [0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 1, 1, 1, 1, 1, 1]

Ta-da 🎉.

Since we now understand this messy conversion, let’s also look at how to convert it back
(Binary → String):

```python
def bits2string(b=None):
    return ''.join([chr(int(x, 2)) for x in b])

```

The idea is very similar: iterate through each binary chunk, convert it back to decimal using int(x, 2), then use `chr()` to recover the character, and finally join everything into a string.

⸻

## AWGN Noise

Let’s skip Hamming coding for now and first look at how AWGN noise is added:

```python
def awgn(signal, snr, seed=False):
    ''' 
    Additive White Gaussian Noise (AWGN) function.
    '''
    
    if seed:
        np.random.seed(1)
        
    sigpower = sum([math.pow(abs(signal[A_I]), 2) for A_I in range(len(signal))])
    sigpower = sigpower / len(signal)
    
    noisepower = sigpower / (math.pow(10, snr/10))
    
    noise = np.random.normal(0, np.sqrt(noisepower), len(signal))
    
    return np.array([noise])

```

Here,

```python
sigpower = sum([math.pow(abs(signal[A_I]), 2) for A_I in range(len(signal))])
sigpower = sigpower / len(signal)
```

iterates through the signal and computes its average power.

Why do we need the signal power?
Because from the definition of SNR:

{{< katex display=true >}}
\mathrm{SNR}{\text{dB}} = 10 \log{10} \left( \frac{P_{\text{signal}}}{P_{\text{noise}}} \right)
{{< /katex >}}

Given the SNR, we can compute the required noise power:

```python
noisepower = sigpower / (math.pow(10, snr/10))
```

We then generate Gaussian noise with that power and add it to the signal.

Example outputs:

```python
SNR = 24 dB

array([[-0.03305892,  0.93648725,  1.07229167,  0.03032389, ...]])

SNR = 6 dB

array([[ 0.15621611,  0.5966272 ,  0.40825385, -0.03236429, ...]])
```

Clearly, higher SNR → less noise; lower SNR → a much messier signal.

⸻

## QAM Modulation

(Should we first talk about what QAM actually is?)

What is QAM?

According to Wikipedia:

{{% hint info %}}
> Quadrature amplitude modulation (QAM) is the name of a family of digital modulation methods and a related family of analog modulation methods widely used in modern telecommunications to transmit information. It conveys two analog message signals, or two digital bit streams, by changing (modulating) the amplitudes of two carrier waves, using the amplitude-shift keying (ASK) digital modulation scheme or amplitude modulation (AM) analog modulation scheme. The two carrier waves are of the same frequency and are out of phase with each other by 90°, a condition known as orthogonality or quadrature. The transmitted signal is created by adding the two carrier waves together. At the receiver, the two waves can be coherently separated (demodulated) because of their orthogonality. 
{{% /hint %}}

(Yeah… that’s long.)

In one sentence: QAM is more spectrally efficient.

For example, 16-QAM can transmit 4 bits per symbol, while AM can only transmit 1 bit per symbol.

![各种调制方法通道容量比较](/img/QAM/QAM_capacity_compare.png "Fig2: Comparision of different modulation methods")

In this implementation, QAM can be summarised in three steps:

1. Split the input bitstream into two halves.
2. Apply amplitude shift keying (ASK).
3. Multiply each branch by its carrier.

A schematic is shown below:

![QAM 调制](/img/QAM/QAM_flow_mod.png "Fig3: QAM mod flowchart")

### QAM Theory

Why does QAM actually work for modulation and demodulation?  
Formulas make this much easier to see.

First, split the input into two equal halves, and compute the ASK amplitudes for each branch: {{< katex >}} A_I(t) {{< /katex >}} and {{< katex >}} A_Q(t) {{< /katex >}}.

{{< katex display=true >}}

I(t) = A_I(t) cos(2\pi f t) \\

Q(t) = A_Q(t) sin(2\pi f t)

{{< /katex >}}

Here, {{< katex >}} f {{< /katex >}} is the carrier frequency.  
Then we add {{< katex >}} I(t) {{< /katex >}} and {{< katex >}} Q(t) {{< /katex >}} together:

{{< katex display=true >}}

I(t) + Q(t) = A_I(t) cos(2\pi f t) + A_Q(t) sin(2\pi f t)

{{< /katex >}}

During demodulation, we multiply the received signal by each carrier:

{{< katex display=true >}}

(I(t) + Q(t)) cos(2\pi f t) = \frac{1}{2} (A_I(t) + A_I(t)cos(4\pi f t) + A_Q(t)sin(4\pi f t))

{{< /katex >}}

{{< katex display=true >}}

(I(t) + Q(t)) sin(2\pi f t) = \frac{1}{2} (A_Q(t) + A_I(t)cos(4\pi f t) - A_Q(t)sin(4\pi f t)) 

{{< /katex >}}

After filtering out the high-frequency components:

{{< katex display=true >}}

Filter((I(t) + Q(t)) cos(2\pi f t)) \approx \frac{1}{2} A_I(t)

{{< /katex >}}

{{< katex display=true >}}

Filter((I(t) + Q(t)) sin(2\pi f t)) \approx \frac{1}{2} A_Q(t)

{{< /katex >}}

And that’s how we successfully recover {{< katex >}} A_I(t) {{< /katex >}} and {{< katex >}} A_Q(t) {{< /katex >}} 🎉.

### QAM Code Implementation

To make the plots look nicer, let’s tweak the initial input string and SNR:

```python
# Signal-to-Noise Ratio (SNR) in dB for the simulation
snr = 24
# Input string to be transmitted. 
input_string = "Communications Systems Communication Channel Simulation and Report. In this project, you will complete a simulation of a wireless communication system. This builds on the tasks in Lab 1 and the lectures cover all the background theory for each part of the task."
# input_string = "a?"
```

To implement dynamic QAM, we want to choose different QAM orders (QAM4, QAM16, or QAM64) depending on SNR.
Here, for snr = 24, it would normally select QAM64 dynamically — but for demonstration purposes, we manually force it to QAM16.

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

Some derived values (mainly to make later calculations easier):

```python

sqrt_M = int(sqrt(M))
size_symbol = int(log2(M))

n_symbol = int(len(input_string) * (8 / size_symbol)) # number of QAM symbols to create

size_symbol, n_symbol

```

For QAM16 with the given message, size_symbol = 4 and n_symbol = 522.
The first means each symbol can carry 4 bits “in one go”; the second means this message needs 522 QAM symbols to transmit.

Next, we define two carriers:

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

Output:

```python
26.3
```

![QAM 载波](/img/QAM/QAM_carrier.png "Fig4: QAM carrier wave")


In the code, you’ll see a variable called rep_symbol.

It’s used below to “repeat” each input symbol and expand it into a time sequence — basically for better sampling and nicer plots.

And `f_s` is the sampling frequency used when plotting these waveforms.

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

Now let’s plot the modulated signals:

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

Output:

![QAM 调制后的 I 分量信号](/img/QAM/QAM_I_mod.png "Fig5: QAM modulated I comp.")

![QAM 调制后的 Q 分量信号](/img/QAM/QAM_Q_mod.png "Fig6: QAM modulated Q comp.")

![QAM 调制后的总信号](/img/QAM/QAM_I+Q_mod.png "Fig7: QAM modulated signal in total")

### QAM Constellation Diagram

To plot the constellation, we use the following variables and function:


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

Output:

```python
[1.88111148+2.08807572j 3.01975263+0.01366766j 2.05328757+1.12644641j 1.83682459+3.0908774j  2.00044805+2.97572787j 2.9585253 +0.84307519j 1.92937708+3.06050133j 1.07733372+1.0006585j  2.98315354+0.84013525j 2.08884394+1.86715759j]

```

![QAM16 星座图(发送前)](/img/QAM/QAM16_cons_b.png "Fig8: QAM16 constellation (send)")


---

## Path Loss Simulation: Hata Model

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

![Hata model](/img/QAM/Hata_model.png "Fig9: Hata model")

As we can see, the farther the distance, the larger the path loss.

## QAM Demodulation

To make demodulation easier, let’s first write a filter:

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

QAM demodulation and the corresponding waveforms:

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

![QAM 解调后的 I 分量信号](/img/QAM/QAM_I_demod.png "Fig10: QAM demod. I comp")

![QAM 解调后的 Q 分量信号](/img/QAM/QAM_Q_demod.png "图11: QAM demod. Q comp")

And the constellation diagram:

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

```python
(500, 526)
(526,)
```

![QAM16 星座图(解调后)](/img/QAM/QAM16_cons_a.png "Fig12: QAM16 constellation(demod.)")

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

```python
['01000011', '01101111', '01101101', '01101101', '01110101', '01101110', '01101001', '01100011', '01100001', '01110100']
Communications Systems Communication Channel Simulation and Report. In this project, you will complete a simulation of a wireless communication system. This builds on the tasks in Lab 1 and the lectures cover all the background theory for each part of the task.

```

## Bit Error Rate (BER)

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

```python
[0, 1, 0, 0, 0, 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1]
[0, 1, 0, 0, 0, 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1]
BER is 0.0
find error at []
```

Perfect!
