# CommsSys (Temporary Name)

A modular, encrypted LoRa-based communication system for sending compressed voice packets between remote nodes via a central Raspberry Pi base station. Each node can either broadcast to all devices or talk privately to one. It’s walkie-talkies, but better, and significantly more overengineered.

---

## System Overview

It has three core components:

- A central base station (Raspberry Pi 5) that acts as the relay hub.
- A set of portable modules (ESP32 + LoRa + mic + speaker).
- LoRa-based radio communication with optional directional antennas.

Audio is compressed, encrypted, and chunked into small packets. These are sent to the base station, which decrypts and routes them according to the header (either public broadcast or targeted private message). The receiving node plays the audio out loud. That’s the whole thing.

---

## Hardware Summary

### Base Station

- Raspberry Pi 5
- LoRa transceiver (SPI or USB-bridge)
- Directional Yagi antenna (if available)
- Optional mic/speaker setup for local testing

### Modules

Each field unit will be built on a dev board first, then PCB later when the design is finalized.

- ESP32 (I2S capable)
- LoRa module (SX1276 or compatible)
- Mic (e.g., MAX9814)
- ADC (if mic is analog)
- Audio amp (PAM8403 or something else that gets loud)
- Speaker (0.5W–1W)
- Push-to-talk button
- LiPo battery + TP4056 charging module

---

## Communication & Packet Structure

### Packet Design (Early Draft)

```
[HEADER][ENCRYPTED AUDIO PAYLOAD]
```

#### Header (16 bytes total)
- 1 byte – Protocol version
- 1 byte – Packet type (Voice / Handshake / Heartbeat)
- 2 bytes – Source ID
- 2 bytes – Target ID
- 2 bytes – Sequence number or timestamp
- 8 bytes – Reserved (HMAC or future expansions)

#### Payload
- Compressed audio data
- Encrypted with AES-128 (CBC mode, probably)

### Modes
- **Public Channel**: Broadcasted by the Pi to all nodes.
- **Private Channel**: Routed only to the intended node based on header.

---

## Audio Processing

### Basic Plan

- Sample rate: 8–16kHz
- Bit depth: 8-bit or 16-bit PCM
- Compression: ADPCM (Ima) or μ-law. Will pick whatever fits better.
- Frame size: ~100–300ms per packet

Audio will be captured in short bursts, compressed, and transmitted. It’s not real-time, but close enough for a project that wasn’t meant to carry high-fidelity opera.

---

## Encryption

The system uses symmetric encryption:

- AES-128 CBC
- Shared key stored in code (for now)
- Optional: Add session key negotiation with Diffie-Hellman once the rest of the system works

---

## LoRa Config (Subject to Change)

| Parameter         | Value                                 |
|------------------|---------------------------------------|
| Frequency         | 409.7500–409.9875 MHz (Prob 409.8488) |
| Bandwidth         | 125 kHz                               |
| Spreading Factor  | 7–10                                  |
| Coding Rate       | 4/5                                   |
| Preamble Length   | 8                                     |
| TX Power          | 17–22 dB                              |

Settings will probably change a dozen times during testing.

---

## Control Logic (System Flow)

1. A user holds the push-to-talk button on a module.
2. Audio is recorded, compressed, encrypted, and sent to the base station via LoRa.
3. The base station decrypts the packet, reads the header, and decides where to forward it.
4. It re-encrypts the payload and sends it to:
   - All nodes (public), or
   - One node (private).
5. The destination node decrypts and plays the audio through its speaker.

---

## Development Plan

| Phase     | Goal                                              |
|-----------|---------------------------------------------------|
| Alpha-0   | RPi <-> ESP32 "Hello World" over LoRa             |
| Alpha-1   | Send compressed dummy audio from ESP32 to RPi     |
| Alpha-2   | Add encryption and real audio recording           |
| Alpha-3   | Implement full relay logic RPi → another ESP32    |
| Alpha-4   | End-to-end test: Node A talks, Node B hears       |
| Beta      | Field testing + range checks                      |
| Final     | PCB design, power optimization, case enclosure    |

---

## Future Notes

- Build a web/CLI interface to manage routing and monitor node status
- Add dynamic channel switching (multiple groups)
- Evaluate latency and try to reduce it
- Optional mesh fallback if the Pi goes down (very optional)
