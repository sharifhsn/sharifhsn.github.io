+++
title = "Circuits"

date = 2022-02-01



[taxonomies]

categories = ["Physics"]

tags = ["phys204"]

+++

Circuits are the method by which all electricity travels throughout the world and power every single device in existence. It is therefore crucial to understand them while understanding electricity.

<!-- more -->

## Electromotive Force (emf) and Current

The **emf** is similar to voltage, but not quite the same. It is the maximum voltage of a battery, signified by \\(Ε\\) (capital epsilon). It represents the maximum amount of joules of energy that can transfer from a battery based on the charge.

**Current** is formed when an electric field parallel to a wire moves free electrons from the positive terminal of a battery to the negative terminal. You can think of it mentally like the flow of charge over time, like water going through a hose, and in fact the equation is \\(I = \frac{Δq}{Δt}\\). For a non-constant flow, this is the average current. Current uses the unit \\(A\\) for ampere.

A **direct current (dc)** has electrons that go through the circuit in the same direction at all times, which is negative to positive. An **alternating current (ac)** alternates direction constantly from moment to moment.

> *Very important note:* electron flow and conventional current \\(I\\) are ***not the same thing***. Conventional current actually goes opposite to electron flow for historical reasons. When we draw current over circuits, we use conventional current notation, so current flows from positive to negative. This is because it was once thought that circuits worked by the passing of positive charges, not negative charges.

## Ohm's Law

**The ratio \\(V/I\\) is a constant, where \\(V\\) is the voltage applied across a piece of material (such as a wire) and \\(I\\) is the current through the material:**

$$\frac{V}{I} = R = constant | V = IR$$

**\\(R\\) is the resistance of the piece of material in ohms (Ω).**

Again using our hose analogy, resistance is like how narrow the hose opening is which lets our water through. Resistance is typically discussed in the context of **resistors**, which are electrical devices that apply resistance to a circuit.

## Resistance and Resistivity

The resistance of a piece of material is dependent on certain qualities. If you think in your head of a small plastic tic-tac as our resistor, this is the formula:

$$R = ρ\frac{L}{A}$$

where \\(L\\) is length, \\(A\\) is cross-sectional area, and \\(ρ\\) is a constant called **resistivity** that is constant to a material. Conductors have very low resistivities, and insulators have very high resistivities.

Resistivity also typically depends on temperature, though in more complicated ways than can be expressed in the above formula. The most common way to express it as:

$$ρ = ρ_0[1 + α(T - T_0)]$$

You can also use resistance instead of resistivity here. The important constant here is \\(α\\), which is the *temperature coefficient of resistivity*, which can be positive or negative depending on how the specific material relates resistivity to temperature.

> Some materials, known as *superconductors*, have resistivity of zero at certain temperatures. This means you can circulate a current indefinitely within that circuit without needing a supply of emf from a battery. They can be used in MRI, maglevs, and computer chips.

## [POWER](https://www.youtube.com/watch?v=chPDTUjnWgA)

**Power** is an important concept when we want to give energy to electrical appliances. The easiest way to think about it on a high level is the change in energy per unit time. Another way to think about it is the flow of voltage transferred.

$$P = \frac{Change\ in\ energy}{Time\ interval} = \frac{(Δq)V}{Δt} = \frac{Δq}{Δt}V = IV$$

The units of power are in watts \\(W\\), which you might remember from lightbulbs. Current multiplied by voltage makes sense; it's the flow multiplied by the quantity, which tells you change in quantity over time. You can also write it as:

$$P = IV = I^2R = \frac{V^2}{R}$$

which is often useful when we don't have one of those elements.

## Alternating Current

Most batteries in the world use ac instead of dc. Therefore, we must note the differences between it and dc. The voltage is not always the same, and fluctuates constantly according to this equation:

$$V = V_0 \sin{2πft}$$

where \\(V_0\\) is the maximum voltage and \\(f\\) is the frequence of isolation. This value is in radians when the sine function is applied.

Current oscillates at the same rate, and so does power. We can simply substitute the above equation for \\(V\\) to evaluate those.

We often want to understand average current and average voltage in relation to power, which are calculated by dividing the maximum of each by \\(\sqrt{2}\\). We can use these two values to get average power through the analogous power equations.

