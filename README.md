# Inter-agent Messaging via CBOR

Community Systems Working Group

2024-10-22

---

## What Exactly is Inter-agent Messaging and Why is it Hard?

Inter-agent messaging refers to the communication between autonomous agents in a distributed system.

Challenges include:
- Avoiding central registries
- Standardizing message formats without a central authority
- Future-proofing message formats
- Allowing for evolution in message formats
- Ensuring fast parsing speeds
- Ensuring messages are self-describing
---

## An example of what not to do

Electronic Data Interchange (EDI) is a standard for exchanging
business documents. Designed for use on mainframes that were less
capable than your smartphone, EDI messages are still in use today.
Here's an example of an EDI 837 Health Care Claim:

```
GS*HC*4405197800*999999999*19991231*0805*1421*X*004010X098A1
ST*837*0021
BHT*0019*00*244579*20061015*1023*CH
NM1*41*2*JONES HOSPITAL*Joe****46*12345
PER*IC*JANE DOE*TE*9005555555*EX*231
NM1*40*2*MEDICARE*****46*00120
HL*1**20*1
PRV*BI*PXC*203BA0200N
[...]
```

Pros:
- It's almost self-describing.

Cons:
- It's a royal pain to parse

---

## Slightly better

Example data message from a Hurricane Hunter aircraft:

```
214 
URNT15 KNHC 220234
AF305 WXWXA 241021204718305    HDOB 35 20241022
022800 2827N 08710W 3926 07743 0429 -185 -325 323035 035 /// /// 03
022830 2828N 08712W 3926 07744 0429 -185 -327 323035 035 /// /// 03
022900 2828N 08715W 3926 07743 0428 -185 -329 323035 035 /// /// 03
022930 2829N 08718W 3926 07744 0429 -185 -328 322035 035 /// /// 03
[...]
$$
;
```

You can make a reasonable guess at tail number, date, time, lat and
long, but the most important bits such as altitude, temperature,
barometric pressure, relative humidity, dew point, wind direction, and
wind speed are all encoded in a way that is not obvious without
digging through Appendix G of the National Hurricane Operations Plan.

Pros:
- It's a little easier to parse than EDI

Cons:
- It's only partially self-describing

---

## Getting there, sort of

Meteorological Aerodrome Reports (METAR) are a standardized format for
providing weather information to pilots.  These are intended to be
human-readable as well as machine-readable.  Here's an example:

```
KSFO 220156Z 29015KT 10SM FEW007 14/13 A3007 RMK AO2 SLP183 T01390128
```

"SFO airport, 22nd day of the month, 01:56 Zulu time, wind from 290
degrees at 15 knots, 10 statute miles visibility, few clouds at 700
feet, temperature 14°C, dew point 13°C, altimeter 30.07 inHg, remark
automatic station readings are sea level pressure 1018.3 hPa,
temperature 13.9°C, dew point 12.8°C."

Pros:
- It's human-readable with a little practice
- It's almost self-describing

Cons:
- It's not easy to parse without a magic decoder ring and prior
  knowledge of positionally-dependent fields

---

## Self-describing formats

RFC-822 Email messages became wildly sucessful:

```
From: John Doe <john@example.com>
To: Mary Smith <mary@example.com>
Subject: Saying Hello
Date: Fri, 21 Nov 1997 09:55:06 -0600
Message-ID: <308d5581-5e19-43fa-b453-b19d904c89c5@example.com>

Message body goes here.
```

Pros:
- It's human-readable
- It's self-describing

Cons:
- It's not very fast to parse


---

## XML

Extensible Markup Language (XML) is a self-describing format that
became popular in the late 1990s but has fallen out of favor:


```xml
<order>
  <customer>John Doe</customer>
  <item>
    <sku>12345</sku>
    <quantity>2</quantity>
  </item>
</order>  
```
Pros:
- It's self-describing
- It's human-readable

Cons:
- It's slow to parse
- It's not very space-efficient
- It's not very bandwidth-efficient
- It's not very CPU-efficient
---

## JSON

JavaScript Object Notation (JSON) has pretty much replaced XML as the
chosen format for new applications.  It is a lightweight, language
independent, self-describing format that is human-readable and faster
to parse than XML:

```json
{
  "order": {
    "customer": "John Doe",
    "item": {
      "sku": "12345",
      "quantity": 2
    }
}

```

Pros:
- It's self-describing
- It's human-readable
- It's faster to parse than XML

Cons:
- It's not very space-efficient
- It's not very bandwidth-efficient
- It's still CPU-intensive to parse

---

