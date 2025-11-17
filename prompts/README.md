# 🌐 Smart IoT Device Communication Protocol (SIoT-P)
# CN Assignment 02 – Computer Networks

**Course:** Computer Networks (Comp-352)  
**Semester:** 5th (Fall 2025)  
**Assignment:** 2  

---

## Group Information
- **Class:** BSAIF23-Green  
- **Group Members:**
  - Name: [Muhammad Abdullah] 
    Reg#: [ B23F0001A1006 ]

  - Name: [Husnain Sajid] 
    Reg#: [ B23F0001A1007 ]

  - Name: [Usba khan] 
    Reg#: [ B23F0922AI190 ]
---

## 📘 Abstract

The **Smart IoT Device Communication Protocol (SIoT-P)** is a custom **application-layer protocol** designed to enable reliable and efficient data exchange between IoT sensor nodes and a central monitoring server.  
This protocol is developed for microcontrollers such as **ESP32** that use sensors like **DHT11** (temperature/humidity) and **MQ2** (gas detection).  
SIoT-P ensures structured communication, reliability through acknowledgments, and scalability for multi-device IoT environments.  

It operates on top of TCP, providing guaranteed message delivery through a binary fixed-format protocol structure similar to standard network protocols (TCP, UDP, HTTP headers).

### 📋 Document Structure

This document provides a complete protocol specification that enables developers to understand and implement SIoT-P. It includes:

1. **Application Functional Requirements** - Detailed functional specifications (Section: ⚙️ Functional Requirements)
2. **Application Non-Functional Requirements** - Quality attributes and constraints (Section: 🧾 Non-Functional Requirements)
3. **Application Architecture** - System components and design (Section: 🏗 Architecture Design)
4. **Message Types** - All protocol message types with codes and actions (Section: 🧠 Protocol Design → Message Types)
5. **Message Format** - Complete binary fixed-format specification with examples (Section: 🧠 Protocol Design → Message Format)
6. **Developer Guidelines** - Implementation parameters, checklists, and code examples (Section: 🧑‍💻 Developer Guidelines)  

---

## 🎯 Objectives

- Design and document a custom **application-layer IoT communication protocol**.  
- Ensure reliable, low-latency data transfer between IoT nodes and server.  
- Define message types, structures, and flow for real-world implementation.  
- Demonstrate communication through packet captures (Wireshark).  
- Provide implementation-ready developer guidelines and extendability for future versions.  

---

## 🧩 Problem Statement

Traditional IoT systems use protocols like **MQTT** or **CoAP**, which can be heavy or require additional broker/server software.  
**SIoT-P** addresses this gap by introducing a **lightweight yet reliable TCP-based protocol** for embedded systems that need:  
- Low overhead  
- High reliability  
- Easy implementation on microcontrollers  

This protocol is designed for projects where real-time monitoring of environmental parameters is essential (e.g., smart homes, labs, and industrial safety systems).

---

## ⚙️ Functional Requirements

1. Each IoT device connects to the central server using TCP.  
2. Devices send periodic sensor readings (temperature, humidity, gas).  
3. Each packet includes device ID, timestamp, and checksum.  
4. The server acknowledges each received message with `ACK`.  
5. If `ACK` is not received within timeout (3s), the device retransmits.  
6. The server may send `CMD` messages to control devices (e.g., change sampling rate).  
7. Multiple devices may communicate concurrently with the same server.  

---

## 🧾 Non-Functional Requirements

| Category | Description |
|-----------|--------------|
| **Reliability** | Uses acknowledgment & retransmission to ensure guaranteed delivery. |
| **Lightweight** | Compact binary fixed-format structure suitable for low-memory IoT boards. |
| **Security** | Supports basic checksum integrity and optional encryption. |
| **Scalability** | Designed to handle multiple devices simultaneously. |
| **Maintainability** | Clear message format allows easy extension. |
| **Interoperability** | Can operate with various sensors and microcontrollers. |

---

## 🏗 Architecture Design

### 🔸 Components

1. **IoT Device (ESP32 Node):**  
   - Collects data from **DHT11** (temperature/humidity) and **MQ2** (gas).  
   - Sends readings to the central server using SIoT-P over TCP.  
   - Waits for acknowledgment or command messages.

2. **Central Server:**  
   - Listens on a predefined TCP port (default: `5050`).  
   - Receives and validates messages from devices.  
   - Sends acknowledgment or control commands.  
   - Stores data in a database for analysis and visualization.

