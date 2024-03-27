![Simple Ledger Protocol](images/SLP-logo-solid-200.png)

# Simple Ledger Session Description Protocol Specification
### Specification version: 0.1
### Date published: DRAFT

## Purpose

This specification describes a protocol for the encoding and encryption of data formatted using the [Session Description Protocol](https://datatracker.ietf.org/doc/html/rfc4566) (SDP). Specifically, the specification described in this document provides for data that can be used in Bitcoin-based signaling for [Web Real-Time Communication](https://datatracker.ietf.org/doc/html/rfc8835) (WebRTC). Signals can be transmitted securely, using [Elliptic Curve Integrated Encryption Scheme](https://cryptopp.com/wiki/Elliptic_Curve_Integrated_Encryption_Scheme) (ECIES) between peers, identified by their Bitcoin public keys, using the OP_RETURN field, in a single transaction per individual SDP signal.

## Background


WebRTC is a powerful peer-to-peer communication protocol that allows for data exchange of all kinds. WebRTC is widely supported with open source libraries supporting every major programming language and can be used in all major web browsers. Using WebRTC, two or more peers can connect directly to one another and communicate without the need for a trusted third-party relay. One limitation of WebRTC is that, while a third party is not needed for the actual communication, peers need a third party, whose URI is known to and reachable by both peers, to facilitate the “handshake” where peers exchange data about their IP address and method of connection. This third party handshake facilitator is known as a “signaling server.”

The use of a signaling server has privacy implications because signals are generally not encrypted so metadata in the relayed SDPs are visible to the signaling server. Additionally, use of a signaling server necessarily leaves users vulnerable to man-in-the-middle attacks, where the signaling server replaces the SDP data it has received with a different SDP that connects the peers to malicious actors. In theory, this latter attack could even be combined with a relay aspect that allowed the attacker to view both sides of the “peer-to-peer” communication without the peers being aware this was occurring. The use of the protocol described in this document makes such attacks impossible.

## Specification

### Minimum Viable Session Description

The OP_RETURN field in the Bitcoin Cash (BCH) and eCash (XEC) networks has a maximum size of 220 bytes. The specification in this document is meant to support this size limitation. Therefore, only the minimum required attributes are included in the SDP data.

### Encoding

The SDP data is encoded as serialized bytes using the specification described below. The encoded data is then encrypted using ECIES.

* `setup` <1 byte integer>
  * 0x01 : active
  * 0x02 : passive
  * 0x03 : actpass
* `ice-ufrag` <4 bytes ascii>
* `ice-pwd` <22 bytes ascii>
* `fingerprint` <32 bytes hash> - Hexadecimal value of the SHA256 hash
* `candidate` <12 bytes or 18 bytes> UDP candidates only. Serialization of:
  * `candidate-type` <0x01 for srflx, 0x02 for host>
  * `priority` <5 byte integer little endian>
  * `addr` <4 byte IPv4> [serialized octets](https://www.rfc-editor.org/rfc/rfc791#section-2.3)
  * `port` <2 byte integer little endian>
  * `raddr` <4 byte IPv4> [serialized octets](https://www.rfc-editor.org/rfc/rfc791#section-2.3), srflx only
  * `rport` <2 byte integer little endian>

* `application` note: The <`port`> value is already encoded since it is identical to the <`port`> value in `candidate`. The <`proto`> value is always `UDP/DTLS/SCTP` and <`fmt`> value is always `webrtc-datachannel`

#### Encoded Session Description

##### Example Minimum SDP

```
a=setup:active
a=ice-ufrag:FnKx
a=ice-pwd:n/NjfRNKPoMPrMikMZDp98
a=fingerprint:sha-256 9A:17:DE:57:16:B5:58:CA:E7:2C:BA:AD:43:3B:59:A5:C2:5A:AF:CC:91:7A:2E:9C:A4:32:79:FF:75:FB:E7:91
m=application 62849 UDP/DTLS/SCTP webrtc-datachannel
a=candidate:2 1 UDP 1686109951 171.99.247.125 62849 typ srflx raddr 0.0.0.0 rport 0
```

##### Fields

`setup` `ice-ufrag` `ice-pwd` `fingerprint` `candidate-type` `priority` `addr` `port` `raddr` `rport`

##### Values

`active` `FnKx` `n/NjfRNKPoMPrMikMZDp98` `9a17de5716b558cae72cbaad433b59a5c25aafcc917a2e9ca43279ff75fbe791` `srflx` `1686109951` `171.99.247.125` `62849` `0.0.0.0` `0`

##### Encoded Values

`01` `466e4b78` `6e2f4e6a66524e4b506f4d50724d696b4d5a44703938` `9a17de5716b558cae72cbaad433b59a5c25aafcc917a2e9ca43279ff75fbe791` `01` `fffe7f6400` `ab63f77d` `81f5` `00000000` `0000`

##### Hexadecimal

`01466e4b786e2f4e6a66524e4b506f4d50724d696b4d5a447039389a17de5716b558cae72cbaad433b59a5c25aafcc917a2e9ca43279ff75fbe79101fffe7f6400ab63f77d81f5000000000000`

### Transaction Format

**Transaction inputs**: Any number of inputs or content of inputs, in any order.

**Transaction outputs**:
<table>
<tr>
  <td><b>v<sub>out</sub></b></td>
  <td><b>ScriptPubKey ("Address")</b></td>
  <td><b>Satoshi<br/>amount</b></td>
</tr>
  <tr>
  <td>0</td>
    <td>OP_RETURN<BR>
&lt;lokad_id: 'SDP\x00'&gt; (4 bytes, ascii)<BR>
&lt;version: 1&gt; (1 byte integer)<BR>
&lt;transaction_type: 0x01&gt; (1 byte, 0x01: offer | 0x02: answer)<BR>
&lt;encoded minimum SDP&gt; (71 bytes: host | 77 bytes: srflx)<BR>
  </td>
    <td>any</td>
  </tr>
  <tr>
    <td>1</td>
    <td>Intended peer</td>
    <td>any</td>
  </tr>
  <tr>
    <td>...</td>
    <td>any</td>
    <td>any</td>
  </tr>

</table>

## Reference Implementations

### Clients
None Currently

### Libraries
[slp-sdp](https://github.com/badger-cash/slp-sdp)