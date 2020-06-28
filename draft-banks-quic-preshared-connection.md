---
title: "QUIC Preshared Connection"
abbrev: "QUIC Preshared Connection"
docname: draft-banks-quic-preshared-connection
date: {DATE}
category: exp
ipr: trust200902
area: Transport
workgroup: QUIC

stand_alone: yes
pi: [toc, sortrefs, symrefs, docmapping]

author:
  -
    ins: N. Banks
    name: Nick Banks
    org: Microsoft Corporation
    email: nibanks@microsoft.com

normative:

  QUIC-TRANSPORT:
    title: "QUIC: A UDP-Based Multiplexed and Secure Transport"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-transport
    author:
      -
        ins: J. Iyengar
        name: Jana Iyengar
        org: Fastly
        role: editor
      -
        ins: M. Thomson
        name: Martin Thomson
        org: Mozilla
        role: editor

  QUIC-TLS:
    title: "Using Transport Layer Security (TLS) to Secure QUIC"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-tls-latest
    author:
      -
        ins: M. Thomson
        name: Martin Thomson
        org: Mozilla
        role: editor
      -
        ins: S. Turner
        name: Sean Turner
        org: sn3rd
        role: editor

--- abstract

This document describes a method for generating a set of information from an
existing connection that can be used to immediately start a new connection
without requiring a TLS handshake.

--- middle

# Introduction

QUIC connections go through a TLS handshake {{!QUIC=I-D.ietf-quic-tls}} to
authenticate each endpoint and securely exchange secrets that are used to
generate the keys used to encrypt the packets.  In addition to the TLS
handshake, QUIC (and possibly the app layer) exchange additional parameters that
control the rest of the protocol behavior.  If all this information can be
collected for the connection by other means the connection can completely skip
the handshake and go directly to exchanging application data.

This can be extremely useful in the scenario where two clients of a server wish
to create a P2P connection between themselves in order to reduce the latency of
information exchange between them and reduce the load on the server.  The server
can be a proxy to exchange this "preshared connection" information.  Then, the
information can be used by each client to immediately start a connection between
each other.

# Conventions and Definitions

{::boilerplate bcp14}

This document uses terms and notational conventions from {{QUIC}}.

# Preshared Connection Transport Parameter

The preshared_connection_support transport parameter can be sent by both a
client and server.  The transport parameter is sent with an empty value; an
endpoint that understands this transport parameter MUST treat receipt of a
non-empty value as a connection error of type TRANSPORT_PARAMETER_ERROR.

Advertising the preshared_connection_support transport parameter indicates that
the endpoint supports creating new connections from preshared connection
information (PSCI).  Both sides must advertise support for the feature for it to
be considered successfully negotiated.

Once negotiated, either endpoint can request to create a new preshared
connection.  An endpoint requests a new preshared connection by sending a
REQUEST_PRESHARED_CONNECTION frame.  The peer then responds with a
NEW_PRESHARED_CONNECTION frame.

# Preshared Connection Information

This document defines PSCI as an explicit set and format of information that can
be used to create a new QUIC connection.  The format uses the same encoding
scheme used by QUIC transport paramters (reference/description TODO).  The
following are the possible parameters (all required):

- IP Address
- UDP Port
- ALPN
- Connection ID
- Traffic Secret
- QUIC Transport Parameters

TODO - Expand on the details of the above parameters.

# Exchanging the Information

The REQUEST_PRESHARED_CONNECTION and NEW_PRESHARED_CONNECTION frames both
contain the PSCI the peer needs in order to be able to connect to the endpoint.
A NEW_PRESHARED_CONNECTION frame can be matched with a previous
REQUEST_PRESHARED_CONNECTION frame via the Traffic Secret parameter.  The
endpoint that sends the REQUEST_PRESHARED_CONNECTION frame is considered the
client in the new preshared connection, and conversely the endpoint that sends
the NEW_PRESHARED_CONNECTION frame is considered the server.

Once the NEW_PRESHARED_CONNECTION frame has been acknowledged the new connection
is considered started and both endpoint can start using it.

Also note, the above frames are the built-in way to directly exchange PSCI for
each endpoint for the new preshared connection.  It is also possible for an out
of band method to be used to exchange the information.  For instance, in the
scenario where two clients of the same server wish to create a P2P connection
between themselves, an application layer protocol between the clients and server
can be used to negotiate and exchange the parameters.

# Starting the Preshared Connection

Once the PSCI is exchanged the new preshared connection is immediately
considered to be started; both endpoints can immediately start sending 1-RTT
packets.  This is accomplished by using the Traffic Secret exchanged in the PSCI
to generate 1-RTT keys just as a normal QUIC connection does, when the traffic
secrets are derived from the TLS handshake.

Both endpoints should immediately start the connection by sending HANDSHAKE_DONE
frames to each other, along with any other application data.  Receipt of the
HANDSHAKE_DONE frames indicate the connection has now been confirmed.

# NAT Punch-through

TODO

# Security Considerations

TODO

# IANA Considerations

This document registers the preshared_connection_support transport parameter in
the "QUIC Transport Parameters" registry established in Section 22.2 of
{{QUIC}}.  The following fields are registered:

Value:
: 0xBEEF

Parameter Name:
: preshared_connection_support

Status:
: Permanent

Specification:
: This document.

Date:
: Date of registration.

Contact:
: IETF QUIC Working Group (quic@ietf.org)

Notes:
: (none)

--- back

# Acknowledgments
{:numbered="false"}

TODO
