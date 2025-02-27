# Lab 4: Binary Phase Shift Keying (BPSK) Via USRP

In phase-shift keying (PSK) modulation, information is encoded on the phase of the transmitted carrier rather than on its amplitude (ASK) or frequency (FSK). In binary phase-shift keying (BPSK) there are two phase values, $0^\circ$ or $180^\circ$, which means that an unmodified carrier is transmitted to represent one binary data value while an inverted carrier is transmitted to represent the other. BPSK is optimum among binary modulation schemes in achieving the lowest average power for a given target bit error ratio (BER).

### Exercise 1: BPSK Transmitter

The BPSK signal is given by: 

$$
A\, g_{\text{TX}}(t) \cos\Bigl(2\pi f_c t + \theta(t)\Bigr).
$$

In this equation,  
- $A$ is a constant corresponding to the transmitted power level,  
- $g_{\text{TX}}(t)$ is a fixed pulse shape,  
- $f_c$ is the carrier frequency, and  
- $\theta(t)$ takes the value of either $0^\circ$ or $180^\circ$ to carry the desired information.

Note that we can also write the equation as: 

$$
\pm A\, g_{\text{TX}}(t) \cos(2\pi f_c t),
$$

where the plus sign corresponds to $\theta = 0^\circ$ and the minus sign to $\theta = 180^\circ$.

We will assume that a new pulse is transmitted every $T$ seconds so that the symbol rate (symbols per second) is $1/T$. For a binary scheme such as BPSK, the bit rate is the same as the symbol rate. Since the pulse $g_{\text{TX}}[n]$ does not carry information, its shape can be chosen to satisfy other criteria. In our case, we desire a pulse shape that provides a rapid spectral roll-off to minimize inter-symbol interference (e.g. a “root-raised-cosine” filter).

### Steps to Form a BPSK Signal

1. **Symbol Mapping:**  
   The input data arrives as a stream of bits. The *MT Generate Bits VI* produces an array of bits. For BPSK the bits are mapped as follows:

   | Bit Value | Phase ($\theta$) | Symbol |
   |-----------|---------------------|--------|
   | 0         | $0^\circ$         | -1     |
   | 1         | $180^\circ$       | +1     |

   **Remark:** The USRP requires a complex-valued input. Make sure to convert the symbols to complex numbers ($+1 + j0$) before sending them as input to the USRP.

2. **Upsampling:**  
   As a first step toward pulse shaping, interpolation is required. We place $L - 1$ zeros after each symbol. This produces a sample interval of 

   $$T_s = \frac{T}{L}$$

   or a sample rate of 

   $$\frac{1}{T_s} = \frac{L}{T}$$

3. **Pulse Shaping:**  
   If the upsampled signal is applied to a filter whose impulse response $g_{\text{TX}}[n]$ is a root-raised-cosine pulse, then each symbol at the filter output will be represented by such a pulse. This pulse shape has a very rapid spectral roll-off so that consecutive transmitted symbols do not interfere with each other.

