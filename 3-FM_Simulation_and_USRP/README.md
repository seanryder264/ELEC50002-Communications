# Lab 3: FM Simulation and USRP

## Exercise 1: FM Modulator

In FM modulation, information about the message signal is contained in the frequency of the modulated carrier waveform. The generalized function for an FM signal is:

$$ s(t) = A_c \cos(2\pi f_c t + \theta_m(t)) $$

where

$$ \theta_m(t) = 2\pi k_f \int_0^t m(\tau) d\tau, $$

- $m(t)$ is the message signal.
- $k_f$ represents the **frequency sensitivity**.

The instantaneous frequency is:

$$ f_i = f_c + k_f m(t) $$

In particular, we will use the following equivalent form for the FM signal:

$$ s(t) = A_c \cos(2\pi f_c t) \cos(\theta_m(t)) - A_c \sin(2\pi f_c t) \sin(\theta_m(t)) $$

In the following figure, you can see an example of a message signal on the left and an FM modulated waveform on the right. Note the change in the frequency of the modulated signal in time.

![image](https://github.com/user-attachments/assets/5b67afa1-258e-4a7c-bd82-7afb35039c61)

The **frequency deviation** is defined as:

$$ \Delta_f = k_f m_f, $$

where $m_f = \max|m(t)|$.

The **modulation index** (deviation ratio) is defined as:

$$ \mu = \frac{\Delta_f}{B}, $$

where \( B \) is the **message bandwidth**.

# Tasks

-  Set the parameters as in the following table and observe the changes in the modulated signal.

| Parameter                 | Value               |
|---------------------------|---------------------|
| **Message signal frequency** | 1 kHz               |
| **Carrier signal frequency** | 10 kHz              |
| **Sample rate**            | 200k               |
| **Samples**               | 1k                 |
| **k_f**                   | [500, 2000, 5000]  |
| **Message signal amplitude** | 1                 |
| **Carrier signal amplitude** | 1                 |

-  Explain the resulting changes in the FM signal and its **PSD** as you change **Δf**. Add the graphs of the FM signal and its **PSD** to your logbook for different **Δf**.

k = 500 :

![image](https://github.com/user-attachments/assets/17b2b5f4-02c4-42b2-b391-a4591b7c3b87)

k = 2000 :

![image](https://github.com/user-attachments/assets/1760d515-e254-4fa1-a4e7-a175c011d971)

k = 5000 :

![image](https://github.com/user-attachments/assets/d86b0017-3194-4f9c-8361-bca4c298bd07)

Block Diagram of FM_Modulator.gvi:

![image](https://github.com/user-attachments/assets/06752a0d-1838-4b12-a4f4-a3573770bcf0)

## Exercise 2: FM Demodulator
One way of demodulating the FM signal is by means of the so-called frequency discrimination, which 
is obtained by: 
1. Transferring the changes in instantaneous frequency into changes in the amplitude of the output 
signal, by differentiating the FM signal. The output of the differentiator is an AM+FM modulated 
signal.
2. Demodulate the AM+FM signal by using an envelope detector (as in Exercise 1b of Lab 2).

**Instructions:**
1. Create a new VI and name it “FMDemodulator.gvi”.
2. Apply the Derivative x(t) found in the function palette on the FM signal. Remember, as you did with 
the integral function, to connect your value of dt to this block to scale appropriately.
3. After the differentiation, the remaining process is very similar to envelope detection in AM 
demodulation. Use exactly the same modules, and you will obtain the message signal. Remember 
to scale the amplitude appropriately.
4. Create a sub-VI and produce two outputs: demodulated signal in time, and demodulated signal 
PSD.

**Tasks:**
1. Explain briefly the mathematical theory behind this demodulation technique.
   
   **Frequency Modulation (FM) Signal:**
   
   $$s_{FM}(t)=A\cos(2\pi f_ct+\Delta f \sin (2\pi f_mt))$$
   
   **Differentiating the FM signal:**
   
   $$s'(t)=A(2\pi f_c+2\pi \Delta f f_m \cos(2\pi f_m t)) \sin(2\pi f_c t + \Delta f \sin(2\pi f_m t))$$
   
   **Envelope Detection (AM Demodulation):**
   
   $$y(t)\propto |s'(t)|$$
   
2. Add the block diagram to your logbook

![image](https://github.com/user-attachments/assets/4f984c4d-b8aa-49db-9c3d-72f8f73dcdc7)

## Exercise 3: FM Simulation
In this part all the sub-VI’s that you have prepared will be used in a single VI, and you will be able to
observe the entire process of FM modulation and demodulation

**Instructions:**
1. Create a new file and name it “FMTopLevel.gvi”.
2. Add all the necessary VIs, and make the necessary connections.
3. Include all the setup in a while loop, and add a stop button.
4. Use the following values:

|     |     |
| --- | --- |
| Message signal frequency | 1kHz |
| Carrier signal frequency | 10kHz |
| Sample frequency | 200kHz |
| Samples | 1k |
| Butterworth low-pass filters | Order:  5 Cut-off : frequency: 1400Hz |
| Message signal amplitude | 2 |
| Carrier signal amplitude | 2 |
| $k_f$ | 1000 |

Tasks:
1. Add the block diagram to your logbook.

![image](https://github.com/user-attachments/assets/f52467e5-93e2-48ea-8322-e14edd66a17c)

2. Set the parameters as in the above table. Observe the demodulated signal and attach the plots of 
the FM signal and the demodulated signal with their corresponding PSDs to your logbook (for the 
demodulated signal, you should also provide a zoomed version, without the initial transient, so that it is 
easy to observe the amplitude of the message).

![image](https://github.com/user-attachments/assets/505141e7-ebe9-435e-8cd2-5f34a093e1c4)

## Exercise 4: FM Communications via USRP
In this exercise, FM communication via USRP will be implemented. Instead of building the code yourself,
you are given a file “FM-TxRx.gvi”. The code in the file should look familiar, but with only a slight
difference: you only have one VI this time. This is no big deal since you could have also combined the AM
communication VIs into a single one. LabView has the ability to execute two (or more) while loops
simultaneously; and thus there is no difference between the two approaches.

**Instructions:**
1. Have a detailed look at the code and compare it with the FM receiver/transmitter developed in the 
previous exercises. Run the VI, adjusting the parameters to reasonable values, and make sure you are 
able to transmit and receive successfully.

![image](https://github.com/user-attachments/assets/cdbccd62-9430-4a9b-b80a-23927127b3df)

### Exercise 4a: Carson’s rule
The bandwidth of an FM signal is difficult to calculate analytically. In the 1920s J.R. Carson provided a 
rule of thumb for approximating the bandwidth:

$$B_{FM}=2(\Delta_f+B)$$

where $B_{FM}$ is the bandwidth of the FM signal, $\Delta_f$ is the frequency deviation of the signal, and $B$ is the 
message bandwidth

**Instructions:**
1. The spectrum shown in the “FM spectrum” graph is identical to the power spectrum of the actual 
transmitted FM signal, except that the carrier will appear at 0 Hz, with the lower sideband on the 
negative-frequency side, and the upper sideband on the positive-frequency side. 
2. To obtain the classic textbook FM spectrum, set the message as a single tone at 1 kHz. Run the 
transmitter and obtain power spectra of the transmitted signal for frequency deviations of 1 kHz, 5 
kHz, and 30 kHz

**Task:**
1. Add the power spectrum graph for each of these cases into your logbook. Be sure to scale the 
horizontal axis so that each spectrum is visible clearly. Annotate your spectra to show the FM signal 
bandwidth (Carson’s rule)

**$\Delta_f = 1kHz$:**

![image](https://github.com/user-attachments/assets/58c3db9f-cfcb-4193-a530-0d31f07672b6)

$B_{FM} = 10$


**$\Delta_f = 5kHz$:**

![image](https://github.com/user-attachments/assets/8497d6b8-70c7-4e5e-807b-b9b045e2018a)

$B_{FM} = 14k$

**$\Delta_f = 30kHz$:**

![image](https://github.com/user-attachments/assets/972b5119-a54f-434b-b7e7-2bae123a302e)

$B_{FM} = 70k$
