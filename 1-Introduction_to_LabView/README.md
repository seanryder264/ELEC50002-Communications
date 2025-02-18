# Lab 1: Introduction to LabView

## Exercise 1: Data Types in LabView

### Challenges

- Initially, some constants were arrays, preventing execution. This was fixed by converting them to scalars.
- When inputting "Chen," an extra `10` appeared at the end of ASCII values, disrupting calculations. We learned that `10` represents "Enter," which was being mistakenly included.
- Some constants were incorrectly set as unsigned integers instead of floating points, leading to calculation errors.

### Task

#### Block Diagram:

![image](https://github.com/user-attachments/assets/167e5c66-df68-441c-a0c9-26a636e3bc9f)


#### Front Panel:

![image](https://github.com/user-attachments/assets/563a3a30-d2c3-4e9d-aa3a-3b10172ea7cb)


### Surname ASCII Values

| Surname | ASCII Values |
|---------|-------------|
| Ryd     | 82 121 100  |
| Che     | 67 104 101  |

---

## Exercise 2: Implementation of the Central Limit Theorem

### Block Diagram:

![image](https://github.com/user-attachments/assets/75cb9247-1605-4387-bc61-570c0af3d3c5)

### Front Panel:

![image](https://github.com/user-attachments/assets/626566f5-90f6-4865-96e2-c37ecf34a56b)

### Front Panel with Stopping Control (1000 iterations):

![image](https://github.com/user-attachments/assets/3503f47a-1631-4b41-891a-e54306d508e5)

### Block Diagram with Standardization:

![image](https://github.com/user-attachments/assets/738d5615-3503-4a95-b0e1-839760f6667c)

We added a statistics function that computed the mean and standard deviation of an array. Subtracting the mean and dividing by the standard deviation standardized the distribution, achieving a mean of 0 and a standard deviation of 1.

$$ z = \frac{x - \mu}{\sigma} $$

---

## Exercise 3: AM Modulator

### Front Panel:

![image](https://github.com/user-attachments/assets/e8d495d5-5ab7-465a-8029-973637e77f18)

### Block Diagram:

![image](https://github.com/user-attachments/assets/182d0ad3-0179-4896-9257-6c4ba59f0e9a)

#### Effects of Modulation Index ($\mu$):

- $$\mu = 0.5$$

![image](https://github.com/user-attachments/assets/ea5a9d09-2e38-44cd-b23a-9370798c0e19)

- $$\mu = 1$$

![image](https://github.com/user-attachments/assets/86d90694-d141-46da-854e-b661719a73c8)

- $$\mu = 1.5$$

![image](https://github.com/user-attachments/assets/c49ad71e-a71e-4aa1-8e2c-eebb7b6f4162)

As the modulation index increases, the modulating signal's amplitude increases, leading to a higher AM signal amplitude.

#### Effects of Message Frequency ($f_m$):

- $$f_m = 1kHz$$

![image](https://github.com/user-attachments/assets/73e8dc6f-d90d-49d1-83f3-00118a239537)

- $$f_m = 2kHz$$

![image](https://github.com/user-attachments/assets/f116675b-785b-41ee-80ae-989e6534df5d)

- $$f_m = 5kHz$$

![image](https://github.com/user-attachments/assets/621673e3-bb96-4089-b6a6-05bf0c8b4433)

The AM signal envelope follows the message signal shape. As the message frequency increases, both the envelope and the AM signal oscillate faster.
