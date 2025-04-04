---
title: SwitchBot Cloud
description: Instructions on how to set up SwitchBot Devices.
ha_category:
  - Hub
  - Plug
  - Remote
  - Switch
ha_release: '2023.10'
ha_iot_class: Cloud Polling
ha_codeowners:
  - '@SeraphicRav'
ha_domain: switchbot_cloud
ha_platforms:
  - switch
ha_config_flow: true
ha_integration_type: integration
---

The SwitchBot Cloud integration allows you to control SwitchBot [devices](https://www.switch-bot.com/) connected through the SwitchBot hub.

## Prerequisites

In order to use this integration, you will need at least a SwitchBot Hub and a SwitchBot account to get a token and secret key from the SwitchBot mobile app in **Profiles** > **Preferences** > **Developer Options**. If **Developer Options** is not present in preferences, tap the App Version (e.g. 6.24) several times (5~15 times) in succession to open the **Developer Options**.

Please note, device names configured in the SwitchBot app are transferred into Home Assistant.

{% include integrations/config_flow.md %}

## Supported devices

- Plug (Wi-Fi only, only available in Japan)
- Plug Mini, both the original and HomeKit-enabled
- Remotes exposed through the different hubs excepted "Others" (State is inferred from previous commands in Home Assistant and might not reflect reality if you use other ways to control the device)

<div class='note warning'>
Only the switch platform is currently supported.
</div>