![image](https://github.com/user-attachments/assets/8e89d2b3-efca-4f27-a4bf-0f924d822e08)


4. **Modulation:**  
   The signal, of the form (+1+j0), $g_{\text{TX}}[n] $, can be sent directly to the USRP. The USRP transmitter will perform the DAC and the multiplication by the carrier $\cos(2\pi f_c t)$.

Following these steps, construct a BPSK transmitter using the guidelines below:

1. **Generate Bits:**  
   Add an *MT Generate Bits (PN Order - Fibonacci)* from the function palette to your template. This creates a pseudorandom sequence of bits. You can control the total number of bits. Note that by default, MT Generate Bits produces the same sequence every run (useful for debugging). To generate a different sequence on each run, connect a random number to the seed in input terminal.

2. **Symbol Mapping and Frame Header:**  
   Convert the integers 0 and 1 from the MT Generate Bits to doubles -1 and +1, respectively. Then, add the **AddFrameHeader** function provided.

3. **Upsampling:**  
   Upsample the array of symbols using the **Upsample** function from the palette. Set the symbol rate ($1/T$) to 10,000 symbols/s and the IQ rate ($1/T_s$) to 200,000 Samples/s. Use these values to calculate the upsampling factor $L$.

4. **Pulse-Shaping Filter Generation:**  
   Use the **MT Generate Filter Coefficients** function to generate the pulse-shaping filter. Set the inputs as follows:  
   - *Modulation Type:* PSK  
   - *Pulse Shaping Samples per Symbol:* $L$  
   - *Pulse Shaping Filter:* “Root Raised Cosine”

5. **Filtering:**  
   Connect the “pulse shaping filter coefficients” output to the “Y” input of a **Convolution** function. Also, connect the output of the **Upsample** function to the “X” input of the **Convolution** function.

6. **Normalization:**  
   Normalize the amplitude of your filtered message signal to a maximum absolute value of 1. The provided **Quick Scale 1D** function finds the maximum of the absolute value.

7. **Waveform Observation:**  
   To observe the baseband waveform, connect the normalized signal to the “Y” input of a **Build Waveform** function. Use the IQ rate to determine the appropriate value for the “dt” terminal, and add an indicator on the panel.

8. **Conversion to Complex Values:**  
   Convert the normalized signal to complex values using the **Real and Imaginary to Complex** function.

9. **Power Spectrum:**  
   To view the baseband power spectrum, connect the normalized signal to the “Y” terminal of the **Cluster Properties** and set the “dt” terminal using the same value from step 7. (The Cluster Properties function is pre-connected to an FFT Power Spectrum for 1 channel (CDB); simply place the “Baseband Power Spectrum” graph on your panel.)

#### Pilots and Channel Estimation

- The **Insert Pilots** function inserts a specific sequence of symbols known to the receiver for estimating the channel transfer function.
- At the receiver, you will later use the **Channel Estimator** to correct for phase and amplitude errors introduced by the channel.

10. **Insert Pilots:**  
    Connect the complex values to the **Insert Pilots** function provided, and then add the **AddFrameHeader (Complex)** function immediately after.

11. **Data Transmission:**  
    Connect the output signal to the “data” node of the **niUSRP Write Tx Data (CDB)** function in the template. Ensure the input to this function is a complex signal.

12. **System Settings:**  
    Assign the following values to the corresponding controls and run the code:
    
    - **Carrier frequency:** 400 MHz  
    - **IQ Rate:** 200 kHz (this sets the value of $1/T_s$)  
    - **Gain:** 0 dB  
    - **Active Antenna:** TX1  
    - **Symbol rate:** 10,000 symbols/s  
    - **Message Length:** 1000 bits  

**Tasks:**

- Calculate the number of samples per symbol, $L$, based on the given values.

$$
L = \frac{\text{IQ rate}}{\text{Symbol rate}} = \frac{200,000 \text{ samples/s}}{10,000 \text{ symbols/s}} = 20.
$$
  
- Add the block diagram and the front panel graphs to your logbook (adjust plots where necessary) and explain your observations.

![image](https://github.com/user-attachments/assets/24741926-3604-4206-bb9f-aa3bc4e7b965)

![image](https://github.com/user-attachments/assets/fdf30a46-6c62-46c3-b0ee-83df745237e6)


- From the spectrum plot, measure the “main lobe” bandwidth of the transmitted signal. Then change the pulse shaping filter control to “none” to create rectangular pulses, run the transmitter again, and compare the spectrum with that for root-raised-cosine pulses. Finally, revert the pulse shaping filter setting to “Root Raised.”

![image](https://github.com/user-attachments/assets/91982a35-b6de-4d7a-9dcd-89604cdd4794)

- Compare the main lobe bandwidth and the spectral roll-off for root-raised-cosine pulses versus rectangular pulses.

---

## Exercise 2: BPSK Receiver

The received BPSK signal is given by:

$$
r(t) = \pm D\, g_{\text{RX}}(t) \cos\Bigl(2\pi f_c t + \phi\Bigr),
$$

where  
- $D$ is a constant (usually much smaller than $A$ in the transmitted signal), and  
- $\phi$ represents the phase difference between the transmitter and receiver carrier oscillators.

If the receiver’s carrier oscillator is set to the same frequency as the transmitter’s, the USRP receiver performs most of the demodulation. The **Fetch Rx Data** function provides a train of output samples, each given by an expression similar to:

$$
\tilde{r}[n] = \pm \frac{D}{2}\, g_{\text{RX}}[n] e^{j\phi}.
$$

The sampling rate $1/T_s$ is set by the receiver’s IQ rate, which provides $M$ samples every $T$ seconds (with $1/T$ being the symbol rate).

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
- Set the Rx and Tx gains as indicated in the table below, run each setting 5 times, and record both the individual and average BER for each configuration.

| Tx Gain (dB) | Rx Gain (dB) |
|--------------|--------------|
| 0            | 0            |
| -35          | -15          |
| -37          | -15          |
| -40          | -15          |

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
- Run the code several times using the same initial configuration as the BPSK system, and explain your observations on the BER performance.
- Compare the performance of BPSK and DPSK, and explain your findings.

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

- Using the same system configuration as the BPSK system without FEC, set the Rx and Tx gains to the following settings, and run each setting 5 times to obtain an average BER:

  | Tx Gain (dB) | Rx Gain (dB) |
  |--------------|--------------|
  | 0            | 0            |
  | -35          | -15          |
  | -37          | -15          |
  | -40          | -15          |

- Compare your BER results from Exercise 2 with those obtained here. Has FEC improved the BER?
- Discuss the trade-offs involved in the error correction coding system. What are the advantages and disadvantages?

---

## Appendix: NI USRP Hardware Diagram

The NI USRP connects to a host PC to create a software defined radio. Incoming signals at the SMA connector inputs are mixed down using a direct-conversion receiver to baseband I/Q components, which are sampled by an analog-to-digital converter (ADC). The digitized I/Q data follows parallel paths through a digital down-conversion (DDC) process that mixes, filters, and decimates the input signal to a user-specified rate. The down-converted samples are then passed to the host computer.

For transmission, baseband I/Q signal samples are synthesized by the host computer and fed to the USRP at a specified sample rate over Ethernet, USB, or PCI Express. The USRP hardware interpolates the incoming signal to a higher sampling rate using a digital up-conversion (DUC) process and then converts the signal to analog with a digital-to-analog converter (DAC). The resulting analog signal is then mixed up to the specified carrier frequency.

The steps above are illustrated in the following figure:

![image](https://github.com/user-attachments/assets/96ecd566-f32e-45f7-b895-23aa0bf0a485)

**Figure A1:** USRP’s internal circuit.