## Series Wiring

All the circuits we've discussed have one device with a straight wire. Often we want to have multiple devices on the same circuit. 

When we have multiple resistors in series, their resistances added up to get the total resistance of a circuit. A very useful method to solve these kinds of problems is to "combine" resistors by adding up their resistances. This becomes helpful when we start working with parallel wiring.

Another way we can think about voltage in this context is that we have voltage that goes through each resistor which is added up to get the final voltage. This will be relevant later when we talk about voltage drops.

## Parallel Wiring

Parallel wiring is wiring where the voltage across each device is the same. Now, this might not necessarily make sense; wouldn't the voltage be divided still? The element that is divided in parallel is actually the *current*, whereas in series it is the current that is preserved and the voltage that is divided. This is useful when we want multiple appliances connected to the same power source that do not interfere with each other with respect to voltage.

Parallel resistors, paradoxically, act as if they have *less* resistance. Instead of adding up the resistances straightforwardly, parallel resistances add up as so:

$$\frac{1}{R_p} = \frac{1}{R_1} + \frac{1}{R_2} + ...$$

A more convenient way to calculate this is

$$R_p = \frac{R_1 \cdot R_2}{R_1 + R_2}$$

for only two circuits, though it gets more complex for more.

When we have multiple parts in series and parallel, the way to solve comes by combining different series and parallel parts together. Mind that the combinations must be *only* series or *only* parallel. The easiest way is to start furthest from the capacitor and work your way there.

## Internal Resistance

We've separated devices into groups of resistors and conducting wire. However, this is a false dichotomy. All materials have some amount of **internal resistance** which resists current. Typically, batteries and wires have extremely low resistance to facilitate current.

## Kirchhoff's Rules

If resistors are in series or parallel, we can combine them fairly easily. There's trouble when we can't do that, however. We must use **Kirchhoff's Rules**, specifically the **junction rule** and the **loop rule** to calculate resistance in these cases.

The junction rule states that the total current directed into a junction must equal the total current directed out of the junction. This is just an extension of conservation of electric charge that we discussed earlier. You can imagine an intersection of cars where you have four cars waiting at the light. They can either turn right or go straight. Either way they go, however, there must be four cars that are leaving the intersection unless something has gone horribly wrong.

The loop rule relates a similar idea but in relation to electric potential. The voltage rises at every capacitor, and it must have drops through every resistor that equal the rise through the resistor. For example, after a 12 V rise through a capacitor, a pass through a 5 Ω and a 1 Ω resistor must drop 10 V and 2 V respectively in order to have it be the same.

Some important notes:

- the choice of direction is arbitrary, we will simply end up with a negative current if it is wrong

- we must indicate the positive and negative directions of every capacitor/resistor in order to find our voltage drops/rises

- our final equation will have the drops on one side and the rises on the other side, with every term either being voltage or a resistance multiplied by the variable \\(I\\)

- the answer is \\(I\\) which is solved for through normal algebra

> In laboratory, there are various devices we use to measure current and voltage. You should remember that current is only the same in series and voltage is only the same in parallel. Therefore, ammeters must be connected in series and voltmeters in parallel.

## Capacitors in Series/Parallel

Capacitors work opposite to resistors when they are in series and in parallel. They add up charge simply when placed in parallel, and have the inverse reciprocal relationship when placed in series. The reason this happens is because charge and capacitance are directly related while current and resistance are inversely related.

One note is that with regards to the charges on their plates, capacitors in series always have charges of the same magnitude because they will equalize out.

## RC Circuits

We often have circuits that have both resistors and capacitors. In this situation, the capacitor's time to charge depends on the resistance:

$$q = q_0[1 - e^{-t/(RC)}]$$

\\(RC\\) here is very simply, the resistance multiplied by the capacitance, which ends up being a unit in seconds known as \\(τ\\), the time constant. That is the amount of time for the capacitor to charge to 63.2%, which is \\(1 - e^{-1}\\).

Discharging works with a similar equation:

$$q = q_0e^{-t/(RC)}$$

where \\(τ\\) represents losing 63.2% charge.