3. **Database (optional):**  
   - Logs all readings with device ID, timestamp, and values.  
   - Supports historical trend analysis or dashboard visualization.  

---

## 📡 Communication Flow

```
+-------------+                   +----------------+
|  IoT Device |                   |    Server      |
+-------------+                   +----------------+
       |                                   |
       |--- REGISTER --------------------->|
       |<-- ACK ----------------------------|
       |--- DATA_PUSH --------------------->|
       |<-- ACK ----------------------------|
       |<-- CMD (optional) -----------------|
       |--- ACK ----------------------------|
```

---

## 🧠 Protocol Design

### 🔸 Message Types

| Type | Code | Direction | Purpose | Actions Taken |
|------|------|------------|----------|---------------|
| `REGISTER` | 0x1 | Device → Server | Register new device with ID and type | **Server:** Validates device ID, checks authorization, adds device to registry, responds with ACK (status: RECEIVED) or ERROR (code: 401 if unauthorized) |
| `DATA_PUSH` | 0x2 | Device → Server | Send sensor readings | **Server:** Validates checksum, parses sensor data, stores in database, responds with ACK (status: RECEIVED) or ERROR (code: 400 if invalid format, 500 if storage fails) |
| `ACK` | 0x3 | Server ↔ Device | Acknowledge successful message delivery | **Receiver:** Confirms message receipt, stops retransmission timer, processes next message in queue. Status codes: 0x01=RECEIVED, 0x02=PROCESSED, 0x03=ERROR |
| `CMD` | 0x4 | Server → Device | Send control commands (e.g., change rate) | **Device:** Parses command type and value, executes command (e.g., updates sampling rate), responds with ACK (status: PROCESSED) or ERROR (code: 400 if invalid command) |
| `ERROR` | 0x5 | Both | Report errors (invalid or corrupted message) | **Receiver:** Logs error code, terminates current operation, may trigger retransmission or connection reset depending on error severity |

---

### 🔸 Message Format (Binary Fixed-Format Protocol)

SIoT-P uses a **binary fixed-format header structure** similar to TCP/IP protocols, with fixed fields at specific byte positions. All multi-byte fields use **big-endian (network byte order)**.

#### 📋 Common Header Structure (Fixed 20 bytes)

All SIoT-P messages start with a fixed 20-byte header:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version| Type  | Flags |Reserved|        Message ID             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Device ID Len | Payload Length (2 bytes)      | Timestamp (4)  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Timestamp (cont.)          |   Checksum     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Reserved (6 bytes)                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Field Definitions:**

| Field | Size | Position | Description |
|-------|------|----------|-------------|
| **Version** | 4 bits | 0 (bits 4-7) | Protocol version (currently `0x1`) |
| **Type** | 4 bits | 0 (bits 0-3) | Message type (see Message Types table) |
| **Flags** | 1 byte | 1 | Control flags (bit 0: ACK required, bit 1: Retransmission) |
| **Reserved** | 1 byte | 2 | Reserved for future use (must be `0x00`) |
| **Message ID** | 2 bytes | 3-4 | Unique message identifier (big-endian) |
| **Device ID Length** | 1 byte | 5 | Length of device ID string (max 32 bytes) |
| **Payload Length** | 2 bytes | 6-7 | Length of payload data (big-endian) |
| **Timestamp** | 4 bytes | 8-11 | Unix timestamp (seconds since epoch, big-endian) |
| **Checksum** | 2 bytes | 12-13 | 16-bit XOR checksum of header + payload |
| **Reserved** | 6 bytes | 14-19 | Reserved for future use (must be `0x00`) |

**Message Type Codes:**

| Code | Type | Value |
|------|------|-------|
| `0x1` | REGISTER | Device registration |
| `0x2` | DATA_PUSH | Sensor data transmission |
| `0x3` | ACK | Acknowledgment |
| `0x4` | CMD | Command from server |
| `0x5` | ERROR | Error message |

#### 📦 Message Format Examples

#### 1️⃣ `REGISTER` Message

**Header (20 bytes):**
```
Version: 0x1
Type: 0x1 (REGISTER)
Flags: 0x01 (ACK required)
Reserved: 0x00
Message ID: 0x0001
Device ID Length: 0x08 (8 bytes)
Payload Length: 0x0001 (1 byte for device_type)
Timestamp: 0x673A2F00 (Unix timestamp)
Checksum: 0xXXXX (calculated)
Reserved: 0x000000000000
```

