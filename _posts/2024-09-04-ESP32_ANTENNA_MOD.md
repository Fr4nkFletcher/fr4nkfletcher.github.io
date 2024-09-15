---
layout: post
title: "Enhancing ESP Board Reception without an External Antenna Connector"
date: 2024-09-14
tags: [esp, rx/tx, antenna, mod, esp32]
---

![Enhancing ESP Board Reception](https://media.istockphoto.com/id/1147102296/vector/evil-wizard-putting-spell-cartoon-pixel-art-character-isolated-on-white-background-old.jpg?s=612x612&w=0&k=20&c=9PhU8LpDn6K0ZF-RKPUlEw7K0xpqGzVOgPD-v0AuqaA=)

If you've purchased an ESP board that lacks a connector for an external antenna and you're experiencing the limits of reception, there's a solution that doesn't involve the complex task of moving a tiny 0603 sized resistor to utilize the external antenna socket on some boards.

This workaround is relatively simple and the required parts are inexpensive, often found on platforms like eBay or AliExpress for just a few dollars.

## Required Components

![Components](/assets/img/1.jpeg)

1. **ESP/CYD of your choosing**
2. **UFL to reverse polarity SMA lead with UFL connector removed and 5mm of shield and center exposed and tinned with solder**
3. **Reverse polarity SMA 2.4GHz antenna**

## Step-by-Step Guide

### 1. Prepare Your Workspace and Tools

Make sure you have a steady hand and the right tools. A fine soldering tip and some magnification can help.

![Step 1](/assets/img/2.jpeg)

### 2. Locate the Antenna Connection Points

Identify the points on your ESP board where the antenna connects. This will be your working area.

![Step 2](/assets/img/3.jpeg)

### 3. Attach the Antenna

Carefully solder the antenna's lead to the designated point on the ESP board. Ensure a solid connection without bridging adjacent contacts.

![Step 3](/assets/img/4.jpeg)

### 4. Test the Connection

Before proceeding further, it's crucial to test the connection. Use a multimeter to ensure there's nothing short.

![Step 4](/assets/img/5.jpeg)

### 5. Seal and Protect

Once you've confirmed the functionality, consider using some non-conductive lacquer to protect the exposed soldered area.

![Step 5](/assets/img/6.jpeg)

### 6. Final Testing

With the external antenna now attached, test the ESP board's reception in its intended environment. You should notice a significant improvement in signal strength and stability.

![Final Result](/assets/img/7.jpeg)

## Conclusion

By following these steps, you can significantly improve the reach of your esp32. For more information and examples, click [here](https://community.home-assistant.io/t/how-to-add-an-external-antenna-to-an-esp-board).
