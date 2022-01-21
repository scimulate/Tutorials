![Synopsis](https://raw.githubusercontent.com/scimulate/Tutorials/cb6b7f51b62ca97ccec203afa3ac2e9e04da33fc/CodeAster/Study_001/images/TestCases.svg)

# Synopsis
This study is a 1/8<sup>th</sup> model (symetric about XY, YZ, and ZX) of a plate with an unloaded central hole. Both x-normal surfaces of the full plate are under simple tension.

**Note:** unless otherwise specified, all units are in MKS.
# Material Properties


```python
alum6061 = DEFI_MATERIAU(ELAS=_F(E=6.89e+9,
                                 NU=3.30e-1,
                                 RHO=2.70e+3))

fieldmat = AFFE_MATERIAU(AFFE=_F(MATER=(alum6061),
                                 TOUT='OUI'),
                         MODELE=model)
```

# Boundary Conditions
Each symmetrical surface has been labeled according to its respective normal axis (```XFix```, ```YFix```, and ```ZFix```). This simplifies assigning displacement boundary conditions to a trivial matter of matching letters ```X```, ```Y```, and ```Z```.


```python
boundary = AFFE_CHAR_MECA(FACE_IMPO=(_F(DX=0.0,
                                        GROUP_MA=('XFix')),
                                     _F(DY=0.0,
                                        GROUP_MA=('YFix')),
                                     _F(DZ=0.0,
                                        GROUP_MA=('ZFix'))),
                          MODELE=model)
```

# Loading
A uniform surface load (```FORCE_FACE```) of 0.1MPa has been applied to the unconstrained x-axis normal surface ```LoadFace```.


```python
load = AFFE_CHAR_MECA(FORCE_FACE=_F(FX=1.0e5,
                                    GROUP_MA=('LoadFace')),
                       MODELE=model)
```

With a cross-sectional area of 1.0e-3m<sup>2</sup> for the 1/8<sup>th</sup> model, this corresponds to a load of 100N (full load of 400N). (**Note:** In this context, a 1/8<sup>th</sup> model will have 1/4<sup>th</sup> of the cross-sectional area.)

# Solution
Along the x-axis and away from the hole, plate stress should approach the applied uniform surface load. On plane YZ, the stress concentration is a function of $r/D$.

![Stress Concentration](https://i.stack.imgur.com/b0GVl.png)

This stress concentration factor, _K_, compounds an already elevated stress on plane YZ due to a reduction in cross-sectional area.

$$
A_{xc} = (\Delta y-r)*\Delta z
$$

$$
\sigma_{nom} = P/A_{xc}
$$

Excluding the stress concentration factor, $r/D=0.25 $ $K \approx 2.45$, the nominal stress on plane YZ should be


```python
dy = 0.10   # [m]
dz = 0.01   # [m]
r = 0.05    # [m]
load = 100  # [N]

area = (dy-r)*dz      # [m**2]
stressNom = load/area # [N/m**2]
print(round(stressNom, 1))
```

    200000.0


Including the stress concentration factor, the maximum stress ($\sigma_{max}$), which should occur where the hole surface meets plane YZ, adheres to the relationship

$$
\sigma_{max} = K\sigma_{nom}.
$$


```python
k = 2.45
stressMax = k*stressNom
print(round(stressMax, 1))
```

    490000.0


$\sigma _{max}$, differs between the textbook calculations and finite element results 10.75%. As expected, the textbook calculations are higher as the empirical curve is likely conservative.
