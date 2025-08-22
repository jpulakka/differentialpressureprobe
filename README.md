# Differential pressure probe

## Background

I wanted to measure air flows and pressure differences in my home's ventilation system. Such pressure differences are small, even 1 Pa (corresponding to a 0.1 mm water column!) matters, and that's not easy to measure. Commercial products are quite expensive. So tein itse ja säästin!

Instead of a fully standalone device like this https://github.com/ardiloot/dif-pressure-meter I wanted some digital detox because I do SW for a living. A **differential pressure probe** can be connected to a multimeter or a data logger. Such modularity is cool, and an analog voltage signal is most ubiquitous and flexible – "portable" in software terms.

## Implementation

All this is just some supporting circuitry around the fabulous Sensirion SDP series sensor. Those sensors detect pressure difference via temperature measurement of a continuous gas stream through the sensor. In other words, these are **flow-through**. Appropriate for many applications, but not all.

From SDP816-125Pa datasheet, we immediately see two things:
1. Sensitivity (V/Pa) depends on Vdd – known as "ratiometric" analog output.
2. Zero Pascal is not zero Volt – it can't be, because the power supply is single-sided, and we need to be able to measure negative pressures too, to differentiate zero from negative, even if we knew the direction of pressure difference in advance and connected the sensor accordingly.

So, while powering Sensirion from a few batteries and connecting AOut to a multimeter would work, the readings wouldn't be directly useful. For nice readings, we need
* Stable PSU
* Appropriate voltage scaling, so that, for example, 1 Pa = 10 mV
* Appropriate bias, so that 0 Pa = 0 V

Here's a simple circuit to do all that. It uses MCP1700-3302E/TO for voltage regulation and MCP6002-I/P dual op-amp as a unity gain buffer. These specific components were chosen because they're available in tangible TO/DIP packages. Also, the op amp is a "minimum surprise" one, meaning
* Unity gain stable
* Rail-to-rail input and output
→ simplicity for design, no need to think about the real world :) 

For power, 3xAAA batteries are good. Since the sensor output voltage is ratiometric, dropping voltage in batteries is a potential problem; everything might still work, kind of, but the readings are wrong! To guard for that, power LED is fed via TL431. The circuit is a bit goofy, operating on the verge of TL431 specs (requires 1 mA cathode current, with 4.33 V battery voltage I measured 0.64 V over the 750R resistor = 0.85 mA (and 1.81 V cathode-anode voltage)), and it might oscillate. But seems to fulfill its purpose: if LED is lit, then the battery voltage is sufficient for our regulator to give out good 3.3 V Vcc.

Note that the power has to be floating wrt. the instrument where the output is connected; "minus" is not "ground" in the output!

## Performance

Connected to a decent multimeter that can measure millivolts, and bias zeroed, this setup should give better than 0.1 Pa (1 mV) accuracy in the low end and 3 % span accuracy, outperforming most commercially available devices, such as ~1000 € Fluke 922 with 1 Pascal "resolution" – if we trust Sensirion's specs and haven't made some coarse mistake, that is.

Demonstrably, it works for measuring measure not only the pressure difference (→ volumetric flow rate) across air vents (a few to a few dozen Pa) and across the building envelope (~1 Pa), but even air flow across rooms, under door gaps – 0.1 … 0.3 Pa in my home – can be clearly, repeatedly measured.
