+++

title = "The Link Layer"

date = 2022-03-28



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

Through all the layers of the internet, the most essential is the **link layer**: converting bits to and from signals, detecting errors, flow control, and addressing.

<!-- more -->

## Encoding

We have spoken mostly through the guise of bits per second being sent across the internet and have left it at that. But what does it mean to send a bit over air waves? This isn't like a traditional electrical circuit or transistor in your computer; the internet uses signals.

The nuances of digital signal processing are beyond these notes, so we will simplify here. We can imagine signals like a pseudo-transistor where low signals are 0 and high signals are 1. However, this causes two issues. One is that time synchronization is impossible for waves that travel at the speed of light, and the other is that because signals are not emitting all the time, we need a way to differentiate between signals and simple noise.

The time synchronization is mandated by a clock standard such as the Manchester encoding, which is slower than the speed of light but allows each link to know exactly when a signal is coming.

## Framing

A **frame** is a group of bits in sequence. Frames are useful for manipulating data at the link layer, but they can escalate small bit errors to big problems. If a frame is too big, then it will error out too often, so we will often have smaller frames.

We also need a way to delineate the beginning and ending of frames. There are two ways, character stuffing and bit stuffing.

With character stuffing, a special meta character will be used to delineate frames. `^` is used for the beginning of frame (BOF) and `$` is used for EOF. If you need a `$` in your text, then simply add an escape character, like another `$` right beforehand.

With bit stuffing, a unique bit sequence `0x7E` or `0b01111110` will delineate frames. If that sequence appears in the data, it will insert a 0 within like `0b011111010`
