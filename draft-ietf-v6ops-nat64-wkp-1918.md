---
title: "Using the Well-Known IPv6 Prefix to Represent Non-Global IPv4 Addresses"
abbrev: "nat64-wkp-1918"
category: std

docname: draft-ietf-v6ops-nat64-wkp-1918-latest
submissiontype: IETF
updates: 6052
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
  RFC1918:
  RFC6052:
  RFC6598:

informative:
  RFC5735:
  RFC6146:
  RFC6147:
  RFC6877:
  RFC6890:
  RFC7050:
  RFC8781:
  RFC8190:
  RFC8215:
  EID5547:
    title: "Errata ID 5547: NAT64 Well-Known Prefix SHOULD NOT be used for Private Use IPv4 Addresses"
    target: https://www.rfc-editor.org/errata/eid5547


...

--- abstract

This document modifies the requirement introduced in Section 3.1 of RFC6052
that IPv4/IPv6 Translators MUST NOT use the Well-Known Prefix 64:ff9b::/96
to represent non-globally reachable IPv4 addresses, such as those defined
in RFC1918 or listed in Section 2.2.2 of RFC6890.
The proposed change enables IPv6-only nodes to reach IPv4-only services
with specific non-globally reachable addresses
by leveraging the Well-Known Prefix.

