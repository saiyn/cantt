# CANTT, a Telemetry Transport for CAN bus

**While based upon ISO-TP (ISO-15765-2), this library does not retain
compatibility with ISO-TP. Read more about the explanation in the section
titled "Compatibility issues with ISO-TP".**

A topic and message protocol/library to send MQTT like messages over CAN bus.
This library is not dependant on the specific CAN bus implementation and has
been tested both with MCP_CAN_lib as well as the CAN implementation in the 
Particle Photon/Electron.

## Usage

See the [usage.ino](examples/usage/usage.ino) file in the
[examples/usage](examples/usage) directory.


## Compatibility issues with ISO-TP (ISO-15765-2)

While trying to build a library that was compatible with ISO-TP, significant 
issues arose from the *Flow Control* frames. These frames are supposed to 
follow a *First* frame as an acknowledge/acceptance from the other side. 
However, this library is supposed to broadcast messages from and to every 
node on the bus. Having each node sending a *Flow Control* frame whenever 
they receive a *First* frame results in lots of strange behaviour, and too 
much unnecessary overhead on the bus.

It was thus decided to skip the *Flow Control* frames and in favour of 
sending the *First* frame and directly after that all of the *Consecutive*
frames.

## License

[BSD 3-Clause License](LICENSE)

## Simple Example

```cpp
CANChannel CAN0(CAN_D1_D2); // Particle Photon/Electron

void callback(long unsigned int id, byte* payload, unsigned int length) {
  // Received frames
  // The payload data is in unparsed form, a "raw" CANTT message.
  // For a publish from CANTT, this would be:
  // BYTE0: 0x03
  // BYTE1, BYTE2: (uint16_t topic length in LSB MSB order) 0x01 0x00 
  // BYTE3..BYTEX: topic
  // BYTEX+1, BYTEX+2: (uint16_t payload length in LSB MSB order) 0x01 0x00 
  // BYTEX+3..BYTEY: payload
}
uint8_t canAvailable() {
  return CAN0.available();
}
uint8_t canRead(CANMessage &msg) {
  CAN0.receive(msg);
  return 0;
}
uint8_t canSend(const CANMessage &msg) {
  CAN0.transmit(msg);
  return 0;
}
CANTT cantt(DEVICE_ID, canAvailable, canRead, canSend, callback);
uint8_t canAvailable() {
  return CAN0.available();
}

void setup() {
  cantt.begin();
}

void loop {
  cantt.cantt();
  // cantt.publish("some/topic", "some kind of data");
}
```

