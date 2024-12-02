title: "SASL Remember Me"
abbrev: "sasl-rememberme"
category: info

docname: draft-bucksch-sasl-rememberme
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: kitten
keyword:
 - sasl
 - rememberme
 - login
 - authentication
venue:
  group: kitten
  type: Working Group
  mail: kitten@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/kitten/
  github: benbucksch/sasl-rememberme
  latest: https://example.com/LATEST

author:
 -
    fullname: Ben Bucksch
    organization: Beonex
    email: ben.bucksch@beonex.com
 -
    fullname: Stephen Farrell
    organization: Trinity College Dublin
    email: stephen.farrell@cs.tcd.ie

normative:

informative:


--- abstract

Introduces a SASL mechanism that allows the application to stay
logged in and re-login without user interaction, after completing a
time-consuming SASL login mechanism that involves the user.

--- middle

# Introduction

A client application might log in to a server the first time using
Passkey or multi-factor authentication. However, these mechanisms
require complicated user interaction. These are too costly to
perform at every login, but can realistically only be
completed once during setup. To stay logged in, this SASL
mechanism allows the server to create a client-specific token that
the client application can then use to log in to the same account,
without user interaction.

The token is opaque to the client and the server can choose how to
implement it. It may be a custom random string that the server
stores, or an application-specific password together with the
username, or a JWT token, or a refresh token which does not expire.

# Creation of the token

The client requests the token from the server using either
1. application-specific commands, like IMAP `REMEMBERME` or
likewise in XMPP, which are to be defined in other standards.
2. the success resppnse of another SASL mechanism.

The client stores the token for this hostname, similar to a
password, in a client-specific storage that is as secure as a
password storage.

# Login using the token

1. If the client needs to log in and has a token for this user and
host, it logs in using the SASL `REMEMBERME` mechanism, by sending:

`REMEMBERME <token>`

2.
  a. If the server accepts the response as valid and allows login,
  it responds with a SASL success response. The user is logged in.

  (ALTERNATIVE EXPIRY: ....
  it responds with a SASL success response, a new token, and the
  expiry time of the new token. The user is logged in.
  To avoid race conditions in clients that open multiple
  connections at the same time, the previously used token MUST
  be valid for at least 30 more seconds. Likewise, if the server
  returned a new token, then it must return the same new token
  in response to the same old token for the next 1 hour.
  Reason: If the client opens 5 connections at the same time,
  using the same token, but the server were to respond with
  5 different new tokens, and it were to allow only 1 of them,
  then the client would not know which one to store, due to race
  conditions in the network response.)

  b. If the response is invalid, the server responds with a
  SASL error and a human-readable error message for the end user.

# IMAP Example

In IMAP, the exchange would be:
```
S: * OK ACME IMAP Server v1.23 is ready
C: 22 CAPABILITY
S: 22 CAPABILITY IMAP4 IMAP4rev1 IMAP4rev2 AUTH=PASSKEY AUTH=REMEMBERME
C: 23 AUTHENTICATE REMEMBERME AEC6576576557
S: 23 OK AUTHENTICATE completed
```

# Requirements on the token

1. The token MUST also allow the server to infer the username. The
token alone must be sufficient to log in.
2. The token MUST not expire.
(ALTERNATIVE EXPIRY: The token MUST be valid for at least 30 days.)
3. The server SHOULD be able to revoke a token of a specific user,
in case this specific user account or device is compromised.
In this case, the login MUST fails and the SASL error message
MUST explain the situation to the user and give instructions
how to remedy the situation.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
