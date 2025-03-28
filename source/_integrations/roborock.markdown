---
title: Roborock
description: Instructions on how to integrate Roborock vacuums into Home Assistant
ha_category:
  - Binary Sensor
  - Number
  - Select
  - Sensor
  - Switch
  - Time
  - Vacuum
ha_iot_class: Local Polling
ha_release: 2023.5
ha_config_flow: true
ha_codeowners:
  - '@humbertogontijo'
  - '@Lash-L'
ha_domain: roborock
ha_platforms:
  - binary_sensor
  - diagnostics
  - number
  - select
  - sensor
  - switch
  - time
  - vacuum
ha_integration_type: integration
---

The Roborock integration allows you to control your [Roborock](https://us.roborock.com/pages/robot-vacuum-cleaner) vacuum while using the Roborock app.

This integration requires a cloud connection to set up the device, but it communicates with the device entirely over your home network.

Once you log in with your Roborock account, the integration will automatically discover your Roborock devices and get the needed information to communicate locally with them. Please ensure your Home Assistant instance can communicate with the local IP of your device. We recommend setting a static IP for your Roborock Vacuum to help prevent future issues. The device communicates on port 58867. Depending on your firewall, you may need to allow communication from Home Assistant to your vacuum on that port.

{% include integrations/config_flow.md %}


## Entities

Roborock devices have a variety of features that are supported on some devices but not on others. Only entities that your device supports will be added to your integration.

### Vacuum

The vacuum entity holds the ability to control most things the vacuum can do, such as start a clean, return to the dock, or set the fan speed.

### Select

Mop mode - Describes how to mop the floor. On some firmware, it is called 'mop route'.

Mop intensity - How hard you would like your vacuum to mop.

### Binary sensor

Mop attached - States if the mop is currently attached.

Mop drying status - Only available on docks with drying capabilites - States if the mop is currently being driven.

Water box attached - States if the water box is currently attached.

Water shortage - States if the water box is low on water - 'Ok' if it has not detected a water shortage.


### Sensor

Cleaning area - How much area the vacuum has cleaned in its current run.  If the vacuum is not currently cleaning, how much area it has cleaned during its last run.

Cleaning time - How long the vacuum has been cleaning for. If the vacuum is not currently cleaning, how long it cleaned for in its last run.

Cleaning progress - Only available on some newer devices - what percent of the current cleaning is completed.

Dock error - Only available on the non-basic docks - The current error of the vacuum or 'Ok' if none exist

Main brush time left - How much time is left before Roborock recommends you replace your main brush.

Mop drying remaining time - Only available on the non-basic docks - How much time is left until the mop is dry and ready to continue cleaning.

Side brush time left - How much time is left before Roborock recommends you replace your side brush.

Filter time left - How much time is left before Roborock recommends you replace your vacuum's air filter.

Status - The current status of your vacuum. This typically describes the action that is currently being run. For example, 'spot_cleaning' or 'docking'.

Last clean begin - the last time that your vacuum started cleaning.

Last clean end - The last time that your vacuum finished cleaning.

Total cleaning time - The lifetime cleaning duration of your vacuum.

Total cleaning area - The lifetime cleaning area of your vacuum.

Vacuum error - The current error with your vacuum, if there is one.

### Time

Do not disturb begin - When _Do not disturb_ is enabled, the vacuum does not run or speak after this point.

Do not disturb end - When _Do not disturb_ is enabled, the vacuum does not run or speak before this point.

### Switch

Child lock - This disables the buttons on the vacuum. Nothing happens when the buttons are pushed.

Status indicator light - This is the LED on the top of your vacuum. The color changes depending on the status of your vacuum.

Do not disturb - This enables _Do not disturb_ during the time frame you have set in the app or on the time entity. When _Do not disturb_ is enabled, the vacuum does not run or speak.

### Number

Volume - This allows you to control the volume of the robot's voice. For example, when it states "Starting cleaning". This allows you to set the volume to 0%, while the app limits it to 20%.


## FAQ

### Can I use the Mi home app with this integration?
No. This integration requires information from your Roborock app to set up and uses Roborock's protocols to communicate with your device. You must have your vacuum synced to the Roborock app.

### Can I block internet access for this device?
As of right now - no. When the vacuum is disconnected from the internet, it will attempt to disconnect itself from Wi-Fi and reconnect itself until it can reach the Roborock servers. We are looking for the best way to handle this and see what can be blocked while still allowing the vacuum to function. 

### What devices are supported?
If you can add your device to the Roborock app - it is supported. However, some older vacuums like the Roborock S5 must be connected using the Mi Home app and can be set up in Home Assistant through the [Xiaomi Miio](/integrations/xiaomi_miio/) integration.

### What features will you support?
We are working on adding a lot of features to the core integration. We have reverse-engineered over [100 commands](https://python-roborock.readthedocs.io/en/latest/api_commands.html). The documentation is currently very bare-bones, and we are looking for users to help us make it more complete. The following are some of the functionalities we plan to add to Home Assistant Core. We ask that you are patient with us as we add them.
- Selective room cleaning
- Dock controls
- Manual vacuum remote control
- Status information such as errors, clean time, consumables, etc.
- Viewing the camera
- Viewing the map

### How can I clean a specific room?
We plan to make the process simpler in the future, but for now, it is a multi-step process.
1. Make sure to first name the rooms in the Roborock app; otherwise, they won't appear in the debug log.
2. [Enable debug logging](/docs/configuration/troubleshooting/#enabling-debug-logging) for this integration and reload it.
3. Search your logs for 'Got home data' and find the attribute rooms.
4. Write the rooms down; they have a name and 6 digit ID.
5. Go to {% my developer_call_service service="vacuum.send_command" title="**Developer Tools** > **Services** > **Vacuum: Send Command**" %}. Select your vacuum as the entity and `get_room_mapping` as the command.
6. Go back to your logs and look at the response to `get_room_mapping`. This is a list of the 6-digit IDs you saw earlier to 2-digit IDs (use the first number, for instance `16` in `[16, '14000663', 12]` ([internal room id, unique room id, room type])). In your original list of room names and 6-digit IDs, replace the 6-digit ID with its pairing 2-digit ID.
7. Now, you have the 2-digit ID that your vacuum uses to describe a room.
8. Go back to {% my developer_call_service service="vacuum.send_command" title="**Developer Tools** > **Services** > **Vacuum: Send Command**" %} then type `app_segment_clean` as your command and `segments` with a list of the 2-digit IDs you want to clean. Then, add `repeat` with a number (ranging from 1 to 3) to determine how many times you want to clean these areas.

Example:
```yaml
service: vacuum.send_command
data:
  command: app_segment_clean
  params:
    - segments:
        - 22
        - 23
      repeat: 2
target:
  entity_id: vacuum.s7_roborock

```