**Payload:**
- Device ID: `ESP32_01` (8 bytes, ASCII)
- Device Type: `0x01` (1 byte: 0x01=EnvNode, 0x02=GasNode, etc.)

**Total Size:** 20 (header) + 8 (device_id) + 1 (device_type) = **29 bytes**

#### 2️⃣ `DATA_PUSH` Message

**Header (20 bytes):**
```
Version: 0x1
Type: 0x2 (DATA_PUSH)
Flags: 0x01 (ACK required)
Reserved: 0x00
Message ID: 0x0002
Device ID Length: 0x08
Payload Length: 0x0006 (6 bytes for sensor data)
Timestamp: 0x673A2F00
Checksum: 0xXXXX
Reserved: 0x000000000000
```

**Payload:**
- Device ID: `ESP32_01` (8 bytes, ASCII)
- Temperature: `0x011C` (2 bytes, big-endian, 28.5°C × 10 = 285 = 0x011C)
- Humidity: `0x0265` (2 bytes, big-endian, 61.3% × 10 = 613 = 0x0265)
- Gas Level: `0x0091` (2 bytes, big-endian, 145 = 0x0091)

**Total Size:** 20 (header) + 8 (device_id) + 6 (sensor_data) = **34 bytes**

#### 3️⃣ `ACK` Message

**Header (20 bytes):**
```
Version: 0x1
Type: 0x3 (ACK)
Flags: 0x00 (no ACK required for ACK)
Reserved: 0x00
Message ID: 0x0003 (matches the acknowledged message ID)
Device ID Length: 0x08
Payload Length: 0x0001 (1 byte for status)
Timestamp: 0x673A2F00
Checksum: 0xXXXX
Reserved: 0x000000000000
```

**Payload:**
- Device ID: `ESP32_01` (8 bytes, ASCII)
- Status: `0x01` (1 byte: 0x01=RECEIVED, 0x02=PROCESSED, 0x03=ERROR)

**Total Size:** 20 (header) + 8 (device_id) + 1 (status) = **29 bytes**

#### 4️⃣ `CMD` Message

**Header (20 bytes):**
```
Version: 0x1
Type: 0x4 (CMD)
Flags: 0x01 (ACK required)
Reserved: 0x00
Message ID: 0x0004
Device ID Length: 0x08
Payload Length: 0x0003 (3 bytes: command_type + value)
Timestamp: 0x673A2F00
Checksum: 0xXXXX
Reserved: 0x000000000000
```

**Payload:**
- Device ID: `ESP32_01` (8 bytes, ASCII)
- Command Type: `0x01` (1 byte: 0x01=UPDATE_RATE, 0x02=RESET, etc.)
- Value: `0x0005` (2 bytes, big-endian, sampling rate in seconds)

**Total Size:** 20 (header) + 8 (device_id) + 3 (command_data) = **31 bytes**

#### 5️⃣ `ERROR` Message

**Header (20 bytes):**
```
Version: 0x1
Type: 0x5 (ERROR)
Flags: 0x00
Reserved: 0x00
Message ID: 0x0005
Device ID Length: 0x08
Payload Length: 0x0002 (2 bytes: error_code)
Timestamp: 0x673A2F00
Checksum: 0xXXXX
Reserved: 0x000000000000
```

**Payload:**
- Device ID: `ESP32_01` (8 bytes, ASCII)
- Error Code: `0x0190` (2 bytes, big-endian: 0x0190 = 400 decimal)

**Error Codes:**
- `0x0190` (400): Invalid message format
- `0x0191` (401): Unauthorized device
- `0x0198` (408): Timeout
- `0x01F4` (500): Internal server error

**Total Size:** 20 (header) + 8 (device_id) + 2 (error_code) = **30 bytes**

#### 🔢 Checksum Calculation

The checksum is a **16-bit XOR** of all bytes in the header (excluding checksum field) and payload:
1. Initialize checksum to `0x0000`
2. XOR with each byte of header (positions 0-11, 14-19, skip 12-13)
3. XOR with each byte of payload
4. Store result in checksum field (positions 12-13, big-endian)

#### 📊 Complete Byte-by-Byte Example: DATA_PUSH Message

**Scenario:** Device `ESP32_01` sending temperature=28.5°C, humidity=61.3%, gas=145

**Complete Message (34 bytes):**

