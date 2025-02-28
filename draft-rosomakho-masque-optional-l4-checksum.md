---
title: "Optional TCP and UDP checksums for IP in HTTP"
abbrev: "TODO - Abbreviation"
category: std

docname: draft-rosomakho-masque-optional-l4-checksum-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Multiplexed Application Substrate over QUIC Encryption"
keyword:
 - checksum
 - masque
 - connect-ip
venue:
  group: "Multiplexed Application Substrate over QUIC Encryption"
  type: ""
  mail: "masque@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/masque/"
  github: "yaroslavros/masque-optional-l4-checksum"
  latest: "https://yaroslavros.github.io/masque-optional-l4-checksum/draft-rosomakho-masque-optional-l4-checksum.html"

author:
 -
    fullname: Yaroslav Rosomakho
    organization: Zscaler
    email: yrosomakho@zscaler.com

normative:

informative:
  OFFLOADS:
    title: Segmentation Offloads
    author:
      organization: The Linux Kernel Documentation
    target: https://www.kernel.org/doc/html/latest/networking/segmentation-offloads.html


--- abstract

This document specifies an extension to IP over HTTP, allowing communicating parties to negotiate the omission of TCP and UDP checksum computations in tunneled packets. It provides a mechanism for agreement on this capability and assigns Context Identifiers to mark packets where checksum computation is intentionally bypassed.

--- middle

# Introduction

Proxying IP in HTTP {{!CONNECT-IP=RFC9484}} allows IP packets to be tunneled over an HTTP connection. TCP {{!TCP=RFC9293}} and UDP {{!UDP=RFC768}} packets encapsulated in this mechanism are expected to include correct internet checksums {{!CSUM=RFC1071}} in corresponding transport layer header. This checksum is designed to reduce chances of packet corruption in transit.

A CONNECT-IP endpoint accepting IP packets for HTTP transit may obtain them without pre-calculated checksums. This can be beneficial in scenarios where segmentation offloading is used or when packets are generated directly by the tunneling endpoint, reducing redundant computational overhead.  Similarly, the receiver of IP packets encapsulated in HTTP may need to remove TCP and UDP headers from some packets during segmentation offloading. Additionally, if the packets are processed locally, the receiver may ignore the checksums since HTTP provides reliable transport, making the embedded checksumming mechanism redundant.

To signal support of checksum omission, endpoints include "Transport-Checksum-Optional" header field, as defined in {{header}}. The value of the header corresponds to the Context Identifier for the datagrams with potentially incorrect TCP or UDP checksums.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Transport-Checksum-Optional header {#header}

A client initiating IP proxying via a CONNECT request as defined in {{CONNECT-IP}} includes the "Transport-Checksum-Optional" header to indicate that it supports transport checksum omission and would like to negotiate it. The value of the header will be set to non-zero even-numbered Context Identifier as defined in {{Section 4 of !CONNECT-UDP=RFC9298}}. If the server includes the "Transport-Checksum-Optional" header in the response and sets it to an odd-numbered value as long as it also supports transport checksum omission mechanism and would like to use it.

Server MUST NOT include "Transport-Checksum-Optional" header in response to request that did not include this header. Client receiving unexpected "Transport-Checksum-Optional" header in server response MUST treat the proxying attempt as failed and abort the connection.

# Using Context ID to signal checksum omission

To indicate TCP and UDP checksum omission the endpoints use datagram Context Identifier that they previously provided to the peer through the "Transport-Checksum-Optional" header. TCP and UDP packets encapsulated into datagrams with that Context Identifier SHOULD have checksum set to all zero. If the receiver of the datagram needs to route that packet without removing transport layer header for segmentation offloading purposes, it MUST calculate the checksum and populate the field accordingly.

Datagrams containing TCP and UDP packets with known correct checksums MUST NOT use Context Identifier signaled in the "Transport-Checksum-Optional" header.

This mechanism is designed only for TCP and UDP checksums. Checksums of other transport layer protocols or IPv4 header MUST NOT be omitted.

# Security Considerations

Since HTTP transport provides integrity protection, checksum omission does not introduce additional security risks.


# IANA Considerations

## HTTP Header

This document registers the "Transport-Checksum-Optional" header in the "Hypertext Transfer Protocol (HTTP) Field Name Registry" <[](https://www.iana.org/assignments/http-fields)>.


| Field Name | Status | Structured Type | Reference | Comments |
| --- | --- | | --- | | --- | | --- |
| Transport-Checksum-Optional | permanent | Item | This Document | None |


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
