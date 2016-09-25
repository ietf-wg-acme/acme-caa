---
title: CA Account URI Binding for CAA Records
abbrev: CAA-URI
docname: draft-landau-acme-caa-latest
date: 2016-09-25
category: info

ipr: trust200902
area: General
workgroup: ACME Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: H. Landau
    name: Hugo Landau
    email: hlandau@devever.net

normative:
  RFC2119:
  RFC3986:
  RFC4648:
  RFC6844:
  RFC7517:
  I-D.ietf-acme-acme:

--- abstract

The CAA DNS record allows a domain to communicate issuance policy to CAs, but
only allows a domain to define policy with CA-level granularity. However, the
CAA specification also provides facilities for extension to admit more
granular, CA-specific policy. This specification defines such a parameter,
allowing specific accounts of a CA to be identified by URI.

--- middle

Introduction
============

This specification defines a parameter for the 'issue' and 'issuewild'
properties of the Certification Authority Authorization (CAA) DNS resource
record {{RFC6844}}, allowing authorization conferred by a CAA policy to be
restricted to specific accounts of a CA. The accounts are identified by URIs.

Terminology
===========

In this document, the key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" are to be
interpreted as described in BCP 14, RFC 2119 {{RFC2119}} and indicate
requirement levels for compliant CAA-URI implementations.

Extensions to the CAA Record: account-uri Parameter
===================================================

A CAA parameter "account-uri" is defined for the 'issue' and 'issuewild'
properties defined by {{RFC6844}}. The value of this parameter, if specified,
MUST be a URI {{RFC3986}} identifying a specific CA account.

"CA account" means an object maintained by a specific CA representing a
specific entity, or group of related entities, which may request the issuance
of certificates.

The presence of this parameter constrains the property to which it is attached.
A CA MUST only consider a property with an "account-uri" parameter to authorize
issuance where the URI specified is an URI that the CA recognises as identifying
the account making a certificate issuance request.

If a certificate issuance request is made to a CA such that no account URI is
available, because the request is made in the absence of any account or the
account has no URI assigned to it, a CA MUST NOT consider any property having
an "account-uri" parameter as authorizing issuance.

If an CA finds multiple CAA records pertaining to it (i.e., having property
'issue' or 'issuewild' as applicable and a domain that the CA recognises as its
own) with different "account-uri" parameters, the CA MUST NOT consider the CAA
record set to authorize issuance unless at least one of the specified account
URIs identifies the account of the CA by which issuance is requested. A
property without an "account-uri" parameter matches any account. A property
with an invalid or unrecognised "account-uri" parameter is
unsatisfiable.

The presence of an "account-uri" parameter does not replace or supercede the
need to validate the domain name specified in an "issue" or "issuewild" record
in the manner described in the CAA specification. CAs MUST still perform such
verification. For example, a CAA property which specifies a domain name
belonging to CA A and an account URI identifying an account at CA B is
unsatisfiable.

Use with ACME
-------------

An ACME {{I-D.ietf-acme-acme}} registration object MAY be identified by setting
the "account-uri" parameter to the URI of the ACME registration object.

Implementations of this specification which also implement ACME MUST recognise
such URIs.

Use without ACME
----------------

This document specifies a general mechanism to identify entities which may
request certificate issuance via URIs. The use of specific kinds of URI
may be specified in future RFCs, and CAs not implementing ACME MAY assign
and recognise their own URIs arbitrarily.

Security Considerations
=======================

This specification describes an extension to the CAA record specification
increasing the granularity at which CAA policy can be expressed. This allows
the set of entities capable of successfully requesting issuance of certificates
for a given domain to be restricted beyond that which would otherwise be
possible, while still allowing issuance for specific accounts of a CA. This
improves the security of issuance for domains which choose to employ it, when
combined with a CA which implements this specification.

URI Ambiguity
-------------

Suppose that CA A recognises "a.example.com" as identifying itself, CA B is a
subsidiary of CA A which recognises both "a.example.com" and "b.example.com" as
identifying itself.

Suppose that both CA A and CA B issue account URIs of the form

  "account-id:1234"

If the CA domain name in a CAA record is specified as "a.example.com" then this
could be construed as identifying account number 1234 at CA A or at CA B. These
may be different accounts, creating ambiguity.

Thus, CAs MUST ensure that the URIs they recognise as pertaining to a specific
account of that CA are unique within the scope of all domain names which they
recognise as identifying that CA for the purpose of CAA record validation.

It is RECOMMENDED that CAs satisfy this requirement by using URIs which include
an authority:

  "https://a.example.com/account/1234"

Authorization Freshness
-----------------------

The CAA specification governs the act of issuance by a CA. In some cases, a CA
may establish authorization for an account to request certificate issuance for
a specific domain separately to the act of issuance itself. Such authorization
may occur substantially prior to a certificate issuance request. The CAA policy
expressed by a domain may have changed in the meantime, creating the risk that
a CA will issue certificates in a manner inconsistent with the presently
published CAA policy.

CAs SHOULD consider adopting practices to reduce the risk of such
circumstances. Possible countermeasures include issuing authorizations with
very limited validity periods, such as an hour, or revalidating the CAA policy
for a domain at certificate issuance time.

DNSSEC
------

Where a domain chooses to secure its nameservers using DNSSEC, the authenticity
of an account URI nomination placed in a CAA record can be assured, providing
that a CA makes all DNS resolutions via an appropriate, trusted
DNSSEC-validating resolver. In this case and so long as control of the
resources identified by the URIs is retained, a domain is protected from the
threat posed by a global adversary capable of performing man-in-the-middle
attacks, which could otherwise forge DNS responses and successfully secure
certificate issuance from a CA where only "domain validation" is used as the
basis for issuance.

Use without DNSSEC
------------------

Where a domain does not secure its nameservers using DNSSEC, or one or more of
the CAs it authorizes do not perform CAA validation lookups using a trusted
DNSSEC-validating resolver, use of the "account-uri" parameter does not confer
additional security against an attacker capable of performing a
man-in-the-middle attack against all validation attempts made by a CA, as such
an attacker could simply fabricate the responses to DNS lookups for CAA
records.

In this case, the "account-uri" mechanism still provides an effective means of
administrative control over issuance, except where control over DNS is
subdelegated (see below).

Restrictions Supercedeable by DNS Delegation
--------------------------------------------

Because CAA records are located during validation by walking up the DNS
hierarchy until one or more records are found, the use of the "account-uri"
parameter, or any CAA policy, is not an effective way to restrict or control
issuance for subdomains of a domain, where control over those subdomains is
delegated to another party (such as via DNS delegation or providing limited
access to manage subdomain DNS records).

IANA Considerations
===================

None. As per the CAA specification, the parameter namespace for the CAA 'issue'
and 'issuewild' properties has CA-defined semantics. This document merely
specifies a RECOMMENDED semantic for a parameter of the name "account-uri".

--- back

Examples
========

The following shows an example DNS zone file fragment which nominates two
account URIs as authorized to issue certificates for the domain "example.com".
Issuance is restricted to the CA "example.net".

    example.com. IN CAA 0 issue "example.net; \
      account-uri=https://example.com/registration/1234"
    example.com. IN CAA 0 issue "example.net; \
      account-uri=https://example.com/registration/2345"

