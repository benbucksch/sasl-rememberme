---
title: "SASL Remember Me"
abbrev: "sasl-rememberme"
category: info

docname: draft-bucksch-sasl-rememberme-latest-00
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

A client application might at first log in to a server using
Passkey or multi-factor authentication. However, these mechanisms
require complicated user interaction. These are too costly to
perform at every login. They can realistically only be
completed once during setup. To stay logged in, this SASL
mechanism allows the server to create a client-specific token that
the client application can then use to log in to the same account,
without further user interaction.

The token is opaque to the client and the server can choose how to
implement it. It may be a custom random string that the server
stores, or an application-specific password generated for this
client together with the username, or a JWT token, or a
refresh token which does not expire.

# Creation of the token

The client requests the token from the server using either
1. application-specific commands, like IMAP `REMEMBERME`,
which are to be defined in other standards.
2. the success response of another SASL mechanism.

The client stores the token, instead of a password,
in a client-specific storage that is as secure as a password
storage.

# Login using the token

If the client needs to log in and has a token for this user and host, uses
the SASL `REMEMBERME` mechanism to log in.  `REMEMBERME` mechanism starts
with the client sending the initial client response, which has the following
format defined using ABNF:

~~~~ abnf
rememberme-client-step1 = token
token                   = 1*OCTET
~~~~

If the server accepts the response as valid and allows login, it responds with
a SASL success response. The user is logged in.  (ALTERNATIVE EXPIRY: ....  it
responds with a SASL success response, a new token, and the expiry time of the
new token. The user is logged in.  To avoid race conditions in clients that
open multiple connections at the same time, the previously used token MUST be
valid for at least 30 more seconds. Likewise, if the server returned a new
token, then it must return the same new token in response to the same old token
for the next 1 hour.  Reason: If the client opens 5 connections at the same
time, using the same token, but the server were to respond with 5 different new
tokens, and it were to allow only 1 of them, then the client would not know
which one to store, due to race conditions in the network response.)

If the response is invalid, the server responds with a SASL error and a
human-readable error message for the end user.

~~~~ abnf
server-final-message = server-error "," server-error-message
        ; Only returned on error. Omitted on success.

server-error = "e=" server-error-value

server-error-value = "invalid-encoding" /
                     "unknown-user" /
                     "invalid-username-encoding" /
                       ; invalid username encoding (invalid UTF-8 or
                       ; SASLprep failed)
                     "other-error" /
                     server-error-value-ext
        ; Unrecognized errors should be treated as "other-error".
        ; In order to prevent information disclosure, the server
        ; may substitute the real reason with "other-error".

server-error-value-ext = value
        ; Additional error reasons added by extensions
        ; to this document.

server-error-message = "m=" server-error-message-value

server-error-message-value = 1*OCTET
        ; Human readable error message in UTF-8
~~~~

# IMAP Example

In IMAP, the exchange would be:

~~~~
S: * OK ACME IMAP Server v1.23 is ready
C: 22 CAPABILITY
S: 22 CAPABILITY IMAP4rev1 IMAP4rev2 AUTH=PASSKEY AUTH=REMEMBERME
C: 23 AUTHENTICATE REMEMBERME QUVDNjU3NjU3NjU1Nwo=
S: 23 OK AUTHENTICATE completed
~~~~

In the above example the token is "AEC6576576557" which is base64-encoded
according to IMAP SASL profile.

# Requirements on the token

1. The token MUST also allow the server to infer the authentication identity (e.g. username or email address).
The token alone must be sufficient to log in.
This SASL mechanism doesn't support separate authorization identities.
2. The token MUST NOT expire.
(ALTERNATIVE EXPIRY: The token MUST be valid for at least 30 days.)
3. The server SHOULD be able to revoke the tokens,
in case this specific user account or device was compromised.
In this case, the authentication MUST fail and the SASL error message
MUST explain the situation to the user and give instructions
how to remedy the situation.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

Clients should treat the token like a password and store it securely.

TODO Security


# IANA Considerations

IANA is requested to add the following entries to the SASL Mechanism registry
established by {{!RFC4422}}:

~~~~
To: iana@iana.org
Subject: Registration of a new SASL mechanism REMEMBERME

SASL mechanism name (or prefix for the family): REMEMBERME
Security considerations: Section YY of [RFCXXXX]
Published specification (optional, recommended): [RFCXXXX]
Person & email address to contact for further information:
    IETF Kitten WG <kitten@ietf.org>
Intended usage: COMMON
Owner/Change controller: IESG <iesg@ietf.org>
Note:
~~~~

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
