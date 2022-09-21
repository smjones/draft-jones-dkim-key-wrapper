%%%
Title = "DKIM Key Wrapper Format"
area = "Internet"
workgroup = ""
keyword = ["DKIM", "public key", "provisioning"]
abbrev = "dkimkeywrapper"
version = "00"

[seriesInfo]
name = "Internet-Draft"
value = "draft-jones-dkim-key-wrapper-00"
stream = "IETF"
status = "experimental"

[[contact]]
initials = "T."
surname = "Herr"
fullname = "Todd Herr"
organization = "ValiMail"

[[author]]
initials = "S."
surname = "Jones, Ed."
fullname = "Steven M Jones"
organization = "DMARC.org"
[author.address]
email = "smj@dmarc.org"

[[author]]
initials = "N."
surname = "Selenu"
fullname = "Nicola Selenu"
organization = "ActiveCampaign"
%%%

.# Abstract {#abstract}

This experimental RFC seeks to document a DKIM key interchange
format intended to avoid provisioning errors where different parties
are generating keys and publishing DNS records for a given domain.


{mainmatter}

# Introduction

DomainKeys Internet Mail (DKIM) requires that an email sending domain
publish DKIM public keys in the Domain Name System (DNS) with very
specific formatting.  But when DKIM keys are published in the DNS with
incorrect formatting, all signature validation will fail and the
entire deployment may be abandoned.

A common reason for this failure mode is that an Email Service
Provider (ESP) or other party will generate the DKIM keys for the
domain owner, who may not be familiar with DNS management. They then may
have to determine how to enter the key data into a user interface
provided by a web hosting company. Often they will include formatting
characters that should be omitted, or include metadata like the TTL in
the payload or RDATA.

This Independent Submission proposes a wrapper format for DKIM public
key information. This format provides a simple block of printable
ASCII text, which is much more easily shared without being
inadvertently corrupted. But because the format is well-documented, a
DNS operator can easily extract the key material and provision the
necessary records.

## Goals

The goal of this submission is to solicit partners for a
proof-of-concept implementation using this format, to complete testing
by mid-2023 at the latest. Experience in the field will show whether
this concept warrants further development.




# Terminology and Definitions

Because this proposal discusses the handling of DKIM key material, the
terminology will be as close to [@!RFC6376] as possible. However as
some aspects of this proposal may concern actors not fully identified
in previous documents, some new elements may be identified.

## Role: Domain Owner

The Domain Owner is the party who controls or owns a given domain, and
presumably is seeking to benefit from deploying DKIM.

## Role: Key Generator

This is the party that has generated the public keypair that will be
used for DKIM signing messages. In all likelihood they will also
fulfill one of the other roles, like the DNS Operator or an Email
Service Provider (ESP).

## Role: DNS Operator

A DNS Operator in this context is the party operating nameservers that
are authoritative for the domain where DKIM is being deployed. The DNS
Operator may be a website hosting company, or a MailBox Provider
(MBP), for example.

## Role: Email Sender

The Email Sender in this context is a third party sending email on
behalf of the Domain Owner.


# Wrapper Format

The information inside the wrapper is a series of key/value pairs
using the JavaScript Object Notation (JSON) format, as described in
[@!RFC8259]. The set of fields used depends on whether a public or
private key is being represented.

This data is then encoded as described below.

## Encoding

A Key Wrapper consists of Base64 encoded text as described in
[@!RFC4648], and SHOULD have no more than 65 characters per line,
appearing between delimiters that identify it as wrapped DKIM key
material. The result is very similar in appearance to the PEM format
used by OpenSSL and similar software.