```
Byte Position:  0    1    2    3    4    5    6    7    8    9   10   11   12   13   14   15   16   17   18   19   20-27        28-33
                +----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+------------+------+
Header:         | 12 | 01 | 00 | 00 | 02 | 08 | 00 | 06 | 67 | 3A | 2F | 00 | XX | XX | 00 | 00 | 00 | 00 | 00 | 00 | ESP32_01   | Data |
                +----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+------------+------+
                |Ver |Type|Flgs|Res |    Message ID    |Len| Payload |        Timestamp        |Chksm|      Reserved      |DeviceID|Sensor|
                |0x1 |0x2 |0x01|0x00|     0x0002       |0x08| 0x0006 |     0x673A2F00          |XXXX |   0x000000000000    |ASCII  |Data  |

Sensor Data (6 bytes):
  Byte 28-29: Temperature = 0x011C (285 = 28.5°C × 10, big-endian)
  Byte 30-31: Humidity    = 0x0265 (613 = 61.3% × 10, big-endian)
  Byte 32-33: Gas Level   = 0x0091 (145, big-endian)
```

**Field Breakdown:**
- **Byte 0:** `0x12` = Version (0x1) in bits 4-7, Type (0x2 = DATA_PUSH) in bits 0-3
- **Byte 1:** `0x01` = Flags (bit 0 set = ACK required)
- **Byte 2:** `0x00` = Reserved
- **Bytes 3-4:** `0x0002` = Message ID (big-endian)
- **Byte 5:** `0x08` = Device ID length (8 bytes)
- **Bytes 6-7:** `0x0006` = Payload length (6 bytes for sensor data, big-endian)
- **Bytes 8-11:** `0x673A2F00` = Unix timestamp (big-endian)
- **Bytes 12-13:** Checksum (calculated after header construction)
- **Bytes 14-19:** Reserved (all zeros)
- **Bytes 20-27:** Device ID `ESP32_01` (8 bytes ASCII)
- **Bytes 28-33:** Sensor data (6 bytes, big-endian)

---

## 🔁 Communication Workflow

### Detailed Message Flow

1. **Connection Establishment:**
   - Device initiates TCP connection to server on port `5050`
   - Server accepts connection and waits for first message

2. **Device Registration:**
   - Device sends `REGISTER` message (Type: 0x1) with device ID and device type
   - Server validates device ID and type
   - Server responds with `ACK` (Type: 0x3) with status `RECEIVED` (0x01)
   - If registration fails, server sends `ERROR` (Type: 0x5) with error code

3. **Data Transmission:**
   - Device periodically sends `DATA_PUSH` messages (Type: 0x2) containing sensor readings
   - Each message includes: device ID, timestamp, temperature, humidity, gas level, and checksum
   - Server validates checksum and message format
   - Server stores data in database (if configured)
   - Server responds with `ACK` (Type: 0x3) confirming receipt
   - If checksum fails, server sends `ERROR` (Type: 0x5) with code `0x0190` (400)

4. **Command Processing:**
   - Server may send `CMD` messages (Type: 0x4) to control device behavior
   - Common commands: `UPDATE_RATE` (0x01), `RESET` (0x02)
   - Device receives `CMD`, executes command, and responds with `ACK` (Type: 0x3)
   - If command is invalid, device sends `ERROR` (Type: 0x5)

5. **Error Handling & Retransmission:**
   - If device doesn't receive `ACK` within 3 seconds, it sets retransmission flag and resends message
   - Maximum retry attempts: 3
   - After 3 failed attempts, device logs error and may attempt reconnection
   - Server validates checksum on every message; invalid checksum triggers `ERROR` response

6. **Connection Termination:**
   - Either party can close TCP connection
   - Device should send final `ACK` before closing (optional)
   - Server logs disconnection event  

---

## 🧑‍💻 Developer Guidelines

### Implementation Parameters

| Parameter | Description |
|------------|--------------|
| **Transport Protocol** | TCP |
| **Port** | 5050 |
| **Encoding** | Binary (big-endian byte order) |
| **Timeout** | 3 seconds |
| **Max Packet Size** | 512 bytes |
| **Header Size** | 20 bytes (fixed) |
| **Checksum** | 16-bit XOR of header + payload |
| **Byte Order** | Big-endian (network byte order) |
| **Testing Tools** | Wireshark, SocketTest, Python socket module |

### Implementation Checklist

