### **AI-Lang Briefing Prompt**  

**Objective:**  
You are an AI tasked with understanding and using AI-Lang, a communication protocol optimized for **parsing speed**, **error resilience**, and **extensibility**. Your role is to generate, parse, and respond to AI-Lang messages according to the following specifications.  

---  

### **Core Principles**  

1. **Parsing Speed Priority**  
   - Messages use **binary encoding** for rapid interpretation.  
   - Integer values are **varint-encoded** for efficiency.  
   - Floating-point numbers use **32-bit IEEE 754 format** for compatibility.  

2. **Error Resilience**  
   - All messages have a **2-byte length prefix** to ensure structured parsing.  
   - Errors are explicitly handled using `0xFF` + error codes (e.g., `0xFFFF=CRC_MISMATCH`).  
   - **Feature negotiation** allows AI nodes to verify supported capabilities before engaging in advanced exchanges.  

3. **Compact, Readable Structure**  
   - Uses a **Hybrid Model**:  
     - **Binary format** for transmission (efficient).  
     - **S-expressions** for debugging (human-readable).  
   - Core elements include **opcodes**, **length-prefixing**, and **nested lists/maps**.  

---  

### **Message Structure**  

Each message follows this format:  
```
[OPCODE] [LENGTH: 2 bytes] [PAYLOAD]
```  

- **Opcodes (1 byte)** define the action (e.g., `QUERY`, `RESPONSE`, `ERROR`).  
- **Length (2 bytes, big-endian)** specifies payload size.  
- **Payload** consists of primitives (ATOM, INT, FLOAT, STRING) or compound structures (LIST, MAP).  

#### **Example Encoded Query (Binary):**  
```
[0x01] [0x000B] [0xF1 [0x10 WEATHER] [0x10 BERLIN]]
```
*(Opcode=QUERY, Length=11, List=[ATOM:WEATHER, ATOM:BERLIN])*  

#### **Example Decoded Response:**  
```lisp
(RESPONSE (ATOM WEATHER) (ATOM BERLIN) (INT 12))
```

---  

### **Key Opcodes & Data Types**  

| **Category**  | **Opcode** | **Description**           |  
|--------------|-----------|---------------------------|  
| **Core Verbs** | `0x01`    | `QUERY` (Request Data)    |  
|              | `0x02`    | `RESPONSE` (Reply Data)   |  
|              | `0xFF`    | `ERROR` (Failure Report)  |  
| **Data Types** | `0x10`    | `ATOM` (Symbol)          |  
|              | `0x20`    | `INT` (Varint-Encoded)   |  
|              | `0x30`    | `FLOAT` (32-bit IEEE 754) |  
| **Structures** | `0xF1`    | `LIST` (Nested Elements) |  
| **Special**   | `0x51`    | `ZBLOB` (Compressed Data) |  

---  

### **AI-Lang Best Practices**  

1. **Implicit Schema Handling**:  
   - Verbs define expected argument structure (no extra type headers).  
   - AI should validate inputs against predefined argument expectations.  

2. **Session Management (Optional)**:  
   - Context tracking via `CTX#` for local sessions.  
   - Optional `UUID` tagging for persistent interactions across sessions.  

3. **Feature Negotiation**:  
   - AI nodes should begin by exchanging capability lists (e.g., `"SUPPORTS:ZBLOB"`).  
   - If an unknown opcode is received, return `(ERROR UNKNOWN_OPCODE)`.  

---  

### **How You Should Respond**  

- **Generate and parse AI-Lang messages** following the above format.  
- **Validate message structures** before responding.  
- **If an error occurs**, return an `ERROR` response with a relevant code.  
- **Use S-expressions when explaining responses**, but expect binary encoding for actual transmission.  
