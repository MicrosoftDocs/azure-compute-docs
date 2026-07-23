---
title: Previous-generation VM size series capacity limitations and migration guidance
description: Learn about capacity limitations for previous-generation Azure VM size series beginning July 2026 and get migration guidance to newer-generation VM families.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/16/2026
ms.author: mattmcinnes
---

# Previous-generation VM size series capacity limitations and migration guidance

As Azure continues to invest in higher-performing, more secure, and modern compute infrastructure, it moves away from older-generation hardware. Azure infrastructure expansion focuses on deploying newer-generation hardware that supports the latest virtual machine (VM) offerings.

As a result, certain previous-generation VM series are subject to capacity limitations starting in July 2026. Plan migrations to newer-generation VM families and series to ensure continued access to current platform capabilities, improved performance, enhanced security, and long-term capacity availability.

## Impacted VM series

The following VM series are affected by these capacity limitations:

| VM category | Impacted VM series |
|---|---|
| Compute optimized | [F](../../sizes/compute-optimized/f-family.md), [Fs](../../sizes/compute-optimized/f-family.md), [Fsv2](../../sizes/compute-optimized/fsv2-series.md) |
| General purpose | [D](../../sizes/general-purpose/d-family.md), [Ds](../../sizes/general-purpose/d-family.md), [Dv2](../../sizes/general-purpose/dv2-series.md), [Dsv2](../../sizes/general-purpose/dsv2-series.md), [Dv3](../../sizes/general-purpose/dv3-series.md), [Dsv3](../../sizes/general-purpose/dsv3-series.md), [Dv4](../../sizes/general-purpose/dv4-series.md), [Dsv4](../../sizes/general-purpose/dsv4-series.md), [Ddv4](../../sizes/general-purpose/ddv4-series.md), [Ddsv4](../../sizes/general-purpose/ddsv4-series.md), [Dav4](../../sizes/general-purpose/dav4-series.md), [Dasv4](../../sizes/general-purpose/dasv4-series.md), [B](../../sizes/general-purpose/b-family.md), [Bs](../../sizes/general-purpose/b-family.md), [Av2](../../sizes/general-purpose/av2-series.md), [Amv2](../../sizes/general-purpose/a-family.md) |
| Memory optimized | [Ev3](../../sizes/memory-optimized/e-family.md), [Esv3](../../sizes/memory-optimized/e-family.md), [Ev4](../../sizes/memory-optimized/ev4-series.md), [Esv4](../../sizes/memory-optimized/esv4-series.md), [Edv4](../../sizes/memory-optimized/edv4-series.md), [Edsv4](../../sizes/memory-optimized/edsv4-series.md), [Eav4](../../sizes/memory-optimized/eav4-series.md), [Easv4](../../sizes/memory-optimized/easv4-series.md), [G](../../sizes/memory-optimized/m-family.md), [Gs](../../sizes/memory-optimized/m-family.md) |
| Storage optimized | [Ls](../../sizes/storage-optimized/l-family.md), [Lsv2](../../sizes/storage-optimized/lsv2-series.md) |

## Recommended migration paths

If you use impacted VM series, migrate to newer-generation VM families. Recommended migration targets generally include v5, v6, and v7 VM series, depending on workload requirements, performance objectives, and storage compatibility considerations.

### Migration recommendations

| Current VM category | Impacted VM series | Recommended target VM series |
|---|---|---|
| General purpose | [Dv3](../../sizes/general-purpose/dv3-series.md), [Dsv3](../../sizes/general-purpose/dsv3-series.md), [Dv4](../../sizes/general-purpose/dv4-series.md), [Dsv4](../../sizes/general-purpose/dsv4-series.md), [Ddsv4](../../sizes/general-purpose/ddsv4-series.md), [Ddav4](../../sizes/general-purpose/d-family.md), [Dav4](../../sizes/general-purpose/dav4-series.md), [Dasv4](../../sizes/general-purpose/dasv4-series.md) | [Dv5](../../sizes/general-purpose/dv5-series.md), [Dv6](../../sizes/general-purpose/d-family.md), [Dv7](../../sizes/general-purpose/d-family.md) (based on workload requirements) |
| Memory optimized | [Ev3](../../sizes/memory-optimized/ev3-esv3-series.md), [Esv3](../../sizes/memory-optimized/ev3-esv3-series.md), [Ev4](../../sizes/memory-optimized/ev4-series.md), [Esv4](../../sizes/memory-optimized/esv4-series.md), [Edv4](../../sizes/memory-optimized/edv4-series.md), [Edsv4](../../sizes/memory-optimized/edsv4-series.md), [Easv4](../../sizes/memory-optimized/easv4-series.md), [Eav4](../../sizes/memory-optimized/eav4-series.md) | [Ev5](../../sizes/memory-optimized/ev5-series.md), [Esv6](../../sizes/memory-optimized/esv6-series.md), [Esv7](../../sizes/memory-optimized/esv7-series.md) (based on workload requirements) |

## Migration resources

To help identify the most suitable replacement VM, review the following resources:

- [VM migration guidance](d-ds-dv2-dsv2-ls-series-migration-guide.md)
- [Migrate to the v6 and v7 VM series](sizes-v6-v7-migration-overview.md)

These resources provide detailed recommendations for alternative virtual machine series, migration planning considerations, and workload-specific guidance.

## Frequently asked questions

### What does "capacity limitations" mean?

Capacity limitations mean that Azure might not approve additional quota requests, new deployments, or capacity expansion requests for impacted VM series due to limited availability of the underlying hardware. These capacity controls don't affect existing running VMs.

### Will my existing VMs be shut down because of lack of additional capacity?

No. Capacity limitations don't affect existing running virtual machines. You can continue operating existing deployments. Capacity limitations primarily affect future growth, expansion, and new capacity requests.

### What should I do if I need additional capacity?

Evaluate migration to newer-generation VM families such as v5, v6, and v7 series. Azure reviews and approves requests for newer-generation virtual machines based on regional capacity availability.

### Why are these VM series affected?

Azure prioritizes investment in newer-generation infrastructure that delivers improved performance, security, reliability, and platform capabilities. Older VM generations depend on hardware platforms that are no longer a focus for infrastructure expansion.

### When do capacity limitations begin?

Capacity limitations for the impacted VM series begin in July 2026.

### Do I need to migrate off Dv3, Dsv3, Ev3, Esv3, Dv4, Dsv4, Ev4, and Esv4 VMs immediately?

No. You don't need to migrate off Dv3, Dsv3, Ev3, Esv3, Dv4, Dsv4, Ev4, or Esv4 VMs immediately because of these capacity limitations. However, growth capacity for these VM series is limited. If you're planning expansion or new deployments, evaluate newer-generation VM series to benefit from greater capacity availability, improved performance, enhanced security, and the latest Azure innovations while avoiding future capacity constraints.

For v1 and v2 VM series, Azure announced retirement for these VMs. Migrate to the recommended target series listed in the [VM migration guidance](d-ds-dv2-dsv2-ls-series-migration-guide.md) to maintain supportability, improve performance, and reduce future operational risk.

### How do I identify the best replacement VM for my workload?

Azure provides VM migration guidance to help you identify the most appropriate replacement VM family and series based on workload requirements, performance needs, and storage compatibility considerations. Review the [VM migration guidance](d-ds-dv2-dsv2-ls-series-migration-guide.md) before selecting a replacement VM.

### Are there pricing differences between previous-generation and newer-generation VM families?

Pricing, performance, storage capabilities, and hardware characteristics might vary across VM generations. Validate workload requirements and review the documentation for the recommended target VM families and series before migration.
