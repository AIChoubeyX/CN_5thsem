# Module 2: Data Link Layer - Complete Guide

## Table of Contents
1. [Framing](#1-framing)
2. [Error Control](#2-error-control)
3. [Error Detection and Correction](#3-error-detection-and-correction)
4. [Flow Control](#4-flow-control)
5. [Data Link Protocols](#5-data-link-protocols)
6. [Sliding Window Protocols](#6-sliding-window-protocols)
7. [HDLC and PPP](#7-hdlc-and-ppp)
8. [Medium Access Control](#8-medium-access-control)
9. [Channel Allocation](#9-channel-allocation)
10. [Multiple Access Protocols](#10-multiple-access-protocols)
11. [Ethernet and Wireless LANs](#11-ethernet-and-wireless-lans)
12. [Exam Questions](#exam-questions)

---

## 1. Framing

### What is Framing?
**Simple Explanation:**  
Imagine you're sending a long letter by breaking it into pages and putting each page in an envelope. Framing is like putting data into envelopes so the receiver knows where one message ends and another begins!

**Technical Definition:**  
Framing is the process of dividing the bit stream from the physical layer into manageable data units called **frames**. Each frame has a specific structure with headers and trailers that help identify the beginning and end of the frame.

### Why Do We Need Framing?

**Without Framing:**
```
01001000010010010010000001110100011010000110010101110010011001010
```
How do you know where one message ends and another begins? Impossible!

**With Framing:**
```
[Start|01001000|End] [Start|01101001|End] [Start|01110010|End]
```
Clear separation! Each frame is distinct.

**Real-World Analogy:**  
Think of a train with multiple carriages. Each carriage (frame) carries passengers (data) and has doors (start/end markers) that show where one carriage begins and ends.

### Frame Structure

```
┌──────────┬──────────┬───────────────┬──────────┬──────────┐
│  Flag    │  Header  │    Payload    │ Trailer  │   Flag   │
│ (Start)  │(Address, │     (Data)    │  (CRC,   │  (End)   │
│          │ Control) │               │  Error)  │          │
└──────────┴──────────┴───────────────┴──────────┴──────────┘
```

**Components Explained:**

1. **Flag (Start Delimiter)**
   - Special bit pattern marking frame start
   - Example: 01111110 (in HDLC)
   - Receiver knows: "New frame starting!"

2. **Header**
   - **Source Address**: Who sent it
   - **Destination Address**: Who should receive it
   - **Control Information**: Frame type, sequence number
   - **Length**: How big is the data

3. **Payload (Data)**
   - The actual data being transmitted
   - Can be 46-1500 bytes (Ethernet)
   - This is what you want to send

4. **Trailer**
   - **CRC (Cyclic Redundancy Check)**: Error detection
   - **Padding**: Extra bits if needed
   - Error checking information

5. **Flag (End Delimiter)**
   - Marks frame end
   - Same pattern as start flag
   - Receiver knows: "Frame complete!"

### Framing Methods

#### 1. Character Count Method
**How it works:**  
First field tells how many characters are in the frame.

```
Frame: | 5 | A | B | C | D |
         ↑
      Count = 5 characters total
```

**Example:**
```
| 8 | H | E | L | L | O | ! | ! |
  ↑
  8 bytes including count
```

**Advantages:**
- ✅ Simple to implement
- ✅ Efficient (no extra flags)

**Disadvantages:**
- ❌ If count gets corrupted, synchronization lost
- ❌ One error breaks all following frames
- ❌ Rarely used in practice

**Example Scenario:**
```
Correct:  [5|A|B|C|D] [4|X|Y|Z]
Corrupted: [9|A|B|C|D] [reads 4 more bytes from next frame!]
```

#### 2. Flag Bytes with Byte Stuffing
**How it works:**  
Use special flag byte (like 0x7E) to mark start and end.

```
[FLAG] Data Data Data [FLAG] [FLAG] Data [FLAG]
  ↑                      ↑      ↑            ↑
Start                  End    Start        End
```

**Problem:** What if data contains the flag byte?

**Solution: Byte Stuffing (Escaping)**
```
Data contains FLAG:  ...Data [FLAG] Data...
Replace with:        ...Data [ESC][FLAG] Data...
```

**If data contains ESC:**
```
Data contains ESC:   ...Data [ESC] Data...
Replace with:        ...Data [ESC][ESC] Data...
```

**Complete Example:**
```
Original Data: A B FLAG C ESC D

After Stuffing: [FLAG] A B ESC-FLAG C ESC-ESC D [FLAG]
                  ↑                                  ↑
                Start                              End

Receiver removes ESC characters to get original data
```

**Advantages:**
- ✅ Resynchronization possible after error
- ✅ Easy to implement
- ✅ Widely used (PPP protocol)

**Disadvantages:**
- ❌ Overhead increases if data contains many flags
- ❌ Processing overhead for stuffing/unstuffing

#### 3. Flag Bits with Bit Stuffing
**How it works:**  
Use bit pattern 01111110 as flag.

**Rule:** Never allow 6 consecutive 1s in data (except flag)

**Bit Stuffing Process:**
- Sender: After five 1s, insert a 0
- Receiver: After five 1s, remove next 0

**Example:**
```
Original:     0 1 1 1 1 1 1 0 1 0
                    ↑
              Five 1s detected

After Stuffing: 0 1 1 1 1 1 0 1 1 0 1 0
                          ↑
                      0 inserted

Receiver sees five 1s, removes next 0, gets original
```

**Complete Frame Example:**
```
Flag:     01111110
Data:     01111111 00111110
Stuffed:  011111011 00111110
Frame:    [01111110] 011111011 00111110 [01111110]
           Start                              End
```

**Advantages:**
- ✅ Works at bit level (more efficient)
- ✅ No restriction on data content
- ✅ Used in HDLC, PPP

**Disadvantages:**
- ❌ Slightly complex implementation
- ❌ Variable frame size

#### 4. Physical Layer Coding Violations
**How it works:**  
Use signal patterns that are normally invalid as frame markers.

**Example in Manchester Encoding:**
```
Normal:  High-to-Low = 1,  Low-to-High = 0
Invalid: High-to-High or Low-to-Low (normally not used)

Frame Marker: Use High-to-High as START
              Use Low-to-Low as END
```

**Advantages:**
- ✅ No overhead in data
- ✅ Easy synchronization
- ✅ Physical layer handles it

**Disadvantages:**
- ❌ Only works with specific encodings
- ❌ Depends on physical layer

### Frame Size Considerations

**Small Frames:**
- ✅ Less retransmission if error
- ✅ Lower latency
- ❌ More overhead (more headers)
- ❌ Less efficient

**Large Frames:**
- ✅ More efficient (less overhead percentage)
- ✅ Higher throughput
- ❌ Entire frame must be retransmitted if error
- ❌ Higher latency

**Optimal Frame Size:**
```
Efficiency = (Data Size) / (Data Size + Header + Trailer)

Example with 1500-byte data, 18-byte overhead:
Efficiency = 1500 / (1500 + 18) = 98.8%

Example with 64-byte data, 18-byte overhead:
Efficiency = 64 / (64 + 18) = 78%
```

### Real-World Examples

**Ethernet Frame:**
```
┌────┬────┬────┬────┬────────┬─────┐
│ 8  │ 6  │ 6  │ 2  │46-1500 │  4  │ bytes
├────┼────┼────┼────┼────────┼─────┤
│Pre-│Dest│Src │Type│ Data   │ CRC │
│amble│MAC│MAC │    │        │     │
└────┴────┴────┴────┴────────┴─────┘
```

**PPP Frame:**
```
┌────┬────┬────┬────────┬────┬────┐
│ 1  │ 1  │ 1  │Variable│ 2  │ 1  │ bytes
├────┼────┼────┼────────┼────┼────┤
│Flag│Addr│Ctrl│ Data   │CRC │Flag│
│7E  │FF  │03  │        │    │ 7E │
└────┴────┴────┴────────┴────┴────┘
```

---

## 2. Error Control

### What is Error Control?
**Simple Explanation:**  
Imagine you're sending a toy by mail. Error control is like having insurance - if the toy breaks during shipping, you can get a new one sent!

**Technical Definition:**  
Error control ensures reliable delivery of frames despite errors in transmission. It involves detecting errors and taking corrective action (retransmission or correction).

### Types of Errors

#### 1. Single-Bit Error
One bit changes during transmission.

```
Sent:     0 1 0 0 1 1 0 0
Received: 0 1 0 1 1 1 0 0
                  ↑
              Error here
```

**Causes:**
- Electrical noise spike
- Brief interference
- Rare in modern systems

#### 2. Burst Error
Multiple consecutive bits affected.

```
Sent:     0 1 0 0 1 1 0 0 1 1 0 1
Received: 0 1 0 1 0 1 1 1 1 1 0 1
                  ↑_______↑
              Burst of errors
```

**Causes:**
- Lightning
- Motor starting
- Microwave interference
- More common than single-bit errors

**Burst Length:** Number of bits from first error to last error

#### 3. Random Errors
Scattered errors throughout data.

```
Sent:     0 1 0 0 1 1 0 0 1 1 0 1 0 0
Received: 0 0 0 0 1 1 1 0 1 1 0 0 0 0
            ↑           ↑       ↑   ↑
         Scattered errors
```

**Causes:**
- General background noise
- Multiple interference sources
- Thermal noise

### Error Control Techniques

#### 1. Error Detection
**Goal:** Determine if frame has errors.

**Methods:**
- Parity Check
- Checksum
- Cyclic Redundancy Check (CRC)

**Process:**
1. Sender calculates error-detection code
2. Appends it to frame
3. Receiver recalculates code
4. Compares with received code
5. If match → No error (probably)
6. If no match → Error detected

#### 2. Error Correction
**Goal:** Fix errors without retransmission.

**Methods:**
- Hamming Code
- Reed-Solomon Code
- Turbo Codes

**Process:**
1. Sender adds redundant bits
2. Receiver uses redundancy to locate and fix errors
3. No retransmission needed

#### 3. Retransmission (ARQ)
**Goal:** Request retransmission when error detected.

**Types:**
- Stop-and-Wait ARQ
- Go-Back-N ARQ
- Selective Repeat ARQ

### Acknowledgments (ACK) and Negative Acknowledgments (NAK)

**ACK (Positive Acknowledgment):**
```
Sender → [Frame 1] → Receiver
Sender ← [ACK 1] ← Receiver
"Got it! Thanks!"
```

**NAK (Negative Acknowledgment):**
```
Sender → [Frame 1 with errors] → Receiver
Sender ← [NAK 1] ← Receiver
"Error! Send again!"
```

### Timeout Mechanism

**What if ACK is lost?**

```
Sender → [Frame 1] → Receiver
Sender ← [ACK 1 - Lost!] ← Receiver

Sender waits... Timer expires!
Sender → [Frame 1 again] → Receiver
Sender ← [ACK 1] ← Receiver
```

**Timer Value:**
- Too short: Unnecessary retransmissions
- Too long: Waste time waiting
- Optimal: Slightly more than round-trip time (RTT)

### Sequence Numbers

**Why needed?**  
Distinguish between new frames and retransmissions.

**Example:**
```
Without sequence numbers:
Frame → Receiver (ACK lost)
Frame → Receiver receives same data twice!
```

```
With sequence numbers:
Frame 1 → Receiver (ACK lost)
Frame 1 → Receiver sees same number, ignores duplicate
```

**Size:** Usually 1-bit (0,1) or multiple bits (0,1,2,3...)

---

## 3. Error Detection and Correction

### Parity Check

#### Simple Parity (Even Parity)
**Rule:** Add 1 parity bit so total number of 1s is even.

**Example 1:**
```
Data: 1 0 1 1 0 1 0
Count of 1s: 4 (even)
Parity bit: 0 (to keep it even)
Transmitted: 1 0 1 1 0 1 0 [0]
```

**Example 2:**
```
Data: 1 0 1 1 0 1 1
Count of 1s: 5 (odd)
Parity bit: 1 (to make it even)
Transmitted: 1 0 1 1 0 1 1 [1]
```

**Detection:**
```
Received: 1 0 1 1 0 1 1 1
Count of 1s: 6 (even) ✓ No error detected

Received: 1 0 0 1 0 1 1 1
            ↑ error
Count of 1s: 5 (odd) ✗ Error detected!
```

**Limitations:**
- Cannot detect even number of errors
- Cannot correct errors
- Simple but weak

#### Two-Dimensional Parity
**Arrange data in rows and columns, add parity to each.**

```
Original Data (12 bits):
1 0 1 1  →  Parity: 1
1 1 0 1  →  Parity: 1
0 1 1 0  →  Parity: 0
↓ ↓ ↓ ↓
Parity:
0 0 0 0

Complete Block:
1 0 1 1 | 1
1 1 0 1 | 1
0 1 1 0 | 0
────────┼──
0 0 0 0 | 0
```

**Error Detection and Correction:**
```
Received with 1 error:
1 0 0 1 | 1   ← Row parity fails
  ↑
1 1 0 1 | 1
0 1 1 0 | 0
────────┼──
0 1 0 0 | 0   ← Column parity fails
    ↑

Intersection shows error location!
Correct it: change 0 to 1
```

**Advantages:**
- ✅ Can detect and correct single-bit errors
- ✅ Can detect burst errors
- ✅ Simple to implement

**Disadvantages:**
- ❌ Cannot correct multiple errors
- ❌ Higher overhead

### Checksum

**How it works:**  
Add all data words, send the sum.

**Example (8-bit checksum):**
```
Data words:
  10110011
  01010101
  11001100
  ────────
  Sum (with carry):
  100100100
  
Wrap around carry:
  00100100
  +       1
  ────────
  00100101 ← Checksum

Transmitted: Data + Checksum
```

**Receiver:**
```
Adds all received words + checksum
If sum = 00000000 (or 11111111), no error
If sum ≠ 00000000, error detected
```

**Internet Checksum (16-bit):**
- Used in IP, TCP, UDP
- Add all 16-bit words
- Wrap around carry
- Take 1's complement

**Advantages:**
- ✅ Simple to compute
- ✅ Reasonable error detection
- ✅ Low overhead

**Disadvantages:**
- ❌ Weak against some error patterns
- ❌ Cannot correct errors
- ❌ May miss errors in specific cases

### Cyclic Redundancy Check (CRC)

**Most Powerful Error Detection Method!**

**How it works:**  
Treat data as a polynomial, divide by generator polynomial.

**Mathematical Basis:**
```
Data (D): 1101 (polynomial: x³ + x² + 1)
Generator (G): 1011 (polynomial: x³ + x + 1)

Append zeros: 1101000 (shift left by 3 for G of length 4)
Divide by G: Get remainder R
Send: D + R
```

**Example Calculation:**
```
Data: 1101
Generator: 1011 (4 bits)
Append 3 zeros: 1101000

Division (XOR, no carries):
        1111
     ────────
1011 | 1101000
       1011
       ────
        1100
        1011
        ────
         1110
         1011
         ────
          1010
          1011
          ────
          001 ← Remainder (CRC)

Transmitted: 1101[001]
             Data CRC
```

**Receiver:**
```
Receives: 1101001
Divides by 1011
If remainder = 000, no error
If remainder ≠ 000, error detected
```

**Standard CRC Polynomials:**

| Name | Polynomial | Bits | Used In |
|------|------------|------|---------|
| CRC-8 | x⁸+x²+x+1 | 8 | ATM |
| CRC-16 | x¹⁶+x¹⁵+x²+1 | 16 | USB |
| CRC-32 | x³²+x²⁶+x²³+...+1 | 32 | Ethernet, ZIP |
| CRC-CCITT | x¹⁶+x¹²+x⁵+1 | 16 | Bluetooth |

**Detection Capability:**
- ✅ All single-bit errors
- ✅ All double-bit errors
- ✅ All odd number of errors
- ✅ All burst errors ≤ r bits (r = CRC length)
- ✅ 99.99% of longer burst errors

**Advantages:**
- ✅ Very powerful error detection
- ✅ Efficient in hardware
- ✅ Widely used (Ethernet, Wi-Fi)
- ✅ Low probability of undetected errors

**Disadvantages:**
- ❌ Cannot correct errors (only detect)
- ❌ More complex than parity
- ❌ Fixed overhead

### Hamming Code (Error Correction)

**Can detect AND correct single-bit errors!**

**Key Concept:**  
Add redundant parity bits at powers-of-2 positions.

**For d data bits:**
```
Number of parity bits r satisfies:
2^r ≥ d + r + 1
```

**Example: 4 data bits**
```
2^r ≥ 4 + r + 1
r = 3 parity bits needed

Total = 7 bits (positions 1-7)
```

**Position Assignment:**
```
Position:    1   2   3   4   5   6   7
             P1  P2  D1  P4  D2  D3  D4
             ↑   ↑       ↑
           Parity      Parity
           bits        bits
```

**Parity Bit Coverage:**
```
P1 (pos 1): Checks positions with bit 0 set in binary
            1, 3, 5, 7 (001, 011, 101, 111)

P2 (pos 2): Checks positions with bit 1 set
            2, 3, 6, 7 (010, 011, 110, 111)

P4 (pos 4): Checks positions with bit 2 set
            4, 5, 6, 7 (100, 101, 110, 111)
```

**Encoding Example:**
```
Data: 1011

Step 1: Place data
Position: 1  2  3  4  5  6  7
          P1 P2 D1 P4 D2 D3 D4
          ?  ?  1  ?  0  1  1

Step 2: Calculate P1 (positions 1,3,5,7)
       3  5  7 → 1, 0, 1
       XOR: 1⊕0⊕1 = 0
       P1 = 0

Step 3: Calculate P2 (positions 2,3,6,7)
       3  6  7 → 1, 1, 1
       XOR: 1⊕1⊕1 = 1
       P2 = 1

Step 4: Calculate P4 (positions 4,5,6,7)
       5  6  7 → 0, 1, 1
       XOR: 0⊕1⊕1 = 0
       P4 = 0

Encoded: 0110011
```

**Error Detection and Correction:**
```
Received: 0110111 (error at position 5)
          0  1  1  0  1  1  1

Check P1: Positions 1,3,5,7 → 0,1,1,1 (XOR=1) ✗
Check P2: Positions 2,3,6,7 → 1,1,1,1 (XOR=0) ✓
Check P4: Positions 4,5,6,7 → 0,1,1,1 (XOR=1) ✗

Error position = P4(4) + P1(1) = 101 binary = 5
Flip bit at position 5: 1→0
Corrected: 0110011
```

**Advantages:**
- ✅ Can correct single-bit errors
- ✅ Can detect double-bit errors
- ✅ No retransmission needed

**Disadvantages:**
- ❌ High overhead (3 parity bits for 4 data bits)
- ❌ Only single-bit correction
- ❌ Complex implementation

---

## 4. Flow Control

### What is Flow Control?
**Simple Explanation:**  
Imagine you're filling a bucket with water from a hose. If you pour too fast, water overflows! Flow control is like adjusting the water speed so the bucket doesn't overflow.

**Technical Definition:**  
Flow control is a mechanism to ensure that a fast sender doesn't overwhelm a slow receiver by controlling the rate of data transmission.

### Why Do We Need Flow Control?

**Problem Without Flow Control:**
```
Fast Sender (100 Mbps) → Slow Receiver (10 Mbps)
                         Receiver buffer overflows!
                         Data lost!
```

**With Flow Control:**
```
Fast Sender → Matches → Receiver
              speed     (No overflow!)
```

**Real-World Analogy:**  
It's like talking to someone. If you speak too fast, they can't understand. You need to speak at a pace they can process!

### Flow Control Mechanisms

#### 1. Stop-and-Wait Protocol

**How it works:**  
Sender sends ONE frame, then WAITS for acknowledgment before sending next frame.

```
Timeline:

Sender: [Send Frame 1] ----→ Wait ←---- [Receive ACK 1] [Send Frame 2]
                                |
Receiver:              [Receive] [Process] [Send ACK] ←--
```

**Detailed Process:**
```
Step 1: Sender transmits Frame 0
Step 2: Sender starts timer
Step 3: Sender STOPS and WAITS
Step 4: Receiver receives Frame 0
Step 5: Receiver processes frame
Step 6: Receiver sends ACK 0
Step 7: Sender receives ACK 0
Step 8: Sender cancels timer
Step 9: Sender sends Frame 1
Repeat...
```

**Example with Numbers:**
```
Sender              Receiver
  |                    |
  |--Frame 0--------→  |
  |                    | (Processing)
  |  ←---------ACK 0---|
  |                    |
  |--Frame 1--------→  |
  |                    | (Processing)
  |  ←---------ACK 1---|
  |                    |
```

**Efficiency Calculation:**
```
Transmission Time (Tt) = Frame Size / Bandwidth
Propagation Time (Tp) = Distance / Signal Speed
Round Trip Time (RTT) = 2 × Tp
Total Time = Tt + RTT

Efficiency = Tt / (Tt + RTT)

Example:
Frame = 1000 bits
Bandwidth = 1 Mbps → Tt = 1 ms
Distance = 10 km
Speed = 200,000 km/s → Tp = 0.05 ms
RTT = 0.1 ms

Efficiency = 1 / (1 + 0.1) = 90.9%
```

**Advantages:**
- ✅ Very simple to implement
- ✅ No receiver buffer needed
- ✅ Ensures receiver isn't overwhelmed
- ✅ Easy error recovery

**Disadvantages:**
- ❌ Very inefficient (sender waits most of the time)
- ❌ Low bandwidth utilization
- ❌ Poor for long-distance/high-speed links
- ❌ Channel remains idle during ACK wait

**When to Use:**
- Short distance networks
- Slow networks
- Simple systems
- Half-duplex links

#### 2. Sliding Window Protocol

**How it works:**  
Sender can send multiple frames before receiving ACK, up to a **window size**.

**Window Concept:**
```
Frames:  [1][2][3][4][5][6][7][8][9][10]
          └─────────┘
         Window Size = 4
         Can send 4 frames without waiting!
```

**Sender Window:**
```
Sent & ACKed | Sent, Not ACKed | Can Send | Can't Send Yet
   [1][2]    |    [3][4][5]     | [6][7]  |   [8][9][10]
             └─── Window Size = 5 ───┘
```

**Receiver Window:**
```
Received & ACKed | Can Receive | Can't Receive Yet
    [1][2][3]    |   [4][5]    |    [6][7][8]
                 └─ Window ──┘
```

**Window Sliding:**
```
Initial:    [1][2][3][4] 5  6  7  8
            └─Window─┘

After ACK 1: 1 [2][3][4][5] 6  7  8
               └─Window─┘
               (Window slides!)

After ACK 2: 1  2 [3][4][5][6] 7  8
                  └─Window─┘
```

**Efficiency:**
```
Window Size (W) = Bandwidth × RTT / Frame Size

Efficiency = W / (1 + 2a)
where a = Propagation Time / Transmission Time

For 100% efficiency: W ≥ 1 + 2a
```

**Example:**
```
Frame = 1000 bits
Bandwidth = 1 Mbps
Distance = 10 km
Tp = 0.05 ms, Tt = 1 ms
a = 0.05/1 = 0.05

Optimal Window = 1 + 2(0.05) = 1.1 ≈ 2 frames
With W=2: Efficiency = 2/(1+0.1) = 181% → 100% (maxed)
```

**Advantages:**
- ✅ Much more efficient than Stop-and-Wait
- ✅ Better bandwidth utilization
- ✅ Good for long-distance links
- ✅ Pipeline transmission

**Disadvantages:**
- ❌ More complex implementation
- ❌ Requires buffer space
- ❌ Sequence number management needed

---

## 5. Data Link Protocols

### Stop-and-Wait ARQ (Automatic Repeat Request)

**Complete Protocol:**

```
Sender Side:
1. Send frame with sequence number
2. Start timer
3. Wait for ACK
4. If ACK received: Send next frame
5. If timeout: Retransmit same frame

Receiver Side:
1. Receive frame
2. Check for errors (CRC)
3. If no error: Send ACK, deliver data
4. If error: Discard frame (sender will timeout)
```

**Sequence Numbers:**  
Use alternating 0 and 1 (1-bit sequence number)

**Normal Operation:**
```
Sender          Receiver
  |               |
  |--Frame 0---→  | (Check: OK)
  |               | (Deliver to upper layer)
  |  ←---ACK 0----| 
  |               |
  |--Frame 1---→  | (Check: OK)
  |               |
  |  ←---ACK 1----|
  |               |
```

**Lost Frame:**
```
Sender          Receiver
  |               |
  |--Frame 0--X   | (Frame lost!)
  | (Timeout!)    |
  |--Frame 0---→  | (Retransmit)
  |  ←---ACK 0----|
  |               |
```

**Lost ACK:**
```
Sender          Receiver
  |               |
  |--Frame 0---→  | (Received)
  |   X←--ACK 0---| (ACK lost!)
  | (Timeout!)    |
  |--Frame 0---→  | (Duplicate! Already have it)
  |               | (Discard, send ACK again)
  |  ←---ACK 0----|
  |               |
```

**Delayed ACK:**
```
Sender          Receiver
  |               |
  |--Frame 0---→  | (Received)
  | (Timeout!)    | (ACK delayed)
  |--Frame 0---→  | (Duplicate!)
  |               |
  |  ←---ACK 0----| (Finally arrives)
  |  ←---ACK 0----| (Second ACK)
  |--Frame 1---→  | (Continue normally)
```

**Corrupted Frame:**
```
Sender          Receiver
  |               |
  |--Frame 0---→  | (CRC error!)
  |               | (Discard silently)
  | (Timeout!)    |
  |--Frame 0---→  | (Retransmit)
  |  ←---ACK 0----|
```

### Go-Back-N ARQ

**How it works:**  
Sender can send N frames without ACK. If error, retransmit from error frame onwards.

**Window Size:**  
Sender: N frames  
Receiver: 1 frame (only accepts in order)

**Normal Operation:**
```
Sender sends: 0, 1, 2, 3, 4 (Window = 5)

Receiver: ACK 0, ACK 1, ACK 2, ACK 3, ACK 4
All frames received in order → All accepted
```

**Frame Lost:**
```
Sender:    0  1  2  3  4  5  6
              ↓  X (Frame 2 lost!)
Receiver:  0  1  3  4  5  6
ACK:       0  1  (3,4,5,6 discarded!)
              Waits for Frame 2

Sender timeout for Frame 2:
Go back to 2: Retransmit 2, 3, 4, 5, 6
              (Even though 3,4,5,6 already sent!)
```

**Timeline Example:**
```
Time   Sender             Receiver
  1    Send 0 ----------→ Receive 0, ACK 0
  2    Send 1 ----------→ Receive 1, ACK 1
  3    Send 2 -----X      Lost!
  4    Send 3 ----------→ Out of order! Discard, ACK 1
  5    Send 4 ----------→ Out of order! Discard, ACK 1
  6    (Timeout for 2)
  7    Send 2 ----------→ Receive 2, ACK 2
  8    Send 3 ----------→ Receive 3, ACK 3
  9    Send 4 ----------→ Receive 4, ACK 4
```

**Sequence Number Size:**
```
For window size N:
Sequence numbers needed = N + 1

Example: Window = 7
Sequence numbers: 0, 1, 2, 3, 4, 5, 6, 7 (8 total)
```

**Advantages:**
- ✅ More efficient than Stop-and-Wait
- ✅ Simple receiver (no buffer for out-of-order)
- ✅ Continuous transmission

**Disadvantages:**
- ❌ Retransmits good frames after error
- ❌ Wastes bandwidth on retransmission
- ❌ Efficiency drops with high error rate

### Selective Repeat ARQ

**How it works:**  
Sender can send N frames. Receiver accepts out-of-order frames and buffers them. Only damaged frames retransmitted.

**Window Size:**  
Sender: N frames  
Receiver: N frames (can accept out of order)

**Normal Operation:**
```
Sender sends: 0, 1, 2, 3, 4
Receiver: Accepts all, ACKs each individually
Buffer: [0][1][2][3][4] → Deliver in order
```

**Frame Lost:**
```
Sender:    0  1  2  3  4  5  6
              ↓  X (Frame 2 lost!)
Receiver:  0  1  -  3  4  5  6
Buffer:    [0][1][_][3][4][5][6]
ACK:       0  1  3  4  5  6 (Individual ACKs!)

Sender timeout for Frame 2:
Retransmit ONLY Frame 2

Receiver gets Frame 2:
Buffer: [0][1][2][3][4][5][6]
Deliver all to upper layer in order!
```

**Timeline Example:**
```
Time   Sender             Receiver
  1    Send 0 ----------→ Receive 0, Buffer[0], ACK 0
  2    Send 1 ----------→ Receive 1, Buffer[1], ACK 1
  3    Send 2 -----X      Lost!
  4    Send 3 ----------→ Receive 3, Buffer[_,3], ACK 3
  5    Send 4 ----------→ Receive 4, Buffer[_,3,4], ACK 4
  6    (Timeout for 2)
  7    Send 2 ----------→ Receive 2, Buffer[2,3,4]
  8                       Deliver 0,1,2,3,4 in order
```

**Sequence Number Size:**
```
For window size N:
Sequence numbers needed = 2N

Example: Window = 4
Sequence numbers: 0-7 (8 total, 3 bits)

Why? To distinguish between:
- Old frame 0 (from previous window)
- New frame 0 (from current window)
```

**Advantages:**
- ✅ Most efficient ARQ protocol
- ✅ Only retransmits damaged frames
- ✅ No waste of good frames
- ✅ Best for high error rates

**Disadvantages:**
- ❌ Complex receiver (needs buffer)
- ❌ More memory required
- ❌ Complex implementation
- ❌ Need more sequence numbers

### Protocol Comparison Table

| Feature | Stop-and-Wait | Go-Back-N | Selective Repeat |
|---------|---------------|-----------|------------------|
| **Sender Window** | 1 | N | N |
| **Receiver Window** | 1 | 1 | N |
| **Sequence Numbers** | 2 | N+1 | 2N |
| **Receiver Buffer** | Not needed | Not needed | Needed |
| **Retransmission** | 1 frame | N frames | 1 frame |
| **Efficiency** | Lowest | Medium | Highest |
| **Complexity** | Simplest | Medium | Most complex |
| **Best Use** | Low speed | Medium speed | High speed |
| **Error Handling** | Retransmit 1 | Retransmit N | Retransmit 1 |

---

## 6. Sliding Window Protocols in Detail

### Window Management

**Sender Window Variables:**
```
- Send Window Size (SWS): Maximum frames that can be sent
- Last Acknowledgment Received (LAR): Last frame ACKed
- Last Frame Sent (LFS): Last frame transmitted

Condition: LFS - LAR ≤ SWS
```

**Receiver Window Variables:**
```
- Receive Window Size (RWS): Maximum frames that can be received
- Last Frame Received (LFR): Last frame delivered to upper layer
- Largest Acceptable Frame (LAF): Last frame receiver can accept

Condition: LAF - LFR ≤ RWS
```

**Example:**
```
SWS = 4, RWS = 4
Sequence numbers: 0-7 (3 bits)

Sender Window:
Frames: 0 1 2 3 4 5 6 7 0 1 2
        └─SWS=4─┘
        Can send 0-3

After sending 0,1,2,3:
LFS = 3, LAR = -1 (none ACKed yet)
LFS - LAR = 4 → Window full! Must wait.

Receive ACK 0:
LAR = 0
LFS - LAR = 3 → Can send 1 more!
Send frame 4, LFS = 4
```

### Sequence Number Wrap-Around

**Problem:**
```
Sequence: 0,1,2,3,4,5,6,7,0,1,2,3...
                        ↑
          Is this old 0 or new 0?
```

**Solution Requirements:**
```
Go-Back-N: Seq# ≥ Window Size + 1
Selective Repeat: Seq# ≥ 2 × Window Size

Why?
Prevents confusion between old and new frames
```

**Example Problem:**
```
Window = 4, Seq# = 0-3 (WRONG!)

Sender sends: 0,1,2,3
All ACKs lost
Sender retransmits: 0,1,2,3
Receiver thinks: These are new frames 0-3!
Accepts duplicates! ✗

Correct: Window = 4, Seq# = 0-4 (5 numbers)
Now old vs new can be distinguished ✓
```

---

## 7. HDLC and PPP

### HDLC (High-Level Data Link Control)

**What is HDLC?**  
A bit-oriented data link protocol developed by ISO, forms the basis for many other protocols.

**Frame Structure:**
```
┌──────┬─────────┬─────────┬─────────────┬──────┬──────┐
│ Flag │ Address │ Control │   Data      │ FCS  │ Flag │
│ 8bit │  8 bit  │  8 bit  │  Variable   │16/32 │ 8bit │
└──────┴─────────┴─────────┴─────────────┴──────┴──────┘
 01111110                                        01111110
```

**Field Details:**

1. **Flag (01111110):**
   - Marks beginning and end
   - Bit stuffing used in data

2. **Address (8 bits):**
   - Identifies secondary station
   - All 1s (11111111) = broadcast

3. **Control (8 bits):**
   - Three types:
     - Information (I-frame): Data transfer
     - Supervisory (S-frame): Flow/error control
     - Unnumbered (U-frame): Link management

4. **Data (Variable):**
   - User data
   - Any length

5. **FCS (Frame Check Sequence):**
   - CRC-16 or CRC-32
   - Error detection

**Control Field Types:**

**I-Frame (Information):**
```
0 | N(S) | P/F | N(R)
↑   ↑↑↑    ↑    ↑↑↑
│   │││    │    └── Next frame expected
│   │││    └─────── Poll/Final bit
│   └─────────────── Sequence number
└──────────────────── I-frame identifier (0)
```

**S-Frame (Supervisory):**
```
1 0 | Type | P/F | N(R)

Types:
00 = RR (Receive Ready) - ACK, ready for more
01 = REJ (Reject) - Go-Back-N NAK
10 = RNR (Receive Not Ready) - Buffer full
11 = SREJ (Selective Reject) - Selective Repeat NAK
```

**U-Frame (Unnumbered):**
```
1 1 | Command/Response | P/F | Command/Response

Used for:
- SABM (Set Asynchronous Balanced Mode) - Initialize
- DISC (Disconnect) - Terminate
- UA (Unnumbered Acknowledgment) - Confirm
- FRMR (Frame Reject) - Report error
```

**HDLC Operation Example:**
```
Station A              Station B
    |                      |
    |----SABM-------→      | (Setup connection)
    |  ←----UA------       | (Acknowledge)
    |                      |
    |----I,0,0------→      | (Data, Send=0, Expect=0)
    |  ←----I,0,1----      | (Data, Send=0, Expect=1)
    |----I,1,1------→      | (Data, Send=1, Expect=1)
    |  ←----RR,2-----      | (Ready, Expect=2)
    |                      |
    |----DISC-------→      | (Disconnect)
    |  ←----UA------       | (Acknowledge)
```

**Advantages:**
- ✅ Reliable data transfer
- ✅ Error and flow control built-in
- ✅ Full-duplex operation
- ✅ Bit-oriented (no character restrictions)

**Disadvantages:**
- ❌ Complex
- ❌ Overhead for short messages
- ❌ Requires both sides to implement fully

### PPP (Point-to-Point Protocol)

**What is PPP?**  
A byte-oriented protocol for point-to-point links, widely used for dial-up and DSL internet connections.

**Frame Format:**
```
┌──────┬──────┬──────┬──────────┬──────────┬──────┬──────┐
│ Flag │Addrs │ Ctrl │ Protocol │   Data   │ FCS  │ Flag │
│  1   │  1   │  1   │    2     │ Variable │  2   │  1   │
└──────┴──────┴──────┴──────────┴──────────┴──────┴──────┘
 7E     FF     03                           CRC-16  7E
```

**Field Explanations:**

1. **Flag:** 01111110 (0x7E)
2. **Address:** 11111111 (0xFF) - Broadcast
3. **Control:** 00000011 (0x03) - Unnumbered
4. **Protocol:** Identifies payload type
   - 0x0021: IP (IPv4)
   - 0x0057: IPv6
   - 0xC021: LCP (Link Control Protocol)
   - 0x8021: NCP (Network Control Protocol)
5. **Data:** Up to 1500 bytes (default)
6. **FCS:** CRC-16 checksum

**PPP Components:**

**1. LCP (Link Control Protocol):**
- Establishes, configures, tests link
- Negotiates options
- Terminates connection

**2. Authentication Protocols:**
- PAP (Password Authentication Protocol)
- CHAP (Challenge Handshake Authentication Protocol)

**3. NCP (Network Control Protocol):**
- Configures network layer protocols
- IPCP for IPv4
- IPV6CP for IPv6

**PPP State Diagram:**
```
    DEAD
      ↓
    Carrier Detected
      ↓
    ESTABLISH (LCP negotiation)
      ↓
    AUTHENTICATE (Optional)
      ↓
    NETWORK (NCP negotiation)
      ↓
    OPEN (Data transfer)
      ↓
    TERMINATE
      ↓
    DEAD
```

**Connection Establishment:**
```
Client                 Server
  |                       |
  |----LCP Config Req--→  | (Propose options)
  |  ←--LCP Config Ack----| (Accept)
  |                       |
  |----Auth Request----→  | (PAP/CHAP)
  |  ←--Auth ACK----------| (Authenticated)
  |                       |
  |----IPCP Config Req-→  | (Request IP)
  |  ←--IPCP Config Ack---| (Assigned IP)
  |                       |
  | ←→  Data Transfer  ←→ |
  |                       |
  |----LCP Terminate---→  | (Disconnect)
  |  ←--LCP Terminate Ack-|
```

**Byte Stuffing in PPP:**
```
Original: AA 7E BB 7D CC

Escape 7E: AA 7D-5E BB 7D CC
Escape 7D: AA 7D-5E BB 7D-5D CC

Transmitted: [7E] AA 7D-5E BB 7D-5D CC [7E]
             Flag                        Flag

Receiver removes escape sequences
```

**Advantages:**
- ✅ Simple and efficient
- ✅ Supports multiple network protocols
- ✅ Built-in authentication
- ✅ Error detection
- ✅ Widely supported

**Disadvantages:**
- ❌ Only point-to-point (no multipoint)
- ❌ No flow control
- ❌ Byte stuffing overhead

**PPP vs HDLC:**

| Feature | HDLC | PPP |
|---------|------|-----|
| Orientation | Bit | Byte |
| Application | Various | Internet access |
| Authentication | No built-in | Yes (PAP/CHAP) |
| Network Layer | Single | Multiple |
| Complexity | More complex | Simpler |
| Use Case | LANs, WANs | Dial-up, DSL |

---

## 8. Medium Access Control (MAC)

### What is MAC?
**Simple Explanation:**  
Imagine a classroom where many students want to talk. MAC is like the rules that decide who can speak when, so everyone doesn't talk at once!

**Technical Definition:**  
MAC (Medium Access Control) sublayer controls how multiple devices share a common communication channel.

### The Problem

**Multiple devices, one channel:**
```
Device A ─┐
Device B ─┤
Device C ─┼──→ Shared Medium (Channel)
Device D ─┤
Device E ─┘

If all transmit together → Collision! → Data corrupted!
```

### MAC Protocols Categories

**1. Channel Partitioning:**
- Divide channel among users
- TDMA, FDMA, CDMA

**2. Random Access:**
- Allow collisions, then recover
- ALOHA, CSMA, CSMA/CD, CSMA/CA

**3. Taking Turns:**
- Devices take turns
- Polling, Token Passing

---

## 9. Channel Allocation

### Static Channel Allocation

#### 1. FDMA (Frequency Division Multiple Access)
**How it works:**  
Divide frequency spectrum into bands, assign each user a band.

```
Frequency
   ↑
   |  User 4  ┃━━━━━━━━━┃
   |  User 3  ┃━━━━━━━━━┃
   |  User 2  ┃━━━━━━━━━┃
   |  User 1  ┃━━━━━━━━━┃
   └────────────────────→ Time
```

**Example:**
```
Total Bandwidth: 100 MHz
Users: 10
Each user gets: 10 MHz continuously

User 1: 0-10 MHz
User 2: 10-20 MHz
...
User 10: 90-100 MHz
```

**Advantages:**
- ✅ No collisions
- ✅ Continuous access
- ✅ Simple

**Disadvantages:**
- ❌ Wastes bandwidth when user idle
- ❌ Fixed allocation (inflexible)
- ❌ Guard bands needed

**Used in:**
- FM radio
- Traditional TV
- Old mobile phones (1G)

#### 2. TDMA (Time Division Multiple Access)
**How it works:**  
Divide time into slots, assign each user a time slot.

```
Time
  ↓
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 1 │ 2 │ 3 │ 4 │ → Repeats
└───┴───┴───┴───┴───┴───┴───┴───┘
  ↑   ↑   ↑   ↑
User User User User
  1    2    3    4
```

**Example:**
```
Frame duration: 10 ms
Users: 5
Each slot: 2 ms

User 1 transmits: 0-2 ms, 10-12 ms, 20-22 ms...
User 2 transmits: 2-4 ms, 12-14 ms, 22-24 ms...
```

**Synchronization Critical:**
```
All users must know:
- When frame starts
- Which slot is theirs
- Slot duration

Use GPS, Network timing, etc.
```

**Advantages:**
- ✅ Single frequency for all
- ✅ No collision in slot
- ✅ Flexible allocation

**Disadvantages:**
- ❌ Requires synchronization
- ❌ Guard time needed
- ❌ Wasted if user has no data

**Used in:**
- GSM (2G mobile)
- Satellite communications
- Digital TV

#### 3. CDMA (Code Division Multiple Access)
**How it works:**  
All users transmit simultaneously on same frequency, each with unique code.

**Mathematical Basis:**
```
Each user has orthogonal code:
User 1: [+1 +1 +1 +1]
User 2: [+1 -1 +1 -1]
User 3: [+1 +1 -1 -1]
User 4: [+1 -1 -1 +1]

These codes are orthogonal:
(User 1) · (User 2) = 1-1+1-1 = 0
```

**Transmission:**
```
Data bit 1: Multiply by code
Data bit 0: Multiply by -code

User 1 sends '1': [+1 +1 +1 +1]
User 2 sends '0': [-1 +1 -1 +1]

Combined signal: [0 +2 0 +2]
(All users transmit together!)
```

**Reception:**
```
Receiver for User 1:
Multiply received signal by User 1's code
Sum the results
If positive: bit was 1
If negative: bit was 0

Other users' signals cancel out (orthogonality)!
```

**Example:**
```
4 users, chip rate = 4

User 1 code: [1  1  1  1]
User 1 sends bit '1': [1  1  1  1]

User 2 code: [1 -1  1 -1]
User 2 sends bit '0': [-1  1 -1  1]

Channel receives: [0  2  0  2]

User 1 receiver:
[0 2 0 2] · [1 1 1 1] = 0+2+0+2 = 4 → Positive → bit '1' ✓

User 2 receiver:
[0 2 0 2] · [1 -1 1 -1] = 0-2+0-2 = -4 → Negative → bit '0' ✓
```

**Advantages:**
- ✅ All users use all frequency, all time
- ✅ No fixed allocation
- ✅ Handles interference well
- ✅ Secure (requires code to decode)

**Disadvantages:**
- ❌ Complex implementation
- ❌ Power control critical
- ❌ Limited by interference

**Used in:**
- 3G mobile networks
- GPS
- Military communications

### Dynamic Channel Allocation

**Assumptions:**
1. Independent stations
2. Single channel
3. Collision detection (sometimes)
4. Continuous or slotted time
5. Carrier sense or no carrier sense

---

## 10. Multiple Access Protocols

### Random Access Protocols

#### 1. Pure ALOHA

**How it works:**  
Transmit whenever you have data. If collision, wait random time and retry.

```
Station A:    ────█████─────────████───
Station B:    ───────███████──────────
Station C:    ─────────────███████────
                     ↑ Collision!
```

**Protocol Steps:**
1. Station has frame to send
2. Transmit immediately
3. Listen for ACK
4. If ACK received: Success!
5. If collision (no ACK): Wait random time
6. Retransmit

**Vulnerable Time:**
```
Time needed for frame transmission: T

Vulnerable period: 2T
(Frame can collide with frames sent T before or T after)

     Frame A
  |─────────────|
|───────────────────| Vulnerable period (2T)
     Can collide with frames here!
```

**Throughput:**
```
Maximum throughput = 18.4% of bandwidth

S = G × e^(-2G)

Where:
S = Throughput
G = Offered load
e = 2.718...

Peak at G = 0.5: S = 0.184 (18.4%)
```

**Example:**
```
Channel capacity: 100 Mbps
Frame size: 1000 bits
Frame time: 10 μs

Maximum useful throughput: 18.4 Mbps
Wasted: 81.6% (collisions + idle)
```

**Advantages:**
- ✅ Very simple
- ✅ Decentralized
- ✅ Works well at low load

**Disadvantages:**
- ❌ Low maximum throughput (18%)
- ❌ Many collisions
- ❌ Inefficient

#### 2. Slotted ALOHA

**How it works:**  
Time divided into slots. Transmit only at beginning of slot.

```
Slots:  |  0  |  1  |  2  |  3  |  4  |
A:         ████
B:              ████
C:                   ████ (collision!)
D:                   ████
```

**Improvement:**
- Vulnerable time reduced to T (from 2T)
- Stations synchronized to slot boundaries

**Throughput:**
```
Maximum throughput = 36.8% of bandwidth

S = G × e^(-G)

Peak at G = 1: S = 0.368 (36.8%)
Double that of pure ALOHA!
```

**Advantages:**
- ✅ Better than pure ALOHA (double)
- ✅ Still simple

**Disadvantages:**
- ❌ Requires synchronization
- ❌ Still only 37% efficiency
- ❌ Wasted slots

#### 3. CSMA (Carrier Sense Multiple Access)

**Key Idea:**  
Listen before transmitting! (Sense the carrier)

**"Listen before you speak!"**

**Types:**

**1-Persistent CSMA:**
```
1. Listen to channel
2. If idle: Transmit immediately (probability = 1)
3. If busy: Wait until idle, then transmit
4. If collision: Wait random time, repeat
```

**Behavior:**
```
Channel:  ─Busy───Idle→
Station A:         TX! (immediately)
Station B:         TX! (immediately)
                   Collision! (both transmit at once)
```

**Non-Persistent CSMA:**
```
1. Listen to channel
2. If idle: Transmit immediately
3. If busy: Wait random time, try again
4. Don't keep monitoring continuously
```

**Behavior:**
```
Channel:  ─Busy───Idle→
Station A:         (maybe transmits)
Station B:         (maybe transmits)
                   Less collisions (random backoff)
```

**P-Persistent CSMA:**
```
1. Listen to channel
2. If idle: Transmit with probability p
           Wait with probability (1-p)
3. If busy: Wait until idle, then step 2
```

**Example (p=0.5):**
```
Channel idle:
Station A: Coin flip → Heads → Transmit
Station B: Coin flip → Tails → Wait next slot
```

**Comparison:**
```
Throughput:
Non-Persistent > P-Persistent > 1-Persistent

Delay:
1-Persistent < P-Persistent < Non-Persistent

Trade-off between throughput and delay!
```

#### 4. CSMA/CD (Collision Detection)

**Used in:** Ethernet (wired LANs)

**How it works:**  
Listen while transmitting. If collision detected, stop immediately!

**Algorithm:**
```
1. Listen to channel
2. If idle:
   a. Start transmitting
   b. Monitor for collision while transmitting
   c. If collision detected:
      - Stop immediately
      - Send jam signal (48 bits)
      - Wait random backoff time
      - Try again
   d. If no collision: Transmission complete!
3. If busy: Wait
```

**Collision Detection:**
```
While transmitting:
  Compare transmitted signal vs received signal
  If different → Collision!

Station A:  ───TX───
Station B:        ───TX───
                    ↓
           Both detect collision!
           Send jam, backoff
```

**Minimum Frame Size:**
```
Must be large enough to detect collision!

Minimum frame size = 2 × Propagation time × Bandwidth

Example:
Distance: 2.5 km
Speed: 200,000 km/s
Propagation time: 12.5 μs
Round trip: 25 μs
Bandwidth: 10 Mbps

Min frame = 25 μs × 10 Mbps = 250 bits = 32 bytes

Ethernet minimum: 64 bytes (512 bits)
```

**Binary Exponential Backoff:**
```
After n collisions:
Wait random time: 0 to (2^n - 1) slot times

Example:
1st collision: Wait 0 or 1 slot time
2nd collision: Wait 0, 1, 2, or 3 slot times
3rd collision: Wait 0-7 slot times
...
10th collision: Wait 0-1023 slot times
After 16: Give up
```

**Advantages:**
- ✅ More efficient than ALOHA
- ✅ Stops on collision (saves bandwidth)
- ✅ Widely used (Ethernet)
- ✅ 80-90% efficiency achievable

**Disadvantages:**
- ❌ Requires collision detection hardware
- ❌ Doesn't work with wireless (hidden terminal)
- ❌ Performance degrades with distance

#### 5. CSMA/CA (Collision Avoidance)

**Used in:** Wi-Fi (wireless LANs)

**Why not CD?**  
In wireless, can't reliably detect collisions (hidden terminal problem)

**How it works:**  
Try to avoid collisions before they happen!

**Algorithm:**
```
1. Listen to channel (Carrier Sense)
2. If idle for DIFS time:
   - Wait random backoff time
   - If still idle: Transmit
3. If busy: Freeze backoff timer
4. After transmission:
   - Wait for ACK
   - If ACK: Success!
   - If no ACK: Collision assumed, retry
```

**RTS/CTS Mechanism:**
```
Sender → [RTS] → Access Point
         (Request to Send)
         
Sender ← [CTS] ← Access Point
         (Clear to Send)
         
Sender → [DATA] → Access Point

Sender ← [ACK] ← Access Point
```

**Timeline:**
```
Station A:
|←DIFS→|←Backoff→|───Frame───|
                   ↑
              Transmit when backoff expires

Station B:
|←DIFS→|←────Backoff────→| (Longer backoff)
         ↑
    Senses A transmitting, freezes backoff
```

**Hidden Terminal Problem:**
```
A ←→ AP ←→ C
A and C can't hear each other!

A transmits → AP
C transmits → AP  (Collision at AP!)
```

**Solution with RTS/CTS:**
```
A → RTS → AP
AP broadcasts CTS (C hears this!)
C: "Okay, channel busy, I'll wait"
A → Data → AP (No collision!)
```

**Advantages:**
- ✅ Works for wireless
- ✅ Handles hidden terminal
- ✅ ACK confirms delivery

**Disadvantages:**
- ❌ Higher overhead (RTS/CTS/ACK)
- ❌ Lower efficiency than CSMA/CD
- ❌ More complex

---

## 11. Ethernet and Wireless LANs

### Ethernet (IEEE 802.3)

**Most Popular LAN Technology!**

**Frame Format:**
```
┌──────────┬─────┬─────┬──────┬────────┬────┬─────┐
│ Preamble │ SFD │ Dst │ Src  │  Type  │Data│ FCS │
│   7 B    │ 1 B │ 6 B │  6 B │   2 B  │    │ 4 B │
└──────────┴─────┴─────┴──────┴────────┴────┴─────┘
```

**Fields:**

**1. Preamble (7 bytes):**
- 10101010 repeated 7 times
- Synchronization
- Allows receiver to lock onto signal

**2. SFD (Start Frame Delimiter, 1 byte):**
- 10101011
- Marks start of actual frame

**3. Destination MAC (6 bytes):**
- Who should receive
- Example: AA:BB:CC:DD:EE:FF
- First bit: Individual (0) or Group (1)

**4. Source MAC (6 bytes):**
- Who sent it
- Unique address burned into NIC

**5. Type/Length (2 bytes):**
- What protocol in data
- 0x0800 = IPv4
- 0x0806 = ARP
- 0x86DD = IPv6

**6. Data (46-1500 bytes):**
- Actual payload
- Min 46 bytes (padding if needed)
- Max 1500 bytes (MTU)

**7. FCS (Frame Check Sequence, 4 bytes):**
- CRC-32
- Error detection

**MAC Address Structure:**
```
AA : BB : CC : DD : EE : FF
└─────┬─────┘ └──────┬──────┘
    OUI           NIC Specific
(Manufacturer)    (Serial)

First bit of first byte:
- 0 = Unicast (single device)
- 1 = Multicast/Broadcast

Special:
FF:FF:FF:FF:FF:FF = Broadcast (all devices)
```

**Ethernet Standards:**

| Standard | Name | Speed | Cable | Max Distance |
|----------|------|-------|-------|--------------|
| 10Base-T | Ethernet | 10 Mbps | Cat3 UTP | 100 m |
| 100Base-TX | Fast Ethernet | 100 Mbps | Cat5 UTP | 100 m |
| 1000Base-T | Gigabit Ethernet | 1 Gbps | Cat5e/6 | 100 m |
| 10GBase-T | 10 Gig Ethernet | 10 Gbps | Cat6a/7 | 100 m |
| 100GBase-SR | 100 Gig | 100 Gbps | Fiber | 100 m |

**Notation Explained:**
```
1000 Base - T
  ↑     ↑    ↑
Speed  Type Cable

Speed: Mbps
Base: Baseband (digital)
T: Twisted Pair
F: Fiber
```

### Wireless LAN (IEEE 802.11 - Wi-Fi)

**Frame Format:**
```
┌─────────┬────────┬────────┬────────┬────────┬─────┬────┐
│Frame Ctl│Duration│Address1│Address2│Address3│ Seq │Data│
│   2 B   │   2 B  │   6 B  │   6 B  │   6 B  │ 2 B │... │
└─────────┴────────┴────────┴────────┴────────┴─────┴────┘
```

**Why More Addresses?**
```
Address 1: Destination (final)
Address 2: Source (original)
Address 3: Router/AP address
Address 4: (Sometimes) for mesh networks

Needed because frames go through AP!
```

**Infrastructure Mode:**
```
Laptop → AP → Server

Laptop sends to AP with:
- Address 1: AP MAC
- Address 2: Laptop MAC
- Address 3: Server MAC
```

**Wi-Fi Standards:**

| Standard | Year | Frequency | Max Speed | Range |
|----------|------|-----------|-----------|-------|
| 802.11b | 1999 | 2.4 GHz | 11 Mbps | 100 m |
| 802.11a | 1999 | 5 GHz | 54 Mbps | 50 m |
| 802.11g | 2003 | 2.4 GHz | 54 Mbps | 100 m |
| 802.11n | 2009 | 2.4/5 GHz | 600 Mbps | 200 m |
| 802.11ac | 2013 | 5 GHz | 1.3 Gbps | 100 m |
| 802.11ax (Wi-Fi 6) | 2019 | 2.4/5 GHz | 9.6 Gbps | 100 m |

**Channel Selection:**
```
2.4 GHz Band:
Channels 1-11 (US), 1-13 (Europe)

Non-overlapping: 1, 6, 11

Channel 1:  [===========]
Channel 6:          [===========]
Channel 11:                 [===========]
```

**Association Process:**
```
1. Client: Scan for APs (Probe Request)
2. APs: Respond (Probe Response)
3. Client: Choose AP, send Auth Request
4. AP: Auth Response
5. Client: Association Request
6. AP: Association Response
7. Connected!
```

**Security:**

**WEP (Wired Equivalent Privacy):**
- ❌ Broken, don't use!
- 40-bit or 104-bit key
- RC4 encryption

**WPA (Wi-Fi Protected Access):**
- Better than WEP
- TKIP encryption
- Still vulnerable

**WPA2:**
- Current standard (until recently)
- AES encryption
- Much more secure

**WPA3:**
- Latest (2018)
- Even stronger encryption
- Protection against offline attacks

### Ethernet vs Wi-Fi Comparison

| Feature | Ethernet | Wi-Fi |
|---------|----------|-------|
| **Medium** | Wired | Wireless |
| **Speed** | Up to 100 Gbps | Up to 10 Gbps |
| **Reliability** | Very high | Medium |
| **Latency** | Very low | Higher |
| **Security** | High (physical) | Medium (encryption) |
| **Mobility** | None | High |
| **Installation** | Complex | Easy |
| **Interference** | None | Common |
| **Cost** | Medium | Low |
| **Range** | 100m per segment | 50-200m |

---

## Exam Questions

### Short Answer Questions (2-3 Marks)

1. What is framing? Why is it necessary in data link layer?
2. Explain bit stuffing with an example.
3. What is the difference between Stop-and-Wait and Sliding Window protocols?
4. What is CSMA/CD? Where is it used?
5. Explain the hidden terminal problem in wireless networks.
6. What is the purpose of CRC in error detection?
7. Differentiate between HDLC and PPP.
8. What is MAC address? How is it different from IP address?
9. Explain TDMA with a diagram.
10. What is the function of preamble in Ethernet frame?

### Medium Answer Questions (5-7 Marks)

1. Explain different framing methods: Character Count, Byte Stuffing, and Bit Stuffing with examples.
2. Describe the Stop-and-Wait ARQ protocol with timeline diagram showing normal operation and frame loss.
3. Compare error detection methods: Parity Check, Checksum, and CRC. Which is most reliable?
4. Explain the working of CSMA/CD protocol with collision detection mechanism.
5. Describe the Ethernet frame format with all fields explained.
6. What is sliding window protocol? Explain with sender and receiver windows.
7. Compare Go-Back-N and Selective Repeat ARQ protocols.
8. Explain HDLC frame structure and different frame types.
9. Describe the association process in Wi-Fi networks.
10. What is CDMA? How does it allow multiple users to share the same frequency?

### Long Answer Questions (10-15 Marks)

1. **Explain all ARQ protocols in detail:**
   - Stop-and-Wait ARQ with all scenarios (normal, lost frame, lost ACK, delayed ACK)
   - Go-Back-N ARQ with window management
   - Selective Repeat ARQ with buffering
   - Include timing diagrams and comparisons

2. **Describe error control mechanisms:**
   - Error detection: Parity, Checksum, CRC
   - Error correction: Hamming Code
   - Include examples and calculations
   - Compare effectiveness

3. **Explain Medium Access Control protocols:**
   - Channel partitioning: FDMA, TDMA, CDMA
   - Random access: ALOHA, CSMA, CSMA/CD, CSMA/CA
   - Taking turns: Polling, Token Passing
   - Compare throughput and efficiency

4. **Compare Ethernet and Wi-Fi:**
   - Frame formats
   - Access methods (CSMA/CD vs CSMA/CA)
   - Standards and speeds
   - Security mechanisms
   - Advantages and disadvantages

5. **Explain HDLC and PPP protocols:**
   - Frame structures
   - Operation modes
   - Connection establishment
   - Use cases
   - Comparison

### Numerical Problems

1. **Calculate efficiency of Stop-and-Wait:**
   - Frame size = 1000 bits
   - Bandwidth = 1 Mbps
   - Distance = 10 km
   - Signal speed = 200,000 km/s

2. **Calculate minimum frame size for CSMA/CD:**
   - Distance = 2.5 km
   - Speed = 200,000 km/s
   - Bandwidth = 10 Mbps

3. **CRC Calculation:**
   - Data: 1101011011
   - Generator: 10011
   - Find CRC and transmitted frame

4. **Hamming Code:**
   - Encode data: 1011
   - Detect and correct error in received: 0110111

5. **Sliding Window:**
   - Window size = 7
   - How many sequence numbers needed?
   - If frames 0-6 sent, ACK 0-2 received, which frames can be sent?

### Diagram-Based Questions

1. Draw and explain the Ethernet frame format with all fields.
2. Draw timing diagram showing Stop-and-Wait ARQ with lost ACK.
3. Draw HDLC frame structure and explain control field types.
4. Illustrate hidden terminal problem and its solution using RTS/CTS.
5. Show how bit stuffing works with example data containing flag sequence.
6. Draw and explain the PPP frame format.
7. Illustrate TDMA time slot allocation for 5 users.
8. Show collision detection in CSMA/CD with timing diagram.
9. Draw state diagram for PPP connection establishment.
10. Illustrate sliding window movement with sequence numbers.

### True/False with Justification

1. CRC can detect and correct errors. (True/False)
2. Ethernet uses CSMA/CA protocol. (True/False)
3. Wi-Fi can detect collisions like Ethernet. (True/False)
4. Go-Back-N is more efficient than Selective Repeat. (True/False)
5. MAC address is 48 bits long. (True/False)
6. PPP provides authentication mechanisms. (True/False)
7. TDMA requires all stations to be synchronized. (True/False)
8. Hamming code can correct multiple bit errors. (True/False)
9. Pure ALOHA is more efficient than Slotted ALOHA. (True/False)
10. Byte stuffing increases frame size. (True/False)

### Scenario-Based Questions

1. **You're designing a network where 10 stations need to share a 100 Mbps channel. Which MAC protocol would you choose and why? Compare TDMA, CDMA, and CSMA/CD for this scenario.**

2. **A file transfer application needs reliable delivery over an unreliable channel with 1% error rate. Would you choose Go-Back-N or Selective Repeat? Justify with calculations.**

3. **Explain what happens when two Wi-Fi devices try to transmit simultaneously. How does CSMA/CA prevent and handle this situation?**

4. **A company is connecting two buildings 500m apart. Should they use Ethernet or Wi-Fi? Consider speed, reliability, security, and cost.**

5. **You receive a frame with CRC that matches. Can you be 100% sure there are no errors? Explain.**

### Comparison Tables to Create

1. Compare ALOHA, Slotted ALOHA, CSMA, CSMA/CD (throughput, efficiency, complexity)
2. Compare Stop-and-Wait, Go-Back-N, Selective Repeat (window size, buffer, retransmission)
3. Compare HDLC and PPP (frame format, use case, features)
4. Compare Ethernet standards (10Base-T, 100Base-TX, 1000Base-T)
5. Compare Wi-Fi standards (802.11a/b/g/n/ac/ax)

### Practice Calculations

1. **Binary Exponential Backoff:**
   - After 3rd collision, what are possible wait times?
   - After 10th collision?

2. **Throughput Calculation:**
   - Window size = 8
   - Propagation time = 20 ms
   - Transmission time = 5 ms
   - Calculate efficiency

3. **Sequence Numbers:**
   - For window size 15, how many bits needed?
   - For Go-Back-N?
   - For Selective Repeat?

---

## Additional Study Tips

### Important Topics to Master:
1. ✅ All three ARQ protocols with timing diagrams
2. ✅ Error detection/correction (especially CRC and Hamming Code)
3. ✅ CSMA/CD vs CSMA/CA (understand why each is used)
4. ✅ Ethernet and Wi-Fi frame formats
5. ✅ MAC protocols comparison
6. ✅ Framing methods (byte stuffing, bit stuffing)
7. ✅ HDLC and PPP protocols

### Must Practice:
- Drawing timing diagrams for ARQ protocols
- CRC calculations
- Hamming code encoding/decoding
- Frame format diagrams
- Efficiency calculations

### Common Mistakes to Avoid:
- ❌ Confusing Go-Back-N with Selective Repeat
- ❌ Wrong sequence number calculations
- ❌ Forgetting bit stuffing in examples
- ❌ Mixing up CSMA/CD (wired) and CSMA/CA (wireless)
- ❌ Incorrect CRC polynomial division

### Memory Tricks:

**ARQ Protocols:**
- **Stop-and-Wait**: One at a time (like waiting in line)
- **Go-Back-N**: Go back to error, resend all (wasteful but simple receiver)
- **Selective Repeat**: Select only errors (efficient but complex receiver)

**MAC Protocols:**
- **FDMA**: Different Frequencies (like radio stations)
- **TDMA**: Different Times (like taking turns)
- **CDMA**: Different Codes (like different languages)

**CSMA variations:**
- **CSMA**: Listen before talking
- **CSMA/CD**: Listen while talking (Ethernet)
- **CSMA/CA**: Avoid talking at same time (Wi-Fi)

---

**Good Luck with Your Data Link Layer Studies! 📚✨**

*Understanding these protocols is key to understanding how networks actually work at the fundamental level. Practice the calculations and diagrams - they appear frequently in exams!*

---

*Document Created: December 2025*
*Subject: Computer Networks (5th Semester)*
*Module 2: Data Link Layer - Complete Guide*


### Short Answer (2-3 Marks)

1. What is framing? Why is it necessary?
2. Explain byte stuffing with an example.
3. What is the difference between error detection and error correction?
4. How does CRC work?
5. Explain the timeout mechanism in error control.

### Medium Answer (5-7 Marks)

1. Explain different framing methods with examples.
2. Describe the Stop-and-Wait ARQ protocol with diagrams.
3. Compare error detection methods: Parity, Checksum, and CRC.
4. Explain Hamming code with encoding example.
5. Describe sliding window protocols.

### Long Answer (10-15 Marks)

1. Explain complete error control mechanisms including detection, correction, and ARQ protocols with diagrams.
2. Describe all framing methods in detail with advantages, disadvantages, and examples.
3. Compare Stop-and-Wait, Go-Back-N, and Selective Repeat ARQ protocols.

---

*Module 2 Complete - Data Link Layer Fundamentals*
*Continue to Module 3 for Network Layer concepts*
