# Home Assistant Guides: Inovelli Black Dimmer Switch - LZW31 - Z-Wave 500 Series

Setup instructions for the [LZW31 Black Series](https://help.inovelli.com/en/collections/5651190-black-series-dimmer-switch) Z-wave dimmer switch

The beauty of Z-Wave is that it creates its own 900Mhz mesh network. Each device is a "node" that can use other "nodes" for range extension and determine the best signal quality path for routing data. The "hub controller", which in this setup is a [USB stick](https://www.getzooz.com/zooz-zst39-z-wave-long-range-usb-stick/) attached to home assistant, is only required to initialize the network. Once your z-wave system is set up, your z-wave nodes can talk directly to each other without the need for the "hub controller". 

### Defintions:
`Z-Wave JS` and `Z-Wave JS UI` are two seperate add-ons, we want to use the latter! They will conflict with each other if both are installed!

`Z-Wave Integration` is an integration that acts as a communication bridge between Home Assistant and `Z-Wave JS` OR `Z-Wave JS UI`. Its defaults will install `Z-Wave JS`. We do not want this!


### 1. Remove Z-Wave JS add-on & Z-Wave Integration

1. If you previously installed Z-Wave JS, we can not simply uninstall it, because Z-Wave Integration will see it missing and automatically re-install it when it goes missing. Uninstall Z-Wave integration, then uninstall Z-Wave JS add-on.



### 2. Initialize Z-Wave on home assistant
```
1. Install add-on "Z-Wave JS UI" and start it
2. Install integration "Z-Wave"
3. A dialog box will show, asking to use the add-on: UNCHECK that box!
4. In the next dialog it will ask for the server. Enter: ws://a0d7b954-zwavejs2mqtt:3000
```

### 3. Install and update firmware

[wiring documentation](https://help.inovelli.com/en/articles/8478836-black-series-dimmer-switch-wiring-schematics) [firmware notes](https://help.inovelli.com/en/articles/8506118-black-series-dimmer-switch-firmware-changelog)
Use Firmware 1.52 for scene controls, use firmware 1.57 for your slave 3 way dimming switch
```
4.  Wire-in LZW31 according to official wiring documentation.
5.  Load Z-Wave JS UI / hamburger menu ☰ / manage nodes ∞ / Inclusion
6.  Triple click the switch config button
7.  The device will sync, but not securely. Update firmware to 1.52 (most feature rich, 3 scene controls), check if you need version 1.57)
8.  Load Z-Wave JS UI / hamburger menu ☰ / manage nodes ∞ / Exclusion
9.  Triple click the switch config button
10.  You should now be able to start the sync process again and sync securely.
```

### 4. Sync Dimming with Smart Bulb Mode + Adaptive Lighting

The biggest selling point for smart bulbs is to change color temperature with time of day. Bright cool lihgt during the day (600k temperature), dimmed warm light (1500k temperature) at night.
```
ENABLE detect_non_ha_changes: Detects and halts adaptations for non-light.turn_on state changes. Needs take_over_control enabled. Caution: Some lights might falsely indicate an ‘on’ state, which could result in lights turning on unexpectedly. Disable this feature if you encounter such issues.
```