This document updates Section 3.1 of RFC6052 ("Restrictions on the Use of the
Well-Known Prefix") to allow packets in which an address is composed of the
Well-Known Prefix and specific non-globally reachable IPv4 addresses to be
translated.

--- middle

# Introduction

Section 3.1 of [RFC6052] prohibits IPv4/IPv6 translators from using the
Well-Known Prefix (WKP, 64:ff9b::/96) to represent non-globally reachable IPv4
addresses, such as those defined in [RFC1918] or listed in Section 2.2.2 of
[RFC6890].

This restriction is relatively straightforward to implement in DNS64 [RFC6147]:
a DNS64 server simply avoids synthesizing an AAAA record using the WKP if the
original A record contains a non-globally reachable IPv4 address. However, this
requirement introduces significant operational challenges for systems that do
not rely on DNS64 and instead use local synthesis such as CLAT (Customer-side
Translator, [RFC6877]), or similar approaches.

Enterprise and other closed networks often require IPv6-only nodes to
communicate with both internal (e.g., using [RFC1918] addresses) and external
(Internet) IPv4-only destinations. The restriction in Section 3.1 of [RFC6052]
prevents such networks from utilizing the WKP and, consequently, from relying
on public DNS64 servers (e.g. forwarding requests for external zones to public
DNS64) which utilize the WKP in order to maximize compatibility.

Using two NAT64 prefixes — the WKP for Internet destinations and a
Network-Specific Prefix (NSP) for non-globally reachable IPv4 addresses — is
not a feasible solution for nodes performing local synthesis or running CLAT.
None of the widely deployed NAT64 Prefix Discovery mechanisms ([RFC7050],
[RFC8781]) provide a method to map a specific NAT64 prefix to the subset of
IPv4 addresses for which it should be used.

According to Section 3 of [RFC7050], a node must use all learned prefixes when
performing local IPv6 address synthesis. Consequently, if a node discovers both
the WKP and the NSP, it will use both prefixes to represent globally reachable
IPv4 addresses. This duplication significantly complicates security policies,
troubleshooting, and other operational aspects of the network.

Combining the WKP with the Local-Use prefix (64:ff9b:1::/48, [RFC8215]) is also
not feasible, as it introduces the same challenges as using the WKP
with the NSP.

Prohibiting the WKP from representing private IPv4 addresses ([RFC1918], [RFC6598]) offers no substantial benefit to IPv6-only or IPv6-mostly deployments.
It also substantially complicates network design and the behavior of nodes.

Given the recent operational experience in deploying IPv6-only and IPv6-mostly
networks, it is desirable to allow translators to use a single prefix
(including the WKP) to represent IPv4 addresses regardless of their
globally reachable or non-globally reachable status.
In particular, allowing translators to use the WKP to represent
private IPv4 addresses ([RFC1918], [RFC6598]) will greatly improve
the utility of the WKP in enterprise networks.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Terminology

This document reuses the Terminology section of [RFC6052] and [RFC8190].


# RFC6052 Update {#update}

This document updates Section 3.1 of [RFC6052] ("Restrictions on the Use of the
Well-Known Prefix") as follows:

OLD TEXT:

===

The Well-Known Prefix MUST NOT be used to represent non-global IPv4 addresses,
such as those defined in [RFC1918] or listed in Section 3 of [RFC5735]. Address
translators MUST NOT translate packets in which an address is composed of the
Well-Known Prefix and a non-global IPv4 address; they MUST drop these packets.

===

NEW TEXT:

===

The Well-Known Prefix MAY be used to represent the non-global IPv4 addresses
listed in [RFC1918] and [RFC6598].

Unmanaged client-side translators (CLATs) MUST translate packets in which an
address is composed of the Well-Known Prefix and these non-globally reachable
IPv4 address by default.

Provider-side translators (PLATs) MUST translate such packets unless configured
otherwise. Because administrators may rely on dropping these packets as an
implicit security policy, PLAT implementations MAY choose not to translate such
packets by default. However, such PLAT implementations MUST provide a
configuration knob to enable translation for these packets.

===

As noted in Erratum 5547 ([eid5547]):

```
IPv4 packets with private destination addresses are routinely translated to IPv4 packets with global destination addresses in NAT44.
Similarly, an IPv6 packet with a destination address representing a private IPv4 address [RFC6052] can be translated to an IPv4 packet with a global destination address by NAT64 [RFC6146].
If a 464XLAT CLAT cannot translate a private IPv4 address to an IPv6 address using the NAT64 /96 prefix and that IPv4 address [RFC6052], then the packet may not be translated to an IPv4 packet with a global address by the 464XLAT PLAT (stateful NAT64). This changes the intent of the sender, and in so doing violates the end to end principle.
```

Removing the requirement introduced in RFC 6052 Section 3.1 addresses this
errata.

# Operational Considerations

There may be cases in which it is desirable to ignore translation of private use
IPv4 addressing due to internal policy or overlapping internal networks.
In such envinronments the operators need to create configurations
which address the filtering of private use IPv4 addressing
if there is an expectation of compliance with
the original section 3.1 of [RFC6052].

## Existing Behavior

Testing and operational experience with existing CLAT implementations (both
mobile and non-mobile) have revealed highly inconsistent behavior regarding the
original restriction in Section 3.1 of [RFC6052]. While some implementations
strictly comply with the original requirement and drop packets destined for
non-globally reachable IPv4 addresses, many other widely deployed CLATs
completely ignore this restriction and translate the packets.

This inconsistency creates significant operational challenges. Network
operators are unable to predictably determine how unmanaged, client-side
devices will handle traffic directed to internal IPv4 services. This
unpredictable dropping or translating of packets on the client side severely
complicates network design, security policies, and troubleshooting.

By formalizing the requirement that unmanaged CLAT implementations MUST
translate these packets by default (as updated in Section 3), and allowing PLAT
devices to translate these packets, this document provides clear, standardized
instructions to implementers. This resolves the current operational ambiguity,
ensuring predictable behavior across all client ecosystems and aligning the
standard with the practical realities of modern IPv6-mostly and IPv6-only
deployments.

Furthermore, where client-side translation and local synthesis are used, it is
currently not feasible to employ more than one translation prefix, especially
if different prefixes must be used for different IPv4 destinations. None of the
widely deployed NAT64 Prefix Discovery mechanisms ([RFC7050], [RFC8781])
provide a method to map a specific NAT64 prefix to a subset of IPv4 addresses
for which it should be used.


## Use of Network Specific or Local-Use Prefix

Use of a network specific prefix or the Local-Use prefix as defined in 	
[RFC8215], 64:ff9b:1::/48, does not preclude the removal of section	
3.1 of [RFC6052] as a MUST requirement.
Whether a network employs a network specific	
prefix or 64:ff9b:1::/48, the behavior of synthesizing a private use	
IPv4 address is not prohibited by [RFC6052]. The changes proposed in	
this document are not impactful to networks using NSPs or the Local-Use	
prefix as defined in [RFC8215].

As discussed in [Introduction](#introduction), utilizing the NSP or
the Local-Use prefix will typically prevent the use of a public DNS64
resolver in the vast majority of cases, as large scale public DNS64 resolvers
use the WKP to maximize compatibility.


# Security Considerations

Legitimizing packets where the IPv6 destination address is composed of the WKP
and a non-globally reachable IPv4 address does not, inherently, introduce new
security considerations. Whether a specific traffic flow between an IPv6-only
source and a non-globally reachable IPv4 destination (or any flow to a
non-globally reachable IPv4 destination) is legitimate is a matter of local
network topology and administrative policy. However, existing NAT64
implementations compliant with RFC 6052 are expected to drop such packets.
Administrators may be relying on this implicit filtering as a built-in security
mechanism to prevent unauthorized access to private IPv4 infrastructure, rather
than implementing explicit security policies. This reliance is particularly
prevalent in managed NAT64 (PLAT) environments.

Modifying the recommended behavior to allow such address compositions may, in
the absence of explicit filtering, enable traffic flows that were previously
prohibited by the translator's default logic.
To mitigate this risk, existing managed PLAT implementations
compliant with RFC 6052 SHOULD NOT alter their default dropping
behavior. As specified in [Update](#update),
implementations which choose to
drop those packets by default MUST provide a configuration knob to
control this functionality, ensuring that the transition to supporting
non-globally reachable addresses is an intentional administrative
action accompanied by a review of local security policies.

Furthermore, administrators should not rely on the internal verification logic
of the translator to enforce security boundaries. Instead, explicit policies
such as access control lists (ACLs), firewall policies or NAT rules must be
used to define authorized traffic patterns through the translator.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank Mikael Abrahamsson, Mohamed Boucadair, Nick Buraglio, Lorenzo Colitti, Brian Carpenter, Goetz Goerisch, Wes Hardaker, Suresh Krishnan, Ted Lemon and Jordi Palet for their helpful comments and suggestions on this document.

# Appendix: Example flow
{: numbered="false"}


To illustrate the updated normative behavior, consider an IPv6-only network
utilizing 464XLAT [RFC6877] where an administrator wishes to provide access to
an internal, IPv4-only corporate service hosted at 10.1.2.3.

## Scenario A: Unmanaged CLAT to Managed PLAT Flow
{: numbered="false"}

An IPv4-only application on an unmanaged client device generates an IPv4 packet
destined for 10.1.2.3.

The local CLAT intercepts the IPv4 packet and synthesizes an IPv6 destination
address by prepending the Well-Known Prefix: 64:ff9b::10.1.2.3.

CLAT Behavior: Under the updated guidance in Section 3, the CLAT MUST translate
this packet by default, ignoring the non-globally reachable nature of the
embedded IPv4 address, and forward the resulting IPv6 packet to the network.

The IPv6 network routes the packet to the managed PLAT (NAT64 gateway).

PLAT Behavior: Upon receiving the packet destined for 64:ff9b::10.1.2.3, the
PLAT evaluates its local configuration:

Permit: If the administrator has explicitly enabled translation for
non-globally reachable addresses (or left the default translation behavior
enabled), the PLAT translates the packet back to IPv4 and forwards it to
10.1.2.3.

Drop: If the administrator relies on a default-drop posture for non-globally
reachable addresses or has explicitly configured an access control list (ACL)
blocking this range, the PLAT drops the packet.

## Scenario B: Native IPv6 Host to Managed PLAT
{: numbered="false"}

An IPv6-capable host (without a local CLAT) needs to communicate with the same
internal service. It acquires the destination address 64:ff9b::10.1.2.3 (e.g.,
via DNS64, local synthesis, or explicit application configuration).

The host transmits the IPv6 packet, which is routed to the PLAT.

PLAT Behavior: The PLAT applies the same configuration logic as in Scenario A.
It MUST translate the packet to IPv4 and forward it to 10.1.2.3 unless local
administrative policy configures it to drop the packet.

## Scenario C: CLAT flows avoiding the PLAT
{: numbered="false"}

An IPv4-only application on an unmanaged client device generates an
IPv4 packet destined for 10.1.2.3.

The local CLAT intercepts the IPv4 packet and synthesizes an IPv6
destination address by prepending the Well-Known Prefix:
64:ff9b::10.1.2.3.

CLAT Behavior: Under the updated guidance in Section 3, the CLAT MUST
translate this packet by default, ignoring the non-globally reachable
nature of the embedded IPv4 address, and forward the resulting IPv6
packet to the network.

The network administrator created the relevant rules to avoid translation
as the destination interface 10.1.2.3 is also configured as dual-stack with
the address 64:ff9b::10.1.2.3.

The IPv6 network more specific routes forward the packet to the IPv6 destination.