An encoded key wrapper might look like the following example:
```
-----BEGIN DKIM KEY-----
eyJ0eXBlIjoiREtJTS1QVUItS0VZIiwibmFtZSI6ImtleTIwMjIwNTE1IiwidiI6Ik
RLSU0xIiwicCI6Ik1JR2ZNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0R05BRENCaVFLQmdR
RGVKRE5hYzZnRlR3akx6eWxERWd3dk9sYk5TREcwN3FZR1h5K3hTY3F5aC9jZWIzek
R0eHZUZGpTbmFkU09adjJtWFRqZGI3N2U1eWlJaWZLbVJBZ2ZhaHF6c0dKUXEyUmRv
c2NZNkdYMThMWmxjNnNYSGJwREU0QzNaTFJoRlFHQzF0T29xU0Z5aEZya0VBeFltV2
Q2cElNajVnNWd2V3lJUm40U3p6Z3hzd0lEQVFBQiIsImsiOiJyc2EifQo=
-----END DKIM KEY-----
```

## Data Fields

The JSON formatted data consists of key/value pairs. Which keys appear
in a given wrapper will depend on what type of key material is being
represented.

This following sections document the names of the fields that appear
in the JSON formatted data. Some fields correspond to the "tags"
defined for the DNS TXT RR binding in [@RFC6376] Section 3.6.

### contact

This field may include contact information for the Key Generator or
the Domain Owner. It is intended to be brief, and might typically
include an organization name, email address, and/or telephone number.

Example:
```
"contact": "Hosting Company Key Services, keys@example.com, +1-123-555-1212"
```

TODO: ABNF

### domain

This is the domain the key was generated for, and is generally an
optional field.

Example:
```
"domain": "example.com"
```

TODO: ABNF

### k

The key type. Historically this has almost always been "rsa", but new
and stronger key types are being used more often on the Internet.

Example:
```
"k": "rsa"
```

TODO: ABNF

### name

The common name of the key. For a DKIM public key where the full DNS
label is "selector._domainkey.example.com", the value of the name field
would be "selector".

Example:
```
"name": "selector"
```

TODO: ABNF

### p

The **p**ublic key data, the actual key material. This is formatted as
described in `[@RFC6376, see section 3.6.1]`.

Example:
```
"p": "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDeJDNac6gFTwjLzylDEgwvOlbNSDG07qYGXy+xScqyh/ceb3zDtxvTdjSnadSOZv2mXTjdb77e5yiIifKmRAgfahqzsGJQq2RdoscY6GX18LZlc6sXHbpDE4C3ZLRhFQGC1tOoqSFyhFrkEAxYmWd6pIMj5g5gvWyIRn4SzzgxswIDAQAB"
```

TODO: ABNF

### r

The p**r**ivate key data. This is formatted in the manner for public key data ("p=" tag) described in `[@RFC6376, see section 3.6.1]`.

Example:
```
"r": "MIIBOwIBAAJBAODmxiUEZdxP34cYT1g2p7NvZkOKwdkOcjQljOFikQgZXSAwTWdgnsedt1V7V37Bc9iO3cUQSESKRGZCCz1CbD0CAwEAAQJBAKZe8Vt26mdVCvVkLWYDYIGjuhHi9s28Gw2qbZJZmRJUVgSG7mJItIN7FMTdjBRU9GoYgbtdnyE36nOiRZUlzEECIQDxoVuwBvwo8xIMBuLdhFrBHjPBBzY+M9y6mgiyi54ksQIhAO5Gu0utP5qg5mKs1WWfbLVnpNKS0djF9a+2ql9ojFNNAiEApOMJoFuD46XLoO1qDuPs0m/7vTNgrp3ReHz4hm6EImECIFucLDSLVpHv3MQBaUZaBiS0xYUEV9P9QFmfZF+sRY9dAiBIdJ7uoXHG8l3zaFX1v0qxUI9KTRB92mDK6nxG+OPzGQ=="
```

TODO: ABNF

### size

This field reflects the size of the key in bits, and is intended for
informational purposes. It is much easier to glance at this field,
than extract the key material and determine the key size
programmatically.

Example:
```
"size": "1024"
```

TODO ABNF

### type

This field indicates the type of key material encoded in this
file. Examples are "DKIM-PUB-KEY" and "DKIM-PRIV-KEY". The value of
this field determines what other fields must be present.