## JSON and binary data

Sending binary data in JSON is awkward and slow.  The way this is
usually done is to encode it in base64, which slows down sending and parsing,
increasing message size and bandwidth usage:

```json
{
  "image": {
    "mime": "image/png",
    "data": "iVBORw0KGgoAAAANSUhEUgAAACMAAAAlCAIAAABH4scTAAAAA3NCSVQICAjb4U/gAAAAWklEQVRIS[...]"
  }
}
```

---

## JSON with encryption and signing

Adding encryption and signing to JSON messages makes things even more
awkward.  You have to add fields for the encrypted data and the
signature, and you have to make sure that the signature is over the
encrypted data, not the plaintext data:

```json
{
  "encrypted": {
    "data": "IyBJbnRlci1hZ2VudCBNZXNzYWdpbmcgdmlhIENCT1IKCkNvbW11bml0eSBTeXN0ZW1zIFdvcmtp[...]",
    "signature": "iVBORw0KGgoAAAANSUhEUgAAACMAAAAlCAIAAABH4scTAAAAA3NCSVQICAjb4U/gAAAAWklEQVRIS[...]"
  }
}
```

By the time you're done doing this, you're not really using JSON any
more.  You're instead using a binary format that is encoded in base64
just so it can be sent as text in a JSON wrapper.  Not a great use of
bandwidth, disk space, or CPU time.

---
  
## Challenges in Decentralized Inter-agent Messaging

Typical requirements for decentralized inter-agent messaging include:
- Allowing for binary data
- Allowing for encryption and signing
- Ensuring messages are self-describing
- Ensuring fast parsing speeds
- Future-proofed message formats that allow for evolution

---

## Why standardize message formats?

In a decentralized system, agents will be written by different people
in different languages at different times for different purposes.  If
they can't understand each other's messages, they can't communicate
and the system will not scale.

Worst case, the system might actually appear to work, but the agents
may be misinterpreting each other's messages and making bad decisions.

---

## What is CBOR and How Does it Solve These Problems?

Concise Binary Object Representation (CBOR) is a binary format that is
"binary JSON".  CBOR was specifically designed to be stable for
decades while at the same time being evolvable.  

CBOR is already in extensive use -- a github code search for "CBOR"
returns over 900,000 results.  It is used in diverse applications:
Internet of Things, automotive, cryptocurrencies, web authentication,
Kubernetes, and more.

CBOR's standards and evolution are maintained by a working group of
the Internet Engineering Task Force (IETF), the same organization that
maintains the standards for the Internet itself. There are currently
31 CBOR-related RFCs, including:

- RFC 7049: Concise Binary Object Representation (CBOR)
- RFC 8152: CBOR Object Signing and Encryption (COSE)
- RFC 8392: CBOR Web Token (CWT)
- RFC 8742: Concise Binary Object Representation (CBOR) Sequences
- RFC 8746: Concise Binary Object Representation (CBOR) Tags for Typed Arrays
- RFC 8943: Concise Binary Object Representation (CBOR) Tags for Date
- RFC 8949: Concise Binary Object Representation (CBOR)
- RFC 9052: CBOR Object Signing and Encryption (COSE): Structures and Process
- RFC 9053: CBOR Object Signing and Encryption (COSE): Initial Algorithms
- RFC 9054: CBOR Object Signing and Encryption (COSE): Hash Algorithms
- RFC 9090: Concise Binary Object Representation (CBOR) Tags for Object Identifiers
- RFC 9164: Concise Binary Object Representation (CBOR) Tags for IPv4 and IPv6 Addresses and Prefixes
- RFC 9277: On Stable Storage for Items in Concise Binary Object Representation (CBOR)
- RFC 9581: Concise Binary Object Representation (CBOR) Tags for Time, Duration, and Period

---
## Alternatives to CBOR

There have been other efforts to create a similar binary format, but
each has their drawbacks. CBOR was designed in part as a reaction to
these.  The alternatives most often mentioned are:

### MessagePack

This format is self-describing, fast, and in fact
influenced the design of CBOR.  MessagePack itself wasn't a good
candidate for moving into the IETF standards process because it
includes early design decisions that are not ideal and are
difficult to change.  I was *this close* to using MessagePack
for Promisegrid before CBOR came along.

### BSON

This format is self-describing, but has issues similar to
those of MessagePack in terms of design and standards support.
Many of the issues stem from the fact that BSON was designed
primarily for use with MongoDB.

### Protocol Buffers

