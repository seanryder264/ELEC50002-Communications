# Lab 4: Binary Phase Shift Keying (BPSK) Via USRP
In phase-shift keying (PSK) modulation, information is encoded on the phase of the transmitted carrier, 
rather than on its amplitude (ASK), or frequency (FSK). In binary phase-shift keying (BPSK) there are 
two phase values, 0! or 180!, which means that an unmodified carrier is transmitted to represent one 
binary data value, while an inverted carrier is transmitted to represent the other. BPSK is optimum 
among binary modulation schemes in achieving the lowest average power for a given target bit error 
ratio (BER).

## Exercise 1: BPSK Transmitter
The BPSK signal is given by:

$$
A g_{TX}(t) \cos (2 \pi f_c t + \theta(t))
$$

In this equation, $A$ is a constant corresponding to the transmitted power level, $g_{TX}(t)$ is a fixed pulse
shape, $f_c$ is the carrier frequency, and $\theta(t)$ takes the value of either 0 or 180 to carry the desired 
information. Note that, we can also write the equation as:

$$
\pm A g_{TX}(t) \cos (2 \pi f_c t)
$$

where the plus sign corresponds to $\theta$ = 0, and the minus sign to $\theta$ = 180. We will assume that a new
pulse is transmitted every $T$ seconds, so that the symbol rate (symbols per second) is $1/T$. For a binary
scheme such as BPSK, the bit rate is the same as the symbol rate. Since the pulse �"#[�] does not
carry information, its shape can be chosen to satisfy other criteria. We desire a pulse shape that provides
a rapid spectral roll-off to minimize inter-symbol interference, e.g. ‘root-raised-cosine' filter.

The steps needed to form a BPSK signal are:
