---
layout: post
title: "Custom Controller Mapping in Unreal Engine"
date: 2025-06-03 19:30 +1100
categories: Unreal Engine
---

Having access to a driving simulation lab, I often work with sim gears like steering wheels and pedals. Mapping them in Unity is as simple as mapping keyboard, mouse, and controller, but it requires more work in Unreal Engine.

# RawInput
[RawInput](https://dev.epicgames.com/documentation/en-us/unreal-engine/rawinput-plugin-in-unreal-engine) is what we use to map unorthdox controllers in Unreal Engine. Unlike Unity, it simply does not detect unorthdox controllers unless we explicitly set up the project.

Enable the RawInput plugin under `Edit - Plugin`.

![Plugin](/assets/2025-06-03-custom-controller-mapping-in-unreal-engine/plugin.png)

For each undetected controller, we provide Unreal Engine with its `Vendor ID`, `Product ID`, `No. Axes` and `No. buttons`.

# Controller ID
`Vendor ID` and `Product ID` are hexadecimal values for each device recognised in `Device Manager`
![Device Manager](/assets/2025-06-03-custom-controller-mapping-in-unreal-engine/device_manager.png)
![Device ID](/assets/2025-06-03-custom-controller-mapping-in-unreal-engine/id.png)

Here, the `Vendor ID` is `0x046D (VID_046D)` and `Product ID` `0xC262(PID_C262)`.

![RawInput ID](/assets/2025-06-03-custom-controller-mapping-in-unreal-engine/rawinput_id.png)
Type in the IDs in `Edit - Project Settings - RawInput`

# Axes and Buttons
Unreal Engine uses `Generic USB Controller Axis/Button` to receive inputs from unrecognised devices. All we need to do is allocate them to each device and memorise which does what because as far as I know there is no way to assign unique name to each device and generic input.

If a controller supports 2 axes and 6 buttons (Fightpad?), simple add 2 axes properties and 6 button properties to the device configuration. Each property will broadcast one axis (or button) input. The order in which each axis and button is assigned to a property seems to be unique for each device.


![input_property](/assets/2025-06-03-custom-controller-mapping-in-unreal-engine/input_property.png)
In this example, the x and y axis input of the controller will be broadcast to `GenericUSBController Axis 1` and `2`. I don't know if 1 will be x - this depends on the controller. Make sure to write it down somewhere once you figure it out.

Unticking `Enabled` will make Unreal Engine ignore the input coming from that axis (or button). This is useful for preserving available inputs. And this does not mess up the inputs; If a button is assigned to index n, it will stay mapped to that index.