**For Device (ESP32) Implementation:**
1. Initialize TCP socket connection to server IP:port (5050)
2. Implement message construction functions for each message type
3. Implement checksum calculation (16-bit XOR)
4. Implement retransmission logic with 3-second timeout
5. Parse incoming ACK and CMD messages
6. Handle ERROR messages and log appropriately
7. Implement sensor reading collection (DHT11, MQ2)
8. Convert sensor values to protocol format (multiply by 10 for temperature/humidity)

**For Server Implementation:**
1. Create TCP server socket listening on port 5050
2. Accept multiple concurrent connections (threading/async)
3. Parse incoming messages using header structure
4. Validate checksum on every received message
5. Implement device registry/database for storing device information
6. Store sensor data in database with timestamps
7. Generate and send ACK messages for valid messages
8. Generate and send ERROR messages for invalid/corrupted messages
9. Implement command generation (CMD messages) for device control
10. Handle connection errors and timeouts gracefully

### Protocol Coherence with Requirements

The SIoT-P protocol design directly addresses all functional and non-functional requirements:

- **Functional Requirement 1 (TCP Connection):** ✓ Protocol uses TCP as transport layer
- **Functional Requirement 2 (Periodic Sensor Readings):** ✓ DATA_PUSH message type enables periodic transmission
- **Functional Requirement 3 (Device ID, Timestamp, Checksum):** ✓ All included in header structure
- **Functional Requirement 4 (ACK Messages):** ✓ ACK message type (0x3) with status codes
- **Functional Requirement 5 (Timeout & Retransmission):** ✓ 3-second timeout specified, retransmission flag in header
- **Functional Requirement 6 (CMD Messages):** ✓ CMD message type (0x4) for server-to-device commands
- **Functional Requirement 7 (Multiple Devices):** ✓ Device ID in header enables concurrent multi-device support

- **Non-Functional: Reliability:** ✓ ACK mechanism and retransmission ensure guaranteed delivery
- **Non-Functional: Lightweight:** ✓ Binary fixed-format header (20 bytes) minimizes overhead
- **Non-Functional: Security:** ✓ Checksum validation and optional encryption support
- **Non-Functional: Scalability:** ✓ Stateless design with device IDs supports multiple concurrent connections
- **Non-Functional: Maintainability:** ✓ Clear message structure with version field for future extensions
- **Non-Functional: Interoperability:** ✓ Standard TCP/IP stack ensures compatibility across platforms

### Example: Python TCP Server (Binary Protocol)

```python
import socket
import struct
import time

def calculate_checksum(header_bytes, payload_bytes):
    """Calculate 16-bit XOR checksum"""
    checksum = 0
    # XOR header (skip checksum field at positions 12-13)
    for i in range(12):
        checksum ^= header_bytes[i]
    for i in range(14, 20):
        checksum ^= header_bytes[i]
    # XOR payload
    for byte in payload_bytes:
        checksum ^= byte
    return checksum & 0xFFFF

def parse_siotp_header(data):
    """Parse SIoT-P header (20 bytes)"""
    if len(data) < 20:
        return None
    
    version_type = data[0]
    version = (version_type >> 4) & 0x0F
    msg_type = version_type & 0x0F
    flags = data[1]
    reserved = data[2]
    msg_id = struct.unpack('>H', data[3:5])[0]  # big-endian
    device_id_len = data[5]
    payload_len = struct.unpack('>H', data[6:8])[0]  # big-endian
    timestamp = struct.unpack('>I', data[8:12])[0]  # big-endian
    checksum = struct.unpack('>H', data[12:14])[0]  # big-endian
    
    return {
        'version': version,
        'type': msg_type,
        'flags': flags,
        'message_id': msg_id,
        'device_id_len': device_id_len,
        'payload_len': payload_len,
        'timestamp': timestamp,
        'checksum': checksum
    }

def create_ack_message(msg_id, device_id):
    """Create ACK message"""
    version_type = (0x1 << 4) | 0x3  # Version 1, Type ACK
    flags = 0x00
    reserved = 0x00
    device_id_bytes = device_id.encode('ascii')
    device_id_len = len(device_id_bytes)
    payload = b'\x01'  # Status: RECEIVED
    payload_len = len(payload)
    timestamp = int(time.time())
    
    # Build header (without checksum)
    header = bytearray(20)
    header[0] = version_type
    header[1] = flags
    header[2] = reserved
    struct.pack_into('>H', header, 3, msg_id)
    header[5] = device_id_len
    struct.pack_into('>H', header, 6, payload_len)
    struct.pack_into('>I', header, 8, timestamp)
    # Checksum field (12-13) left as 0 for calculation
    
    # Calculate checksum
    checksum = calculate_checksum(header, device_id_bytes + payload)
    struct.pack_into('>H', header, 12, checksum)
    
    return bytes(header) + device_id_bytes + payload

# Server setup
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('0.0.0.0', 5050))
server.listen(5)

print("SIoT-P Server Listening on Port 5050...")

while True:
    conn, addr = server.accept()
    print(f"Connection from {addr}")
    
    try:
        while True:
            # Receive header (20 bytes)
            header_data = conn.recv(20)
            if len(header_data) < 20:
                break
            
            header = parse_siotp_header(header_data)
            if not header:
                break
            
            # Receive device ID and payload
            device_id = conn.recv(header['device_id_len']).decode('ascii')
            payload = conn.recv(header['payload_len'])
            
            # Verify checksum
            calculated_checksum = calculate_checksum(header_data, device_id.encode('ascii') + payload)
            if calculated_checksum != header['checksum']:
                print("Checksum mismatch!")
                continue
            
            print(f"Received: Type={header['type']}, Device={device_id}, ID={header['message_id']}")
            
            # Send ACK
            ack_msg = create_ack_message(header['message_id'], device_id)
            conn.send(ack_msg)
            
    except Exception as e:
        print(f"Error: {e}")
    finally:
        conn.close()
```

