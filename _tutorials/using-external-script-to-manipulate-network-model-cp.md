---
published: true
date: '2026-03-02 20:34 +0800'
title: Using External Scripts to manipulate network models in Crosswork Planning
author: Fung Lim
---
# Tutorial: Using External Scripts to Manipulate Network Models in Crosswork Planning

Crosswork Planning's Collector framework supports [running external scripts against a network model](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration/m-collectors-in-cp.html#run-external-scripts). An external script takes an existing plan file produced by an upstream collector, modifies it programmatically using the **OPM Python Library**, and writes the result back out. The Collector framework handles all the file plumbing — it passes the source plan file path and the output plan file path as command-line arguments. This gives you a powerful extension point: any transformation you can express in Python can be inserted into an automated collection chain.

This tutorial teaches you how to build, deploy, and verify an external script within the Crosswork Planning Collector. As a concrete example, we use a script — **`update_interface_metric.py`** — that copies IPv6 IGP and TE metrics into their IPv4 counterparts, a common need in dual-stack networks where the IPv6 metric is authoritative but downstream analysis tools only read IPv4 columns. The same workflow applies to any custom transformation you need to perform on a network model.

---

## Overview

| | |
|---|---|
| **Objective** | Learn how to use Crosswork Planning external scripts to programmatically manipulate a network model as part of a collection chain |
| **Example use case** | Copying IPv6 IGP/TE metric values into IPv4 IGP/TE metric fields in a dual-stack network model |
| **Script** | `update_interface_metric.py` |
| **Mechanism** | Crosswork Planning **External Script** collector, integrated into an existing collection chain |
| **Audience** | Network planners and operations engineers who want to extend Crosswork Planning with custom model transformations |
| **Time to complete** | ~15 minutes |

---

## Background

### What Is an External Script?

An external script is a user-written program (typically Python) that the Crosswork Planning Collector framework can execute as a step in a collection chain. The framework:

1. Runs an upstream collector to produce a plan file.
2. Invokes your script, passing the source plan file path and an output plan file path as command-line arguments.
3. Takes your script's output plan file and feeds it to the next step in the chain.

Because the script receives a full network model via the **OPM Python Library**, you can read, modify, or augment any element — nodes, interfaces, circuits, demands, metrics, and more — then write the result back. This makes external scripts ideal for tasks such as:

- Copying or transforming metric values between tables
- Enriching models with data from external systems (CMDBs, traffic databases)
- Normalizing naming conventions or tagging interfaces
- Pruning or filtering model elements for specific analysis scenarios

### Example Use Case: IPv6 Metric Synchronization

To illustrate the workflow, this tutorial uses a practical scenario common in dual-stack networks.

Accurate interface metrics are essential for capacity planning and "what-if" simulations. Crosswork Planning's predictive simulation engine relies on the IGP metric and TE metric columns in the plan file to model shortest-path routing, traffic distribution, and failure scenarios.

In networks running IS-IS with both IPv4 and IPv6 address families, operators often configure distinct metrics per address family. The SR-PCE collector faithfully records these into separate tables:

- **IPv4 metrics** → `IGPMetric` and `TEMetric` columns on the Interfaces table
- **IPv6 metrics** → `ipv6IGPMetric` and `ipv6TEMetric` columns on the IPv6-IGP metric table

If your network's authoritative routing metric is the IPv6 metric (a common pattern in IPv6-first or dual-stack deployments), then the IPv4 metric columns used by Crosswork Planning's simulation engine may contain stale or default values. This leads to inaccurate capacity plans and misleading failure simulations.

The `update_interface_metric.py` script solves this by automatically copying the IPv6 values into the IPv4 columns on every collection run.

---

## Prerequisites

Before you begin, ensure you have:

- A working **Cisco Crosswork Planning 7.2** (or later) deployment
- An existing collection configuration that includes the **SR-PCE collector** (or IGP database collector) discovering your dual-stack network
- The `update_interface_metric.py` script file (provided below or in your lab files)
- Familiarity with the Crosswork Planning Collection UI ([Collection Setup and Administration Guide](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration.html))

