# Lab 3: FM Simulation and USRP

## Exercise 1: FM Modulator

In this exersice we are going to make a FM modulator.

>Frequency Modulation is a process of encoding information on one carrier wave by changing its frequency. The frequency of the carrier wave is changed according to the frequency of the modulating signal. - geeksforgeeks.org

From the Lab Documents. 

---

The generalized function for an FM signal is:
 
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

---

Block Diagram of FM_Modulator.gvi:

![image](https://github.com/user-attachments/assets/06752a0d-1838-4b12-a4f4-a3573770bcf0)

| Parameter                 | Value               |
|---------------------------|---------------------|
| **Message signal frequency** | 1 kHz               |
| **Carrier signal frequency** | 10 kHz              |
| **Sample rate**            | 200k               |
| **Samples**               | 1k                 |
| **k_f**                   | [500, 2000, 5000]  |
| **Message signal amplitude** | 1                 |
| **Carrier signal amplitude** | 1                 |

The above are the values we used.

Changing the values of k we observed the following

k = 500 :

![image](https://github.com/user-attachments/assets/17b2b5f4-02c4-42b2-b391-a4591b7c3b87)

k = 2000 :

![image](https://github.com/user-attachments/assets/1760d515-e254-4fa1-a4e7-a175c011d971)

k = 5000 :

![image](https://github.com/user-attachments/assets/d86b0017-3194-4f9c-8361-bca4c298bd07)

- **Time-Domain Plots**
  he carrier wave is shifting between higher and lower frequencies more dramatically as $k_f$ increases  
  As $k_f$ increases, the rate at which the signal crosses zero fluctuates more rapidly—sometimes packed closely together, sometimes stretched out - but the amplitude stays constant.

- **PSD Plots**  
  As $k_f$ increases, the spectrum broadens around the carrier.  
  - For $k_f = 500$, the **PSD** is relatively narrow.  
  - For $k_f = 5000$, a much wider spread and more side lobes.

These observations confirm that **increasing $k_f$ (thus $\Delta f$)** directly **increases** the FM signal’s instantaneous-frequency swing **and widens its spectrum** in the frequency domain.

## Exercise 2: FM Demodulator

In this exercise we will create a _demodulator_. One method for demodulation is "_frequency descrimination_". The lab instructions explaining it as follows:
>One way of demodulating the FM signal is by means of the so-called frequency discrimination, which 
is obtained by: 
>1. Transferring the changes in instantaneous frequency into changes in the amplitude of the output 
signal, by differentiating the FM signal. The output of the differentiator is an AM+FM modulated 
signal.
>2. Demodulate the AM+FM signal by using an envelope detector (as in Exercise 1b of Lab 2).

We created the demodulator in “FMDemodulator.gvi”

---

We implement the demodulator using the _frequency descriminaton_ method mentioned prior, following the details given in the lab instructions:

![image](https://github.com/user-attachments/assets/f2761099-14d5-453a-a796-737251dabf71)

** Differentiation on the left

Then we created a Sub-VI for it with

Inputs:
- FM Signal
-  Order (for Butterworth filter)
-   Cutoff Frequency (for Butterworth filter)
  
Outputs:
- Demodulated Message Signal
- Demodulated Message PSD

### Mathematical Theory of Frequency Discrimination
   
   **Frequency Modulation (FM) Signal:**
   
   $$s_{FM}(t)=A\cos(\omega_c t+ k_f\int_0^t m(\tau)d\tau)$$
   
   **Differentiating the FM signal:**
   
   $$s'(t)=-A(\omega_c + 2 \pi k_f m(t)) \sin(\omega_c t + 2 \pi k_f \int_0^t m(\tau)d\tau)$$
   
   **Envelope Detection (AM Demodulation):**
   
   $$y(t)\propto |s'(t)|$$

   (similar to the AM signal: $A_c(1 + m(t)) \cos( \omega_c t )$ )

![image](https://github.com/user-attachments/assets/34472b36-2421-4262-bee0-fce93a225ed1)
> https://wiki.analog.com/_detail/university/courses/electronics/fmd_f5_3.png?id=university%3Acourses%3Aelectronics%3Aelectronics_lab_fm_detectors


## Exercise 3: FM Simulation

In this exercise we will put together the modules we made in exercise 1 and 2 to observer the whole process of frequency modulation and demodulation

We connected the inputs, VI's and ouputs, then encompased the whole thing in a while loop so we could update the inputs and get outputs without having to stop and run each time. Ultimately saving the module in "FMTopLevel.gvi".

![image](https://github.com/user-attachments/assets/f52467e5-93e2-48ea-8322-e14edd66a17c)

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

Setting the input parameters to the ones in the table above. We observed the demodulated singal:

![image](https://github.com/user-attachments/assets/505141e7-ebe9-435e-8cd2-5f34a093e1c4)

We had to remove the first 0.001 seconds due to transients. That which is likely caused by the initial jump in frequency from starting and the Butterworth low-pass filter taking time to settle. We observe that the peak frequency of the demodulated signal is 1k matching that of the message signal. The demodulated signal is flipped and about 20000x bigger than the input message signal. This can be explained by the demodulation technique:
$$s'(t)=-A_c(\omega_c + 2 \pi k_f m(t)) \sin(\omega_c t + 2 \pi k_f \int_0^t m(\tau)d\tau)$$

The amplitude of the demodulated signal will be 
$$-A_c 2 \pi k_f A_m = - 2 \times 2 \pi \times 1000 \times 2 = - 25 120$$

I'm not entierly sure why the FM and demodulated signal are double in amplitude

## Exercise 4: FM Communications via USRP

From the Lab Instructions:
>In this exercise, FM communication via USRP will be implemented. Instead of building the code yourself,
you are given a file “FM-TxRx.gvi”. The code in the file should look familiar, but with only a slight
difference: you only have one VI this time. This is no big deal since you could have also combined the AM
communication VIs into a single one. LabView has the ability to execute two (or more) while loops
simultaneously; and thus there is no difference between the two approaches.

The given VI is similar to the ones built prior, though now everything is in a single file. There are also various functions to interface with the USRP. The modulator is contained in the while loop at the top of the diagram, and the demodulator in the bottom while loop. As before, the modulator appears to be working with the general equation of an FM signal. The message signal $𝑚(𝑡)$ is used to calculate $𝜃_𝑚(𝑡)$, which then gets passed into a sinusoid. That data is then transmitted by the USRP. The USRP receives the signal and passes it to the demodulator in the lower while loop. This appears to be using frequency discrimination like out demodulator in Exercise 2. We can see the received signal is (1) differentiated, and then (2) passed into a circuit similar to our AM envelope detection demodulator from Lab 2 Exercise 1b. These are the two key steps involved in the frequency discrimination method
for FM demodulation.

We run the VI as instructed, adjusting the value of frequency deviation (= delta_f) to 30kHz, which was successful:

![image](https://github.com/user-attachments/assets/cdbccd62-9430-4a9b-b80a-23927127b3df)

### Exercise 4a: Carson’s rule

From Lab Instructions: 
> The bandwidth of an FM signal is difficult to calculate analytically. In the 1920s J.R. Carson provided a 
rule of thumb for approximating the bandwidth:

>$$B_{FM}=2(\Delta_f+B)$$

>where $B_{FM}$ is the bandwidth of the FM signal, $\Delta_f$ is the frequency deviation of the signal, and $B$ is the 
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

# Exercise 5: Listening to FM Radio Using USRP (Bonus)

In this step, you are expected to detect and listen to local radio stations through your USRP.

## Detecting Stations

1. Open **Find Radio Station.gvi** file provided. This basic code reveals the power densities of frequencies around the carrier frequency that is entered.
2. Set the carrier frequency to **90 MHz** and the IQ rate to **20 MHz**. Examine the frequency plot obtained. The center of the graph (0) represents the chosen carrier frequency (**90 MHz**), and the bandwidth is proportional to the chosen IQ Rate (**±10 MHz** in this case).

## Task

- Capture the data of the plot to explore it. Zoom in and try to find peaks on the graph. Each peak represents an **FM signal** detected by the USRP antenna.
- Change the parameters in Step 2 if necessary and copy the obtained graph together with the chosen values to your logbook.
- Determine which FM station has the strongest signal.

![image](https://github.com/user-attachments/assets/e15b00a1-2b3b-4d2b-8d9e-6e87e1f8a34f)

**Hint:** If you are unable to see peaks in the plotted data, try to **increase the gain** to **10, 25, and 30**. The signal for FM radio stations can be weak inside the laboratory.

**Hint:** If you get the *'overflow errors'* message when running the VI, try to **decrease the number of samples**.

![image](https://github.com/user-attachments/assets/925df02e-f435-421b-b581-1fbedf44702a)

## Instructions

1. From the analysis of the spectrum, choose an **FM frequency** that has a strong signal. If the previous step fails, try **98.8 MHz – BBC 1**.
2. Open **FM Music.gvi**. Look at the code and observe that a module is added to play the demodulated waveform.
3. Input the **chosen carrier frequency** and a **coherent bandwidth (IQ Rate)** to tune into an FM radio station.

## Task

- **Plug in an earphone** into the computer, turn the volume up, and (hopefully!) enjoy the music!

**Hint:** When an earphone is plugged into the computer, LabView might have trouble detecting the correct **soundcard settings** to play audio.


![image](https://github.com/user-attachments/assets/8237a96c-b81f-4cdc-8208-839d10f19da1)



https://github.com/user-attachments/assets/3ebb0a32-7a5d-4f3e-833d-0810f6827703