---

## ⚠️ Error Handling & Security

### Error Types
| Code (Hex) | Code (Decimal) | Description | Action |
|------------|-----------------|-------------|--------|
| `0x0190` | 400 | Invalid message format | Send `ERROR` message with code |
| `0x0191` | 401 | Unauthorized device | Send `ERROR` and reject connection |
| `0x0198` | 408 | Timeout waiting for ACK | Retransmit message |
| `0x01F4` | 500 | Internal server error | Send `ERROR`, log error, retry |

### Security Features
- Basic checksum verification for integrity.  
- Optional symmetric encryption for secure deployments.  
- Device authentication via pre-registered IDs.

---

## 🚀 Future Enhancements

- **Encryption Layer:** Add AES-128 or TLS for secure transmission.  
- **QoS Levels:** Implement multiple delivery guarantees (like MQTT).  
- **Over-the-Air Updates:** Server-initiated firmware updates.  
- **REST API Gateway:** Integrate SIoT-P with cloud dashboards.  
- **Offline Mode:** Local caching when connection unavailable.  

---

## 📂 Repository Structure

```
CN_Assignment_02/
│
├── README.md              ← Main protocol and documentation
├── docs/                  ← PDF + supplementary reports
├── code/                  ← ESP32 / Python demo code
├── captures/              ← Wireshark screenshots
├── assets/                ← Architecture & flow diagrams
└── prompts/               ← ChatGPT prompt logs (Honor Code)
```

---

## 📜 Honor Code Statement

This project was developed solely for the **Computer Networks (Comp-352)** CCP under the supervision of **Engr. Adnaan Iqbaal**.  
All AI assistance was used responsibly and documented under `/prompts/` as per institutional policy.  
Work is original and has not been shared with other groups.  

---

## 📅 Submission Details

- **Deadline:** November 18, 2025 (11:59 PM PKT)  
- **Repository Privacy:** Private  
- **Shared With:** `adnaaniqbaal`  
- **Last Commit Before:** Deadline (no commits after this will be accepted)

---

## 🧠 Learning Outcomes

By completing this activity, students demonstrate:
- Understanding of **application-layer network design**.  
- Skills in **protocol specification and message structuring**.  
- Ability to apply **modern tools** (GitHub, Wireshark, ESP32).  
- Awareness of **real-world networking and IoT challenges**.

---

## 📚 References

- RFC 7252 – Constrained Application Protocol (CoAP)  
- MQTT v3.1.1 OASIS Standard  
- Espressif Systems – ESP32 Wi-Fi Programming Guide  
- Python `socket` Documentation (docs.python.org)

---

## 🧩 Maintainers

**Muhammad Abdullah (B23F0001AI006)**  
Department of Artificial Intelligence  
Pak-Austria Fachhochschule: Institute of Applied Sciences and Technology (PAF-IAST)

---

**© 2025 – Smart IoT Device Communication Protocol (SIoT-P)**  
_All rights reserved for academic use under PAF-IAST guidelines._