---

## Understanding the Example Script

Let's examine `update_interface_metric.py` to see how an external script interacts with the OPM Python Library.

### Full Script

```python
"""Copy IPv6 IGP/TE metrics to IP metrics in a plan file

Used as a Crosswork Planning external script. The Collector framework
passes positional arguments: base planfile path, output planfile path,
and possibly extra args (ignored).
"""

import sys
from com.cisco.wae.opm.network import Network

def copy_ipv6_metrics(src, dest):
    """Read a planfile, copy IPv6 metrics to IP metrics, and write the result."""
    network = Network(src)
    copy_count = 0

    for node in network.model.nodes:
        for interface in node.interfaces:
            """rpc_record is used for v6 metrics since this is not exposed in OPM on node.interfaces"""
            rec = interface.rpc_record
            v6igp = rec.ipv6IGPMetric
            v6te = rec.ipv6TEMetric

            if isinstance(v6igp, int) and v6igp != interface.igp_metric:
                print("Copying IPv6 IGP Metric for {} {}: {} -> {}".format(
                    node.name, interface.name, v6igp, interface.igp_metric))
                interface.igp_metric = v6igp
                copy_count += 1

            if isinstance(v6te, int) and v6te != interface.te_metric:
                print("Copying IPv6 TE Metric for {} {}: {} -> {}".format(
                    node.name, interface.name, v6te, interface.te_metric))
                interface.te_metric = v6te
                copy_count += 1

    network.write(dest)
    print("Total number of interface metrics updated: {}".format(copy_count))


if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("Usage: {} <base_planfile> <output_planfile> [extra args...]".format(sys.argv[0]))
        sys.exit(1)

    base_planfile = sys.argv[1]
    output_planfile = sys.argv[2]
    # Extra positional args from the Collector framework are ignored.

    copy_ipv6_metrics(base_planfile, output_planfile)
```

### How It Works

The script uses the **Crosswork Planning OPM Python Library** (`com.cisco.wae.opm.network`) to interact with the plan file. Here is what each section does:

| Step | Code | Purpose |
|------|------|---------|
| 1 | `Network(src)` | Opens and loads the source plan file into an OPM Network object |
| 2 | `network.model.nodes` → `node.interfaces` | Iterates over every node and interface in the model |
| 3 | `interface.rpc_record` | Accesses the low-level RPC record to read IPv6 metric fields that are not directly exposed on the OPM interface object |
| 4 | `rec.ipv6IGPMetric`, `rec.ipv6TEMetric` | Reads the IPv6 IGP metric and TE metric values |
| 5 | `interface.igp_metric = v6igp` | Copies the IPv6 value into the IPv4 IGP metric field (only when the value is a valid integer and differs from the current value) |
| 6 | `network.write(dest)` | Writes the modified network model to the output plan file |

> **Note:** The `rpc_record` accessor is used because `ipv6IGPMetric` and `ipv6TEMetric` are not exposed as first-class properties on the OPM `interface` object.

### Collector Framework Arguments

When the Collector framework invokes any external script, it passes these positional arguments:

| Argument | Description | Used by this script? |
|----------|-------------|----------------------|
| `argv[1]` | Source plan file | ✅ Yes |
| `argv[2]` | Output plan file | ✅ Yes |
| `argv[3]` | Device access authentication file | ❌ Ignored |
| `argv[4]` | Global network access configuration file | ❌ Ignored |
| `argv[5]` | Home directory | ❌ Ignored |
| `argv[6]` | Path to user-uploaded external files | ❌ Ignored |
| `argv[7]` | Path to archive root directory | ❌ Ignored |

Since this script only reads from and writes to plan files (no live device access needed), it only uses `argv[1]` and `argv[2]`. Other scripts — for example, one that enriches the model with live device data — could use `argv[3]` and `argv[4]` for authentication and network access configuration.

