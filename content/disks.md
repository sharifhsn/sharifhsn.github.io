+++

title = "Disks"

date = 2022-04-14



[taxonomies]

categories = ["Operating Systems Design"]

tags = ["cs416"]

+++

Persistent storage is essential to every system that we can think of, and we can hardly think of a computer without it. However, managing **disks** is not as simple as it might appear on a surface level.

<!-- more -->

## I/O

Before we dive deep into disks, we must first consider the general concept of **input/output**, or **I/O**. This is a general class of devices which externally connect to the computer and provide input and output. They include keyboards, mice, displays, speakers, etc. and are essential to any typical use of a computer.

All of these devices are very different, so how does the operating system communicate with each of them? In order to talk to external devices, the OS treats every device as a **canonical device** with three components: a status register, a command buffer, and a data buffer.

The status register tells the OS what the current status of the device is. There might be many types of status for a device, but the basic one we will consider is `BUSY`. If a device is busy, the OS will not command it to do anything and will wait until it is free. Once it's free, the OS will put a command in the command buffer, and optionally some relevant data in the data buffer. The device will become busy again performing the command that the OS has given it.

Note that the device here is agnostic to the actual use of the device. The OS uses the same general procedure for every device.

One nice thing about I/O theoretically is that because its duties are separate from the computer, it can perform operations while the CPU is busy doing something else, increasing performance. However, this might not necessarily be the case. In order to initiate I/O, the CPU must, as previously stated, modify the command and data buffers. This can use up valuable CPU time.

In order to mitigate this, most computers have a separate **DMA engine** for direct memory access. This engine will communicate with I/O while the CPU is busy doing something else.

Although we do have this canonical device, the computer still needs to know what the different commands are and the type of data to put into the hardware. In order to communicate with the hardware effectively, all modern operating systems use **device drivers**. These are essentially low-level wrappers to the device which provide a convenient API for operating systems to use. In fact, almost all code in an operating system is devoted to these device drivers!

## Hard Disks

For most of computing history, **HDDs** or hard disk drives have been the hardware for persistent storage, and their internal structure is important to making sure they are performant. Recently, **SSDs** or solid state drives have increased in popularity for faster storage access. However, they are still expensive for large amounts of data and HDDs are still used commonly, so let's discuss them.

The basic hardware for a hard disk is a platter with several rotating wheels and an arm which moves between each wheel. Most hard disks are composed of thousands of such platters. Each wheel is sectioned into blocks of data. In order  to access the data, the arm must **seek** the correct wheel and the wheel must **rotate** to bring the correct data block for the arm to read, which will **transfer** the data to the computer. These three steps form the basis of reading and writing from a hard disk. Seeking and rotating are slow and tend to be the bottleneck of access, while transferring is typically fast. The time taken for I/O is defined as

$$
T_{I/O} = T_{seek} + T_{rotation} + T_{transfer}
$$

and the rate of I/O is

$$
R_{I/O} = \frac{Size_{Transfer}}{T_{I/O}}
$$

These facts mean that when composing data, we want to reduce the amount of seeks and rotations as much as possible. This makes **sequential** data very important. When data is accessed sequentially, it tends to be on the same wheel and next to each other, so no seeks and very little rotation needs to be done.

## Scheduling

Like with CPU cycles, disk requests can also be scheduled and reordered. This becomes important when trying to optimize for sequential data instead of random access.

The most obvious way to schedule a disk is to schedule requests that have the shortest seek time from the current wheel, so either on the same wheel or a close-by one. This increases performance, but it will starve requests that happen to be far away. We want to have some amount of fairness in our scheduling.

The currently accepted way to do this is **SCAN** or **C-SCAN**, also known as *the elevator method*. Essentially, the disk will be swept from beginning to end to check for any requests. If there is a request, then it will be performed. This preserves sequential data as the sweep goes in order, but makes sure that far away requests are also done. C-SCAN is an improvement over SCAN. Where SCAN simply goes back and forth like a real elevator, C-SCAN will treat the disk like a circle and sweep in the same direction. This prevents the sectors at the ends from being starved.


