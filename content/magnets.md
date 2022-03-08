+++
title = "Magnets"

date = 2022-02-11



[taxonomies]

categories = ["Physics"]

tags = ["phys204"]

+++

The question of how magnets work has [long puzzled many](https://www.youtube.com/shorts/8bhYMnHb5JY). We will endeavor to answer all of those questions today.

<!-- more -->

## Magnetic Fields

Magnets work similarly to electric charges where like poles repel, and unlike poles attract. However, magnets do not ever exist in isolation. Every magnet has a north and south pole, while you can have an isolated positive or negative charge.

Every magnet has a magnetic field around it, similar to the electric field around electric charges. The direction of the magnetic field is shown by the north pole. Every magnetic field points from south to north; this is why compasses point north. They are based off of the magnetic field of the polarity of the Earth's own magnetic field.

You can apply many of the concepts of electric fields to magnetic fields, such as field lines and the way they curve and indicate field direction.

## Magnetic Force

Charges experience electric force in a field, and they experience **magnetic force** in a magnetic field. However, a few conditions must be met:

1. The charge must be moving.

2. The velocity of the charge must have a component perpendicular to the direction of the magnetic field.

To visualize the second rule, imagine the magnetic force like a buffeting wind around the charge. The wind only has any force if it has something to push on. If you hold a piece of paper parallel to the direction to the wind, it'll barely move because there's nothing to push.

> This is such an important point it's in its own section. **Right Hand Rule No. 1 (RHR1)** will tell us the direction of a magnetic force based on the magnetic field and velocity of charge. Your right thumb points in the direction of the charge, and the fingers point in the direction of the magnetic field. The palm faces in the direction of the magnetic force.
>
> This is only applies to *positive charges*. Negative charges will have the direction of the force as opposite.

The magnitude \\(B\\) of a magnetic field is defined in teslas \\(T\\) by

$$B = \frac{F}{|q_0|(v \sin{θ})}$$

where \\(F\\) is the magnitude of the magnetic force on test charge \\(q_0\\) with velocity \\(v\\\).

## Motion

We have been comparing electric fields and magnetic fields a lot, but in regards to motion through them they are quite different. Electric fields attract towards its directions, while magnetic fields direct in the other perpendicular direction. If left to its devices, a moving charge would perpetually move in a circle remaining perpendicular to the magnetic field.

This means that the work done by a magnetic field is also different. In fact, the magnetic force *cannot* do work, it can only change the direction of the particle, not its speed nor its energy.

The force required to keep a charge moving in a circle is:

$$F_c = \frac{mv^2}{r} or\ r = \frac{mv}{|q|B}$$

## Currents in a Magnetic Field

We've discussed charges moving through a magnetic field; what about a current? A current is just a collection of moving charges, after all. If we have a wire with a current running through it placed between two magnets, then we can consider the direction of the current as the charge direction in RHR1 to calculate the direction of the magnetic force.

We can use a bit of a trick to get the calculation of magnetic force for a current. We can rearrange the earlier equation for a magnetic field to get the equation for force. Current is the same thing as charge over time, so \\(\frac{Δq}{Δt}\\). Length of the current is the same thing as velocity multiplied by time \\((\frac{m}{s} \cdot s)\\). If we multiply these, the \\(Δt\\) will cancel out and we will be left with the same expression as \\\(|q_0|v\\)! Our new equation for force is:

$$F = ILB \sin{θ}$$

The angle \\(θ\\) here maximizes current when perpendicular and is zero when parallel, just as with a single charge.

## Torque

As a refresher, **torque** describes the rate of change of the angular momentum of an object. Current-carrying wires have magnetic force applied to them which causes them to move. If the wire are in a loop, then the magnetic force is applied as torque which causes the loop to rotate. Its resting state is when the normal of the loop is aligned with the magnetic field. You can think of it like a compass needle, which will turn until it reaches its resting state of pointing towards the north pole.

To calculate the torque, we get the force of each side of the loop turning, which is half the width and so half the force. Summed together we get:

$$τ = NIAB \sin{θ}$$

\\(N\\) here is the number of loops in the wire and \\(A\\) is the area that the loops make. When the loop is parallel with the magnetic field it experiences the greatest torque, and when it is perpendicular it experiences none.

The collective expression \\(NIA\\) is known as the **magnetic moment** of the coil in \\(A \cdot m^2\\).

Motors operate using this principle of a coil turning in a magnetic field.

## Magnetic Fields in a Current

Current-carrying wires create their own kinds of magnetic fields.

> The second Right-Hand Rule (RHR2) is easier to remember, as it's just a thumbs-up. The thumb points towards the direction of the current, and the fingers curl in the direction of the magnetic field generated.

The magnetic field magnitude is given by the following equation:

$$B = \frac{μ_0I}{2πr}$$

with \\(μ_0\\) representing the *permeability of free space* with the value \\(μ_0 = 4π × 10^{-7} T \cdot m/A\\).  This equation is for an infinitely long, straight wire, which is not necessarily the case.

Because of this property, currents can affect each other magnetically. Currents in the same direction are attrated to each other.

Currents in a loop have a slightly different magnetic field equation to the straight wire:

$$B = N\frac{μ_0I}{2R}$$

in the center of the loop, where the field is strongest.

A useful visual for the magnetic field of a current loop is a bar magnet placed in the middle of the loop. If we use RHR2 here, the north pole is on the palm and the south pole is on the back of the hand.

## Ampère's Law

**For any current geometry that produces a magnetic field that does not change in time,**

$$ΣB_{||}Δl = μ_0I$$

**where \\(Δl\\) is a small segment of length along a closed path of arbitrary shape around the current, \\(B_{||}\\) is the component of the magnetic field parallel to \\(Δl\\), \\(I\\) i sthe net current passing through the surface bounded by the path, and the \\(μ_0\\) is the permeability of free space. The symbol \\(Σ\\) indicates the sum of all \\(B_{||}Δl\\) terms must be taken around the closed path.**
