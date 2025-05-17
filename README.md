# Digital PLL design notes

## Introduction

There are few publications on the internet regarding analog PLL design.
Digital PLL design articles are mostly from hardware perspective.
Very little comprehensive material is available on "software" DPLL design.
Here are some links:

[Link 1](https://wirelesspi.com/phase-locked-loop-pll-in-a-software-defined-radio-sdr/)

[Link 2](https://liquidsdr.org/blog/pll-howto/)

[Link 3](https://www.dsprelated.com/showarticle/967.php)

[Link 4](https://www.dsprelated.com/showarticle/973.php)

Unfortunately, in many cases, whole parts of derivation are omitted and
the PLL structure assumption are undocumented. Often only the closed-loop
response is shown/analyzed without providing formulas for just the loop filter.

Here I'll try to do more thorough analysis of second- and third-order digital
"software" PLL from the ground up. Including discretization and full formulas + plots.

## Second-order digital PLL design

The following continuous-time structure is assumed:

![img1](imgs/dpll_2_ct.png)

There are no gain/proportional coefficients included, as a software phase detector can output
the phase directly. Transfer function of the open-loop system is:

$$
H_{open}=\frac{s \tau_{2} + 1}{s \tau_{1}}\frac{1}{s}= \frac{s \tau_{2} + 1}{s^{2} \tau_{1}}
$$

Closed-loop system is therefore described by:

$$
H_{closed}=\frac{H_{open}}{1 + H_{open}}=\frac{s \tau_{2} + 1}{s^{2} \tau_{1} + s \tau_{2} + 1}=
\frac{\frac{1}{\tau_{1}} \left(s \tau_{2} + 1\right)}{s^{2} + \frac{s \tau_{2}}{\tau_{1}} + \frac{1}{\tau_{1}}}
$$

We can identify the denominator with the standard form: ${s^{2} + 2\zeta\omega_{n} + \omega^{2}_{n}}$
and therefore express $\tau_{1}$ and $\tau_{2}$ in terms of the design parameters $\zeta$ (damping factor)
and $\omega_{n}$ (natural frequency):

$$
\tau_{1}=\frac{1}{\omega_{n}^{2}};
\tau_{2}=\frac{2 \zeta}{\omega_{n}}
$$

Using the bilinear transform we can discretize the closed-loop transfer function:

$$
H_{zclosed}=\frac{\left(z + 1\right) \left(2 \tau_{2} \left(z - 1\right) + z + 1\right)}{4 \tau_{1} \left(z - 1\right)^{2} + 2 \tau_{2} \left(z - 1\right) \left(z + 1\right) + \left(z + 1\right)^{2}}
$$

From this we can compute the discrete filter coefficients:

$$
b_{0}=\frac{2 \tau_{2} + 1}{4 \tau_{1} + 2 \tau_{2} + 1}, b_{1}=\frac{2}{4 \tau_{1} + 2 \tau_{2} + 1}, b_{2}=\frac{1 - 2 \tau_{2}}{4 \tau_{1} + 2 \tau_{2} + 1}
$$

$$
a_{0}=1, a_{1}=\frac{2 - 8 \tau_{1}}{4 \tau_{1} + 2 \tau_{2} + 1}, a_{2}=\frac{4 \tau_{1} - 2 \tau_{2} + 1}{4 \tau_{1} + 2 \tau_{2} + 1}
$$

Similarly we can obtain the discrete approximation for just the loop filter:

$$
F_{z}=\frac{\tau_{2} \left(z - 1\right) + \frac{z}{2} + \frac{1}{2}}{\tau_{1} \left(z - 1\right)}
$$

and the filter coefficients:

$$
b_{0}=\frac{2 \tau_{2} + 1}{2 \tau_{1}}, b_{1}=\frac{1 - 2 \tau_{2}}{2 \tau_{1}}
$$
$$
a_{0}=1, a_{1}-1
$$

Example:

```python
samplerate = 1000 #Hz
loop_bandwidth = 50 #Hz
omega_n=2*math.pi*loop_bandwidth/samplerate # 0.31415
zeta = 1/math.sqrt(2) # 0.707
tau_1 = 1 / (omega_n * omega_n) # 10.1321
tau_2 = 2 * zeta / omega_n # 4.5016
```

Closed-loop discrete filter coefficients:

```python
b = [0.19795842428558091, 0.039579165327638284, -0.15837925895794264]
a = [1.0, -1.5645039861011998, 0.6436623167564764]
```

Discrete loop filter coefficients:

```python
b = [0.49363631582128226, -0.39494027181038893]
a = [1.0, -1.0]
```

And the final analysis results:

![img2](imgs/dpll_2_plots.png)

## Third-order digital PLL design

TODO

