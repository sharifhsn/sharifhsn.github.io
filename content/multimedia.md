+++
title = "Multimedia Networking"

date = 2022-04-07



[taxonomies]

categories = ["internet Technology"]

tags = ["cs352"]

+++

Streaming multimedia is a relatively new concept in internet technology, but now has become central to the daily media consumption of people around the world.

<!-- more -->

## Multimedia

In contrast to the ordinary messages that encode text, **multimedia** streaming uses audio or video. Most of the traffic on the internet today is dominated by multimedia streaming, so it's a big deal. Multimedia streaming is different from other kinds of streaming in that latency is of greatest importance, but it is also loss tolerant; if the streaming video is a bit distorted, it's not a big deal. When the delay hits above 150 ms users begin to complain.

VoIP is a special case where there is media being streamed *both* ways, where two people speak on the same connection through data networks.

We must handle signals differently because we don't have a simple Huffman coding like with text. There are three elements at play that determine the quality of the signal:

- Sampling: how often do you sample the signal for data?

- Quantization: how many levels/bits represent each sample?

- Compression: how much are the quantized values compressed?

In order to minimize the amount of data being sent, video streaming will often only send the changes from one frame to the next. Otherwise, the level of fidelity would be infeasible on current internet technology.

## Audio

Audio is generally ranges from 20 Hz to 22.05 kHz, and speech goes between 200 Hz and 8kHz. Sampling generally occurs up to 8kHz to capture speech, the most important element of audio in general, though it is possible to capture 16kHz for high-fidelity audio.

Quantization can either be 8 bits or 16 bits, which means there are either \\(2^8\\) or \\(2^{16}\\) levels. That's a difference between 256 and 65,536!

The bit rate of samples typically depends on the type of audio being sent. For simple VoIP it will typically be 64Kbps, because it quantized at 8 bits and sampled 8k times per second. Audio will generally be 705.6Kbps from 16 bits of quantization and 44.1k samples per second. Stereo is the most fidelity because it is audio in both ears, so double the bit rate at 1.4112 Mbps. By removing frequencies that the human ear cannot distinguish, the amount of data can be reduced dramatically without altering the end sound.

## Video

Images are 2D arrays of pixels at heart. The amount of pixels is *resolution*, which is typically expressed through a 16:9 aspect ratio as with 1080p or 4K/UHD.

Every color has 8 bits, and there are three colors RGB so each pixel will contain 24 bits. That means the raw data for a color image even of size 320×240 will be 7680 bytes! That's a lot for one image. Compression through JPEG, Gif, etc. is typically used to help here.

Video is composed of images that are displayed at a certain *frame rate*. Movies run at 24 FPS, though some rare movies display at 48 FPS. Video games are commonly at either 30 FPS or 60 FPS, but can go even higher.

If we imagine a 4k television displaying color images at 60 FPS video, the amount of data transmitted would be \\(4096 \cdot 2160 \cdot 60 \cdot 24\\) which is 12 Gbps!!! We obviously need some kind of compression.

## Streaming

The client will typically downloading the first portion of the multimedia and begin consuming it. The rest is downloaded while the client consumes the data, so you don't need to wait for the whole file to be downloaded. If you've ever watched a YouTube video, you can see this in real time as the gray bar overlaid on top of the timeline of the video. The client has a buffer to store that downloaded video which is constantly drained to show the video and filled at times corresponding to network delays.

However, this can cause *buffering* when the network delay is variable and the client consumes all of its data before the server can produce more. The video will be buffered until more data is received, and you see a still frame on the video while a loading symbol plays over it.

## RTP

UDP streaming is the traditional type of streaming. As multimedia does not necessarily need to be reliable, UDP is better for reducing latency which is most important for the consumer. The packet has the timestamp of the multimedia so it can be ordered correctly in the client buffer. It also needs certain information like the encoding or whether the audio is for left or right microphone. The protocol used for multimedia that incorporates these is called **RTP** or **Real-time Transport Protocol**.

RTP is a flexible protocol that allows for many types of multimedia to be transmitted.

The header includes a timestamp field which increments per sample, which allows for correct timing. It also uses sequence numbers to distinguish between packet loss and silence.

## HTTP

Most people stream video over their browsers nowadays, which uses HTTP. You might remember that HTTP uses TCP, which is slower than UDP but more reliable. The streaming is slower but more convenient.

## DASH

**Dynamic Adaptive Streaming** allows for good bandwidth over HTTP. The client is the one that adjusts the request rate from multiple content formats/encoding. For example, the client can request data at a lower bit rate or quality like 480p if it feels that 720p is too slow.

The MPD or Media Presentation Descriptor is sent over HTTP and gives information about the different format segments. The client can then request each segment, and the segments can come from different ASes, which works well for CDNs. This is how ads are served over YouTube; the client is the one that decides whether to get an ad or not. This is also why blocking ads on YouTube can be done by an extension to your browser.

## DNS

The ID for every YouTube video is a BASE64 11 character string, giving a space of \\(64^{11}\\) possible videos. The ID is mapped to 192 DNS host names so that multiple servers can be accessed for videos. This ID is a special hash of the video ID.

Multiple ASes can broadcast the fact that they are all reachable to a BGP router, and a BGP router can then choose the shortest path. This is called **anycast**. This means that for accessing video servers, you can pick the country where the video is hosted quite easily. Let's say you want to access a Japanese video in Japan. It's obviously best to access the Japan server, so that's the closest. However, all 192 servers that host the video will broadcast, so that if the Japan server goes down in an earthquake or something, the other servers are still available for the BGP router to pick from, so they can access, for example, the South Korean server.

## VoIP

Traditional telephones work over PSTN (Public Switched Telephone Network), which is autonomous and only designed for telephones. However, as the Internet has gradually taken over the world, a demand was created to include telephony in the Internet. Thus, the **VoIP** (Voice IP) protocol was born.

The voice sound waves are converted into packets and transported as audio through lossy RTP, similar to other forms of multimedia.

In order to establish the connection to begin with, **SIP** or Session Initiation Protocol sets up the VoIP session between two hosts. SIP URLS are similar to email addresses that identify users on a network. Most of the work is lent to the caller in order to type in some identifying information as they typically have a display/keyboard. The URL is mapped to an IP address similar to DNS and email.
