---
v: 3

title: OSCORE-capable Proxies
abbrev: OSCORE-capable Proxies
docname: draft-ietf-core-oscore-capable-proxies-latest

# stand_alone: true

ipr: trust200902
area: Internet
wg: CoRE Working Group
kw: Internet-Draft
cat: std
submissiontype: IETF
updates: 8613

coding: utf-8
pi:    # can use array (if all yes) or hash here

  toc: yes
  sortrefs:   # defaults to yes
  symrefs: yes

author:
      -
        ins: M. Tiloca
        name: Marco Tiloca
        org: RISE AB
        street: Isafjordsgatan 22
        city: Kista
        code: SE-16440
        country: Sweden
        email: marco.tiloca@ri.se
      -
        ins: R. Höglund
        name: Rikard Höglund
        org: RISE AB
        street: Isafjordsgatan 22
        city: Kista
        code: SE-16440
        country: Sweden
        email: rikard.hoglund@ri.se

normative:
  RFC2119:
  RFC7252:
  RFC8174:
  RFC8724:
  RFC8613:
  I-D.ietf-core-oscore-groupcomm:
  I-D.ietf-core-href:
  I-D.ietf-schc-8824-update:

informative:
  RFC7030:
  RFC7641:
  RFC8742:
  RFC9200:
  I-D.ietf-core-groupcomm-bis:
  I-D.ietf-core-groupcomm-proxy:
  I-D.ietf-core-observe-multicast-notifications:
  I-D.ietf-core-coap-pubsub:
  I-D.ietf-core-oscore-edhoc:
  I-D.ietf-lake-edhoc:
  I-D.ietf-core-transport-indication:
  I-D.ietf-ace-key-groupcomm-oscore:
  I-D.ietf-core-coap-pm:
  I-D.ietf-ace-coap-est-oscore:
  I-D.amsuess-core-cachable-oscore:
  I-D.amsuess-t2trg-onion-coap:
  LwM2M-Core:
    author:
      org: Open Mobile Alliance
    title: Lightweight Machine to Machine Technical Specification - Core, Approved Version 1.2, OMA-TS-LightweightM2M_Core-V1_2-20201110-A
    date: 2020-11
    target: http://www.openmobilealliance.org/release/LightweightM2M/V1_2-20201110-A/OMA-TS-LightweightM2M_Core-V1_2-20201110-A.pdf
  LwM2M-Transport:
    author:
      org: Open Mobile Alliance
    title: Lightweight Machine to Machine Technical Specification - Transport Bindings, Approved Version 1.2, OMA-TS-LightweightM2M_Transport-V1_2-20201110-A
    date: 2020-11
    target: http://www.openmobilealliance.org/release/LightweightM2M/V1_2-20201110-A/OMA-TS-LightweightM2M_Transport-V1_2-20201110-A.pdf
  LwM2M-Gateway:
    author:
      org: Open Mobile Alliance
    title: Lightweight Machine to Machine Gateway Technical Specification - Approved Version 1.1, OMA-TS-LWM2M_Gateway-V1_1-20210518-A
    date: 2021-05
    target: https://www.openmobilealliance.org/release/LwM2M_Gateway/V1_1-20210518-A/OMA-TS-LWM2M_Gateway-V1_1-20210518-A.pdf
  TOR-SPEC:
    author:
      org: Tor Project
    date: false
    title: Tor Specifications
    target: https://spec.torproject.org/


--- abstract

Object Security for Constrained RESTful Environments (OSCORE) can be used to protect CoAP messages end-to-end between two endpoints at the application layer, also in the presence of intermediaries such as proxies. This document defines how to use OSCORE for protecting CoAP messages also between an origin application endpoint and an intermediary, or between two intermediaries. Also, it defines how to secure a CoAP message by applying multiple, nested OSCORE protections, e.g., both end-to-end between origin application endpoints, as well as between an application endpoint and an intermediary or between two intermediaries. Thus, this document updates RFC 8613. The same approach can be seamlessly used with Group OSCORE, for protecting CoAP messages when group communication with intermediaries is used.

--- middle

# Introduction # {#intro}

The Constrained Application Protocol (CoAP) {{RFC7252}} supports the presence of intermediaries, such as forward-proxies and reverse-proxies, which assist origin clients by performing requests to origin servers on their behalf, and forwarding back the related responses.

CoAP supports also group communication scenarios {{I-D.ietf-core-groupcomm-bis}}, where clients can send a one-to-many request targeting all the servers in the group, e.g., by using IP multicast. Like for one-to-one communication, group settings can also rely on intermediaries {{I-D.ietf-core-groupcomm-proxy}}.

The protocol Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}} can be used to protect CoAP messages between two endpoints at the application layer, especially achieving end-to-end security in the presence of (non-trusted) intermediaries. When CoAP group communication is used, the same can be achieved by means of the protocol Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}.

For a number of use cases (see {{sec-use-cases}}), it is required and/or beneficial that communications are secured also between an application endpoint (i.e., a CoAP origin client/server) and an intermediary, as well as between two adjacent intermediaries in a chain. This especially applies to the communication leg between the CoAP origin client and the adjacent intermediary acting as next hop towards the CoAP origin server.

In such cases, and especially if the origin client already uses OSCORE to achieve end-to-end security with the origin server, it would be convenient that OSCORE is used also to secure communications between the origin client and its next hop. However, the original specification {{RFC8613}} does not define how OSCORE can be used to protect CoAP messages in such communication leg, which would require to consider also the intermediary as an "OSCORE endpoint".

This document fills this gap, and updates {{RFC8613}} as follows.

* It defines how to use OSCORE for protecting a CoAP message in the communication leg between: i) an origin client/server and an intermediary; or ii) two adjacent intermediaries in an intermediary chain. That is, besides origin clients/servers, it allows also intermediaries to be possible "OSCORE endpoints".

* It admits a CoAP message to be secured by multiple, nested OSCORE protections applied in sequence, as an "OSCORE-in-OSCORE" process. For instance, this is the case when the message is OSCORE-protected end-to-end between the origin client and origin server, and the result is further OSCORE-protected over the leg between the current and next hop (e.g., the origin client and the adjacent intermediary acting as next hop towards the origin server).

This document does not specify any new signaling method to guide the message processing on the different endpoints. In particular, every endpoint is always able to understand what steps to take on an incoming message depending on the presence of the OSCORE Option, as exclusively included or instead combined together with CoAP options intended for an intermediary.

The approach defined in this document can be seamlessly adopted also when Group OSCORE is used, for protecting CoAP messages in group communication scenarios that rely on intermediaries.

## Terminology ## {#terminology}

{::boilerplate bcp14}

Readers are expected to be familiar with the terms and concepts related to CoAP {{RFC7252}}, OSCORE {{RFC8613}}, and Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}. This document especially builds on concepts and mechanics related to intermediaries such as CoAP forward-proxies and reverse-proxies.

In addition, this document uses the following terms.

* Source application endpoint: an origin client producing a request, or an origin server producing a response.

* Destination application endpoint: an origin server intended to consume a request, or an origin client intended to consume a response.

* Application endpoint: a source or destination application endpoint.

* Source OSCORE endpoint: an endpoint protecting a message with OSCORE or Group OSCORE.

* Destination OSCORE endpoint: an endpoint unprotecting a message with OSCORE or Group OSCORE.

* OSCORE endpoint: a source/destination OSCORE endpoint. An OSCORE endpoint is not necessarily also an application endpoint with respect to a certain message.

* Proxy-related options: either of the following (set of) CoAP options used for proxying a CoAP request. These CoAP options are defined in {{RFC7252}} and {{I-D.ietf-core-href}}.

   - The Proxy-Uri Option or the Proxy-Cri Option. These are relevant when using a forward-proxy.

   - The set of CoAP options comprising the Proxy-Scheme Option or the Proxy-Scheme-Number Option, together with any of the Uri-* Options. This is relevant when using a forward-proxy.

   - One or more Uri-Path Options, when used not together with the Proxy-Scheme Option or the Proxy-Scheme-Number Option. This is relevant when using a reverse-proxy.

* OSCORE-in-OSCORE: the process by which a message protected with (Group) OSCORE is further protected with (Group) OSCORE. This means that, if such a process is used, a successful decryption/verification of an OSCORE-protected message might yield an OSCORE-protected message.

# Use Cases # {#sec-use-cases}

The approach defined in this document has been motivated by a number of use cases, which are summarized below.

## CoAP Group Communication with Proxies # {#ssec-uc1}

CoAP supports also one-to-many group communication, e.g., over IP multicast {{I-D.ietf-core-groupcomm-bis}}, which can be protected end-to-end between origin client and origin servers by using Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}.