---

## Step-by-Step Deployment

### Step 1: Prepare the Script File

Save the script as `update_interface_metric.py`. No additional dependencies or supporting files are needed — the OPM library is pre-installed in Crosswork Planning's Python environment.

> **Tip:** If you have additional helper modules, you can bundle everything into a `.zip` archive. For this script, a single `.py` file is sufficient.

### Step 2: Open Your Collection Configuration

1. Log in to the **Crosswork Planning** UI.
2. Navigate to the collection you want to modify. This should be an existing collection that uses the **SR-PCE collector** (or IGP database collector) for topology discovery.
3. Click **Edit** to modify the collection configuration.

If you don't have an existing collection, create a new one following the [Configure a collection](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration/m-create-network-models.html#configure_collections) guide, selecting SR-PCE as the basic topology collector.

### Step 3: Add the External Script

On the **Configure** page of your collection:

1. Decide where in the collection chain to place the script. The recommended placement is **after** the SR-PCE (or IGP database) collector and any LSP/PCEP collectors, but **before** DARE aggregation. You can add it under:
   - **Basic topology** — if you want it to run right after topology discovery
   - **Advanced modeling** — if you have LSP or PCEP collectors and want metrics updated after those run
   - **Traffic and Demands** — if you want it to run later in the chain

2. Click **+ Add external script** under the appropriate section.

### Step 4: Configure the External Script Parameters

Fill in the following fields:

| Option | Value |
|--------|-------|
| **Collector name** | `Copy IPv6 Metrics` (or any descriptive name) |
| **Is source a plan file?** | Leave **unchecked** (the source comes from an upstream collector) |
| **Source** | Select the upstream collector whose output should be processed (e.g., `SR-PCE`, `PCEP LSP`, or an aggregator) |
| **Input file** | Click **Browse** and upload `update_interface_metric.py` |
| **Executable script** | `update_interface_metric.py` |
| **Script language** | **Python** |
| **Timeout** | `30` minutes (default; adjust if your network model is very large) |

> **Important:** The **Source** field determines which collector's output plan file is fed to your script as `argv[1]`. Choose the collector that produces the most complete model with IPv6 metrics already populated.

### Step 5: Preview and Save

1. Click **Next** to proceed to the Preview page.
2. Review the collection chain. You should see your external script listed after its source collector. A typical chain looks like:

   ```
   SR-PCE Collector → PCEP LSP Collector → Copy IPv6 Metrics → DARE Aggregation
   ```

   Or with the IGP database collector:

   ```
   IGP Database Collector → LSP Collector → Copy IPv6 Metrics → DARE Aggregation
   ```

3. Click **Create** (for a new collection) or **Save** (for an edit) to apply the configuration.

### Step 6: Schedule and Run the Collection

