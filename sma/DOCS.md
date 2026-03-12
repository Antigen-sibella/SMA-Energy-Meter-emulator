# SMA Energy Meter emulator

This home assistant add-on can emulate the existence of one or more SMA Energy Meters on the local network. This makes it possible to use the data from other meter types and integrate them with your SMA inverter.

# features

* Emulate meter based on mqtt messages
* Auto discover HomeWizard meters and emulate meters based on there measurements.

# Configuration

## How to use MQTT

If you have a mqtt broker configured in home assistant you do not need to configure anything. Otherwise fill in the mqtt configuration in the configuration tab.

The add-on will subscribe to the following mqtt topic: `sma/emeter/<NUMERIC_METER_ID>/state`. When receiving the first message the emulator wil start the emulation of the energy meter with the provided <NUMERIC_METER_ID>. The emulator wil send a udp packet every 1000ms. The content of the packet wil stay the same until it gets updated by the next mqtt message.

```json
{
  "powerIn": 125.5,       // total power consumption in W (required)
  "powerOut": 80.3,       // total power production in W (required)
  "energyIn": 5000.603,   // total consumed energy in kWh (required)
  "energyOut": 2000.707,  // total produced energy in kWh (required)

  "powerInL1": 50.0,      // per-phase power consumption L1 in W (optional)
  "powerInL2": 40.0,      // per-phase power consumption L2 in W (optional)
  "powerInL3": 35.5,      // per-phase power consumption L3 in W (optional)
  "powerOutL1": 30.0,     // per-phase power production L1 in W (optional)
  "powerOutL2": 25.0,     // per-phase power production L2 in W (optional)
  "powerOutL3": 25.3,     // per-phase power production L3 in W (optional)

  "energyInL1": 1600.0,   // per-phase consumed energy L1 in kWh (optional)
  "energyInL2": 1700.0,   // per-phase consumed energy L2 in kWh (optional)
  "energyInL3": 1700.603, // per-phase consumed energy L3 in kWh (optional)
  "energyOutL1": 650.0,   // per-phase produced energy L1 in kWh (optional)
  "energyOutL2": 675.0,   // per-phase produced energy L2 in kWh (optional)
  "energyOutL3": 675.707, // per-phase produced energy L3 in kWh (optional)

  "destinationAddresses": [
    "192.168.1.34" // ip-address(es) to send the packets to. Should be the ip of the inverter. Leave empty to use multicast.
  ]
}
```

Per-phase fields (`L1`/`L2`/`L3`) are **optional**. If omitted, phase values are reported as 0 in the UDP packet. The total fields (`powerIn`, `powerOut`, `energyIn`, `energyOut`) are always required.

## How to use with HomeWizard meters

Enable the HomeWizard functionality in the configuration. On startup the addon will try to find the homewizard meters on the local network. When a meter is found(it can take a few minutes) a serial number will be assigned and printed to the log output. To speed up the process for the next startup you can add the hostname in the configuration in the field "HomeWizard manual addresses". 

If your homewizard meter is not automatically detected you can manually add it by entering the ip address of the meter(s) in the field "HomeWizard manual addresses". 

If the meter is not detected by the inverter you can add the ip address of your inverter in the field "HomeWizard destination ip addresses". 

# Home Assistant

example of a service call to publish the mqtt message:

```yaml
service: mqtt.publish
data:
  payload_template: |-
    {
      "powerIn": {{states('sensor.power_consumed_from_grid') | float}},
      "powerOut": {{states('sensor.power_returned_to_grid') | float}},
      "energyIn": {{states('sensor.energy_grid_consumed_helper') | float}},
      "energyOut": {{states('sensor.energy_grid_returned_helper') | float}},
      "powerInL1": {{states('sensor.power_consumed_from_grid_l1') | float(0)}},
      "powerInL2": {{states('sensor.power_consumed_from_grid_l2') | float(0)}},
      "powerInL3": {{states('sensor.power_consumed_from_grid_l3') | float(0)}},
      "powerOutL1": {{states('sensor.power_returned_to_grid_l1') | float(0)}},
      "powerOutL2": {{states('sensor.power_returned_to_grid_l2') | float(0)}},
      "powerOutL3": {{states('sensor.power_returned_to_grid_l3') | float(0)}},
      "destinationAddresses": [
          "192.168.1.34"
        ]
    }
  topic: sma/emeter/1/state
```

> **Note**: The `| float(0)` filter ensures missing sensors default to 0. Remove the per-phase lines if your meter does not provide phase-level data.
