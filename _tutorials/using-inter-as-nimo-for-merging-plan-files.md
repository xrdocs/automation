---
published: false
date: '2023-08-06 10:23 +0800'
title: Using inter-as-nimo for merging plan files
author: Fung Lim
excerpt: Using inter-as-nimo to merge plan files
---
## A New Post

Plan files from different Autonomous Systems (AS) can be merged using the inter-as-nimo. The inter-as-nimo resolves any conflicts across the plan files. Plan files in native format are supported.

In addition, inter-as-nimo may be used to merge plan files from the same AS. Use cases could include networks regions that may only be collected via specific methods (e.g. topo-igp-nimo in one region, and topo-bgpls-xtc-nimo in another region). 

Note: WAE does not support the collection of multiple basic networks on the same server. If multiple basic networks need to be collected concurrently, they will require separate WAE server instances.