Valid values are currently "DKIM-PUB-KEY" and "DKIM-PRIV-KEY". The use
of any other value means that the entire record MUST be ignored.

Example:
```
"type": "DKIM-PUB-KEY"
```

TODO: ABNF

### v

The version of the DKIM key. This is the same value as specified for
the "v=" tag in `[RFC6376, see section 3.6.1]`, with all the
corresponding normative constraints.

Example:
```
"v": "DKIM1"
```
TODO: ABNF

## Record Types

There are two record types defined, which correspond to public and
private keys. Each type requires a specific set of data fields. Any
fields not included for a given record type below MUST be ignored.

### DKIM-PUB-KEY

This record type presents DKIM public key data. It is the type of
record that would be given to a DNS Operator for publication as a DNS
TXT RR.

Required fields:
* k
* name
* p
* type

Optional fields:
* contact
* domain
* size

Example:
```
{
  "contact": "Hosting Company Key Services, keys@example.com, +1-123-555-1212",
  "domain": "example.com",
  "k": "rsa",
  "name": "selector",
  "p": "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDeJDNac6gFTwjLzylDEgwvOlbNSDG07qYGXy+xScqyh/ceb3zDtxvTdjSnadSOZv2mXTjdb77e5yiIifKmRAgfahqzsGJQq2RdoscY6GX18LZlc6sXHbpDE4C3ZLRhFQGC1tOoqSFyhFrkEAxYmWd6pIMj5g5gvWyIRn4SzzgxswIDAQAB",
  "size": "2048",
  "type": "DKIM-PUB-KEY"
}
```

### DKIM-PRIV-KEY

This record type presents the DKIM private key data. It is the type of
record that would be given to an Email Sender, who needs the private
key to produce DKIM signatures for email messages being sent.

Required fields:
* k
* name
* r
* type

Optional fields:
* contact
* domain
* size

Example:
```
{
  "contact": "Hosting Company Key Services, keys@example.com, +1-123-555-1212",
  "domain": "example.com",
  "k": "rsa",
  "name": "selector",
  "r": "MIIBOwIBAAJBAODmxiUEZdxP34cYT1g2p7NvZkOKwdkOcjQljOFikQgZXSAwTWdgnsedt1V7V37Bc9iO3cUQSESKRGZCCz1CbD0CAwEAAQJBAKZe8Vt26mdVCvVkLWYDYIGjuhHi9s28Gw2qbZJZmRJUVgSG7mJItIN7FMTdjBRU9GoYgbtdnyE36nOiRZUlzEECIQDxoVuwBvwo8xIMBuLdhFrBHjPBBzY+M9y6mgiyi54ksQIhAO5Gu0utP5qg5mKs1WWfbLVnpNKS0djF9a+2ql9ojFNNAiEApOMJoFuD46XLoO1qDuPs0m/7vTNgrp3ReHz4hm6EImECIFucLDSLVpHv3MQBaUZaBiS0xYUEV9P9QFmfZF+sRY9dAiBIdJ7uoXHG8l3zaFX1v0qxUI9KTRB92mDK6nxG+OPzGQ=="
  "size": "2048",
  "type": "DKIM-PRIV-KEY"
}
```

# IANA Considerations

TODO IANA

# Security Considerations

TODO Security Considerations


{backmatter}

# Examples

TODO Examples

```
foo._domainkey.example.com IN TXT ( "blah blah blah; "
                                    "blor blor blor; "
                                    "foo bar baz quux; )
```


# Acknowledgements

[@Todd Herr]

Todd Herr solicited interested parties to shepherd the DKIM Enablement
topic at the M3AAWG meeting in February 2022, and the authors put
their names forward. This proposal came out of a suggestion made in a
M3AAWG Slack channel, and subsequent discussions between these
gentlemen.



# Change Log

\[RFC Editor: Please remove this section prior to publicaion.\]

TODO Change Log


