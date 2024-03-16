### Home Assistant Guides: Inovelli Black Dimmer Switch - LZW31 - Z-Wave 500 Series

Setup instructions for the [LZW31 Black Series](https://help.inovelli.com/en/collections/5651190-black-series-dimmer-switch) Z-wave dimmer switch

1. Install add-on "Z-Wave JS UI"
2. Allow it to install Z-Wave Integration
3. Do NOT allow Z-Wave integration to install Z-Wave JS add-on (not the same as Z-wave JS UI), these will conflict with each other

4.  Wire-in LZW31 according to official [wiring documentation](https://help.inovelli.com/en/articles/8478836-black-series-dimmer-switch-wiring-schematics)
5.  Load Z-Wave JS UI / hamburger menu ☰ / manage nodes ∞ / Inclusion
6.  Triple click the switch config button
7.  The device will sync, but not securely. Update firmware to 1.52 (most feature rich, 3 scene controls), check [here](https://help.inovelli.com/en/articles/8506118-black-series-dimmer-switch-firmware-changelog) if you dont need latest (and last) version 1.57)
