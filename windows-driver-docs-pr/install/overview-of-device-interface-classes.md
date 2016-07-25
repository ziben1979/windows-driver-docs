---
title: Overview of Device Interface Classes
description: Overview of Device Interface Classes
ms.assetid: e463e3f0-cbc8-490e-a7c4-4837d43c20e3
keywords: ["interface classes WDK device installations", "device interfaces WDK device installations", "interfaces WDK device", "device interface classes WDK device installations"]
---

# Overview of Device Interface Classes


## <a href="" id="ddk-introduction-to-device-interfaces-dg"></a>


Any driver of a physical, logical, or virtual device to which user-mode code can direct I/O requests must supply some sort of name for its user-mode clients. Using the name, a user-mode application (or other system component) identifies the device from which it is requesting I/O.

In Windows NT 4.0 and earlier versions of the NT-based operating system, drivers named their device objects and then set up symbolic links in the registry between these names and a user-visible Win32 logical name.

Starting with Windows 2000, drivers do not name device objects. Instead, they make use of *device interface classes*. A device interface class is a way of exporting device and driver functionality to other system components, including other drivers, as well as user-mode applications. A driver can register a device interface class, then enable an instance of the class for each device object to which user-mode I/O requests might be sent.

Each device interface class is associated with a GUID. The system defines GUIDs for common device interface classes in device-specific header files. Vendors can create additional device interface classes.

For example, three different types of mouse devices could be members of the same device interface class, even if one connects through a USB port, a second through a serial port, and the third through an infrared port. Each driver registers its device as a member of the interface class GUID\_DEVINTERFACE\_MOUSE. This GUID is defined in the header file *Ntddmou.h*.

Typically, drivers register for only one interface class. However, drivers for devices that have specialized functionality beyond that defined for their standard interface class might also register for an additional class. For example, a driver for a disk that can be mounted should register for both its disk interface class (GUID\_DEVINTERFACE\_DISK) and the mountable device class (MOUNTDEV\_MOUNTED\_DEVICE\_GUID).

When a driver registers an instance of a device interface class, the I/O manager associates the device and the device interface class GUID with a symbolic link name. The link name is stored in the registry and persists across system starts. An application that uses the interface can query for instances of the interface and receive a symbolic link name representing a device that supports the interface. The application can then use the symbolic link name as a target for I/O requests.

Do not confuse device interfaces with the interfaces that drivers can export in response to an [**IRP\_MN\_QUERY\_INTERFACE**](https://msdn.microsoft.com/library/windows/hardware/ff551687) request. That IRP is used to pass routine entry points between kernel-mode drivers.

 

 

[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20%5Bdevinst\devinst%5D:%20Overview%20of%20Device%20Interface%20Classes%20%20RELEASE:%20%287/22/2016%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/default.aspx. "Send comments about this topic to Microsoft")



