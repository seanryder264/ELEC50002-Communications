# Lab 2: AM Simulation and USRP

## Exercise 1: AM Demodulators

### Exercise 1a: Coherent Detection

Block Diagram:  
![image](https://github.com/user-attachments/assets/a8ee0c3f-0564-4f30-954e-a1123da78a50)

$$s(t)=\[A_C+A_Mcos(2f_Mt)\]cos(2f_Ct)A_Ccos(2f_Ct)$$

[DSBSC Demodulators](https://www.tutorialspoint.com/analog_communication/analog_communication_dsbsc_demodulators.htm)  
In summary, this is how coherent detection works:  

1. Multiply the incomming AM signal $s(t)=A_c[1+m(t)]cos(\omega_ct)$ by locally generated $cos(\omega_Ct)$.
2. Use a low-pass filter to remove the high-frequency terms near $2\omega_c$.
3. The remaining baseband signal is $\frac{A_c}2[1+m(t)]$.
4. Eliminate DC and apply gain as needed to get m(t).

AM signals can be written as $s(t)=A_c[1+m(t)]cos(\omega_ct)$, $m(t)$ is the message signal. AM signal is now multiplied by a carrier signal

$$x(t)=s(t)\cdot \cos(\omega_ct) = [A_c(1+m(t))\cos(\omega_ct)]\cos(\omega_ct)$$

Using the triganometric identity

$$ \cos(a)\cos(a)=\cos^2(a)=\frac12[1+cos(2\omega_ct)] $$

we can rewrite:

$$x(t)=\frac{A_c}2[1+m(t)]+\frac{A_c}2[1+m(t)]cos(2\omega_ct)$$

After low passing, the high-frequency component is eliminated. We have:  
$$y(t)=\frac{A_c}2+\frac{A_c}2m(t)$$ 
Notice that $y(t)$ includes:
- A DC offset $\frac{A_c}2$
- The message term $\frac{A_c}2m(t)$
A simple coupling capacitor or high-pass stage can remove the DC offset, yeilding

$$\tilde{m}(t)=\frac{A_c}2m(t)$$

which is just a scaled version of $m(t)$. From there, any additional gain block can adjust the amplitude so that the recovered signal matches the original message level.

Phase and frequency must be matched for local carrier signal and incoming AM signal. If there is a frequency or phase offset, the multiplication no longer cleanly produces a low‐frequency baseband term. Instead, you get mixing products that shift the message in frequency or introduce other distortions. That is why, in practice, synchronous detection requires **carrier synchronisation** or a **carrier recovery** technique.

### Exercise 1b: Envelope Detection

Block Diagram:  
![image](https://github.com/user-attachments/assets/a90e3f16-af07-45fa-8916-664b1606e123)

Rectify, low pass, remove dc offset

Envelope detector recovers the baseband (low‐frequency) “envelope” of an AM signal simply by rectifying the signal and then following its peak amplitude with a relatively slow RC filter.

As before, assume a standard AM signal of the form

$$s(t)=A_c[1+m(t)]\cos(\omega_ct)$$

For an envelope detector to work properly, we require that
- $1+m(t)$ does not go negative
- $\omega_c$ is much larger that the highest frequency present in $m(t)$

[The Envelope Detector](https://www.winlab.rutgers.edu/~crose/322_html/envelope_detector.html)![image](https://github.com/user-attachments/assets/e47af9fa-1673-48ae-97ed-f7ebe8cac000)
  
Diode Rectification:  
The diode passes only the possitive portions of $s(t)$. Mathematically, you can think of it as approximately
$$s_{\text{rect}}(t)=\text{max}(s(t),0)$$
Because $s(t)$ is a sine carrier multiplied by the positive envelope $[1+m(t)]$, for sufficently low modulation index, $s(t)$ will be positive over part of each carrier cycle.

Next $s_+(t)$ is fed into a low-pass filter the "smooths out" the rapid carrier-frequency ripples and tries to follow only the slower envelope variations. Conceptually:
$$s_{\text{env}}(t)=\text{LPF}(s_+(t))$$
- THe rectified signal s_+(t) has a fundimental component at $\omega_c$ (and higher harmonics around $\omega_c$, $2\omega_c$, $\dots$), plus a slowly varying envelope that changes at the rate of the message frequencies
- A low-pass filter with a cutoff frequency above ther highest message frequency but well below the carrier frequency $\omega_c$ removes (most of) the carrier ripple.

In hardware, one often uses a simple diode + RC circuit:
- The capacitor charges quickly up to the peaks (following the envelope),
- Then discharges only slightly between peaks if the time constat $RC$ is chosen so that
$$\frac1{\omega_c}<<RC<<\frac1{f_{\text{max}}(m)}$$
where $f_{\text{max}}(m)$ is the highest frequency in the message $m(t)$.

If the modulation index is such that $1+m(t)\ge 0$ then the ideal envelope of the AM signal is just

$$\text{Envelope}[s(t)]=A_c[1+m(t)]$$
After rectification and low-pass filtering the ouptut $s_{env}(t)$ will approximate that envelope
$$s_{env}(t)=A-c[1+m(t)]$$

DC Offset Removal:
You might notice that $A_c[1+m(t)]$ has a DC component $=A_c$. Often we only want to recover $m(t)$ itself. One can remove the DC offset by a coupling capacitor or a simple high-pass stage, giving
$$\tilde{m}(t)=A_cm(t)$$

## Exercise 2: AM Simulation

![image](https://github.com/user-attachments/assets/c642cdf1-ce69-4661-8d3d-16f1baaaf71f)

Message signal amplitude: 1![image](https://github.com/user-attachments/assets/c74f8d4c-6c1b-4911-a580-bd0145f2c68f)


Message signal amplitude: 2![image](https://github.com/user-attachments/assets/01a849e9-3515-4bbc-a78b-190d7bf98d0e)
 
Message signal amplitude: 3![image](https://github.com/user-attachments/assets/89279d59-b61c-49c0-81d2-1612bd4c7f79)
 
Message signal amplitude: 4![image](https://github.com/user-attachments/assets/7583433c-d68f-4beb-96d4-1602e06a341a)


**Envelope detection relies on the carrier never going to zero** (the modulation index staying below 1). Coherent detection, on the other hand, can still demodulate correctly even if the carrier amplitude is partially or fully “overridden” by the modulating signal.  

1. Envelope deteection outputs a signal whose amplitude (and DC offset) scales directly with the message amplitude. As the message amplitude grows large relative to the carrier amplitude, you begin to see the familiar "over-modulation" effect:
- The envelope is no longer a perfect replica of the baseband, because the modulated waveform dips below zero  and inverts the envelope.
- Thsi shows up as distorion in both time-domain waveform (which can get "pinched" at the peaks) and the PSD (extra frequency content).
2. Coherent detection (multiplying by a locally generated carrier and low-pass filtering) does not require the envelope to remain stricly positive. As you increase the message amplitude, the coherent demodulator continues to recover the baseband accuretly - provided you have the correct carrier phase and frequency in the reciever.
  - In your time domain plots, the coherent demodulated signal remains a clean sinusoid (centered around zero or with a mild DC offset, depending on your processing).
  - Its PSD is also relatively undistorted and does not blow up in amplitude the same way the envelope detector output does.
  
There is a clear difference once the **message amplitude** (and thus the **modulation index**) becomes large. The envelope detector starts to fail (or at least distort) under “over‐modulation,” whereas the coherent detector still tracks the message correctly.

## 

## 

## 

## Exercise 3: AM Communication by USRP	

Transmitter:!![image](https://github.com/user-attachments/assets/644acbd7-5259-4712-b9bd-cd2540ef0e7e)


A convenient way to think about the **NI USRP‑2900** (and most NI‑USRP devices) is as a **generic, wideband “RF front end”** that you drive (or read) with **baseband I/Q signals** from LabVIEW. Internally, it has A/D and D/A converters plus an FPGA that handle the necessary **digital upconversion** (on transmit) and **digital downconversion** (on receive) so that you can operate at a user‐specified carrier frequency and sample rate. At a high level, here is how the transmit/receive paths work:

Transmitter Path:
1. Baseband I/Q Samples (from LabVIEW)
   You generate your signal numerically (for example, an AM-modulated waveform at baseband), whcih LabVIEW sends as a stream of I/Q samples to the NI USRP device over USB or Ethernet.
2. Interpolation and Upconversion
   Inside the USPR, an on-board FPGA and DACs take those baseband samples, interpolate them (to match the hradwaree DAC rate), and preform a numerically controlled ocsillator (NCO) mix to shift them up to a chosen intermediate frequency (IF) or directly to RF.
3. RF Front End
   A local oscillator (LO) in the USPR mixes the signal to the final carrier frequency you specified in LabVIEW (e.g, 10MHz, 2.45 GHz, ect.).
   The front end then filters and amplifies (gain setting) the signal.
4. Antenna Output
   Finally, the devices sends the amplified RF signal out through the selected atenna port.

In other words, although you are creating an “AM” (or any other) waveform in software, it’s actually just **I/Q baseband data** to the hardware. The USRP’s FPGA \+ RF chain handle all the analog mixing and upconversion to get that waveform out over the air.

Receiver:![image](https://github.com/user-attachments/assets/5573cbb7-0f86-4f0f-b946-d511f45c4599)

**Reciever Path**
1. RF Front End (Downconversion)
   On receive, the user sets a carrier frequency and gain. The USRP tunes its internal local oscillator to the desired frequency, then mixes the incoming RF signals down to an IF or baseband. It also filters out signals outside of your chosen bandwidth.
2. Digitization and Decimation
   The downconverted signal is sampled by high-speed ACDs. The on-board FPGA then decimates the ADC data to your requested "IQ rate", removing unnecessary bandwidth and producuing a stream of I/Q samples at baseband.
3. I/Q Data to LabVIEW
   These baseband samples are sent back to the host (LabVIEW). Inside LabVIEW, you can then prefrom AM demodulation, coherent detection, or any othe signal processing.

**Putting together:**

A concise way to see what is happening is that LabVIEW VIs produce (and consume) **baseband** signals, while the NI 2900 handles all the **RF up/down conversion** internally. In other words, you send “complex baseband samples” to the NI 2900, which then:

1. **Upconverts** them to 400 MHz (the carrier frequency) at the specified IQ rate (500 kS/s) and  
2. **Transmits** the resulting RF signal out of the TX1 antenna.

On the **receive** side, the NI 2900:

1. **Tunes** to the same 400 MHz carrier,  
2. **Downconverts** whatever arrives at RX2 back to baseband (again at 500 kS/s),  
3. **Demodulates** if you have the hardware configured to do so (or simply returns baseband I/Q data, depending on the example), and finally  
4. Streams the result back into LabVIEW for display and further processing.

Hence, from LabVIEW’s point of view, you only need to provide (on transmit) or process (on receive) **baseband** data. The NI 2900’s FPGA and front‐end circuitry take care of mixing, filtering, gain, etc. at the RF level, so effectively “the USRP does all the modulation and demodulation on its own” and the VIs just pass complex data to or from the device.

Transmitted 5kHz Signal:  
![image](https://github.com/user-attachments/assets/6f905b88-c473-4439-8edc-836e4188e6ff)
 
Demodulated 5kHz Signal:  
![image](https://github.com/user-attachments/assets/b2ad758a-7af3-406f-b97b-1074983e2bf8)


Noise 20dB Signal:![image](https://github.com/user-attachments/assets/7d9078ef-fbde-4c48-98eb-6faedd359594)