This communication model can be assisted by intermediaries such as a CoAP forward-proxy or reverse-proxy, which relays a group request to the origin servers. If Group OSCORE is used, the proxy is intentionally not a member of the OSCORE group. Furthermore, {{I-D.ietf-core-groupcomm-proxy}} defines a signaling protocol between origin client and proxy, to ensure that responses from the different origin servers are forwarded back to the origin client within a time interval set by the client, and that they can be distinguished from one another.

In particular, it is required that the proxy identifies the origin client as allowed-listed, before forwarding a group request to the servers (see {{Section 4 of I-D.ietf-core-groupcomm-proxy}}). This requires a security association between the origin client and the proxy, which would be convenient to provide with a dedicated OSCORE Security Context between the two, since the client is possibly using also Group OSCORE with the origin servers.

## CoAP Observe Notifications over Multicast # {#ssec-uc2}

The Observe extension for CoAP {{RFC7641}} allows a client to register its interest in "observing" a resource at a server. The server can then send back notification responses upon changes to the resource representation, all matching with the original observation request.

In some applications, such as pub-sub {{I-D.ietf-core-coap-pubsub}}, multiple clients are interested to observe the same resource at the same server. Hence, {{I-D.ietf-core-observe-multicast-notifications}} defines a method that allows the server to send a multicast notification to all the observer clients at once, e.g., over IP multicast. To this end, the server synchronizes the clients by providing them with a common "phantom observation request", against which the following multicast notifications will match.

In case the clients and the server use Group OSCORE for end-to-end security and a proxy is also involved, an additional step is required (see {{Section 12 of I-D.ietf-core-observe-multicast-notifications}}). That is, clients are in turn required to provide the proxy with the obtained "phantom observation request", thus enabling the proxy to receive the multicast notifications from the server.

Therefore, it is preferable to have a security association also between each client and the proxy, to especially ensure the integrity of that information provided to the proxy (see {{Section 15.3 of I-D.ietf-core-observe-multicast-notifications}}). Like for the use case in {{ssec-uc1}}, this would be conveniently achieved with a dedicated OSCORE Security Context between a client and the proxy, since the client is also using Group OSCORE with the origin server.

## LwM2M Client and External Application Server # {#ssec-uc3}

The Lightweight Machine-to-Machine (LwM2M) protocol {{LwM2M-Core}} enables a LwM2M Client device to securely bootstrap and then register at a LwM2M Server, with which it will perform most of its following communication exchanges. As per the transport bindings specification of LwM2M {{LwM2M-Transport}}, the LwM2M Client and LwM2M Server can use CoAP and OSCORE to secure their communications at the application layer, including during the device registration process.

Furthermore, Section 5.5.1 of {{LwM2M-Transport}} specifies that:

{:quote}
> OSCORE MAY also be used between LwM2M endpoint and non-LwM2M endpoint, e.g., between an Application Server and a LwM2M Client via a LwM2M server. Both the LwM2M endpoint and non-LwM2M endpoint MUST implement OSCORE and be provisioned with an OSCORE Security Context.

In such a case, the LwM2M Server can practically act as forward-proxy between the LwM2M Client and the external Application Server. At the same time, the LwM2M Client and LwM2M Server must continue protecting communications on their leg using their Security Context. Like for the use case in {{ssec-uc1}}, this also allows the LwM2M Server to identify the LwM2M Client, before forwarding its request outside the LwM2M domain and towards the external Application Server.

## LwM2M Gateway # {#ssec-uc4}

The specification {{LwM2M-Gateway}} extends the LwM2M architecture by defining the LwM2M Gateway functionality. That is, a LwM2M Server can manage end IoT devices "behind" the LwM2M Gateway. While it is outside the scope of such specification, it is possible for the LwM2M Gateway to use any suitable protocol with its connected end IoT devices, as well as to carry out any required protocol translation.

Practically, the LwM2M Server can send a request to the LwM2M Gateway, asking to forward it to an end IoT device. With particular reference to the CoAP protocol and the related transport binding specified in {{LwM2M-Transport}}, the LwM2M Server acting as CoAP client sends its request to the LwM2M Gateway acting as CoAP server.

If CoAP is used in the communication leg between the LwM2M Gateway and the end IoT devices, then the LwM2M Gateway fundamentally acts as a CoAP reverse-proxy (see {{Section 5.7.3 of RFC7252}}). That is, in addition to its own resources, the LwM2M Gateway serves the resources of each end IoT device behind itself, as exposed under a dedicated URI path. As per {{LwM2M-Gateway}}, the first URI path segment is used as "prefix" to identify the specific IoT device, while the remaining URI path segments specify the target resource at the IoT device.

As per Section 7 of {{LwM2M-Gateway}}, message exchanges between the LwM2M Server and the L2M2M Gateway are secured using the LwM2M-defined technologies, while the LwM2M protocol does not provide end-to-end security between the LwM2M Server and the end IoT devices. However, the approach defined in this document makes it possible to achieve both goals, by allowing the LwM2M Server to use OSCORE for protecting a message both end-to-end for the targeted end IoT device as well as for the LwM2M Gateway acting as reverse-proxy.

## Further Use Cases

The approach defined in this document can be useful also in the following use cases relying on a proxy.

* A server aware of a suitable cross proxy can rely on it as a third-party service, in order to indicate transports for CoAP available to that server (see {{Section 4 of I-D.ietf-core-transport-indication}}).

   From a security point of view, it would be convenient if the proxy could provide suitable credentials to the client, as a general trusted proxy for the system. At the same time, it can be desirable to limit the use of such a proxy to a set of clients which have permission to use it, and that the proxy can identify through a secure communication association.

   However, in order for OSCORE to be an applicable security mechanism for this scenario, OSCORE has to be terminated at the proxy. That is, it would be required for a client and the proxy to share a dedicated OSCORE Security Context and to use it for protecting their communication leg.

* The method specified in {{I-D.ietf-core-coap-pm}} relies on the Performance Measurement Option to enable network telemetry for CoAP communications. This makes it possible to efficiently measure Round-Trip Time and message losses, both end-to-end and hop-by-hop. In particular, on-path probes such as intermediary proxies can be deployed to perform measurements hop-by-hop.

   When OSCORE is used in deployments including on-path probes, an inner Performance Measurement Option is protected end-to-end between the two application endpoints and enables end-to-end measurements between those. At the same time, an outer Performance Measurement Option allows also hop-by-hop measurements to be performed by reying on an on-path probe.

   Therefore, it is preferable to have a secure association with an on-path probe, in order to also ensure the integrity of the hop-by-hop measurements exchanged with the probe.

* The method specified in {{I-D.ietf-ace-coap-est-oscore}} enables public-key certificate enrollment for Internet of Things deployments. This leverages payload formats defined in Enrollment over Secure Transport (EST) {{RFC7030}}, while relying on CoAP for message transfer and on OSCORE for message protection.

   In real-world deployments, an EST server issuing public-key certificates may reside outside a constrained network that includes devices acting as EST clients. In particular, the EST clients are expected to support only CoAP, while the EST server in a non-constrained network is expected to support only HTTP. This requires a CoAP-to-HTTP proxy to be deployed between the EST clients and the EST server, in order to map CoAP messages with HTTP messages across the two networks.

   Even in such a scenario, the EST server and every EST client can still effectively use OSCORE to protect their communications end-to-end. At the same time, it is desirable to have an additional secure association between the EST client and the CoAP-to-HTTP proxy, especially in order for the proxy to identify the EST client before forwarding EST messages out of the CoAP boundary of the constrained network and towards the EST server.

* A proxy may be deployed to act as an entry point to a firewalled network, which only authenticated clients can join. In particular, authentication can rely on the used secure communication association between a client and the proxy. If the proxy could share a dedicated OSCORE Security Context with each client, the proxy can rely on it to identify the client, before forwarding its messages to any other member of the firewalled network.

* The approach defined in this document does not pose a limit to the number of OSCORE protections applied to the same CoAP message.

   This enables more privacy-oriented scenarios based on proxy chains, where the origin client protects a CoAP request first by using the OSCORE Security Context shared with the origin server, and then by using different OSCORE Security Contexts shared with the different hops in the chain. Once received at a chain hop, the request would be stripped of the OSCORE protection associated with that hop before being forwarded to the next one.

   Building on that, it is also possible to enable the operation of hidden services and clients through onion routing with CoAP {{I-D.amsuess-t2trg-onion-coap}}, similarly to how Tor (The Onion Router) {{TOR-SPEC}} enables it for TCP-based protocols.

# Message Processing # {#sec-message-processing}

As mentioned in {{intro}}, this document introduces the following two main deviations from the original OSCORE specification {{RFC8613}}.