1. Configure a **collection schedule** — either run immediately or set a recurring interval. For details, see [Schedule a collection](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration/m-create-network-models.html#configure_schedules).
2. Trigger the collection and wait for it to complete.

### Step 7: Verify the Results

After the collection completes:

1. Open the resulting plan file in **Crosswork Planning Design**.
2. Navigate to the **Interfaces** table.
3. Compare the **IGPMetric** column values with the **ipv6IGPMetric** values in the IPv6-IGP metric table. They should now match for all interfaces that had valid IPv6 metrics.
4. Similarly, verify the **TEMetric** values.

To review the script's execution log:

1. From the main menu, choose **Administration > Show Tech**.
2. Click the **Microservices** tab.
3. Request logs for the `collection-service`.
4. Download and inspect the log files. Look for output lines like:

   ```
   Copying IPv6 IGP Metric for node-1 GigabitEthernet0/0/0/0: 100 -> 10
   Copying IPv6 TE Metric for node-1 GigabitEthernet0/0/0/0: 200 -> 20
   Total number of interface metrics updated: 42
   ```

---

## Using the Updated Plan File for Analysis

With the IPv6 metrics now correctly reflected in the IPv4 columns, you can use Crosswork Planning Design's full suite of simulation and analysis tools with accurate metric data:

### Capacity Planning

- Run **worst-case analysis** to identify links at risk of congestion under the authoritative IPv6 metric topology.
- Use **traffic simulations** to verify that IGP shortest-path routing provides adequate capacity under the authoritative metrics.

### Failure Simulation

- Simulate single and multi-failure scenarios (SRLG failures, node failures) to verify network resiliency with the correct metrics.

### What-If Analysis

- Model metric changes ("what if we increase the metric on this link?") to optimize traffic distribution.
- Evaluate the impact of adding or removing links with accurate baseline metrics.

---

## Troubleshooting

| Symptom | Possible Cause | Resolution |
|---------|---------------|------------|
| Script runs but no metrics are updated | IPv6 metrics are not populated by the upstream collector | Verify that the SR-PCE collector is discovering IPv6 metrics. Check that your IGP protocol selection includes `isisv6` or that dual-stack discovery is enabled. |
| Script fails with import error | OPM library not available | Ensure the script runs within the Crosswork Planning environment, not standalone. The `com.cisco.wae.opm.network` library is provided by the platform. |
| Script times out | Very large network model | Increase the **Timeout** value in the external script configuration. |
| Metrics are overwritten on next collection run | Expected behavior | The external script runs on every collection cycle, so metrics are re-synchronized automatically. |
| IPv6 metric shows as `None` | Interface has no IPv6 metric configured | The script only copies metrics that are valid integers. Interfaces without IPv6 metrics are left unchanged. |

---

## API Reference

This script uses the following components from the Crosswork Planning OPM Python Library (see [API documentation](https://developer.cisco.com/docs/crosswork/planning/)):

| OPM Class / Method | Purpose |
|---------------------|---------|
| `com.cisco.wae.opm.network.Network(plan_file)` | Opens and loads a plan file into an OPM Network object |
| `network.model.nodes` | Iterable collection of all node objects in the network model |
| `node.interfaces` | Iterable collection of all interface objects on a given node |
| `interface.rpc_record` | Accesses the underlying RPC record for fields not directly exposed in OPM |
| `rpc_record.ipv6IGPMetric` | IPv6 IGP metric value for the interface |
| `rpc_record.ipv6TEMetric` | IPv6 TE metric value for the interface |
| `interface.igp_metric` | Read/write property for the IPv4 IGP metric |
| `interface.te_metric` | Read/write property for the IPv4 TE metric |
| `network.write(dest_file)` | Writes the modified network model to a plan file |

---

## Summary

In this tutorial you learned how to:

1. **Understand external scripts** — How the Crosswork Planning Collector framework integrates user-written scripts into a collection chain, passing plan file paths as arguments
2. **Use the OPM Python Library** — How to open a plan file, traverse the network model, read and modify element properties, and write the result
3. **Deploy an external script** — How to configure the Crosswork Planning Collector to run your script as part of an automated collection chain
4. **Verify results** — How to confirm that your script's transformations are correctly applied to the output plan file

The IPv6-to-IPv4 metric copy demonstrated here is just one example. The same pattern — open, transform, write — applies to any custom manipulation of the network model. Refer to the [OPM Python API documentation](https://developer.cisco.com/docs/crosswork/planning/) and the [external script documentation](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration/m-collectors-in-cp.html#run-external-scripts) to explore what else you can build.

---

## References

- [Run an external script against a network model](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration/m-collectors-in-cp.html#run-external-scripts) — Cisco Crosswork Planning 7.2 Collection Setup and Administration Guide
- [Crosswork Planning OPM Python API](https://developer.cisco.com/docs/crosswork/planning/) — Developer documentation

- [Getting started with Crosswork Planning Collector](https://xrdocs.io/automation/tutorials/getting-started-crosswork-planning-collector) — xrdocs.io tutorial on collection setup