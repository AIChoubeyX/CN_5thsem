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

*[Continuing in next response due to length...]* 

## Exam Questions

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