1. An "OSCORE endpoint" as a producer/consumer of an OSCORE Option, can be not only an application endpoint (i.e., an origin client or server), but also an intermediary such as a proxy.

   Hence, OSCORE can be used between an origin client/server and a proxy, as well as between two proxies in an intermediary chain.

2. A CoAP message can be secured by multiple OSCORE protections applied in sequence. Therefore, the final result is a message with nested OSCORE protections, as the output of an "OSCORE-in-OSCORE" process. Hence, following a decryption, the resulting message might legitimately include an OSCORE Option, and thus have in turn to be decrypted.

   The most common case is expected to consider a message protected with up to two OSCORE layers, i.e.: i) an inner layer, protecting the message end-to-end between the origin client and the origin server acting as application endpoints; and ii) an outer layer, protecting the message between a certain OSCORE endpoint and the other OSCORE endpoint adjacent in the intermediary chain.

   However, a message can also be protected with a higher, arbitrary number of nested OSCORE layers, e.g., in scenarios relying on a longer chain of intermediaries. For instance, the origin client can sequentially apply multiple OSCORE layers to a request, each of which to be consumed and removed by one of the intermediaries in the chain, until the origin server is reached and it consumes the innermost OSCORE layer.

   An OSCORE endpoint SHOULD define the maximum number of OSCORE layers that it is able to apply (remove) when processing an outgoing (incoming) CoAP message. The defined limit has to appropriately reflect the security requirements of the application. At the same time, it is typically bounded by the maximum number of OSCORE Security Contexts that can be active at the endpoint, and by the number of intermediary OSCORE endpoints that have been explicitly set up by the communicating parties.

   If its defined limit is reached when processing a CoAP message, an OSCORE endpoint MUST NOT perform any further OSCORE processing on that message. For an outgoing request, this results in aborting the message sending altogether. For an incoming request, this results in replying with a 4.01 (Unauthorized) error response, which the OSCORE endpoint protects by applying the same OSCORE layers that it successfully removed from the corresponding incoming request, but in the reverse order than the one according to which they were removed (see {{outgoing-responses}}).

{{sec-examples}} provides a number of examples where the approach defined in this document is used to protect message exchanges.

## General Rules on Protecting Options {#general-rules}

Let us consider a sender endpoint that, when protecting an outgoing message, applies the i-th OSCORE layer in sequence, by using the OSCORE Security Context shared with another OSCORE endpoint X.

In addition to the CoAP options specified as class E in {{RFC8613}} or in the document defining them, the sender endpoint MUST encrypt and integrity-protect the following CoAP options. That is, even if they are originally specified as class U or class I for OSCORE, such options are processed like if they were specified as class E.

* Any CoAP option such that both the following conditions hold.

   1. The sender endpoint has added the option to the message either:

      * Following a message protection previously performed for an OSCORE endpoint different than X; or

      * Before a message protection previously performed for an OSCORE endpoint different than X, where the option was treated as Class U or I.

   2. The option is not intended to be consumed by the other OSCORE endpoint X.

   Examples of such CoAP options are:

   - The OSCORE Option present as the result of the OSCORE layer immediately previously applied for an OSCORE endpoint different than X, when the sender endpoint is an origin endpoint.

   - The EDHOC Option defined in {{I-D.ietf-core-oscore-edhoc}}, when the sender endpoint is the EDHOC Initiator.

   - The Request-Hash Option defined in {{I-D.amsuess-core-cachable-oscore}}, when X is not an origin endpoint).

* Any CoAP option such that all the following conditions hold.

   1. The sender endpoint has added the option to the message.

   2. The option is intended to be consumed by the other OSCORE endpoint X.

   3. At the other OSCORE endpoint X, the option does not play a role in processing the message before having removed the i-th OSCORE layer or in removing the i-th OSCORE layer altogether.

   Examples of such CoAP options are:

   - The Proxy-Uri, Proxy-Scheme, Uri-Host, and Uri-Port Options defined in {{RFC7252}}.

   - The Proxy-Cri and Proxy-Scheme-Number Options defined in {{I-D.ietf-core-href}}.

   - The Listen-To-Multicast-Notifications Option defined in {{I-D.ietf-core-observe-multicast-notifications}}.

   - The Multicast-Timeout, Response-Forwarding, and Group-ETag Options defined in {{I-D.ietf-core-groupcomm-proxy}}.

* Any CoAP option such that the sender endpoint has not added the option to the message.

   Examples of such CoAP options are:

   - The OSCORE Option present as the result of the OSCORE layer immediately previously applied for an OSCORE endpoint different than X, when the sender endpoint is not an origin endpoint.

   - The EDHOC Option defined in {{I-D.ietf-core-oscore-edhoc}}, when the sender endpoint is not the EDHOC Initiator.

## Processing an Outgoing Request {#outgoing-requests}

The rules from {{general-rules}} apply when processing an outgoing request message, with the following addition.

When an application endpoint applies multiple OSCORE layers in sequence to protect an outgoing request, and it uses an OSCORE Security Context shared with the other application endpoint, then the first OSCORE layer MUST be applied by using that Security Context.

## Processing an Incoming Request {#incoming-requests}

Upon receiving a request REQ, the recipient endpoint performs the actions described in the following steps. {{sec-incoming-req-diag}} provides an overview as a state diagram.

1. If REQ includes proxy-related options, the endpoint moves to step 2. Otherwise, the endpoint moves to step 3.

2. The endpoint proceeds as defined below, depending on which of the two following conditions holds.

   * REQ includes either of the following (set) of CoAP options: the Proxy-Uri Option; the Proxy-Cri Option; the Proxy-Scheme Option or the Proxy-Scheme-Number Option, together with any of the Uri-* Options.

      If the endpoint is not configured to be a forward-proxy, it MUST stop processing the request and MUST respond with a 5.05 (Proxying Not Supported) error response to (the previous hop towards) the origin client, as per {{Section 5.10.2 of RFC7252}}. This may result in protecting the error response over that communication leg, as per {{outgoing-responses}}.

      Otherwise, the endpoint MUST check whether forwarding this request to (the next hop towards) the origin server is an authorized operation. This check can be based, for instance, on the specific OSCORE Security Context that the endpoint used to decrypt the incoming message, before performing this step.

      In case the authorization check fails, the endpoint MUST stop processing the request and MUST respond with a 4.01 (Unauthorized) error response to (the previous hop towards) the origin client, as per {{Section 5.10.2 of RFC7252}}. This may result in protecting the error response over that communication leg, as per {{outgoing-responses}}.

      Instead, in case the authorization check succeeds, the endpoint consumes the proxy-related options as per {{Section 5.7.2 of RFC7252}}. In particular, the endpoint checks whether the authority (host and port) of the request URI identifies the endpoint itself. In such a case, the endpoint moves to step 1.

      Otherwise, the endpoint forwards REQ to (the next hop towards) the origin server according to the request URI, unless differently indicated in REQ, e.g., by means of any of its CoAP options. For instance, a forward-proxy does not forward a request that includes proxy-related options together with the Listen-To-Multicast-Notifications Option (see {{Section 12 of I-D.ietf-core-observe-multicast-notifications}}).

      If the endpoint forwards REQ to (the next hop towards) the origin server, this may result in (further) protecting REQ over that communication leg, as per {{outgoing-requests}}.

      After that, the endpoint does not take any further action.

   * REQ includes one or more Uri-Path Options, but not the Proxy-Scheme Option or the Proxy-Scheme-Number Option.

      If the endpoint is not configured to be a reverse-proxy or its resource targeted by the Uri-Path Options is not intended to support reverse-proxy functionalities, then the endpoint proceeds to step 3.

      Otherwise, the endpoint MUST check whether forwarding this request to (the next hop towards) the origin server is an authorized operation. This check can be based, for instance, on the specific OSCORE Security Context that the endpoint used to decrypt the incoming message, before performing this step.

      In case the authentication check fails, the endpoint MUST stop processing the request and MUST respond with a 4.01 (Unauthorized) error response to (the previous hop towards) the origin client, as per {{Section 5.10.2 of RFC7252}}. This may result in protecting the error response over that communication leg, as per {{outgoing-responses}}.

      Otherwise, the endpoint consumes the Uri-Path Options as per {{Section 5.7.3 of RFC7252}}, and forwards REQ to (the next hop towards) the origin server, unless differently indicated in REQ, e.g., by means of any of its CoAP options.

      If the endpoint forwards REQ to (the next hop towards) the origin server, this may result in (further) protecting REQ over that communication leg, as per {{outgoing-requests}}.

      After that, the endpoint does not take any further action.

      Note that, when forwarding REQ, the endpoint might not remove all the Uri-Path Options originally present, e.g., in case the next hop towards the origin server is a further reverse-proxy.

