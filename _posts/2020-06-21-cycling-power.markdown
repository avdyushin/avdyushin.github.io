---
date: 2020-06-21T20:00:24+02:00
title: "Cycling Power"
math: true
---

## The physics behind cycling

To move forward with constant speed $$V$$ you have to provide energy (power) to overcome total resistive force:

$$
P  = F_r \cdot V = ( F_{downhill} + F_{rolling} + F_{drag} ) \cdot V
$$

### Gravity

Cycling uphill or downhill force:

$$
F_{downhill} = m \cdot g \cdot sin(\theta)
$$

where:

- $$m$$ - weight of cyclist and bike;
- $$g = 9.80665~m/s^2$$ - earth-surface gravitational acceleration.

### Rolling resistance

$$
F_{rolling} = C_{rr} \cdot m \cdot g \cdot cos(\theta)
$$

where:

- $$C_{rr}$$ - coefficient of rolling resistance.

The coefficient of rolling resistance of the air filled tires on dry road:

$$
C_{rr} = 0.005 + \frac 1 p \left( 0.01 + 0.0095 \left(\frac V {100}\right)^2 \right)
$$

where:

- $$p$$ - the wheel pressure (Bar);
- $$V$$ - the velocity (km/h).

The angle $$\theta$$ can be calculated using elevation gain and total distance:

$$
tan(\theta) = \frac H L \Rightarrow \theta = arctan\left(\frac H L\right)
$$

where:

- $$H$$ - height (opposite side);
- $$L$$ - length (adjacent side).

### Aerodynamic Drag

Drag force:

$$
F_{drag} = \frac 1 2 \cdot \rho \cdot (V - V_w)^2 \cdot C_d \cdot A
$$

where:

- $$\rho$$ - the density of the air;
- $$V$$ - the speed of the bike;
- $$V_W$$ - the speed of the wind;
- $$A$$ - the projected frontal area of the cyclist and bike;
- $$C_d$$ - the drag coefficient.

Approximated body surface area can be estimated from the measurement of the body height and body mass
(Du Bois & Du Bois, 1916; Shuter & Aslani, 2000):

$$
A = 0.00949 \cdot (H/100)^{0.655} \cdot m^{0.441}
$$

where:

- $$H$$ - the body height in $$m$$;
- $$m$$ - the body mass in $$kg$$.

Drag coefficient in cycling can be related to the body mass also and depends on cyclist position.

#### Density

The density of the air is its mass per unit volume:

$$\rho = \frac m V$$

where:

- $$m$$ - the mass;
- $$V$$ - the volume.

It decreases with increasing altitude and changes with variation in temperature or humidity.

The density of dry air:

$$
\rho = \frac {p_0 M} {R T_0} \left(1 - \frac {Lh}{T_0}\right)^{gM/RL-1}
$$

where air specific constants:

- $$p_0 = 101325~Pa$$ - sea level standard pressure;
- $$T_0 = 288.15~K$$ - sea level standard temperature;
- $$M = 0.0289654~kg/mol$$ - molar mass of dry air;
- $$R = 8.31447~J/(mol \cdot K)$$ - ideal gas constant;
- $$g = 9.80665~m/s^2$$ - earth-surface gravitational acceleration;
- $$L = 0.0065~K/m$$ - temperature lapse rate.

Density close to the ground is:

$$
\rho_0 = \frac {p_0 M} {R T_0}
$$

At sea level and at 15℃, air has $$1.225~kg/m^3$$.

Using exponential approximation:

$$
\rho = \rho_0 e^{(\frac {gM}{RL} - 1) \cdot ln(1 - \frac {Lh}{T_0})} \approx \rho_0 e^{-(\frac {gMh} {R T_0} - \frac {Lh} {T_0})}
$$

Thus:

$$
\rho \approx \rho_0 e^{-h / H_n}
$$

where:

$$
\frac 1 H_n = \frac {gM} {R T_0} - \frac L T_0
$$

So $$H_n = 10.4~km$$.

### Coefficients Table

#### Rolling resistance coefficient:

| Tire type | $$C_{rr}$$ |
| --------- | ---------- |
| Bicycle   |      0.006 |
| Road bike |      0.004 |

#### Surface area and drag coefficient of cyclist:

| Position        | $$A~m^2$$ | $$C_d$$ | $$C_d A$$ |
|:----------------|-----------|---------|-----------|
| Back Up         |     0.423 |   0.655 |     0.277 |
| Back Horizontal |     0.370 |   0.638 |     0.236 |
| Back Down 1     |     0.339 |   0.655 |     0.222 |
| Back Down 2     |     0.334 |   0.641 |     0.214 |
| Elbows          |     0.381 |   0.677 |     0.258 |
| Froome          |     0.344 |   0.677 |     0.233 |
| Top Tube 1      |     0.371 |   0.644 |     0.239 |
| Top tube 2      |     0.355 |   0.611 |     0.217 |
| Top Tube 3      |     0.345 |   0.588 |     0.203 |
| Top Tube 4      |     0.333 |   0.604 |     0.201 |
| Pantani         |     0.343 |   0.618 |     0.212 |
| TT* & TT Helmet |     0.370 |   0.641 |     0.237 |
| TT Top Tube     |     0.331 |   0.568 |     0.188 |
| TT & Helmet     |     0.374 |   0.679 |     0.254 |
| Superman        |     0.244 |   0.615 |     0.150 |

\* Time trial

### Conclusion

Based on all this formulas we are able to calculate power effort, burned calories and fat loss of bike ride activity.

## Links
- http://bikecalculator.com
- http://thecraftycanvas.com/library/online-learning-tools/physics-homework-helpers/incline-force-calculator-problem-solver/
- https://www.gribble.org/cycling/power_v_speed.html
- https://www.researchgate.net/publication/51660070_Aerodynamic_drag_in_cycling_Methods_of_assessment
- https://www.sciencedirect.com/science/article/pii/S0167610518305762
