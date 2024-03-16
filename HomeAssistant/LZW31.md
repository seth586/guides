### Home Assistant Guides: Inovelli Black Dimmer Switch - LZW31 - Z-Wave 500 Series

Setup instructions for the [LZW31 Black Series](https://help.inovelli.com/en/collections/5651190-black-series-dimmer-switch) Z-wave dimmer switch

THe beauty of Z-Wave is that it creates its own 900Mhz mesh network. Each device is a "node" that can use other "nodes" for range extension and determine the best signal quality path for routing data. The "hub controller", which in this setup is a [USB stick](https://www.getzooz.com/zooz-zst39-z-wave-long-range-usb-stick/) attached to home assistant, is only required to initialize the network. Once your z-wave system is set up, your z-wave nodes can talk directly to each other without the need for the "hub controller". 

1. Install add-on "Z-Wave JS UI"
2. Allow it to install Z-Wave Integration
3. Do NOT allow Z-Wave integration to install Z-Wave JS add-on (not the same as Z-wave JS UI), these will conflict with each other


   

5.  Wire-in LZW31 according to official [wiring documentation](https://help.inovelli.com/en/articles/8478836-black-series-dimmer-switch-wiring-schematics)
6.  Load Z-Wave JS UI / hamburger menu ☰ / manage nodes ∞ / Inclusion
7.  Triple click the switch config button
8.  The device will sync, but not securely. Update firmware to 1.52 (most feature rich, 3 scene controls), check [here](https://help.inovelli.com/en/articles/8506118-black-series-dimmer-switch-firmware-changelog) if you dont need latest (and last) version 1.57)
9.  Load Z-Wave JS UI / hamburger menu ☰ / manage nodes ∞ / Exclusion
10.  Triple click the switch config button
11.  You should now be able to start the sync process again and sync securely.
