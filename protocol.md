# GamepadOverIp protocol specification

# 1. Introduction

## 1.1. Purpose

The `GamepadOverIp` protocol is a protocol that allows the communication between a client and a server to send gamepad inputs over the network. The protocol is designed to be simple and easy to implement.

## 1.2. Version

This document describes the version `0.1` of the protocol.

## 1.3. Scope

The protocol is mainly going to be used in combination with other gaming related streaming protocols like [NVIDIA GameStream](https://www.nvidia.com/en-us/support/gamestream/). While the protocol is designed to be used in a local network, mainly for gaming purposes, it can be used in other scenarios as well.

# 2. Definitions

## 2.1. Terms

The term "client" refers to the device that sends the gamepad inputs to the server. The term "server" refers to the device that receives the gamepad inputs from the client.

The term "packet" refers to a unit of data that is sent between the client and the server.

The term "flow" refers to a sequence of packets that are sent between the client and the server to achieve a specific goal.

## 2.2. Data types

### 2.2.1. `ProtocolVersion`

The `ProtocolVersion` is a data type that represents the version of the protocol. It is two bytes long. The first byte represents the major version, and the second byte represents the minor version. Both bytes are unsigned integers.

### 2.2.2. `UUID`

The `UUID` is a data type that represents a universally unique identifier. It is 16 bytes long, encoded as two 64-bit integers. The first integer represents the most significant bits, and the second integer represents the least significant bits.

### 2.2.3. `GamepadType`

The gamepad type refers to the identifier of the gamepad type in this specification. It is encoded as a single unsigned byte.

### 2.2.4. `Joystick`

The `Joystick` is a data type that represents the state of a joystick. It is four bytes long. The first two bytes represent the x-axis, and the second two bytes represent the y-axis. Both values are signed integers.

## 2.3. Gamepads

Each sub-section describes a gamepad. The title of the sub-section is the name of the gamepad and the identifier of the gamepad type.

### 2.3.1. PowerA Enhanced Wired Controller for Nintendo Switch - `0x00`

#### Data

| Field           | Type                        | Description                                                                                                                          |
| --------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `buttons`       | `UInt32`                    | The state of the buttons. The state of the buttons is represented as a bitfield. Refer to the button mapping below for this gamepad. |
| `leftJoystick`  | [`Joystick`](#224-joystick) | The state of the left joystick.                                                                                                      |
| `rightJoystick` | [`Joystick`](#224-joystick) | The state of the right joystick.                                                                                                     |

#### Button mapping (bitfield)

| Bit | Button      |
| --- | ----------- |
| 0   | A           |
| 1   | B           |
| 2   | X           |
| 3   | Y           |
| 4   | L           |
| 5   | R           |
| 6   | ZL          |
| 7   | ZR          |
| 8   | PLUS        |
| 9   | MINUS       |
| 10  | HOME        |
| 11  | CAPTURE     |
| 12  | D-PAD UP    |
| 13  | D-PAD DOWN  |
| 14  | D-PAD LEFT  |
| 15  | D-PAD RIGHT |
| 16  | LEFT STICK  |
| 17  | RIGHT STICK |

# 3. Protocol

## 3.1. Packet structure

Every packet should have the following structure:

| Field | Type    | Description                   |
| ----- | ------- | ----------------------------- |
| `id`  | `UByte` | The identifier of the packet. |
| `...` | `...`   | The data of the packet.       |

## 3.2. Encryption

The protocol supports encryption. The encryption is done using RSA. The key exchange is done outside the protocol.

It is decided during the handshake whether the communication is going to be encrypted or not. If it is decided that the communication is going to be encrypted, the client should start to send, and expect from the server, encrypted packets directly after the [HandshakeAck](#342-handshakeack---0x01) packet is received by the client.

In the [Handshake](#341-handshake---0x00) packet, the client should send a boolean value that indicates whether the client is set up to use encryption. The server should respond with a boolean value that indicates whether encryption is going to be used. The server can refuse to use encryption even if the client is set up to use encryption. In that case, the client should not send encrypted packets and should not expect encrypted packets from the server.

## 3.3. Flows

### 3.3.1. Handshake

The client should open a TCP connection to the server on port `17843` (port can be overridden by the user). The client should send a [Handshake](#341-handshake---0x00) packet to the server. Upon reception, the server should send a [HandshakeAck](#342-handshakeack---0x01) packet to the client.

### 3.3.2. Setup

Once the handshake is done, the client can set up gamepads. Before starting any streaming, the client should send an [AddGamepad](#343-addgamepad---0x02) packet to the server for each gamepad that the client wants to use. For each gamepad, the client should generate a unique identifier.

When a client is done with a gamepad, the client should send a [RemoveGamepad](#345-removegamepad---0x04) packet to the server. After this packet, any streaming packet sent with the identifier of the removed gamepad should be ignored by the server.

### 3.3.3. Streaming

After the setup of a controller is done, the client can start sending gamepad inputs to the server. However, the streaming flow isn't done over TCP. The client should send [GamepadInput](#344-gamepadinput---0x03) packets over UDP to the server. The server should listen for gamepad inputs on port `17843` (port can be overridden by the user).

## 3.4. Packets

### 3.4.1. `Handshake` - `0x00`

This serverbound packet is used to initiate the connection.

The server can support multiple versions of the protocol. The client should send the version of the protocol it uses. No direct version negotiation is supported.

| Field             | Type                                      | Description                                    |
| ----------------- | ----------------------------------------- | ---------------------------------------------- |
| `protocolVersion` | [`ProtocolVersion`](#221-protocolversion) | The version of the protocol.                   |
| `encryption`      | `Boolean`                                 | Whether the client is setup to use encryption. |

### 3.4.2. `HandshakeAck` - `0x01`

This clientbound packet is sent after a [Handshake](#341-handshake---0x00) packet is received.

| Field        | Type      | Description                        |
| ------------ | --------- | ---------------------------------- |
| `encryption` | `Boolean` | Whether encryption should be used. |

### 3.4.3. `AddGamepad` - `0x02`

This serverbound packet is used to add a gamepad to the server.

| Field        | Type                              | Description                    |
| ------------ | --------------------------------- | ------------------------------ |
| `identifier` | [`UUID`](#222-uuid)               | The identifier of the gamepad. |
| `type`       | [`GamepadType`](#223-gamepadtype) | The type of the gamepad.       |

### 3.4.4. `GamepadInput` - `0x03`

This serverbound packet is sent by the client regularly to update the state of the gamepad.

Since every type of gamepad have different layouts, capabilities, etc, the `data` field is different for each gamepad type. Refer to the gamepad type sub-sections for more information.

| Field        | Type                | Description                    |
| ------------ | ------------------- | ------------------------------ |
| `identifier` | [`UUID`](#222-uuid) | The identifier of the gamepad. |
| `data`       | `Byte[]`            | The data of the gamepad.       |

### 3.4.5. `RemoveGamepad` - `0x04`

This serverbound packet is used to remove a gamepad from the server.

| Field        | Type                | Description                    |
| ------------ | ------------------- | ------------------------------ |
| `identifier` | [`UUID`](#222-uuid) | The identifier of the gamepad. |
