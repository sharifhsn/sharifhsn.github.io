+++
title = "Mirrors"

date = 2022-03-08



[taxonomies]

categories = ["Physics"]

tags = ["phys204"]

+++

Mirrors can interact with light in interesting ways. In fact, reflection and refraction are probably most people's engagement with laws of physics surrounding light in their daily life.

<!-- more -->

## Reflection

On a plane mirror, like the kind that you're used to in daily life, *the angle of incidence equals the angle of reflection*. To clarify some terms: *incidence* refers to the ray that is going towards the mirror, and *reflection* refers to the ray that is coming out of the mirror. The *normal* is a perpendicular line, in this case to the plane of the mirror.

Because of this characteristic of plane mirrors, the reflection of an object (or **image**) will be upright and the same size as the object. The image will be "inside" the mirror, which is called *virtual*. The distance "within" the mirror to the image is the same distance from the mirror to the object. This is useful for real life because we want to see most objects the same as they actually exist. However, in physics, sometimes we want to manipulate these variables, and for that we use **spherical mirrors**.

A spherical mirror is a mirror that is curved such that it could be cut out of a sphere. It can be either concave or convex (front and back of a spoon).

There are two points that are significant to a mirror, the focal point \\(F\\) and the center of curvature \\(C\\). If the mirror was part of a sphere, then the center of curvature would be the center of that sphere. The focal point is where parallel rays intersect after reflection. Both points are on the *principal axis* of the mirror. The focal point is halfway between \\(C\\) and the center of the mirror.

In order to determine how the image of an object looks when reflected off of a mirror, it is useful to construct a *ray diagram*. The ray diagram is based on three rules (you only need two).

1. an incident ray parallel to the principal axis reflects through \\(F\\)

2. an incident ray that passes through \\(F\\) reflects parallel to the principal axis

3. an incident ray which passes through \\(C\\) reflects on itself

## Magnification

The calculation of the distance and size of an image are governed by the following equations:

$$
\frac{1}{f} = \frac{1}{d_o} + \frac{1}{d_i}
$$

where \\(d_o\\) is the distance from the mirror to the object and \\(d_i\\) is the same for the image. 

The magnification \\(m\\) of an image is the ratio of the image size to the object size:

$$
m = \frac{h_i}{h_o} = -\frac{d_i}{d_o}
$$

Now, you might be wondering about how the sign of each value affects these equations. The answer is, a LOT. To think about mirrors intuitively, think about what each sign must mean.

- \\(f\\) is positive for concave mirrors because the focal point is front, but negative for convex mirrors because it's behind the mirror.

- \\(d_o\\) is positive for a real object (in front of the mirror), but negative for a virtual object. A virtual object can't generally exist, but you can simulate it using reflected light.

- \\(d_i\\) is the same, but a virtual image makes more sense.

- \\(m\\) is obvious, positive for upright and negative for inversion

## Refraction

If you put a straw in your water, the straw will strangely seem to bend at the water's edge and go at an angle. The reason for this is **refraction**: light will change direction when it passes through a medium. The change in angle is described by Snell's Law:

$$
n_1\sin{θ_1} = n_2\sin{θ_2}
$$

where \\(θ_1\\) is the *angle of incidence*, \\(θ_2\\) is the *angle of refraction*, and \\(n_1\\) and \\(n_2\\) are the *indices of refraction* for the corresponding media. The index of refraction is expressed as the ratio between \\(c\\) and the speed of light in the material. If light is very fast in the material, \\(n\\) will be close to 1.

When light passes from a medium with a higher index of refraction to a lower one, there's an angle at which the refracted light will be parallel to the media edge, known as the *critical angle* \\(θ_c\\):

$$
\sin{θ_c} = \frac{n_2}{n_1}
$$

At any angle greater than the critical angle, all the light will be reflected and none will be refracted.

> The reason that this is the case can be derived from math. Let's say that light passes from water (1.33) into air (1.00). The critical angle would be equivalent to \\(\sin^{-1}{\frac{1.00}{1.33}}\\) or 48.75°.
> 
> What would happen if we tried to find the angle of refraction based on Snell's Law if the incident angle is 50°, so greater than the critical angle?
> 
> $$
> 1.33 \cdot \sin{50°} = 1.00 \cdot \sin{θ_2}
> $$
> 
> $$
> \sin{θ_2} = 1.33 \cdot \sin{50°} = 1.019
> $$
> 
> But there's a problem here! The function \\(\sin^{-1}{}\\) is not defined for \\(θ\\) greater than 1, so the angle does not exist. This means that refraction does not occur, the light is only reflected.

## Lens

The properties of magnification and refraction are used in lens that are located in glasses, telescopes, microscopes, and even your own eyeball! A **lens** has two mirrors on either side. If the mirrors are convex, the lens is *converging*, and if they are concave, the lens is *diverging*. Every lens has *two* focal points corresponding to each mirror.

The ray diagram for a converging lens follows similar rules to a concave mirror, but instead of thinking of reflected rays, the rays are refracted through the lens.

1. When a ray passes through the axis of a lens parallel, it will refract towards the other focal point.

2. When a ray passes through a focal point, it will refract parallel through the other side.

3. When a ray passes through the center of the lens, it will not bend.

The rules are the same for a diverging lens, though it may not appear that way. All of the rules for a diverging lens are based on a focal point on the other side of the mirror, but fundamentally have the same property.

All of the formulas are the same for lens like mirrors, however, the meaning of the signs changes in this context.

- \(f\) is positive for converging lens because the focal point is front, but negative for diverging lens because it's behind the lens

- \(d_o\) is positive for a real object (same side as light), but negative for a virtual object. A virtual object can't generally exist, but you can simulate it using reflected light.

- \(d_i\) is positive for a real image on the *opposite* side of the light, and negative for a virtual image on the same side of the light.

- \(m\) is obvious, positive for upright and negative for inversion
