# Lab 4: Binary Phase Shift Keying (BPSK) Via USRP

From the Lab Instructions: 
>In phase-shift keying (PSK) modulation, information is encoded on the phase of the transmitted carrier rather than on its amplitude (ASK) or frequency (FSK). In binary phase-shift keying (BPSK) there are two phase values, $0^\circ$ or $180^\circ$, which means that an unmodified carrier is transmitted to represent one binary data value while an inverted carrier is transmitted to represent the other. BPSK is optimum among binary modulation schemes in achieving the lowest average power for a given target bit error ratio (BER).

### Exercise 1: BPSK Transmitter

In this exercise we make a Binary Phase Shift Keying (BPSK) Transmitter. Which encodes encodes binary information into $0^\circ$ or $180^\circ$, dependent on zero or one. In this scheme the bits are transmitted as pulses, and to limit interference between bits (or symbols) we choose the shape of the polse to have a rapid specral roll off (how quickly the power diminishes outside its main bandwidth). We will achieve this with a root-raised-cosine filter

In the Lab Instructions this is explained mathematically:
>The BPSK signal is given by: 

>$$
A\, g_{\text{TX}}(t) \cos\Bigl(2\pi f_c t + \theta(t)\Bigr).
$$

>In this equation,  
>- $A$ is a constant corresponding to the transmitted power level,  
>- $g_{\text{TX}}(t)$ is a fixed pulse shape,  
>- $f_c$ is the carrier frequency, and  
>- $\theta(t)$ takes the value of either $0^\circ$ or $180^\circ$ to carry the desired information.

>Note that we can also write the equation as: 

>$$
\pm A\, g_{\text{TX}}(t) \cos(2\pi f_c t),
$$

>where the plus sign corresponds to $\theta = 0^\circ$ and the minus sign to $\theta = 180^\circ$.