Mentioned here because it is widely used, but
Google's format is not self-describing.  It was designed
primarily for speed between applications that are tightly
version-controlled (ideally managed by the same legal entity)
rather than decentralized use cases.

---

## CBOR Basics: Strings

You might use an online CBOR encoder/decoder to follow along with
this.  There is a CBOR encoder/decoder at [https://cbor.me/](https://cbor.me/), and a JSON-to-CBOR converter at
[https://cbor.williamchong.cloud/](https://cbor.williamchong.cloud/)

Instead of using punctuation to delimit fields, CBOR uses a byte
prefix for each field that expresses the field type and length (very
fast to parse). The first three bits of the first byte describe the
type, and the remaining bits describe the length.  

For example:

- "a" is major type 3 (string), length 1, followed by the ASCII code for "a" (0x61):
```
            type  len   content
    binary: 011 00001   0110 0001
    hex:    0x61        0x61
```

- "aaa" is major type 3, length 3, followed by the ASCII codes for three "a"s:
```
            type  len   content
    binary: 011 00011   0110 0001 0110 0001 0110 0001
    hex:    0x63        0x61 0x61 0x61
```

---

## CBOR Basics: Integers

Small integers are even simpler. The type code for integers is zero,
and for small numbers the number itself is stored in the length field.
For example:

- The integer 0 is just 0x00:
```
            type  len/content
    binary: 000 00000
    hex:    0x00
```

- The integer 1 is just 0x01:
```
            type  len/content
    binary: 000 00001
    hex:    0x01
```

---

## CBOR Basics: Larger values

For integer values greater than 24, the length field is set to 24 for
1-byte integers, 25 for 2-byte integers, 26 for 3-byte integers, and
27 for 4-byte integers.  The integer itself follows.  For example:

- The integer 25 (one byte) is 0x1819:
```
            type  len   content
    binary: 000 11001   0001 1001
    hex:    0x18        0x19
    dec:    24          25
```

- The integer 65,535 (two bytes) is 0x1a00ffff:
```
            type  len   content
    binary: 000 11010   1111 1111 1111 1111
    hex:    0x19        0xff 0xff
    dec:    25          65,535
```

---
## CBOR Basics: Other types

Similar prefixing and length-encoding techniques are used for strings,
arrays, and maps.  The complete specification is in [RFC
7049](https://www.rfc-editor.org/rfc/rfc7049.html).  

The set of major types is:

- 0: unsigned integer
- 1: negative integer
- 2: byte string
- 3: text string
- 4: array
- 5: map
- 6: tag
- 7: floating point, simple values, and "break" (end of array or map)

---

## CBOR Tags

CBOR has a concept of "tags" that allow a protocol designer to compose
specialized data types out of the basic types.  Tags are a way to
extend the basic types in a way that is backwards-compatible.  For
example, the tag 0 is "date string", which is a string that represents
a date.  If you see a CBOR tag 0, you know that the data that follows
is a date string.  

The CBOR tags registry is maintained by the IANA, the same organization
that maintains the registries for IP addresses and port numbers.
Anyone can register a new tag at https://www.iana.org/assignments/cbor-tags/cbor-tags.xhtml

The CBOR convention is that if you don't understand a tag, you can
ignore it and still parse the rest of the message.  This is similar in
spirit to the way that a web browser can render a web page even if it
doesn't understand a particular HTML tag.

---

## CBOR Magic numbers

CBOR has a "magic number" that is used to identify the beginning of a
CBOR message.  This is useful for protocols that might have to
distinguish between CBOR messages and other types of messages, like
JSON, plain text, or whatever might completely replace CBOR one day.
RFC 8949 defines tag number 55799 for this purpose.  When composed as
the beginning of a CBOR message using type 6 (tag), this is what it
looks like:

```
            type content
    dec:    6   55799
    binary: 110  1101 1010 1110 0111
    hex:    0xd9d9f7
```

In particular, 0xd9d9f7 is not a valid Unicode character, and is not a
distinguishing mark for any frequently used file types.  

See [magic numbers](https://en.wikipedia.org/wiki/File_format#Magic_number) on
Wikipedia for more information about how magic numbers are used to
recognize data types in the wild, including for the 'file' command in
Unix and Linux.

---
## CBOR as stable storage

In addition to having a good wire format for messaging, it's important
in a decentralized system to have a good format for storing data on
disk.  

[RFC 9277] describes how to use CBOR for stable storage.  The goal is
that you can write a CBOR message to disk, and then read it back in
even if the software that wrote the message is no longer available.

This "difference in time" problem is similar in many ways to the
"difference in space" problem of inter-agent messaging.  The solution
is similar: a self-describing format that is stable over time.

---
## Order matters in a message

In some applications, it is important that the same data always
appears in the same order in a message or on disk.  This problem
arises because in JSON and most languages, the order of keys in a map
is not guaranteed.  For example, these are equivalent in JSON:

```
{"a": 1, "b": 2}
{"b": 2, "a": 1}
```

For PromiseGrid, this is a huge problem -- those two messages will
hash to different values so:

- we can't reliably store and retrieve them by hash
- we can't do prefix-based routing
- we can't rely on simple byte-sequence completion for e.g. LLM
  conversation continuation

---

## Deterministically encoded CBOR

[RFC 8949](https://www.rfc-editor.org/rfc/rfc8949.html#name-deterministically-encoded-c)
describes an optional way to encode CBOR messages so that the order of
keys in a map is always the same.  

In PromiseGrid, we use deterministically encoded CBOR for all messages
and storage.  This allows us to:

- predictably hash messages in a repeatable way
- use prefix-based routing to the correct agent
- use sequence completion to e.g. continue an LLM chain of messages

---
## CBOR support in programming languages

The good news is that all of the low-level work of encoding and
decoding CBOR is already done for you.  There are libraries for
pretty much every programming language that you might want to use.
For instance, saying "hello world" in CBOR in Go is as simple as:

```go
// encode a string in CBOR
data, _ := cbor.Marshal("hello world")
```

We can examine the output of that in a few ways just to prove that
we've learned something from these slides:

```go
// show the message data in hex
for _, b := range data {
  fmt.Printf("%02x ", b)
}
fmt.Println()
// Output: 6b 68 65 6c 6c 6f 20 77 6f 72 6c 64

// show the first byte in binary
fmt.Printf("%08b\n", data[0])
// Output: 01101011

// show the CBOR major type
fmt.Printf("%d\n", data[0]>>5)
// Output: 3
```

---

## PromiseGrid messaging before CBOR

What I was thinking before CBOR:

```
{hash} {arg1} {arg2} ... {result}
```

For instance, if we wanted to send a message to an agent promising
that 2 + 2 = 4, we might send:

```
3822beed17dbef1235f9782ba0ae6184f0e987d29b46550db8cc79ff82e62256 2 2 4
```

...where the hash is the SHA-256 hash of the 'add' agent.

Pros:
- It's evolvable

Cons:
- It's not self-describing
- It's slow to parse

---

## PromiseGrid messaging with JSON

If we wanted to use JSON, we might send:

```json
{
  "10-hash": "3822beed17dbef1235f9782ba0ae6184f0e987d29b46550db8cc79ff82e62256",
  "20-args": [2, 2],
  "30-result": 4
}
```

Pros:
- It's self-describing
- It's human-readable

Cons:
- It's slow to parse
- It's not very efficient in space, bandwidth, or CPU
- Binary data is awkward 
- Encryption and signing would be awkward
- Using the NN- format for keys is a hack and JSON libraries won't
  preserve ordering that way anyway

---

## PromiseGrid messaging with CBOR

Translating the JSON message to CBOR, using the 55799 tag as a magic number, and adding tags for hash, args, and result:

```
d9d9f7a 102710 3822beed17dbef1235f9782ba0ae6184f0e987d29b46550db8cc79ff82e62256 102720 2 102720 2 102730 4
```

...where:
- the hash is the SHA-256 hash of the 'add' agent.
- the tag 102710 is the hash tag
- the tag 102720 is the args tag
- the tag 102730 is the result tag

Pros:
- It's self-describing
- It's fast to parse
- It's space-efficient

Cons:
- It's not human-readable (we'll need tools to convert to/from JSON for debugging, wireshark plugins, etc.)

---

## Conclusion

Here's how we can use CBOR to speed up PromiseGrid development:

- Use CBOR for inter-agent messaging.
- Use CBOR for data storage.
- Use **deterministically encoded CBOR** for messages and storage.
- Use the [CBOR Web Token (CWT)](https://www.rfc-editor.org/rfc/rfc8392.html) format for
  authentication and authorization.
- Use the [CBOR Object Signing and Encryption (COSE)](https://www.rfc-editor.org/rfc/rfc8152.html) format for
  message signing and encryption.
- Use the 55799 tag as a magic number to identify CBOR messages in
  PromiseGrid, allowing us to eventually switch to a different message
  format if CBOR is ever replaced by something better.

The goal is to make PromiseGrid as future-proof as possible, and to
avoid the up-front design work (and pitfalls) involved in designing
our own message format.

## Questions?