3. The endpoint proceeds as defined below, depending on which of the two following conditions holds.

   * REQ does not include an OSCORE Option.

      If the endpoint does not have an application to handle REQ, it MUST stop processing the request and MAY respond with a 4.00 (Bad Request) error response to (the previous hop towards) the origin client. This may result in protecting the error response over that communication leg, as per {{outgoing-responses}}.

      Otherwise, the endpoint delivers REQ to the application.

   * REQ includes an OSCORE Option.

      If REQ includes any Uri-Path Options, the endpoint MUST stop processing the request and MAY respond with a 4.00 (Bad Request) error response to (the previous hop towards) the origin client. This may result in protecting the error response over that communication leg, as per {{outgoing-responses}}.

      Otherwise, the endpoint MUST check whether decrypting the request is an authorized operation, in view of the (previous hop towards the) origin client being the alleged request sender. This check can be based, for instance, on considering the source addressing information of the request, and then asserting whether the OSCORE Security Context indicated by the OSCORE Option is not only available to use, but also present in a local list of OSCORE Security Contexts that are usable to decrypt a request from the alleged request sender.

      In case the authorization check fails, the endpoint MUST stop processing the request and MUST respond with a 4.01 (Unauthorized) error response to (the previous hop towards) the origin client, as per {{Section 5.10.2 of RFC7252}}. This may result in protecting the error response over that communication leg, as per {{outgoing-responses}}.

      Instead, in case the authorization check succeeds, the endpoint decrypts REQ using the OSCORE Security Context indicated by the OSCORE Option, i.e., REQ* = dec(REQ). After that, the possible presence of an OSCORE Option in the decrypted request REQ* is not treated as an error situation.

      If the OSCORE processing results in an error, the endpoint MUST stop processing the request and performs error handling as per {{Section 8.2 of RFC8613}} or {{Sections 8.2 and 9.4 of I-D.ietf-core-oscore-groupcomm}}, in case OSCORE or Group OSCORE is used, respectively. In case the endpoint sends an error response to (the previous hop towards) the origin client, this may result in protecting the error response over that communication leg, as per {{outgoing-responses}}.

      Otherwise, REQ takes REQ*, and the endpoint moves to step 1.

## Processing an Outgoing Response {#outgoing-responses}

The rules from {{general-rules}} apply when processing an outgoing response message, with the following additions.

When an application endpoint applies multiple OSCORE layers in sequence to protect an outgoing response, and it uses an OSCORE Security Context shared with the other application endpoint, then the first OSCORE layer MUST be applied by using that Security Context.

The sender endpoint protects the response by applying the same OSCORE layers that it removed from the corresponding incoming request, but in the reverse order than the one according to which they were removed.

In case the response is an error response, the sender endpoint protects it by applying the same OSCORE layers that it successfully removed from the corresponding incoming request, but in the reverse order than the one according to which they were removed.

## Processing an Incoming Response {#incoming-responses}

The recipient endpoint removes the same OSCORE layers that it added when protecting the corresponding outgoing request, but in the reverse order than the one according to which they were removed.

When doing so, the possible presence of an OSCORE Option in the decrypted response following the removal of an OSCORE layer is not treated as an error situation, unless it occurs after having removed as many OSCORE layers as were added in the outgoing request. In such a case, the endpoint MUST stop processing the response.

# Caching of OSCORE-Protected Responses # {#sec-response-caching}

Although not possible as per the original OSCORE specification {{RFC8613}}, cacheability of OSCORE-protected responses at proxies can be achieved. To this end, the approach defined in {{I-D.amsuess-core-cachable-oscore}} can be used, as based on Deterministic Requests protected with the pairwise mode of Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} used end-to-end between an origin client and an origin server. The applicability of this approach is limited to requests that are safe (in the RESTful sense) to process and do not yield side effects at the origin server.

In particular, this approach requires both the origin client and the origin server to have already joined the correct OSCORE group. Then, starting from the same plain CoAP request, different clients in the OSCORE group are able to deterministically generate a same request protected with Group OSCORE, which is sent to a proxy for being forwarded to the origin server. The proxy can effectively cache the resulting OSCORE-protected response from the server, since the same plain CoAP request will result again in the same Deterministic Request and thus will produce a cache hit.

When using this approach, the following also applies in addition to what is defined in {{sec-message-processing}}, when processing incoming messages at a proxy that implements caching of responses.

* Upon receiving a request from (the previous hop towards) the origin client, the proxy checks if specifically the message available during the execution of step 2 in {{incoming-requests}} produces a cache hit.

   That is, such a message: i) is exactly the one to be forwarded to (the next hop towards) the origin server, if no cache hit has occurred; and ii) is the result of an OSCORE decryption at the proxy, if OSCORE is used on the communication leg between the proxy and (the previous hop towards) the origin client.

* Upon receiving a response from (the next hop towards) the origin server, the proxy first removes the same OSCORE layers that it added when protecting the corresponding outgoing request, as defined in {{incoming-responses}}.

   Then, the proxy stores specifically that resulting response message in its cache. That is, such a message is exactly the one to be forwarded to (the previous hop towards) the origin client.

The specific rules about serving a request with a cached response are defined in {{Section 5.6 of RFC7252}}, as well as in {{Section 7 of I-D.ietf-core-groupcomm-proxy}} for group communication scenarios.

# Establishment of OSCORE Security Contexts

Like the original OSCORE specification {{RFC8613}}, this document is not devoted to any particular approach that two OSCORE endpoints use for establishing an OSCORE Security Context.

At the same time, the following applies, depending on the two peers using OSCORE or Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} to protect their communications.

* When using OSCORE, the establishment of the OSCORE Security Context can rely on the authenticated key establishment protocol EDHOC {{I-D.ietf-lake-edhoc}}.

   Assuming that OSCORE has to be used both between the two origin application endpoints as well as between the origin client and the first proxy in the chain, it is expected that the origin client first runs EDHOC with the first proxy in the chain, and then with the origin server through the chain of proxies (see the example in {{sec-example-edhoc}}).

   Furthermore, the additional use of the combined EDHOC + OSCORE request defined in {{I-D.ietf-core-oscore-edhoc}} is particularly beneficial in this case (see the example in {{sec-example-edhoc-comb-req}}), and especially when relying on a long chain of proxies.

* The use of Group OSCORE is expected to be limited between the origin applications endpoints, e.g., between the origin client and multiple origin servers. In order to join the same OSCORE group and obtain the corresponding Group OSCORE Security Context, those endpoints can use the approach defined in {{I-D.ietf-ace-key-groupcomm-oscore}} and based on the ACE framework for authentication and authorization in constrained environments {{RFC9200}}.

  For the purposes of this document, there is no need for a proxy to also be a member of the OSCORE group whose Group OSCORE Security Context is used by the origin application endpoints for protecting communications end-to-end.

# CoAP Header Compression with SCHC

The method defined in this document enables and results in the possible protection of the same CoAP message with multiple, nested OSCORE layers. Especially when this happens, it is desirable to compress the header of protected CoAP messages, in order to improve performance and ensure that CoAP is usable also in Low-Power Wide-Area Networks (LPWANs).

To this end, it is possible to use the Static Context Header Compression and fragmentation (SCHC) framework {{RFC8724}}. In particular, {{I-D.ietf-schc-8824-update}} specifies how to use SCHC for compressing headers of CoAP messages, also when messages are protected with OSCORE. The SCHC Compression/Decompression is applicable also in the presence of CoAP proxies, and especially to the two following cases.

* In case OSCORE is not used at all, the SCHC processing occurs hop-by-hop, by relying on SCHC Rules that are consistently shared between two adjacent hops.

* In case OSCORE is used only end-to-end between the application endpoints, then an Inner SCHC Compression/Decompression and an Outer SCHC Compression/Decompression are performed (see {{Section 8.2 of I-D.ietf-schc-8824-update}}). In particular, the following holds.

   The SCHC processing occurs end-to-end as to the Inner SCHC Compression/Decompression. This relies on Inner SCHC Rules that are shared between the two application endpoints, which act as OSCORE endpoints and share the used OSCORE Security Context.

   The SCHC processing occurs hop-by-hop as to the Outer SCHC Compression/Decompression. This relies on Outer SCHC Rules that are shared between two adjacent hops.

When using the method defined in this document, and thus enabling also an intermediary proxy to be an OSCORE endpoint, the SCHC processing above is generalized as specified below.

When processing an outgoing CoAP message, a sender endpoint proceeds as follows.

