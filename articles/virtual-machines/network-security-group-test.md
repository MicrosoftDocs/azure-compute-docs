---
title: Network security group test
description: Learn how to check if a security rule is blocking traffic to or from your virtual machine (VM) using network security group test in the Azure portal.
author: halkazwini
ms.author: halkazwini
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 09/20/2024
# Customer intent: "As a network administrator, I want to test security rules for inbound and outbound connections on my virtual machine, so that I can ensure proper traffic flow and resolve connectivity issues effectively."
---

# Network security group test

You can use [network security groups](/azure/virtual-network/network-security-groups-overview) to filter and control inbound and outbound network traffic to and from your virtual machines (VMs). You can also use [Azure Virtual Network Manager](/azure/virtual-network-manager/overview) to apply admin security rules to your VMs to control network traffic.

In this article, you learn how to use **Network security group test** to check if a security rule is blocking traffic to or from your virtual machine by checking what security rules are applied to your VM traffic.

## Prerequisites

- An Azure account with an active subscription. If you don't have one, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

- Sign in to the [Azure portal](https://portal.azure.com) with your Azure account.

- An Azure virtual machine (VM). If you don't have one, create [a Linux VM](./linux/quick-create-portal.md) or [a Windows VM](./windows/quick-create-portal.md).

## Test inbound connections

In this section, you test if RDP connections are allowed to your VM from a remote IP address. 

1. In the search box at the top of the portal, enter *virtual machines*. Select **Virtual machines** from the search results.

    :::image type="content" source="./media/network-security-group-test/virtual-machines-portal-search.png" alt-text="Screenshot of searching for virtual machines in the Azure portal." lightbox="./media/network-security-group-test/virtual-machines-portal-search.png":::

1. Select the VM that you want to test.

1. Under **Connect**, select **Connect**.

    > [!NOTE]
    > The virtual machine must be in running state.

1. Under **Troubleshoot**, select **Test network security groups**.

    :::image type="content" source="./media/network-security-group-test/test-network-security-groups.png" alt-text="Screenshot that shows how to access network security groups test in the Azure portal." lightbox="./media/network-security-group-test/test-network-security-groups.png":::

1. Select **Inbound connections**. The following options are available for **Inbound** tests:

    | Setting | Value |
    | --- | --- |
    | Source type | - **My IP address**: your public IP address that you're using to access the Azure portal. <br> - **Any IP address**: any source IP address. <br> - **Other IP address/CIDR**: Source IP address or address prefix. <br> - **Service tag**: Source [service tag](/azure/virtual-network/service-tags-overview). |
    | IP address/CIDR | The IP address or address prefix that you want to use as the source. <br><br> **Note**: You see this option if you select **Other IP address/CIDR** for **Source type**. |
    | Service tag | The service tag that you want to use as the source. <br><br> **Note**: You see this option if you select **Service tag** for **Source type**. |
    | Service type | List of predefined services available for the test. <br><br> **Notes**:<br> - If you select a predefined service, the service port number and protocol are automatically selected. <br> - If you don't see the port and protocol information that you want, select **Custom**, and then enter the port number and select the protocol that you want. |
    | Port | VM port number. <br><br> **Note**: If you select one of the predefined services, the correct port number is automatically selected. <br>Manually enter the port number when you select **Custom** for **Service type**. |
    | Protocol | Connection protocol. Available options are: **Any**, **TCP**, and **UDP**. <br><br> **Note**:  If you select one of the predefined services, the correct protocol used by the service is automatically selected. <br>Manually select the protocol when you select **Custom** for **Service type**. |

1. To test if RDP connection is allowed to the VM from a remote IP address, select the following values:

    | Setting | Value |
    | --- | --- |
    | Source type | Select **My IP address**. |
    | Service type | Select **RDP**. |
    | Port | Leave the default of **3389**. |
    | Protocol | Leave the default of **TCP**. |

    :::image type="content" source="./media/network-security-group-test/inbound-test.png" alt-text="Screenshot of inbound network security group test in the Azure portal." lightbox="./media/network-security-group-test/inbound-test.png":::

1. Select **Run test**.

    After a few seconds, you see the details of the test:
    -    If RDP connections are allowed to the VM from the remote IP address, you see **Traffic status: Allowed**.
    -    If RDP connections are blocked, you see **Traffic status: Denied**. In the Summary section, you see the security rules that are blocking the traffic.

    :::image type="content" source="./media/network-security-group-test/inbound-result.png" alt-text="Screenshot of inbound network security group test result." lightbox="./media/network-security-group-test/inbound-result.png":::

    To allow the RDP connection to the VM from the remote IP address, add to the network security group a security rule that allows RDP connections from the remote IP address. This security rule must have a higher priority than the one that's blocking the traffic. For more information, see [Create, change, or delete a network security group](/azure/virtual-network/manage-network-security-group).

## Test outbound connections

In this section, you test your VM can have connect to the internet. 

1. In the search box at the top of the portal, enter *virtual machines*. Select **Virtual machines** from the search results.

    :::image type="content" source="./media/network-security-group-test/virtual-machines-portal-search.png" alt-text="Screenshot of searching for virtual machines in the Azure portal." lightbox="./media/network-security-group-test/virtual-machines-portal-search.png":::

1. Select the VM you want to test.

1. Under **Connect**, select **Connect**.

    > [!NOTE]
    > The virtual machine must be in running state.

1. Under **Troubleshoot**, select **Test network security groups**.

    :::image type="content" source="./media/network-security-group-test/test-network-security-groups.png" alt-text="Screenshot that shows how to access network security groups test in the Azure portal." lightbox="./media/network-security-group-test/test-network-security-groups.png":::

1. Select **Outbound connections**. The following options are available for **Outbound** tests:

    | Setting | Value |
    | --- | --- |
    | Service type | List of predefined services available for the test. <br><br> **Notes**:<br> - If you select a predefined service, the service port number and protocol are automatically selected. <br> - If you don't see the port and protocol information that you want, select **Custom**, and then enter the port number and select the protocol that you want. |
    | Port | VM port number. <br><br> **Note**: If you select one of the predefined services, the correct port number is automatically selected. <br>Manually enter the port number when you select **Custom** for **Service type**. |
    | Protocol | Connection protocol. Available options are: **Any**, **TCP**, and **UDP**. <br><br> **Note**:  If you select one of the predefined services, the correct protocol used by the service is automatically selected. <br>Manually select the protocol when you select **Custom** for **Service type**. |
    | Destination type | - **My IP address**: your public IP address that you're using to access the Azure portal. <br> - **Any IP address**: any source IP address. <br> - **Other IP address/CIDR**: Source IP address or address prefix. <br> - **Service tag**: Source [service tag](/azure/virtual-network/service-tags-overview). |
    | IP address/CIDR | The IP address or address prefix that you want to use as the destination. <br><br> **Note**: You see this option if you select **Other IP address/CIDR** for **Source type**. |
    | Service tag | The service tag that you want to use as the destination. <br><br> **Note**: You see this option if you select **Service tag** for **Source type**. |

1. To test if the VM can connect to the internet, select the following values:

    | Setting | Value |
    | --- | --- |
    | Service type | Select **Custom**. |
    | Port | Leave the default of **50000**. |
    | Protocol | Leave the default of **Any**. |
    | Destination type | Select **Any IP address**. |

    :::image type="content" source="./media/network-security-group-test/outbound-test.png" alt-text="Screenshot of outbound network security group test in the Azure portal." lightbox="./media/network-security-group-test/outbound-test.png":::

1. Select **Run test**.

    After a few seconds, you see the details of the test:
    -    If connections to the internet are allowed from the VM, you see **Traffic status: Allowed**.
    -    If connections to the internet are blocked, you see **Traffic status: Denied**. In the Summary section, you see the security rules that are blocking the traffic.

    :::image type="content" source="./media/network-security-group-test/outbound-result.png" alt-text="Screenshot of outbound network security group test result." lightbox="./media/network-security-group-test/outbound-result.png":::

    To allow internet connections from the VM, add to the network security group a security rule that allows connections to the internet service tag. This security rule must have a higher priority than the one that's blocking the traffic. For more information, see [Create, change, or delete a network security group](/azure/virtual-network/manage-network-security-group).

## Related content

- To learn how to troubleshoot VM connections, see [Troubleshoot connections with Azure Network Watcher](/azure/network-watcher/network-watcher-connectivity-portal).
- To learn more about network security groups, see [Network security groups overview](/azure/virtual-network/network-security-groups-overview).
