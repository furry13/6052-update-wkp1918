---
title: "NAT64 WKP"
abbrev: "nat64-wkp-1918"
category: std

docname: draft-ietf-v6ops-nat64-wkp-1918-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "IPv6 Operations"
keyword:
  - ipv6
  - nat64
  - wkp
venue:
  group: "IPv6 Operations"
  type: "Working Group"
  mail: "v6ops@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/v6ops/"
  github: "furry13/6052-update-wkp1918"

author:
 -
    name: Warren Kumari
    ins: W. Kumari
    organization: Google, LLC
    email: warren@kumari.net

 -
    fullname: Jen Linkova
    organization: Google, LLC
    email: furry13@gmail.com

normative:
  RFC6052:

informative:
  RFC7050:
  RFC8781:
  RFC8215:


...

--- abstract

This document removes the requirement introduced in Section 3.1 of RFC6052 that the NAT64 Well-Known Prefix 64:FF9B::/96 MUST NOT be used to represent non-global IPv4 addresses, such as those defined in [RFC1918] or listed in Section 3 of [RFC5735].

--- middle

# Introduction

Section 3.1 of [RFC6052] prohibits IPv4/IPv6 translators from using the Well-Known Prefix (WKP, 64:FF9B::/96) to represent non-global IPv4 addresses, such as those defined in [RFC1918] or listed in Section 3 of [RFC5735].

This restriction is relatively straightforward to implement in DNS64 [RFC6147]: a DNS64 server simply avoids synthesizing an AAAA record using the WKP if the original A record contains a non-global IPv4 address.
However, this requirement introduces significant operational challenges for systems that do not rely on DNS64 and instead use local synthesis such as CLAT (Customer-side Translator, [RFC6877]), or similar approaches.

Enterprise and other closed networks often require IPv6-only nodes to communicate with both internal (e.g., using RFC1918 addresses) and external (Internet) IPv4-only destinations.
The restriction in Section 3.1 of RFC6052 prevents such networks from utilizing the WKP and, consequently, from relying on public DNS64 servers which utilize the WKP in order to maximize compatibility.


Using two NAT64 prefixes — the WKP for Internet destinations and a Network-Specific Prefix (NSP) for non-global IPv4 addresses — is not a feasible solution for nodes performing local synthesis or running CLAT.
None of the widely deployed NAT64 Prefix Discovery mechanisms ([RFC7050], [RFC8781]) provide a method to map a specific NAT64 prefix to a subset of IPv4 addresses for which it should be used.

According to Section 3 of [RFC7050], a node must use all learned prefixes when performing local IPv6 address synthesis.
Consequently, if a node discovers both the WKP and the NSP, it will use both prefixes to represent global IPv4 addresses.
This duplication significantly complicates security policies, troubleshooting, and other operational aspects of the network.

Prohibiting the WKP from representing non-global IPv4 addresses offers no substantial benefit to IPv6-only or IPv6-mostly deployments.
Simultaneously, it substantially complicates network design and the behavior of nodes.

Given the recent operational experience in deploying IPv6-only and IPv6-mostly networks, it is desirable to allow translators to use a single prefix (including the WKP) to represent all IPv4 addresses, regardless of their global or non-global status.
This simplification would greatly improve the utility of the WKP in enterprise networks.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Terminology

This document reuses the Terminology section of [RFC6052].


# RFC6052 Update

This document updates Section 3.1 of [RFC6052] ("Restrictions on the Use of the Well-Known Prefix") as follows:

OLD TEXT:

===

The Well-Known Prefix MUST NOT be used to represent non-global IPv4 addresses, such as those defined in [RFC1918] or listed in Section 3 of [RFC5735].
Address translators MUST NOT translate packets in which an address is composed of the Well-Known Prefix and a non- global IPv4 address; they MUST drop these packets.

===

NEW TEXT:

===

The Well-Known Prefix MAY be used to represent non-global IPv4 addresses, such as those defined in [RFC1918] or listed in Section 3 of [RFC5735].
Address translators MUST translate packets in which an address is composed of the Well-Known Prefix and a non- global IPv4 address; they MUST NOT drop these packets.

===

As noted in eid5547:

```
IPv4 packets with private destination addresses are routinely translated to IPv4 packets with global destination addresses in NAT44.
Similarly, an IPv6 packet with a destination address representing a private IPv4 address [RFC6052] can be translated to an IPv4 packet with a global destination address by NAT64 [RFC6146].
If a 464XLAT CLAT cannot translate a private IPv4 address to an IPv6 address using the NAT64 /96 prefix and that IPv4 address [RFC6052], then the packet may not be translated to an IPv4 packet with a global address by the 464XLAT PLAT (stateful NAT64). This changes the intent of the sender, and in so doing violates the end to end principle.
```

Removing the requirement introduced in RFC 6052 Section 3.1 addresses this errata.

# Operational Considerations

There may be cases when it is desirable to ignore translation of private use IPv4 addressing due to internal policy or overlapping internal networks. It is important to note, however, that overlapping networks in IPv6 translated addresses are also overlapping in IPv4, and so behavior will be similar across protocols in the vast majority of use cases. In environments reliant on [RFC7050] may be required to create configurations which address the filtering of private use IPv4 addressing if there is an expectation of compliance with the original section 3.1.

## Existing Behavior

Testing of existing non-mobile CLAT implementations has shown that there is significant lack of support for compliance with the original test of [RFC6052] section 3.1, indicating the operational behaviors of devices utilizing a client side translator (CLAT) are aligned with the proposed text at present, and that compliance with the existing text will cause potential operational overhead as adjustments to current practice will be required.

Further, where client side translation and local synthesis is used, it is currently not possible to employ more than one translation prefix, as none of the widely deployed NAT64 Prefix Discovery mechanisms ([RFC7050], [RFC8781]) provide a method to map a specific NAT64 prefix to a subset of IPv4 addresses for which it should be used.

## Use of Network Specific Prefix

Use of a network specific prefix such as provided by [RFC8215] does not preclude the removal of section 3.1 as a MUST requirement. If a network employs a network specific prefix the behavior of synthesizing a private use IPv4 address is not prevented by standard. The use of a network specific prefix implies the existence of a local mechanism for synthesizing IPv6 addresses based on that specific prefix, and thereby rules out use of a public DNS64 resolver in the vast majority of cases, as large scale public DNS64 resolvers use the WKP to maximize compatibility.



# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank .... for their helpful comments and suggestions on this document.