* The sender endpoint performs one Inner SCHC Compression for each OSCORE layer applied to the outgoing message. Each Inner SCHC Compression occurs before protecting the message with that OSCORE layer, and relies on the SCHC Rules that are shared with the other OSCORE endpoint.

* The sender endpoint performs exactly one Outer SCHC Compression. This occurs after having performed all the intended OSCORE protections of the outgoing message, and relies on the SCHC Rules that are shared with the (next hop towards the) recipient application endpoint.

That is, with respect to the SCHC Compression/Decompression processing, the following holds.

* An Inner SCHC Compression is intended for a recipient OSCORE endpoint, which will: first, decrypt an incoming message with the OSCORE Security Context shared with the other OSCORE endpoint; and then, perform the corresponding Inner SCHC Decompression, by relying on the SCHC Rules shared with the other OSCORE endpoint.

* An Outer SCHC Compression is intended for the (next hop towards the) recipient application endpoint, which will: first, perform a corresponding Outer SCHC Decompression on an incoming message, by relying on the SCHC Rules shared with the (previous hop towards the) recipient application endpoint; then, perform a new Outer SCHC Compression on the result, by relying on the SCHC Rules shared with the (next hop towards the) recipient application endpoint; and, finally, send the result to the (next-hop towards the) recipient application endpoint.

Note that the generalization above does not alter the core approach, design choices, and features of the SCHC Compression/Decompression applied to CoAP headers.

# Security Considerations

The same security considerations from CoAP {{RFC7252}} apply to this document. The same security considerations from {{RFC8613}} and {{I-D.ietf-core-oscore-groupcomm}} apply to this document, when using OSCORE or Group OSCORE to protect exchanged messages.

Further security considerations to take into account are inherited from the specifically used CoAP options, extensions, and methods employed when relying on OSCORE or Group OSCORE.

This document does not change the security properties of OSCORE and Group OSCORE. That is, given any two OSCORE endpoints, the method defined in this document provides them with the same security guarantees that OSCORE and Group OSCORE provide in the case where such endpoints are specifically application endpoints.

## Preserving Location Anonimity

Before decrypting an incoming request (see step 3 in {{incoming-requests}}), the recipient endpoint checks whether decryption is authorized, in the light of the alleged request sender and the OSCORE Security Context to use. This is particularly relevant for an origin server that expects to receive messages protected end-to-end by origin clients, but only if sent by a reverse-proxy as its adjacent hop.

In that setup, the authorization check prevents a malicious sender endpoint C from associating the addressing information of the origin server S with their shared OSCORE Security Context CTX. Making such an association would compromise the location anonimity of the origin server, as otherwise afforded by the reverse-proxy.

That is, if C gains knowledge of some addressing information ADDR, then C might send a request directly addressed to ADDR and protected with CTX. A response protected with CTX would prove that ADDR is in fact the addressing information of S.

However, after performing and failing the authorization check on the received request, S replies with a 4.01 (Unauthorized) error response that is not protected with CTX, hence preserving the location anonimity of the origin server.

# IANA Considerations

This document has no actions for IANA.

--- back

# Examples # {#sec-examples}

This section provides a number of examples where the approach defined in this document is used to protect message exchanges.

The presented examples build on the example shown in {{Section A.1 of RFC8613}}, and illustrate an origin client requesting the alarm status from an origin server, through a forward-proxy.

The abbreviations "REQ" and "RESP" are used to denote a request message and a response message, respectively.

## Example 1

In the example shown in {{fig-example-client-proxy}}, message exchanges are protected with OSCORE over the following legs.

* End-to-end, between the client and the server, using the OSCORE Security Context CTX_C_S. The client uses the OSCORE Sender ID 0x5f when using OSCORE with the server.

* Between the client and the proxy, using the OSCORE Security Context CTX_C_P. The client uses the OSCORE Sender ID 0x20 when using OSCORE with the proxy.