Implementing this we multiplied the output of *MT Generate Bits (PN Order - Fibonacci)* by 2 then subtracted 1 
![image](https://github.com/user-attachments/assets/6d2aadb9-c2a6-44eb-8825-904e0ac2f114)
to map the bits to symbols as such:
 | Bit Value | Phase ($\theta$) | Symbol |
   |-----------|---------------------|--------|
   | 0         | $0^\circ$         | -1     |
   | 1         | $180^\circ$       | +1     |

We then added the frame header. Next we upsample the signal, so that the symbols are seperated by a number of zeros. Running it through a root raised cosine filter, makes each symbol have a pulse shape with a rapid spectal roll off (preventing interference). Then convolving it we get a waveform transmitting infomation without interference.

![image](https://github.com/user-attachments/assets/8e89d2b3-efca-4f27-a4bf-0f924d822e08)

finaly we normailise so that the amplitude is 1 maximum. The USRP converts the signal from digital to analogue and multiplies by the carrier $\cos(2\pi f_c t)$

From the lab instructions: 

>#### Pilots and Channel Estimation

>- The **Insert Pilots** function inserts a specific sequence of symbols known to the receiver for estimating the channel transfer function.
>- At the receiver, you will later use the **Channel Estimator** to correct for phase and amplitude errors introduced by the channel.

Setting the values to the following:

- **Carrier frequency:** 400 MHz  
- **IQ Rate:** 200 kHz (this sets the value of $1/T_s$)  
- **Gain:** 0 dB  
- **Active Antenna:** TX1  
- **Symbol rate:** 10,000 symbols/s  
- **Message Length:** 1000 bits  

Calculating number of samples per symbol, $L$, based on the given values:

$$
L = \frac{\text{IQ rate}}{\text{Symbol rate}} = \frac{200,000 \text{ samples/s}}{10,000 \text{ symbols/s}} = 20.
$$
  
We observe:

 ![image](https://github.com/user-attachments/assets/6b0e76e8-afba-4279-8dfe-568532246e6b)


![image](https://github.com/user-attachments/assets/47da2774-05ec-42f4-a871-50616db1e3c9)

It looks fairly low in magnitude with a -40dB bandwidth of about 15kHz, and a more typical -3dB badwidth of about 8-10kHz 

https://ntrs.nasa.gov/api/citations/20120008631/downloads/20120008631.pdf
>The rolloff factor is a measure of the excess bandwidth of the filter, i.e., the bandwidth
occupied beyond the Nyquist bandwidth of 1/2T, where 1/T is symbol rate.
sp the Nyquist bandwidth is 10,000/2 = 5000Hz

This is the minimum bandwidth required to avoid intersymbol interference
reading further from nasa it seems ideal for the bandwidth should approach 5 for efficiency but possibly at the sacrifice of clarity of symbols
so a bandwidth of 8-10kHz is good effeciency wise, but might be unclear with a weaker signal.

![image](https://github.com/user-attachments/assets/1f54c948-81c5-493a-bfb9-df19184deb6b)

WiRRC filtering, the waveform appears smooth, with gradual transitions between symbols, 
while rectangular pulses create a blocky signal with abrupt jumps.

RRC filtering concentrates most of the signal's energy near the center frequency, rolling off quickly to minimize interference. 
rectangular pulses result in a wider spread of energy, producing strong ripples and inefficient bandwidth usage.

A smooth waveform and a well-contained spectrum indicate effective RRC filtering, 
whereas sharp steps and a broader, more rippled spectrum suggest the effects of unfiltered pulses

In short, **RRC filtering** keeps the signal more contained within its bandwidth and suppresses side lobes,  
whereas **rectangular pulses** lead to a broader spectrum with stronger side lobes.


---

## Exercise 2: BPSK Receiver

In this exercise we make a reciever for the modulator we made

From the Lab Instructions:

>The received BPSK signal is given by:

>$$
r(t) = \pm D\, g_{\text{RX}}(t) \cos\Bigl(2\pi f_c t + \phi\Bigr),
$$

>where  
>- $D$ is a constant (usually much smaller than $A$ in the transmitted signal), and  
>- $\phi$ represents the phase difference between the transmitter and receiver carrier oscillators.

>If the receiver’s carrier oscillator is set to the same frequency as the transmitter’s, the USRP receiver performs most of the demodulation. The **Fetch Rx Data** function provides a train of output samples, each given by an expression similar to:

>$$
\tilde{r}[n] = \pm \frac{D}{2}\, g_{\text{RX}}[n] e^{j\phi}.
$$

>The sampling rate $1/T_s$ is set by the receiver’s IQ rate, which provides $M$ samples every $T$ seconds (with $1/T$ being the symbol rate).

### Steps to Obtain the Transmitted Signals

1. **Channel Estimation:**  
   Remove phase ambiguity caused by the channel and the USRP oscillators. The channel estimator reads the received pilots (inserted in the Tx signal) and performs a least-square estimation of the channel transfer function. Its inverse is then applied to the received signal to remove the phase offset.

2. **Matched Filtering:**  
   Use a root-raised-cosine filter as the matched filter. Its impulse response, $/text{g^*_{RX}[n]}$, is matched to the transmitted pulse shape $g_{\text{TX}}[n]$. This filter maximizes the signal-to-noise ratio in the presence of additive noise.

3. **Pulse Synchronization:**  
   The output of the matched filter is an analog baseband signal that must be sampled once per symbol time (every $T$ seconds). Due to filtering, propagation delays, and channel distortion, it is necessary to determine the optimal sampling time. Use the **PulseAlign** sub-VI provided for this alignment.

4. **Sampling:**  
   Sample the aligned baseband waveform using the **Decimate** function at index 0 and every $T$ seconds thereafter.

5. **Detection:**  
   Examine each sampled value to determine whether it represents a symbol of +1 or -1.

6. **Symbol Mapping:**  
   Convert the detected symbol values to bits.

### Constructing the BPSK Receiver

1. **Frame Synchronization (Complex):**  
   Add the **FrameSync (Complex)** sub-VI after the output of **Fetch Rx Data**. Connect the output to the “Sampled Input” terminal and leave remaining inputs unconnected. This module locates the header marking the start of the Tx signal and discards any preceding symbols.

2. **Channel Estimation:**  
   Connect the “Aligned Samples” output from **FrameSync (Complex)** to the **Channel Estimator** function to eliminate phase offset.

3. **Matched Filtering:**  
   Use the **MT Generate Filter Coefficients** function (as in the transmitter) with the following settings:  
   - *Modulation Type:* PSK  
   - *Pulse Shaping Filter:* “Root Raised Cos”  
   - *Matched Samples per Symbol:* $M$ (calculated from the IQ rate $1/T_s$ and the symbol rate $1/T$)

4. **Convolution:**  
   Extract the real part of the channel-estimated signal using the **Complex to Real and Imaginary** function and convolve it with the matched filter coefficients using the **Convolution** function.

5. **Power Spectrum Observation:**  
   Connect the convolution output to the “Y” terminal of the **Cluster Properties**. Set the “dt” terminal as determined in step 7 of the transmitter section.

6. **Pulse Alignment:**  
   Place the **PulseAlign (Real)** sub-VI on your block diagram. Wire the convolution output to the “input signal” and connect the $M$ samples per symbol to the “receiver sampling factor” terminal.

7. **Sampling via Decimation:**  
   Once the baseband waveform is aligned, use the **Decimate (single shot)** function to sample the signal. (The decimating factor is $M$.)

8. **Frame Synchronization (Real):**  
   Immediately after decimation, add a **FrameSync (Real)** sub-VI using the same inputs and outputs as in step 1.

9. **Detection and Bit Mapping:**  
   Use a For Loop to examine each element of the “Aligned Samples” to decide if it is a 1 or 0.  
   *Hint:* Use the **Greater Than 0?** function inside the loop, and then convert the Boolean output to integers using a **Boolean to Integer** function.

10. **Extract Output Bits:**  
    Connect the output array to the ‘array’ input of an **Array Subset** function (set the ‘index’ to 0 and ‘length’ to the message length control). Display the result as ‘Output bits’ on the receiver front panel.

11. **Bit Error Ratio (BER) Measurement:**  
    Automate the BER measurement using the **MT Calculate BER** function. In its Configure ribbon, select “PN Fibonacci” and set the “BER trigger threshold” to 0.4. Connect indicators to the “BER” and “trigger found?” outputs, and feed the “Output bits” to its input. (If using a random seed with MT Generate Bits, consider coding your own BER calculator.)

12. **Receiver Settings:**  
    Set the following control values:
    
    - **Carrier frequency:** 400 MHz  
    - **IQ Rate:** 200 kHz (this sets $1/T_s$)  
    - **Gain:** 0 dB  
    - **Active Antenna:** RX2  
    - **Symbol rate:** 10,000 symbols/s  
    - **Message Length:** 1000 bits  

**Tasks:**

- Run the transmitter and receiver several times. Record the observed BER values.

![image](https://github.com/user-attachments/assets/0a7f9a87-6bdc-4c86-a84b-71b0f8b902a2)

- Set the Rx and Tx gains as indicated in the table below, run each setting 5 times, and record both the individual and average BER for each configuration.

| Tx Gain (dB) | Rx Gain (dB) |
|--------------|--------------|
| 0            | 0            |
| -35          | -15          |
| -37          | -15          |
| -40          | -15          |

| Data                                         | Average BER  |
|----------------------------------------------|-----------|
| 0, 0, 0, 0, 0                             | 0       |
| 0.456954, 0.489496, 0.484328, 0.47548, 0.483131 | 0.4733  |
| 0.454142, 0.476731, 0.465487, 0.493868, 0.488525 | 0.4825 |
| 0.490375, 0.475501, 0.50431, 0.473515, 0.498864  | 0.4889 |


---

## Exercise 3: Differential Phase Shift Keying (DPSK)

In DPSK, the transmitter sends the difference between adjacent bits rather than the bits themselves. The following table illustrates how the difference is obtained for various symbol pairs. Notice that the encoded output sequence is obtained by

$$
b_n = b_{n-1} \times a_n.
$$

The sequences are as follows:

| Description                                 | Sequence                         |
|---------------------------------------------|----------------------------------|
| Information symbols $\{a_n\}$             | 1, -1, -1, 1, -1, -1, 1, 1        |
| Intermediate sequence $\{b_n\}$            | 1, 1, -1, 1, 1, -1, 1, 1          |
| Differentially encoded sequence $\{b_n\}$ | 1, 1, -1, 1, 1, -1, 1, 1, 1        |
| Output of lowpass filter (polarity)         | +, -, -, +, -, -, +, +             |
| Decision                                    | 1, -1, -1, 1, -1, -1, 1, 1          |

### Constructing the DPSK System

1. **Duplicate the BPSK System:**  
   Make new copies of your BPSK transmitter and receiver to create the DPSK system.

2. **Differential Encoder (Transmitter):**  
   Design and add a differential encoder to the BPSK transmitter. Insert it after the **AddFrameHeader** function (the frame header is also differentially encoded).

3. **Modify Transmitter Functions:**  
   Remove the **Insert Pilots** and **AddFrameHeader (Complex)** functions that were placed before the **niUSRP Write Tx Data** module. Instead, convert the output of the **Quick Scale 1D** to complex values and connect it directly to the **niUSRP Write Tx Data** module.

4. **Modify Receiver Functions:**  
   At the receiver, remove the **Channel Estimator** and **FrameSync (Complex)** functions that were placed after the **niUSRP Fetch Rx Data** module. Connect the received data directly to the **Convolution** function.  
   **Remark:** Since the received data is complex, ensure you use the appropriate functions (e.g. **Convolution (CDB)**, **PulseAlign (Complex)**, and **FrameSync (Complex)**) in subsequent stages.

5. **Differential Decoder (Receiver):**  
   Design and add a differential decoder to the BPSK receiver. Place it immediately after the **Decimate** function (the receiver’s sampler) and before **FrameSync (Complex)**.  
   **Remark:** Since the received data is complex-valued at this stage, design your decoder to form the product of the current sample and the complex conjugate of the previous sample, then take the real part of the result.

**Tasks:**

- Add the block diagram of your DPSK system to your logbook.
![image](https://github.com/user-attachments/assets/dfd49a78-1f2a-4eed-91f1-fbd38e6436b1)
![image](https://github.com/user-attachments/assets/32e02b4c-8f51-4344-ae5c-6fac29d20d16)

- Run the code several times using the same initial configuration as the BPSK system, and explain your observations on the BER performance.

| Tx Gain (dB) | Rx Gain (dB) |
|--------------|--------------|
| 0            | 0            |
| -35          | -15          |
| -37          | -15          |
| -40          | -15          |

| Data                                         | Average BER  |
|----------------------------------------------|-----------|
| 0, 0, 0, 0, 0                             | 0       |
| 0.456954, 0.489496, 0.484328, 0.47548, 0.483131 | 0.4779  |
| 0.454142, 0.476731, 0.465487, 0.493868, 0.488525 | 0.4753 |
| 0.490375, 0.475501, 0.50431, 0.473515, 0.498864  | 0.4889 |
  
We did not find much change in preformance, although very accurate at high gain, it quickly falls off.
This could potentially be down to the narrow bandwidth of our transmitted signal. Im not clear which one we would expect to be better:
- BPSK relies on a good channel estimator to guess its carrier wave and transfer function
- With DPSK there will be carry over bit errors if one bit is read wrong 

---

## Exercise 4: Error Correction Coding (Bonus)

In this exercise, forward error correction (FEC) is performed by adding redundancy to the transmitted information. In FEC, the transmitter sends each data bit three times (forming a triplet). Due to channel noise, the receiver might observe different triplets and then decode the bit using a majority vote decoder as shown below:

| Triplet Received | Bit Decoded |
|------------------|-------------|
| 000              | 0           |
| 001              | 0           |
| 010              | 0           |
| 011              | 1           |
| 100              | 0           |
| 101              | 1           |
| 110              | 1           |
| 111              | 1           |

An error in any one of the three samples is corrected by the majority vote. To implement FEC, the transmitter sends each bit of the message three times consecutively. The receiver then demodulates and decodes these bits using the table above.

### Steps to Implement FEC

1. **Duplicate and Rename:**  
   Duplicate your BPSK.gvi file from the previous exercise and rename it as BPSK_FEC.gvi.

2. **FEC Encoder (Transmitter):**  
   Design an FEC encoder that triples each message bit from *MT Generate Bits (PN Order - Fibonacci)*. For example, if the original bits are 011, they should be converted to 000111111.

3. **Adjust Receiver Length:**  
   At the receiver, modify the “length” node of the **Array Subset** function to be three times the message length.

4. **FEC Decoder (Receiver):**  
   Design an FEC decoder that decodes the bits from the **Array Subset** function output according to the table above.

5. **Connect to BER Measurement:**  
   Connect the output of your FEC decoder to the **MT Calculate BER** function (or your own BER calculator).

**Tasks:**
![image](https://github.com/user-attachments/assets/9dc07ba5-355a-4aea-b3d1-0906fe826adf)
![image](https://github.com/user-attachments/assets/5902829a-71c3-487e-9495-ee19bd813404)

- Using the same system configuration as the BPSK system without FEC, set the Rx and Tx gains to the following settings, and run each setting 5 times to obtain an average BER:

  | Tx Gain (dB) | Rx Gain (dB) |
  |--------------|--------------|
  | 0            | 0            |
  | -35          | -15          |
  | -37          | -15          |
  | -40          | -15          |

| Data                                         | Average BER |
|----------------------------------------------|-----------|
| 0, 0, 0, 0, 0 | 0 |
| 0.148751, 0.489818, 0.495288, 0.141333, 0.475138 | 0.3500656 |
| 0.490566, 0.488688, 0.490482, 0.489362, 0.497343 | 0.4912882 |
| 0.497446, 0.493435, 0.473259, 0.482185, 0.494541 | 0.4881732 |


- Compare your BER results from Exercise 2 with those obtained here. Has FEC improved the BER?

It has for the -30 Tx gain however no change is obvious in the other three 
- Discuss the trade-offs involved in the error correction coding system. What are the advantages and disadvantages?

More robust transmission -> no need for retransmittion
Increased Bandwidth to maintain same symbol rate -> the bit rate must increase by a factor of 3 and so the bandwidth must also triple this decrease the power effiecency of the system 
Conversly increased latency -> if the rate is not increased then the information will take more time to transmit

---

## Appendix: NI USRP Hardware Diagram

The NI USRP connects to a host PC to create a software defined radio. Incoming signals at the SMA connector inputs are mixed down using a direct-conversion receiver to baseband I/Q components, which are sampled by an analog-to-digital converter (ADC). The digitized I/Q data follows parallel paths through a digital down-conversion (DDC) process that mixes, filters, and decimates the input signal to a user-specified rate. The down-converted samples are then passed to the host computer.

For transmission, baseband I/Q signal samples are synthesized by the host computer and fed to the USRP at a specified sample rate over Ethernet, USB, or PCI Express. The USRP hardware interpolates the incoming signal to a higher sampling rate using a digital up-conversion (DUC) process and then converts the signal to analog with a digital-to-analog converter (DAC). The resulting analog signal is then mixed up to the specified carrier frequency.

The steps above are illustrated in the following figure:

![image](https://github.com/user-attachments/assets/96ecd566-f32e-45f7-b895-23aa0bf0a485)

**Figure A1:** USRP’s internal circuit.
