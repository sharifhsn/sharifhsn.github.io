+++
title = "Waves"

date = 2022-03-29



[taxonomies]

categories = ["Physics"]

tags = ["phys204"]

+++

Light is curious because it is both a particle and a wave, which makes it behave strangely in certain contexts. When projected through a slit, some strange fringes will appear, which is the subject of the famous double slit experiment/

## Interference

When waves combine together, it is called **interference**. This interference can be either *constructive* or *destructive*. When light is projected through a slit, depending on the angle from the slit to the wall behind it, the light waves will either constructively interfere or destructively interfere, creating bright and dark fringes, respectively.

The *position* of the bright fringes is given by the following equation:

$$
d\sin{θ} = mλ ⇒ m = 0, 1, 2, 3...
$$

where \\(d\\) is the distance *between the slits*, \\(θ\\) is the angle of interference from the central max onward, and \\(m\\) is the order of the interference fringe. For bright fringes, the order of the interference fringes begins at 0 for the central bright fringe, then increments onward for each surrounding fringe. Note that there is no functional difference between a fringe on either side of the central fringe as long as they are the same order away from it.

Dark fringes have a similar equation:

$$
d\sin{θ} = (m + \frac{1}{2})λ
$$

The ½ is important here because the location alternates with the bright fringes. \\(m\\) is also different because it starts at the first dark fringe surrounding the bright fringe. You can think like this:

... **3** *2* **2** *1* **1** *0* **0** *0* **1** *1* **2** *2* **3** ...

where the italicized numbers are dark fringes and the bold are bright fringes.

## Thin Film

Interference has a curious corollary with refraction if done through a thin film. Light that is transmitted between two media is partially reflected and partially passes through if passing from a medium of lower to higher index of refraction e.g. air to water. It is fully reflected if from higher to lower. The partially reflected light (but *not* the fully reflected light) undergoes a phase shift by a half wavelength.

Let's say we want to find the distance that the light travels in the medium, the "height" of the medium layer \\(t\\). If the light underwent a phase shift, the equation for interference is actually flipped:

$$
2t = (m + \frac{1}{2})λ_{film}
$$

for constructive interference, and vice versa for destructive interference. However, if there is no net phase shift, then use the normal equations.

Note that \\(λ_{film}\\) here is actually

$$
λ_{film} = \frac{λ_{vacuum}}{n_{film}}
$$

# 

## Diffraction

When a wave passes through a tiny slit, it spreads out. The slit has to be really small, as in on the order of the wavelength of the light, in order for this to happen, with more bending with a smaller slit.

Diffraction has interference as well, so the equation for the position of dark fringes is similar, although different:

$$
W\sin{θ} = mλ ⇒ m = 1,2,3...
$$

where \\(W\\) is the width of the slit. **Importantly, \\(θ\\) is different here!!!** It is the angle between the center of the slit and the center of the dark fringe in question. \\(m\\) is also different in that it *cannot* be 0! The first dark fringe in diffraction has order 1, because at order 0 there is constructive interference.

Diffraction grating has different properties than single slit diffraction, however, and is more similar to typical interference, following the constructive interference equation for bright fringes. Be careful about \\(d\\) in this context! It refers to the distance between each slit, which you might have to derive yourself from other values.