~~~~~~~~~~~ aasvg
Client  Proxy  Server
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_S   |       |
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_P   |       |
  |       |       |
  +------>|       |     Code: 0.02 (POST)
  | POST  |       |    Token: 0x8c
  |       |       |   OSCORE: [kid:0x20, Partial IV:31]
  |       |       |     0xff
  |       |       |  Payload: {Code: 0.02 (POST),
  |       |       |            OSCORE: [kid:0x5f, Partial IV:42],
  |       |       |            Uri-Host: "example.com",
  |       |       |            Proxy-Scheme: "coap",
  |       |       |            0xff,
  |       |       |            {Code: 0.01 (GET),
  |       |       |             Uri-Path: "alarm_status"
  |       |       |            } // Encrypted with CTX_C_S
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
  |     Decrypt   |
  |     REQ with  |
  |     CTX_C_P   |
  |       |       |
  |       +------>|     Code: 0.02 (POST)
  |       | POST  |    Token: 0x7b
  |       |       | Uri-Host: "example.com"
  |       |       |   OSCORE: [kid:0x5f, Partial IV:42]
  |       |       |     0xff
  |       |       |  Payload: {
  |       |       |            Code: 0.01 (GET),
  |       |       |            Uri-Path: "alarm_status"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |
  |       |     Decrypt
  |       |     REQ with
  |       |     CTX_C_S
  |       |       |
  |       |     Encrypt
  |       |     RESP with
  |       |     CTX_C_S
  |       |       |
  |       |<------+     Code: 2.04 (Changed)
  |       |  2.04 |    Token: 0x7b
  |       |       |   OSCORE: -
  |       |       |     0xff
  |       |       |  Payload: {Code: 2.05 (Content),
  |       |       |            0xff,
  |       |       |            "0"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |
  |     Encrypt   |
  |     RESP with |
  |     CTX_C_P   |
  |       |       |
  |<------+       |     Code: 2.04 (Changed)
  |  2.04 |       |    Token: 0x8c
  |       |       |   OSCORE: -
  |       |       |     0xff
  |       |       |  Payload: {Code: 2.04 (Changed),
  |       |       |            OSCORE: -,
  |       |       |            0xff,
  |       |       |            {Code: 2.05 (Content),
  |       |       |             0xff,
  |       |       |             "0"
  |       |       |            } // Encrypted with CTX_C_S
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_P   |       |
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_S   |       |
  |       |       |

Square brackets [ ... ] indicate content of compressed COSE object.
Curly brackets { ... } indicate encrypted data.
~~~~~~~~~~~
{: #fig-example-client-proxy title="Use of OSCORE between Client-Server and Client-Proxy"}

## Example 2

In the example shown in {{fig-example-proxy-server}}, message exchanges are protected with OSCORE over the following legs.

* End-to-end between the client and the server, using the OSCORE Security Context CTX_C_S. The client uses the OSCORE Sender ID 0x5f when using OSCORE with the server.

* Between the proxy and the server, using the OSCORE Security Context CTX_P_S. The proxy uses the OSCORE Sender ID 0xd4 when using OSCORE with the server.

~~~~~~~~~~~ aasvg
Client  Proxy  Server
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_S   |       |
  |       |       |
  +------>|       |         Code: 0.02 (POST)
  | POST  |       |        Token: 0x8c
  |       |       |     Uri-Host: "example.com"
  |       |       | Proxy-Scheme: "coap"
  |       |       |       OSCORE: [kid:0x5f, Partial IV:42]
  |       |       |         0xff
  |       |       |      Payload: {Code: 0.01 (GET),
  |       |       |                Uri-Path: "alarm_status"
  |       |       |               } // Encrypted with CTX_C_S
  |       |       |
  |     Encrypt   |
  |     REQ with  |
  |     CTX_P_S   |
  |       |       |
  |       +------>|         Code: 0.02 (POST)
  |       | POST  |        Token: 0x7b
  |       |       |       OSCORE: [kid:0xd4, Partial IV:31]
  |       |       |         0xff
  |       |       |      Payload: {Code: 0.02 (POST),
  |       |       |                Uri-Host: "example.com",
  |       |       |                OSCORE: [kid:0x5f, Partial IV:42],
  |       |       |                0xff,
  |       |       |                {Code: 0.01 (GET),
  |       |       |                 Uri-Path: "alarm_status"
  |       |       |                } // Encrypted with CTX_C_S
  |       |       |               } // Encrypted with CTX_P_S
  |       |       |
  |       |     Decrypt
  |       |     REQ with
  |       |     CTX_P_S
  |       |       |
  |       |     Decrypt
  |       |     REQ with
  |       |     CTX_C_S
  |       |       |
  |       |     Encrypt
  |       |     RESP with
  |       |     CTX_C_S
  |       |       |
  |       |     Encrypt
  |       |     RESP with
  |       |     CTX_P_S
  |       |       |
  |       |<------+         Code: 2.04 (Changed)
  |       |  2.04 |        Token: 0x7b
  |       |       |       OSCORE: -
  |       |       |         0xff
  |       |       |      Payload: {Code: 2.04 (Changed),
  |       |       |                OSCORE: -,
  |       |       |                0xff,
  |       |       |                {Code: 2.05 (Content),
  |       |       |                 0xff,
  |       |       |                 "0"
  |       |       |                } // Encrypted with CTX_C_S
  |       |       |               } // Encrypted with CTX_P_S
  |       |       |
  |     Decrypt   |
  |     RESP with |
  |     CTX_P_S   |
  |       |       |
  |<------+       |         Code: 2.04 (Changed)
  |  2.04 |       |        Token: 0x8c
  |       |       |       OSCORE: -
  |       |       |         0xff
  |       |       |      Payload: {Code: 2.05 (Content),
  |       |       |                0xff,
  |       |       |                "0"
  |       |       |               } // Encrypted with CTX_C_S
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_S   |       |
  |       |       |

Square brackets [ ... ] indicate content of compressed COSE object.
Curly brackets { ... } indicate encrypted data.
~~~~~~~~~~~
{: #fig-example-proxy-server title="Use of OSCORE between Client-Server and Proxy-Server"}

## Example 3

In the example shown in {{fig-example-client-proxy-server}}, message exchanges are protected with OSCORE over the following legs.

* End-to-end between the client and the server, using the OSCORE Security Context CTX_C_S. The client uses the OSCORE Sender ID 0x5f when using OSCORE with the server.

* Between the client and the proxy, using the OSCORE Security Context CTX_C_P. The client uses the OSCORE Sender ID 0x20 when using OSCORE with the proxy.

* Between the proxy and the server, using the OSCORE Security Context CTX_P_S. The proxy uses the OSCORE Sender ID 0xd4 when using OSCORE with the server.

~~~~~~~~~~~ aasvg
Client  Proxy  Server
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_S   |       |
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_P   |       |
  |       |       |
  +------>|       |    Code: 0.02 (POST)
  | POST  |       |   Token: 0x8c
  |       |       |  OSCORE: [kid:0x20, Partial IV:31]
  |       |       |    0xff
  |       |       | Payload: {Code: 0.02 (POST),
  |       |       |           OSCORE: [kid:0x5f, Partial IV:42],
  |       |       |           Uri-Host: "example.com",
  |       |       |           Proxy-Scheme: "coap",
  |       |       |           0xff,
  |       |       |           {Code: 0.01 (GET),
  |       |       |            Uri-Path: "alarm_status"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |          } // Encrypted with CTX_C_P
  |       |       |
  |     Decrypt   |
  |     REQ with  |
  |     CTX_C_P   |
  |       |       |
  |     Encrypt   |
  |     REQ with  |
  |     CTX_P_S   |
  |       |       |
  |       +------>|    Code: 0.02 (POST)
  |       | POST  |   Token: 0x7b
  |       |       |  OSCORE: [kid:0xd4, Partial IV:31]
  |       |       |    0xff
  |       |       | Payload: {Code: 0.02 (POST),
  |       |       |           Uri-Host: "example.com",
  |       |       |           OSCORE: [kid:0x5f, Partial IV:42],
  |       |       |           0xff,
  |       |       |           {Code: 0.01 (GET),
  |       |       |            Uri-Path: "alarm_status"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |          } // Encrypted with CTX_P_S
  |       |       |
  |       |     Decrypt
  |       |     REQ with
  |       |     CTX_P_S
  |       |       |
  |       |     Decrypt
  |       |     REQ with
  |       |     CTX_C_S
  |       |       |
  |       |     Encrypt
  |       |     RESP with
  |       |     CTX_C_S
  |       |       |
  |       |     Encrypt
  |       |     RESP with
  |       |     CTX_P_S
  |       |       |
  |       |<------+    Code: 2.04 (Changed)
  |       |  2.04 |   Token: 0x7b
  |       |       |  OSCORE: -
  |       |       |    0xff
  |       |       | Payload: {Code: 2.04 (Changed),
  |       |       |           OSCORE: -,
  |       |       |           0xff,
  |       |       |           {Code: 2.05 (Content),
  |       |       |            0xff,
  |       |       |            "0"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |          } // Encrypted with CTX_P_S
  |       |       |
  |     Decrypt   |
  |     RESP with |
  |     CTX_P_S   |
  |       |       |
  |     Encrypt   |
  |     ERSP with |
  |     CTX_C_P   |
  |       |       |
  |<------+       |    Code: 2.04 (Changed)
  |  2.04 |       |   Token: 0x8c
  |       |       |  OSCORE: -
  |       |       |    0xff
  |       |       | Payload: {Code: 2.04 (Changed),
  |       |       |           OSCORE: -,
  |       |       |           0xff,
  |       |       |           {Code: 2.05 (Content),
  |       |       |            0xff,
  |       |       |            "0"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |          } // Encrypted with CTX_C_P
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_P   |       |
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_S   |       |
  |       |       |

Square brackets [ ... ] indicate content of compressed COSE object.
Curly brackets { ... } indicate encrypted data.
~~~~~~~~~~~
{: #fig-example-client-proxy-server title="Use of OSCORE between Client-Server, Client-Proxy and Proxy-Server"}

## Example 4 # {#sec-example-edhoc}

In the example shown in {{fig-example-edhoc}}, message exchanges are protected over the following legs.

* End-to-end, between the client and the server, using the OSCORE Security Context CTX_C_S. The client uses the OSCORE Sender ID 0x5f when using OSCORE with the server.

* Between the client and the proxy, using the OSCORE Security Context CTX_C_P. The client uses the OSCORE Sender ID 0x20 when using OSCORE with the proxy.

The example also shows how the client establishes an OSCORE Security Context CTX_C_P with the proxy and CTX_C_S with the server, by using the key establishment protocol EDHOC {{I-D.ietf-lake-edhoc}}.

~~~~~~~~~~~ aasvg
Client  Proxy  Server
  |       |       |
  +------>|       |     Code: 0.02 (POST)
  | POST  |       |    Token: 0xf3
  |       |       | Uri-Path: ".well-known"
  |       |       | Uri-Path: "edhoc"
  |       |       |     0xff
  |       |       |  Payload: (true, EDHOC message_1)
  |       |       |
  |<------+       |     Code: 2.04 (Changed)
  |  2.04 |       |    Token: 0xf3
  |       |       |     0xff
  |       |       |  Payload: EDHOC message_2
  |       |       |
Establish |       |
CTX_C_P   |       |
  |       |       |
  +------>|       |     Code: 0.02 (POST)
  | POST  |       |    Token: 0x82
  |       |       | Uri-Path: ".well-known"
  |       |       | Uri-Path: "edhoc"
  |       |       |     0xff
  |       |       |  Payload: (C_R, EDHOC message_3)
  |       |       |
  |     Establish |
  |     CTX_C_P   |
  |       |       |
  |<------+       |
  |  ACK  |       |
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_P   |       |
  |       |       |
  +------>|       |     Code: 0.02 (POST)
  | POST  |       |    Token: 0xbe
  |       |       |   OSCORE: [kid:0x20, Partial IV:0]
  |       |       |     0xff
  |       |       |  Payload: {Code: 0.02 (POST),
  |       |       |            Uri-Host: "example.com",
  |       |       |            Uri-Path: ".well-known",
  |       |       |            Uri-Path: "edhoc",
  |       |       |            Proxy-Scheme: "coap",
  |       |       |            0xff,
  |       |       |            (true, EDHOC message_1)
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
  |     Decrypt   |
  |     REQ with  |
  |     CTX_C_P   |
  |       |       |
  |       +------>|     Code: 0.02 (POST)
  |       | POST  |    Token: 0xa5
  |       |       | Uri-Host: "example.com",
  |       |       | Uri-Path: ".well-known"
  |       |       | Uri-Path: "edhoc"
  |       |       |     0xff
  |       |       |  Payload: (true, EDHOC message_1)
  |       |       |
  |       |<------+     Code: 2.04 (Changed)
  |       |  2.04 |    Token: 0xa5
  |       |       |     0xff
  |       |       |  Payload: EDHOC message_2
  |       |       |
  |     Encrypt   |
  |     RESP with |
  |     CTX_C_P   |
  |       |       |
  |<------+       |     Code: 2.04 (Changed)
  |  2.04 |       |    Token: 0xbe
  |       |       |   OSCORE: -
  |       |       |     0xff
  |       |       |  Payload: {Code: 2.04 (Changed),
  |       |       |            0xff,
  |       |       |            EDHOC message_2
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
Establish |       |
CTX_C_S   |       |
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_P   |       |
  |       |       |
  +------>|       |     Code: 0.02 (POST)
  | POST  |       |    Token: 0xb9
  |       |       |   OSCORE: [kid:0x20, Partial IV:1]
  |       |       |     0xff
  |       |       |  Payload: {Code: 0.02 (POST),
  |       |       |            Uri-Host: "example.com",
  |       |       |            Uri-Path: ".well-known",
  |       |       |            Uri-Path: "edhoc",
  |       |       |            Proxy-Scheme: "coap",
  |       |       |            0xff,
  |       |       |            (C_R, EDHOC message_3)
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
  |     Decrypt   |
  |     REQ with  |
  |     CTX_C_P   |
  |       |       |
  |       +------>|     Code: 0.02 (POST)
  |       | POST  |    Token: 0xdd
  |       |       | Uri-Host: "example.com",
  |       |       | Uri-Path: ".well-known"
  |       |       | Uri-Path: "edhoc"
  |       |       |     0xff
  |       |       |  Payload: (C_R, EDHOC message_3)
  |       |       |
  |       |     Establish
  |       |     CTX_C_S
  |       |       |
  |       |<------+
  |       |  ACK  |
  |       |       |
  |<------+       |
  |  ACK  |       |
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_S   |       |
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_P   |       |
  |       |       |
  +------>|       |     Code: 0.02 (POST)
  | POST  |       |    Token: 0x8c
  |       |       |   OSCORE: [kid:0x20, Partial IV:2]
  |       |       |     0xff
  |       |       |  Payload: {Code: 0.02 (POST),
  |       |       |            OSCORE: [kid:0x5f, Partial IV:0],
  |       |       |            Uri-Host: "example.com",
  |       |       |            Proxy-Scheme: "coap",
  |       |       |            0xff,
  |       |       |            {Code: 0.01 (GET),
  |       |       |             Uri-Path: "alarm_status"
  |       |       |            } // Encrypted with CTX_C_S
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
  |     Decrypt   |
  |     REQ with  |
  |     CTX_C_P   |
  |       |       |
  |       +------>|     Code: 0.02 (POST)
  |       | POST  |    Token: 0x7b
  |       |       | Uri-Host: "example.com",
  |       |       |   OSCORE: [kid:0x5f, Partial IV:0]
  |       |       |     0xff
  |       |       |  Payload: {Code: 0.01 (GET),
  |       |       |            Uri-Path: "alarm_status"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |
  |       |     Decrypt
  |       |     REQ with
  |       |     CTX_C_S
  |       |       |
  |       |     Encrypt
  |       |     RESP with
  |       |     CTX_C_S
  |       |       |
  |       |<------+     Code: 2.04 (Changed)
  |       |  2.04 |    Token: 0x7b
  |       |       |   OSCORE: -
  |       |       |     0xff
  |       |       |  Payload: {Code: 2.05 (Content),
  |       |       |            0xff,
  |       |       |            "0"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |
  |     Encrypt   |
  |     RESP with |
  |     CTX_C_P   |
  |       |       |
  |<------+       |     Code: 2.04 (Changed)
  |  2.04 |       |    Token: 0x8c
  |       |       |   OSCORE: -
  |       |       |     0xff
  |       |       |  Payload: {Code: 2.04 (Changed),
  |       |       |            OSCORE: -,
  |       |       |            0xff,
  |       |       |            {Code: 2.05 (Content),
  |       |       |             0xff,
  |       |       |             "0"
  |       |       |            } // Encrypted with CTX_C_S
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_P   |       |
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_S   |       |
  |       |       |

Square brackets [ ... ] indicate content of compressed COSE object.
Curly brackets { ... } indicate encrypted data.

(A, B) indicates a CBOR sequence [RFC8742]
       of two CBOR data items A and B.
~~~~~~~~~~~
{: #fig-example-edhoc title="Use of OSCORE between Client-Server and Proxy-Server, with OSCORE Security Contexts established through EDHOC"}

## Example 5 # {#sec-example-edhoc-comb-req}

In the example shown in {{fig-example-edhoc-comb-req}}, message exchanges are protected over the following legs.

* End-to-end, between the client and the server. The client uses the OSCORE Sender ID 0x5f when using OSCORE with the server.

* Between the client and the proxy. The client uses the OSCORE Sender ID 0x20 when using OSCORE with the proxy.

The example also shows how the client establishes an OSCORE Security Context CTX_C_P with the proxy and CTX_C_S with the server, by using the key establishment protocol EDHOC {{I-D.ietf-lake-edhoc}}.

In particular, the client relies on the EDHOC + OSCORE request defined in {{I-D.ietf-core-oscore-edhoc}} and denoted as COMB\_REQ, in order to transport the last EDHOC message_3 and the first OSCORE-protected application CoAP request combined together.

~~~~~~~~~~~ aasvg
Client  Proxy  Server
  |       |       |
  +------>|       |     Code: 0.02 (POST)
  | POST  |       |    Token: 0xf3
  |       |       | Uri-Path: ".well-known"
  |       |       | Uri-Path: "edhoc"
  |       |       |     0xff
  |       |       |  Payload: (true, EDHOC message_1)
  |       |       |
  |<------+       |    Code: 2.04 (Changed)
  |  2.04 |       |   Token: 0xf3
  |       |       |    0xff
  |       |       | Payload: EDHOC message_2
  |       |       |
Establish |       |
CTX_C_P   |       |
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_P   |       |
  |       |       |
Prepare   |       |
COMB_REQ  |       |
for P     |       |
from REQ  |       |
  |       |       |
  +------>|       |     Code: 0.02 (POST)
  | POST  |       |    Token: 0x82
  |       |       |   OSCORE: [kid:0x20, Partial IV:0]
  |       |       |    EDHOC: -
  |       |       |     0xff
  |       |       |  Payload: EDHOC message_3, // Intended for P
  |       |       |           {Code: 0.02 (POST),
  |       |       |            Uri-Host: "example.com",
  |       |       |            Uri-Path: ".well-known",
  |       |       |            Uri-Path: "edhoc",
  |       |       |            Proxy-Scheme: "coap",
  |       |       |            0xff,
  |       |       |            (true, EDHOC message_1)
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
  |     Establish |
  |     CTX_C_P   |
  |       |       |
  |     Rebuild   |
  |     REQ from  |
  |     COMB_REQ  |
  |       |       |
  |     Decrypt   |
  |     REQ with  |
  |     CTX_C_P   |
  |       |       |
  |       +------>|     Code: 0.02 (POST)
  |       | POST  |    Token: 0xa5
  |       |       | Uri-Host: "example.com",
  |       |       | Uri-Path: ".well-known"
  |       |       | Uri-Path: "edhoc"
  |       |       |     0xff
  |       |       |  Payload: (true, EDHOC message_1)
  |       |       |
  |       |<------+    Code: 2.04 (Changed)
  |       |  2.04 |   Token: 0xa5
  |       |       |    0xff
  |       |       | Payload: EDHOC message_2
  |       |       |
  |     Encrypt   |
  |     RESP with |
  |     CTX_C_P   |
  |       |       |
  |<------+       |     Code: 2.04 (Changed)
  |  2.04 |       |    Token: 0x82
  |       |       |   OSCORE: -
  |       |       |     0xff
  |       |       |  Payload: {Code: 2.04 (Changed),
  |       |       |            0xff,
  |       |       |            EDHOC message_2
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_P   |       |
  |       |       |
Establish |       |
CTX_C_S   |       |
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_S   |       |
  |       |       |
Prepare   |       |
COMB_REQ  |       |
for S     |       |
from REQ  |       |
  |       |       |
Encrypt   |       |
REQ with  |       |
CTX_C_P   |       |
  |       |       |
  +------>|       |     Code: 0.02 (POST)
  | POST  |       |    Token: 0x83
  |       |       |   OSCORE: [kid:0x20, Partial IV:1]
  |       |       |     0xff
  |       |       |  Payload: {Code: 0.02 (POST),
  |       |       |            Uri-Host: "example.com",
  |       |       |            OSCORE: [kid:0x5f, Partial IV:0],
  |       |       |            EDHOC: -,
  |       |       |            Proxy-Scheme: "coap",
  |       |       |            0xff,
  |       |       |            EDHOC message_3, // Intended for S
  |       |       |            {
  |       |       |             Code: 0.01 (GET),
  |       |       |             Uri-Path:"alarm_status"
  |       |       |            } // Encrypted with CTX_C_S
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
  |     Decrypt   |
  |     REQ with  |
  |     CTX_C_P   |
  |       |       |
  |       +------>|     Code: 0.02 (POST)
  |       | POST  |    Token: 0xa6
  |       |       | Uri-Host: "example.com",
  |       |       |   OSCORE: [kid:0x5f, Partial IV:0]
  |       |       |    EDHOC: -
  |       |       |     0xff
  |       |       |  Payload: EDHOC message_3, // Intended for S
  |       |       |           {
  |       |       |            Code: 0.01 (GET),
  |       |       |            Uri-Path: "alarm_status"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |
  |       |     Establish
  |       |     CTX_C_S
  |       |       |
  |       |     Rebuild
  |       |     REQ from
  |       |     COMB_REQ
  |       |       |
  |       |     Decrypt
  |       |     REQ with
  |       |     CTX_C_S
  |       |       |
  |       |     Encrypt
  |       |     RESP with
  |       |     CTX_C_S
  |       |       |
  |       |<------+     Code: 2.04 (Changed)
  |       |  2.04 |    Token: 0xa6
  |       |       |   OSCORE: -
  |       |       |     0xff
  |       |       |  Payload: {Code: 2.05 (Content),
  |       |       |            0xff,
  |       |       |            "0"
  |       |       |           } // Encrypted with CTX_C_S
  |       |       |
  |     Encrypt   |
  |     RESP with |
  |     CTX_C_P   |
  |       |       |
  |<------+       |     Code: 2.04 (Changed)
  |  2.04 |       |    Token: 0x83
  |       |       |   OSCORE: -
  |       |       |     0xff
  |       |       |  Payload: {Code: 2.04 (Changed),
  |       |       |            OSCORE: -,
  |       |       |            0xff,
  |       |       |            {Code: 2.05 (Content),
  |       |       |             0xff,
  |       |       |             "0"
  |       |       |            } // Encrypted with CTX_C_S
  |       |       |           } // Encrypted with CTX_C_P
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_P   |       |
  |       |       |
Decrypt   |       |
RESP with |       |
CTX_C_S   |       |
  |       |       |

Square brackets [ ... ] indicate content of compressed COSE object.
Curly brackets { ... } indicate encrypted data.

(A, B) indicates a CBOR sequence [RFC8742]
       of two CBOR data items A and B.
~~~~~~~~~~~
{: #fig-example-edhoc-comb-req title="Use of OSCORE between Client-Server and Proxy-Server, with OSCORE Security Contexts established through EDHOC using the EDHOC + OSCORE request"}

# State Diagram of the Incoming Request Processing # {#sec-incoming-req-diag}

{{fig-incoming-request-diagram}} overviews the processing of an incoming request, as specified in {{incoming-requests}}. The dotted boxes indicate ending states where the processing terminates.

~~~~~~~~~~~ aasvg
             +-----------------------------------------------+
Incoming --->|        Are there proxy-related options?       |<-------+
request      +-----------------------------------------------+        |
              |                           ^          |                |
             YES        ..........        |          NO               |
              |         : Return :        |          |                |
              |         : 5.05   :        |          |                |
              |         :........:        |          |                |
              |             ^             |          |                |
              |             |             |          |                |
              |             NO            |          |                |
              v             |             |          v                |
+--------------+ YES    +---------+       |  +----------------+       |
| Is there the |------->| Am I a  |       |  | Is there an    |       |
| Proxy-Uri or |        | forward |       |  | OSCORE Option? |       |
| Proxy-Cri    |  +---->| proxy?  |       |  +----------------+       |
| Option?      |  |     +---------+       |   ^   |       |           |
+--------------+  |       |               |   |   NO     YES          |
   |              |      YES              |   |   |       |           |
   NO             |       |               |   |   |       |           |
   |              |       |               |   |   |       |           |
   |              |       |               |   |   |       |           |
   |              |       |  ..........   |   |   |       |           |
   |              |       |  : Return :   |   |   |       |           |
   |              |       |  : 4.01   :   |   |   |       v           |
   |              |       |  :........:   |   |   |    +-----------+  |
   |              |       |      ^        |   |   |    | Are there |  |
   |              |       |      |        |   |   |    | Uri-Path  |  |
   |             YES      |      NO       |   |   |    | Options?  |  |
   v              |       v      |        |   |   |    +-----------+  |
+---------------------+ +---------------+ |   |   |     |         |   |
| Is there the        | | Is forwarding | |   |   |    YES        NO  |
| Proxy-Scheme or     | | the request   | |   |   |     |         |   |
| Proxy-Scheme-Number | | an authorized | |   |   |     v         |   |
| Option, together    | | operation?    | |   |   |   ..........  |   |
| with the Uri-Host   | +---------------+ |   |   |   : Return :  |   |
| or Uri-Port Option? |           |       |   |   |   : 4.00   :  |   |
+---------------------+          YES      |   |   |   ..........  |   |
   |                              |       |   |   |               |   |
   NO                             |       |   |   |               |   |
   |                              |       |   |   |               |   |
   |                              v       |   |   |               v   |
   |                  +---------------+   |   |   | +---------------+ |
   |                  | Consume the   |   |   |   | | Is decrypting | |
   |                  | proxy-related |   |   |   | | the request   | |
   |                  | options       |   |   |   | | an authorized | |
   |                  +---------------+   |   |   | | operation?    | |
   |                              |       |   |   | +---------------+ |
   |                              |       |   |   |    |         |    |
   |                              |       |   |   |    NO       YES   |
   |                              |       |   |   |    |         |    |
   |                              |      YES  |   |    |         |    |
   |                              v       |   |   |    |         |    |
   |            +--------------------------+  |   |    |         |    |
   |            | Does the authority       |  |   |    v         |    |
   |            | (host and port) of the   |  |   |  ..........  |    |
   |            | request URI identify me? |  |   |  : Return :  |    |
   |            +--------------------------+  |   |  : 4.01   :  |    |
   |                              |           |   |  :........:  |    |
   |                              NO          |   |              |    |
   |                              |           |   |              v    |
   |                              |           |   |     +---------+   |
   v                              v           |   |     | Decrypt |   |
+---------------------+    ...............    |   |     +---------+   |
| There are Uri-Path  |    : Forward the :    |   |         |         |
| Options, without    |    : request     :    |   |         |         |
| the Proxy-Scheme or |    :.............:    |   |         v         |
| Proxy-Scheme-Number |           ^           |   | +----------+      |
| Option              |           |           |   | | Success? |-YES -+
+---------------------+           |           |   | +----------+
   |                              |           |   |         |
   |                              |           |   |         NO
   |                              |           |   |         |
   |                              |           |   |         v
   |                              |           |   |    ................
   |       ..........    +---------------+    |   |    : OSCORE error :
   |       : Return :    | Consume the   |    |   |    : handling     :
   |       : 4.01   :    | proxy-related |    |   |    :..............:
   |       :........:    | options       |    |   |
   |            ^        +---------------+    |   v
   |            |                 ^           |  +--------------+
   |            NO                |           |  | Is there an  |
   |            |                 |           |  | application? |
   |       +---------------+      |           |  +--------------+
   |       | Is forwarding |      |           |        |       |
   |       | the request   |-YES--+           |       YES      NO
   |       | an authorized |                  |        |       |
   |       | operation?    |                  |        |       v
   |       +---------------+                  |        |     ..........
   |            ^                             |        |     : Return :
   |            |                             |        |     : 4.00   :
   |           YES                            |        |     :........:
   v            |                             |        v
+------------------------+                    |      ..................
| Am I a reverse proxy   |                    |      : Deliver the    :
| using the indicated    |-NO-----------------+      : request to the :
| resource for proxying? |                           : application    :
+------------------------+                           :................:
~~~~~~~~~~~
{: #fig-incoming-request-diagram title="Processing of an incoming request." artwork-align="center"}

# Document Updates # {#sec-document-updates}

RFC EDITOR: PLEASE REMOVE THIS SECTION.

## Version -00 to -01 ## {#sec-00-01}

* Clarified examples of Class U/I CoAP options that become encrypted.

* Considered also the CoAP Options Proxy-Cri and Proxy-Scheme-Number.

* Added reference to Onion CoAP as use case.

* Required to set a limit on OSCORE layers that can be added/removed.

* A forward-proxy consumes a request when identified by the request URI.

* Added authorization check before decrypting a request.

* Fixes in the examples of message exchange.

* Updated state diagram of the incoming request processing.

* Updated references.

* Editorial fixes and improvements.

# Acknowledgments # {#acknowledgments}
{: numbered="no"}

The authors sincerely thank {{{Christian Amsüss}}}, {{{Peter Blomqvist}}}, {{{David Navarro}}}, and {{{Göran Selander}}} for their comments and feedback.

The work on this document has been partly supported by VINNOVA and the Celtic-Next project CRITISEC; and by the H2020 project SIFIS-Home (Grant agreement 952652).
