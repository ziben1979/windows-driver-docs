---
title: Interface to the Native 802.11 802.1X Module
description: Interface to the Native 802.11 802.1X Module
ms.assetid: 8af78e5b-c9d9-4f07-8f07-f4a156ffdb9e
keywords: ["post-association operations WDK Native 802.11 IHV Extensions DLL", "802.1X module WDK networking"]
---

# Interface to the Native 802.11 802.1X Module


**Important**  The [Native 802.11 Wireless LAN](native-802-11-wireless-lan4.md) interface is deprecated in Windows 10 and later. Please use the WLAN Device Driver Interface (WDI) instead. For more information about WDI, see [WLAN Universal Windows driver model](wifi-universal-driver-model.md).

 

After the operating system receives an NDIS\_STATUS\_DOT11\_ASSOCIATION\_COMPLETION indication from the Native 802.11 miniport driver, it calls the [*Dot11ExtIhvPerformPostAssociate*](https://msdn.microsoft.com/library/windows/hardware/ff547492) function to initiate a post-association operation by the IHV Extensions DLL.

While it performs the post-association operation or after the operation has completed, the IHV Extensions DLL can use the extensible authentication protocol (EAP) algorithms that are supported by the operating system to authenticate the user with the access point (AP). In this situation, the IHV Extensions DLL interfaces with the 802.1X module of the Native 802.11 framework for the processing of EAP packets that are sent by the AP in the EAP over LAN (EAPOL) format.

For more information about the EAPOL format, refer to Clause 7 of the IEEE 802.1X-2001 standard.

For more information about the 802.1X module and the Native 802.11 framework, see [Native 802.11 Software Architecture](native-802-11-software-architecture.md).

When interfacing the 802.1X module for user authentication, the IHV Extensions DLL must follow these guidelines:

-   For Windows Vista, the IHV Extensions DLL can initiate 802.1X authentication operations through the 802.1X module only for infrastructure basic service set (BSS) network connections.

-   The IHV Extensions DLL must register with the operating system to receive EAPOL packets. In this situation, the DLL must call the [**Dot11ExtSetEtherTypeHandling**](https://msdn.microsoft.com/library/windows/hardware/ff547587) function and add the IEEE EAPOL EtherType (0x888E) to the list of registered EtherTypes that are passed in through the *pusRegistration* parameter. After the EtherType is registered, the operating system forwards received EAPOL packets to the IHV Extensions DLL through calls to the [*Dot11ExtIhvReceivePacket*](https://msdn.microsoft.com/library/windows/hardware/ff547513) IHV Handler function.

    For more information about registering EtherTypes, see [IEEE EtherType Handling](ieee-ethertype-handling.md).

-   While it is performing the post-association operation, the IHV Extensions DLL initiates the 802.1X authentication operation by calling the [**Dot11ExtStartOneX**](https://msdn.microsoft.com/library/windows/hardware/ff547610) function. When this function is called, the operating system does the following:

    -   Display the properties page for the configuration of the 802.1X authentication. This information includes the EAP algorithm used for the authentication.
    -   Prompt the user for credentials.
    -   Send an EAPOL-Start packet to the AP to initiate the 802.1X authentication.

    The IHV Extensions DLL can call **Dot11ExtStartOneX** either within the call to [*Dot11ExtIhvPerformPostAssociate*](https://msdn.microsoft.com/library/windows/hardware/ff547492) or after the function call returns.

-   The IHV Extensions DLL can call the [**Dot11ExtStartOneX**](https://msdn.microsoft.com/library/windows/hardware/ff547610) function only after the Native 802.11 miniport driver has completed an association operation with the AP. In this situation, the IHV Extensions DLL must not call the **Dot11ExtStartOneX** function under any of the following conditions:
    -   Before the operating system calls [*Dot11ExtIhvPerformPostAssociate*](https://msdn.microsoft.com/library/windows/hardware/ff547492). The operating system calls this function after the miniport driver has successfully completed an association operation. For more information about this operation, see [Association Operations](association-operations.md).
    -   After the operating system calls [*Dot11ExtIhvStopPostAssociate*](https://msdn.microsoft.com/library/windows/hardware/ff547521). The operating system calls this function after the miniport driver has completed a disassociation operation with the AP. For more information about this operation, see [Disassociation Operations](disassociation-operations.md).
    -   After the operating system calls [*Dot11ExtIhvAdapterReset*](https://msdn.microsoft.com/library/windows/hardware/ff547434). The operating system calls this function after the miniport driver has completed a disconnection operation with the basic service set (BSS) network. For more information about this operation, see [Disconnection Operations](disconnection-operations.md).
-   While the 802.1X authentication operation is in progress, the IHV Extensions DLL can cancel the operation by calling [**Dot11ExtStopOneX**](https://msdn.microsoft.com/library/windows/hardware/ff547614).

-   While the 802.1X authentication operation is in progress, the IHV Extensions DLL must call [**Dot11ExtProcessOneXPacket**](https://msdn.microsoft.com/library/windows/hardware/ff547541) to forward EAPOL packets to the operating system for processing.
    **Note**  The IHV Extensions DLL is responsible for processing EAPOL-Key packets received from the AP. The DLL must not pass these packets to the operating system through calls to [**Dot11ExtProcessOneXPacket**](https://msdn.microsoft.com/library/windows/hardware/ff547541).

     

-   When the 802.1X authentication operation completes, the operating system calls the [*Dot11ExtIhvOneXIndicateResult*](https://msdn.microsoft.com/library/windows/hardware/ff547482) IHV Handler function. After this function is called, the IHV Extensions DLL is responsible for processing all EAPOL packets received from the AP, such as the EAPOL-Key packets used for derivation of the cipher keys.

-   If the 802.1X authentication operation completed successfully, the operating system passes the MPPE-Send-Key value to the [**DOT11\_MSONEX\_RESULT\_PARAMS**](https://msdn.microsoft.com/library/windows/hardware/ff548698) structure pointed to by the *pDot11MsOneXResultParams* parameter of [*Dot11ExtIhvOneXIndicateResult*](https://msdn.microsoft.com/library/windows/hardware/ff547482). The MPPE-Send-Key value pointed to by the **pbMPPESendKey** member of DOT11\_MSONEX\_RESULT\_PARAMS is derived through the authentication process and is used by the IHV Extensions DLL when sending EAPOL-Key packets to the AP. This key is encrypted and should be decrypted by calling the **CryptUnprotectData** function that is documented in the Windows SDK.

-   The algorithm that is used to derive the cipher keys is dependent upon the implementation of the independent hardware vendor (IHV). The IHV Extensions DLL can support standard key derivation algorithms, such as the algorithm defined in Clause 8.5 of the IEEE 802.11i-2004 standard, as well as it can support a proprietary key derivation algorithm.

-   After it derives the keys, the IHV Extensions DLL can call the following functions to download the cipher keys to the Native 802.11 miniport driver, which manages the wireless LAN (WLAN) adapter.

    -   [**Dot11ExtSetDefaultKey**](https://msdn.microsoft.com/library/windows/hardware/ff547578)

    -   [**Dot11ExtSetDefaultKeyId**](https://msdn.microsoft.com/library/windows/hardware/ff547584)

    -   [**Dot11ExtSetKeyMappingKey**](https://msdn.microsoft.com/library/windows/hardware/ff547597)

-   The IHV Extensions DLL completes the post-association operation by calling the [**Dot11ExtPostAssociateCompletion**](https://msdn.microsoft.com/library/windows/hardware/ff547530) function. After the post-association operation completes, the IHV Extensions DLL can initiate another 802.1X authentication operation if the DLL determines that the user must be reauthenticated.

The following figure shows the sequence of events when the IHV Extensions DLL initiates an 802.1X authentication operation during a post-association operation.

![diagram illustrating the series of events when the ihv extensions dll initiates an 802.1x authentication operation during a post-association operation](images/ihv-ext-802.1x.png)

 

 





