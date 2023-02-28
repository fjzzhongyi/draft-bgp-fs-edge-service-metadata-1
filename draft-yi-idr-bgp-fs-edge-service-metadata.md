---
title: Distribution of Service Metadata in BGP FlowSpec
abbrev: Service Metadata in BGP FlowSpec
docname: draft-yi-idr-bgp-fs-edge-service-metadata-lastest
obsoletes:
updates:
date:
category: std
submissionType: IETF

ipr: trust200902
area: Routing Area
workgroup: IDR
keyword: Internet-Draft

author:
 -
  ins: X.Yi
  name: Xinxin Yi
  organization: China Unicom
  email: yixx3@chinaunicom.cn
  role: editor
  city: Beijing
  country: China
 -
  ins: T.He
  name: Tao He
  organization: China Unicom
  email: het21@chinaunicom.cn
  role: editor
  city: Beijing
  country: China
 -
  ins: H.Shi
  name: Hang Shi
  organization: Huawei Technologies
  email: shihang9@huawei.com
  role: editor
  city: Beijing
  country: China
 -
  ins: X.Ding
  name: Xiangfeng Ding
  organization: Huawei Technologies
  email: dingxiangfeng@huawei.com
  city: Beijing
  country: China
 -
  ins: H.Wang
  name: Haibo Wang
  organization: Huawei Technologies
  email: rainsword.wang@huawei.com
  city: Beijing
  country: China

normative:
  RFC8955:
  RFC8956:
  I-D.ietf-idr-5g-edge-service-metadata:
  I-D.ietf-idr-ts-flowspec-srv6-policy:

informative:

...

--- abstract

In edge computing, a service may be deployed on multiple instances within one or more sites, called edge service. The edge service is associated with an ANYCAST IP address, and the route of it along with service metadata can be collected by a central controller. The controller may process the metadata and distribute the result to ingress routers using BGP FlowSpec. The service metadata can be used by ingress routers to make path selections not only based on the routing cost but also the running environment of the edge services. This document describes a mechanism to distribute the information of the service routes and related service metadata using BGP FlowSpec.


--- middle

# Introduction {#intro}

Many modern services deploy their service instances in multiple sites to get better response time and resource utilization. These sites are often geographically distributed to serve the user demand. For some services such as VR/AR and intelligent transportation, the QoE will depend on both the network metrics and the compute metrics. For example, if the nearest site is overloaded due to the demand fluctuation, then steer the user traffic to another light-loaded sites may improve the QoE. To steer the traffic to the best site, the computing metadata of the site needs to be collected.

{{I-D.ietf-idr-5g-edge-service-metadata}} describes the BGP extension of distributing service route with network and computing-related metrics. The router connected to the site will received the service routes and service metadata sent from devices inside the edge site, and then generates the corresponding routes and distributes them to ingress routers. However, the route with service metadata on the router connected to the site can be also collected by a central controller using BGP LS. Then the central controller may process the metadata and distributes the result to the ingress router using BGP FlowSpec.

This document defines an extension of BGP FlowSpec to carry the service metadata along with the service route which is received from the controller. Using the service metadata and the service route, the ingress router can calculate the best site for the traffic, giving each user the best QoE.


## Terminology

## Requirements Language

{::boilerplate bcp14-tagged}

# BGP FlowSpec Extension for Service Metadata

The goal of the BGP FlowSpec extension is to distribute the information of the service route and metadata. A service is identified by an prefix and this information is carried using the existing Destination Prefix Component specified in {{RFC8955}} and {{RFC8956}}. {{I-D.ietf-idr-ts-flowspec-srv6-policy}} defines that the Color Extended Community and BGP Prefix-SID attribute is carried in the context of the FlowSpec NLRI.

In addition to that, this document proposes to carry the service metadata attribute(See {{fig-example}}). The ingress router can compare the compute metric of different sites and steer the traffic into the best one using the SR policy. The metadata can be original values defined in {{I-D.ietf-idr-5g-edge-service-metadata}} or an aggregated one calculated using original values.

~~~
   +------------+
   |  BGP FS    |
   | Controller |
   +------------+
      | FlowSpec route to Ingress:
      |   NLRI: Destination Prefix
      |   Redirect to IPv6 Nexthop: Egress's Address
      |   Policy Color: C1
      |   PrefixSID: End.X1
      |   Service Metadata: Compute metric
      |          .-----.
      |         (       )
      V     .--(         )--.
+-------+  (                 )  +------+          +---------+
|       |_( SRv6 Core Network )_|      | (End.X1) |         |
|Ingress| ( ================> ) |Egress|----------|   Site  |
+-------+  (SR List<S1,S2,S3>)  +------+          +---------+
            '--(         )--'
                (       )
                 '-----'
~~~
{: #fig-example title="Example of using BGP FlowSpec to distribute the service route and metadata"}


## Metadata Path Attribute TLV

The Metadata Path Attribute TLV is the same as defined in {{I-D.ietf-idr-5g-edge-service-metadata}}, including the following three sub-TLVs:

1. Site Preference Index sub-TLV indicates the preference to choose the site.
2. Capacity Index sub-TLV indicates the capability of a site. One Edge Site can be in full capacity, reduced capacity, or completely out of service.
3. Load Measurement sub-TLV indicates the load level of the site.

## Aggregated Metric Path Attribute TLV {#metadata}

The Aggregated Metric Path Attribute is a newly defined TLV(See {{fig-Aggregated-Metric-Attribute}}). It contains a single aggregated value which is calculated by the controller using the original metrics such as site preference, capacity and load measurement.

~~~
   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |    Aggregated Metadata Type   |            Length             |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |            Aggregated Metric Value (4 octets)                 |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-Aggregated-Metric-Attribute title="Aggregated Metric Path Attribute TLV format"}

- Type: identify the Aggregated Metadata Attribute, to be assigned by IANA.
- Length: the total number of the octets of the value field.
- Value: value of Aggregated Computing metric.


# Security Considerations

TBD


# IANA Considerations

This document requires IANA to assign the following code points from the registry called "BGP Path Attributes":


| Value | Description | Reference |
|-------|-------------|-----------|
| TBD1  | Aggregated Metadata Type | {{metadata}} |
