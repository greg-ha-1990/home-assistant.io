---
title: MQTT
description: Instructions on how to setup MQTT within Home Assistant.
ha_category:
  - Hub
  - Update
featured: true
ha_release: pre 0.7
ha_iot_class: Local Push
ha_config_flow: true
ha_codeowners:
  - '@emontnemery'
  - '@jbouwh'
ha_domain: mqtt
ha_platforms:
  - alarm_control_panel
  - binary_sensor
  - button
  - camera
  - climate
  - cover
  - device_tracker
  - diagnostics
  - event
  - fan
  - humidifier
  - image
  - light
  - lock
  - number
  - scene
  - select
  - sensor
  - siren
  - switch
  - tag
  - text
  - update
  - vacuum
  - water_heater
ha_integration_type: integration
ha_quality_scale: gold
---

MQTT (aka MQ Telemetry Transport) is a machine-to-machine or "Internet of Things" connectivity protocol on top of TCP/IP. It allows extremely lightweight publish/subscribe messaging transport.

{% include integrations/config_flow.md %}

Your first step to get MQTT and Home Assistant working is to choose a broker.

## Choose a MQTT broker

### Run your own

The most private option is running your own MQTT broker.

The recommended setup method is to use the [Mosquitto MQTT broker add-on](https://github.com/home-assistant/hassio-addons/blob/master/mosquitto/DOCS.md).

<div class='note warning'>

Neither ActiveMQ MQTT broker nor the RabbitMQ MQTT Plugin are supported, use a known working broker like Mosquitto instead.
There are [at least two](https://issues.apache.org/jira/browse/AMQ-6360) [issues](https://issues.apache.org/jira/browse/AMQ-6575) with the ActiveMQ MQTT broker which break MQTT message retention.

</div>

### Use a public broker

The Mosquitto project runs a [public broker](https://test.mosquitto.org). This is the easiest to set up, but there is no privacy as all messages are public. Use this only for testing purposes and not for real tracking of your devices or controlling your home. To use the public mosquitto broker, configure the MQTT integration to connect to broker `test.mosquitto.org` on port 1883 or 8883.

## Broker configuration

MQTT broker settings are configured when the MQTT integration is first set up and can be changed later if needed.

Add the MQTT integration, then provide your broker's hostname (or IP address) and port and (if required) the username and password that Home Assistant should use. To change the settings later, follow these steps:

1. Go to **{% my integrations title="Settings > Devices & Services" %}**.
2. Select the MQTT integration.
3. Select **Configure**, then **Re-configure MQTT**.

<div class='note'>

If you experience an error message like `Failed to connect due to exception: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed`, then turn on `Advanced options` and set [Broker certificate validation](/integrations/mqtt/#broker-certificate-validation) to `Auto`.

</div>

### Advanced broker configuration

Advanced broker configuration options include setting a custom client ID, setting a client certificate and key for authentication and enabling TLS validation of the brokers certificate for. To access the advanced settings, open the MQTT broker settings, switch on `Advanced options` and click `Next`. The advanced options will be shown by default if there are advanced settings active already.

<div class='note info'>

Advanced broker options are accessible only when advanced mode is enabled (see user settings), or when advanced broker settings are configured already.

</div>

#### Alternative client ID

You can set a custom MQTT client ID, this can help when debugging. Mind that the client ID must be unique. Leave this settings default if you want Home Assistant to generate a unique ID.

#### Keep alive

The time in seconds between sending keep alive messages for this client. The default is 60 seconds. The keep alive setting should be minimal 15 seconds.

#### Broker certificate validation

To enable a secure the broker certificate should be validated. If your broker uses a trusted certificate then choose `Auto`. This will allow validation against certifite CAs bundled certificates. If a self-signed certificate is used, select `Custom`. A custom PEM encoded CA-certificate can be uploaded. Click `NEXT` to show the control to upload the CA certificate.
If the server certificate does not match the hostname then validation will fail. To allow a connection without the verification of the hostname, turn the `Ignore broker certificate validation` switch on.

#### MQTT Protocol

The MQTT protocol setting defaults to version `3.1.1`. If your MQTT broker supports MQTT version 5 you can set the protocol setting to `5`.

#### Securing the the connection

With a secure broker connection it is possible to use a client certificate for authentication. To set the client certificate and private key turn on the option `Use a client certificate` and click "Next" to show the controls to upload the files. Only a PEM encoded client certificates together with a PEM encoded private key can be uploaded. Make sure the private key has no password set.

#### Using WebSockets as transport

You can select `websockets` as transport method if your MQTT broker supports it. When you select `websockets` and click `NEXT`, you will be able to add a WebSockets path (default = `/`) and WebSockets headers (optional). The target WebSockets URI: `ws://{broker}:{port}{WebSockets path}` is built with `broker`, `port` and `ws_path` (WebSocket path) settings.
To configure the WebSocketS headers supply a valid JSON dictionary string. E.g. `{ "Authorization": "token" , "x-header": "some header"}`. The default transport method is `tcp`. The WebSockets transport can be secured using TLS and optionally using user credentials or a client certificate.

<div class='note'>

A configured client certificate will only be active if broker certificate validation is enabled.

</div>

## Configure MQTT options

To change the settings, follow these steps:

1. Go to **{% my integrations title="Settings > Devices & Services" %}**.
2. Select the MQTT integration.
3. Select **Configure**, then **Re-configure MQTT**.
4. To open the MQTT options page, select **Next**.

### Discovery options

MQTT discovery is enabled by default. Discovery can be turned off. The prefix for the discovery topic (default `homeassistant`) can be changed here as well.
See also [MQTT Discovery section](#mqtt-discovery)

### Birth and last will messages

Home Assistant's MQTT integration supports so-called Birth and Last Will and Testament (LWT) messages. The former is used to send a message after the service has started, and the latter is used to notify other clients about a disconnected client. Please note that the LWT message will be sent both in case of a clean (e.g. Home Assistant shutting down) and in case of an unclean (e.g. Home Assistant crashing or losing its network connection) disconnect.

By default, Home Assistant sends `online` and `offline` to `homeassistant/status`.

MQTT Birth and Last Will messages can be customized or disabled from the UI. To do this, click on "Configure" in the integration page in the UI, then "Re-configure MQTT" and then "Next".

## Testing your setup

The `mosquitto` broker package ships commandline tools (often as `*-clients` package) to send and receive MQTT messages. For sending test messages to a broker running on `localhost` check the example below:

```bash
mosquitto_pub -h 127.0.0.1 -t homeassistant/switch/1/on -m "Switch is ON"
```

Another way to send MQTT messages manually is to use the **MQTT** integration in the frontend. Choose "Settings" on the left menu, click "Devices & Services", and choose "Configure" in the "Mosquitto broker" tile. Enter something similar to the example below into the "topic" field under "Publish a packet" and press "PUBLISH" .

1. Go to **{% my integrations title="Settings > Devices & Services" %}**.
2. Select the Mosquitto broker integration, then select **Configure**.
3. Enter something similar to the example below into the **topic** field under **Publish a packet**. Select **Publish**.

```bash
   homeassistant/switch/1/power
```

and in the Payload field

```bash
   ON
```

In the "Listen to a topic" field, type `#` to see everything, or "homeassistant/switch/#" to just follow a published topic, then press "START LISTENING". The messages should appear similar to the text below:

```bash
Message 23 received on homeassistant/switch/1/power/stat/POWER at 12:16 PM:
ON
QoS: 0 - Retain: false
Message 22 received on homeassistant/switch/1/power/stat/RESULT at 12:16 PM:
{
    "POWER": "ON"
}
QoS: 0 - Retain: false
```

For reading all messages sent on the topic `homeassistant` to a broker running on localhost:

```bash
mosquitto_sub -h 127.0.0.1 -v -t "homeassistant/#"
```

## Sharing of device configuration

MQTT entities can share device configuration, meaning one entity can include the full device configuration and other entities can link to that device by only setting mandatory fields.
The mandatory fields were previously limited to at least one of `connection` and `identifiers`, but have now been extended to at least one of `connection` and `identifiers` as well as the `name`.

## Naming of MQTT Entities

For every configured MQTT entity Home Assistant automatically assigns a unique `entity_id`. If the `unique_id` option is configured, you can change the `entity_id` after creation, and the changes are stored in the Entity Registry. The `entity_id` is generated when an item is loaded the first time.

If the `object_id` option is set, then this will be used to generate the `entity_id`.
If, for example, we have configured a `sensor`, and we have set `object_id` to `test`, then Home Assistant will try to assign `sensor.test` as `entity_id`, but if this `entity_id` already exits it will append it with a suffix to make it unique, for example, `sensor.test_2`.

This means any MQTT entity which is part of a device will [automatically have it's `friendly_name` attribute prefixed with the device name](https://developers.home-assistant.io/docs/core/entity/#has_entity_name-true-mandatory-for-new-integrations)

Unnamed `binary_sensor`, `button`, `number` and `sensor` entities will now be named by their device class instead of being named "MQTT binary sensor" etc.
It's allowed to set an MQTT entity's name to `None` (use `null` in YAML) to mark it as the main feature of a device.

Note that on each MQTT entity, the `has_entity_name` attribute will be set to `True`. More details [can be found here](https://developers.home-assistant.io/docs/core/entity/#has_entity_name-true-mandatory-for-new-integrations).

## MQTT Discovery

The discovery of MQTT devices will enable one to use MQTT devices with only minimal configuration effort on the side of Home Assistant. The configuration is done on the device itself and the topic used by the device. Similar to the [HTTP binary sensor](/integrations/http/#binary-sensor) and the [HTTP sensor](/integrations/http/#sensor). To prevent multiple identical entries if a device reconnects, a unique identifier is necessary. Two parts are required on the device side: The configuration topic which contains the necessary device type and unique identifier, and the remaining device configuration without the device type.

{% details "Entity integrations supported by MQTT discovery" %}

- [Alarm control panel](/integrations/alarm_control_panel.mqtt/)
- [Binary sensor](/integrations/binary_sensor.mqtt/)
- [Button](/integrations/button.mqtt/)
- [Camera](/integrations/camera.mqtt/)
- [Cover](/integrations/cover.mqtt/)
- [Device Tracker](/integrations/device_tracker.mqtt/)
- [Device Trigger](/integrations/device_trigger.mqtt/)
- [Event](/integrations/event.mqtt/)
- [Fan](/integrations/fan.mqtt/)
- [Humidifier](/integrations/humidifier.mqtt/)
- [Image](/integrations/image.mqtt/)
- [Climate/HVAC](/integrations/climate.mqtt/)
- [Lawn Mower](/integrations/lawn_mower.mqtt/)
- [Light](/integrations/light.mqtt/)
- [Lock](/integrations/lock.mqtt/)
- [Number](/integrations/number.mqtt/)
- [Scene](/integrations/scene.mqtt/)
- [Select](/integrations/select.mqtt/)
- [Sensor](/integrations/sensor.mqtt/)
- [Siren](/integrations/siren.mqtt/)
- [Switch](/integrations/switch.mqtt/)
- [Update](/integrations/update.mqtt/)
- [Tag Scanner](/integrations/tag.mqtt/)
- [Text](/integrations/text.mqtt/)
- [Vacuum](/integrations/vacuum.mqtt/)
- [Water Heater](/integrations/water_heater.mqtt/)

{% enddetails %}

MQTT discovery is enabled by default, but can be disabled. The prefix for the discovery topic (default `homeassistant`) can be changed.
See the [MQTT Options sections](#configure-mqtt-options)

### Discovery messages

#### Discovery topic

The discovery topic needs to follow a specific format:

```text
<discovery_prefix>/<component>/[<node_id>/]<object_id>/config
```

- `<discovery_prefix>`: The Discovery Prefix defaults to `homeassistant`. This prefix can be [changed](#discovery-options).
- `<component>`: One of the supported MQTT integrations, eg. `binary_sensor`.
- `<node_id>` (*Optional*):  ID of the node providing the topic, this is not used by Home Assistant but may be used to structure the MQTT topic. The ID of the node must only consist of characters from the character class `[a-zA-Z0-9_-]` (alphanumerics, underscore and hyphen).
- `<object_id>`: The ID of the device. This is only to allow for separate topics for each device and is not used for the `entity_id`. The ID of the device must only consist of characters from the character class `[a-zA-Z0-9_-]` (alphanumerics, underscore and hyphen).

The `<node_id>` level can be used by clients to only subscribe to their own (command) topics by using one wildcard topic like `<discovery_prefix>/+/<node_id>/+/set`.

Best practice for entities with a `unique_id` is to set `<object_id>` to `unique_id` and omit the `<node_id>`.

#### Discovery payload

The payload must be a serialized JSON dictionary and will be checked like an entry in your `configuration.yaml` file if a new device is added, with the exception that unknown configuration keys are allowed but ignored. This means that missing variables will be filled with the integration's default values. All configuration variables which are *required* must be present in the payload. The reason for allowing unknown documentation keys is allow some backwards compatibility, software generating MQTT discovery messages can then be used with older Home Assistant versions which will simply ignore new features.

Subsequent messages on a topic where a valid payload has been received will be handled as a configuration update, and a configuration update with an empty payload will cause a previously discovered device to be deleted.

A base topic `~` may be defined in the payload to conserve memory when the same topic base is used multiple times.
In the value of configuration variables ending with `_topic`, `~` will be replaced with the base topic, if the `~` occurs at the beginning or end of the value.

Configuration variable names in the discovery payload may be abbreviated to conserve memory when sending a discovery message from memory constrained devices.

It is encouraged to add additional information about the origin that supplies MQTT entities via MQTT discovery by adding the `origin` option (can abbreviated to `o`) to the discovery payload. Note that these options also support abbreviations. Information of the origin will be logged to the core event log when an item is discovered or updated.

{% configuration_basic %}
name:
  description: The name of the application that is the origin the discovered MQTT item. This option is required.
sw_version:
  description: Software version of the application that supplies the discovered MQTT item.
support_url:
  description: Support URL of the application that supplies the discovered MQTT item.
{% endconfiguration_basic %}


{% details "Supported abbreviations" %}

```txt
    'act_t':               'action_topic',
    'act_tpl':             'action_template',
    'atype':               'automation_type',
    'aux_cmd_t':           'aux_command_topic',
    'aux_stat_tpl':        'aux_state_template',
    'aux_stat_t':          'aux_state_topic',
    'av_tones':            'available_tones',
    'avty':                'availability',
    'avty_mode':           'availability_mode',
    'avty_t':              'availability_topic',
    'avty_tpl':            'availability_template',
    'away_mode_cmd_t':     'away_mode_command_topic',
    'away_mode_stat_tpl':  'away_mode_state_template',
    'away_mode_stat_t':    'away_mode_state_topic',
    'b_tpl':               'blue_template',
    'bri_cmd_t':           'brightness_command_topic',
    'bri_cmd_tpl':         'brightness_command_template',
    'bri_scl':             'brightness_scale',
    'bri_stat_t':          'brightness_state_topic',
    'bri_tpl':             'brightness_template',
    'bri_val_tpl':         'brightness_value_template',
    'clr_temp_cmd_tpl':    'color_temp_command_template',
    'bat_lev_t':           'battery_level_topic',
    'bat_lev_tpl':         'battery_level_template',
    'chrg_t':              'charging_topic',
    'chrg_tpl':            'charging_template',
    'clr_temp_cmd_t':      'color_temp_command_topic',
    'clr_temp_stat_t':     'color_temp_state_topic',
    'clr_temp_tpl':        'color_temp_template',
    'clr_temp_val_tpl':    'color_temp_value_template',
    'clrm':                'color_mode',
    'clrm_stat_t':         'color_mode_state_topic',
    'clrm_val_tpl':        'color_mode_value_template',
    'cln_t':               'cleaning_topic',
    'cln_tpl':             'cleaning_template',
    'cmd_off_tpl':         'command_off_template',
    'cmd_on_tpl':          'command_on_template',
    'cmd_t':               'command_topic',
    'cmd_tpl':             'command_template',
    'cod_arm_req':         'code_arm_required',
    'cod_dis_req':         'code_disarm_required',
    'cod_trig_req':        'code_trigger_required',
    'cont_type':           'content_type',
    'curr_temp_t':         'current_temperature_topic',
    'curr_temp_tpl':       'current_temperature_template',
    'dev':                 'device',
    'dev_cla':             'device_class',
    'dir_cmd_t':           'direction_command_topic',
    'dir_cmd_tpl':         'direction_command_template',
    'dir_stat_t':          'direction_state_topic',
    'dir_val_tpl':         'direction_value_template',
    'dock_t':              'docked_topic',
    'dock_tpl':            'docked_template',
    'e':                   'encoding',
    'en':                  'enabled_by_default',
    'ent_cat':             'entity_category',
    'ent_pic':             'entity_picture',
    'err_t':               'error_topic',
    'err_tpl':             'error_template',
    'evt_typ':             'event_types',
    'fanspd_t':            'fan_speed_topic',
    'fanspd_tpl':          'fan_speed_template',
    'fanspd_lst':          'fan_speed_list',
    'flsh_tlng':           'flash_time_long',
    'flsh_tsht':           'flash_time_short',
    'fx_cmd_t':            'effect_command_topic',
    'fx_cmd_tpl':          'effect_command_template',
    'fx_list':             'effect_list',
    'fx_stat_t':           'effect_state_topic',
    'fx_tpl':              'effect_template',
    'fx_val_tpl':          'effect_value_template',
    'exp_aft':             'expire_after',
    'fan_mode_cmd_tpl':    'fan_mode_command_template',
    'fan_mode_cmd_t':      'fan_mode_command_topic',
    'fan_mode_stat_tpl':   'fan_mode_state_template',
    'fan_mode_stat_t':     'fan_mode_state_topic',
    'frc_upd':             'force_update',
    'g_tpl':               'green_template',
    'hs_cmd_t':            'hs_command_topic',
    'hs_cmd_tpl':          'hs_command_template',
    'hs_stat_t':           'hs_state_topic',
    'hs_val_tpl':          'hs_value_template',
    'ic':                  'icon',
    'img_e':               'image_encoding',
    'img_t':               'image_topic',
    'init':                'initial',
    'hum_cmd_t':           'target_humidity_command_topic',
    'hum_cmd_tpl':         'target_humidity_command_template',
    'hum_stat_t':          'target_humidity_state_topic',
    'hum_state_tpl':       'target_humidity_state_template',
    'json_attr':           'json_attributes',
    'json_attr_t':         'json_attributes_topic',
    'json_attr_tpl':       'json_attributes_template',
    'l_ver_t':             'latest_version_topic',
    'l_ver_tpl':           'latest_version_template',
    'lrst_t':              'last_reset_topic',
    'lrst_val_tpl':        'last_reset_value_template',
    'max':                 'max',
    'min':                 'min',
    'max_mirs':            'max_mireds',
    'min_mirs':            'min_mireds',
    'max_temp':            'max_temp',
    'min_temp':            'min_temp',
    'max_hum':             'max_humidity',
    'min_hum':             'min_humidity',
    'mode':                'mode',
    'mode_cmd_tpl':        'mode_command_template',
    'mode_cmd_t':          'mode_command_topic',
    'mode_stat_tpl':       'mode_state_template',
    'mode_stat_t':         'mode_state_topic',
    'modes':               'modes',
    'name':                'name',
    'o':                   'origin',
    'obj_id':              'object_id',
    'off_dly':             'off_delay',
    'on_cmd_type':         'on_command_type',
    'ops':                 'options',
    'opt':                 'optimistic',
    'osc_cmd_t':           'oscillation_command_topic',
    'osc_cmd_tpl':         'oscillation_command_template',
    'osc_stat_t':          'oscillation_state_topic',
    'osc_val_tpl':         'oscillation_value_template',
    'pct_cmd_t':           'percentage_command_topic',
    'pct_cmd_tpl':         'percentage_command_template',
    'pct_stat_t':          'percentage_state_topic',
    'pct_val_tpl':         'percentage_value_template',
    'ptrn':                'pattern',
    'pl':                  'payload',
    'pl_arm_away':         'payload_arm_away',
    'pl_arm_home':         'payload_arm_home',
    'pl_arm_custom_b':     'payload_arm_custom_bypass',
    'pl_arm_nite':         'payload_arm_night',
    'pl_arm_vacation':     'payload_arm_vacation',
    'pl_prs':              'payload_press',
    'pl_rst':              'payload_reset',
    'pl_avail':            'payload_available',
    'pl_cln_sp':           'payload_clean_spot',
    'pl_cls':              'payload_close',
    'pl_disarm':           'payload_disarm',
    'pl_dir_fwd':          'payload_direction_forward',
    'pl_dir_rev':          'payload_direction_reverse',
    'pl_home':             'payload_home',
    'pl_inst':             'payload_install',
    'pl_lock':             'payload_lock',
    'pl_loc':              'payload_locate',
    'pl_not_avail':        'payload_not_available',
    'pl_not_home':         'payload_not_home',
    'pl_off':              'payload_off',
    'pl_on':               'payload_on',
    'pl_open':             'payload_open',
    'pl_osc_off':          'payload_oscillation_off',
    'pl_osc_on':           'payload_oscillation_on',
    'pl_paus':             'payload_pause',
    'pl_stop':             'payload_stop',
    'pl_strt':             'payload_start',
    'pl_stpa':             'payload_start_pause',
    'pl_ret':              'payload_return_to_base',
    'pl_rst_hum':          'payload_reset_humidity',
    'pl_rst_mode':         'payload_reset_mode',
    'pl_rst_pct':          'payload_reset_percentage',
    'pl_rst_pr_mode':      'payload_reset_preset_mode',
    'pl_toff':             'payload_turn_off',
    'pl_ton':              'payload_turn_on',
    'pl_trig':             'payload_trigger',
    'pl_unlk':             'payload_unlock',
    'pos_clsd':            'position_closed',
    'pos_open':            'position_open',
    'pr_mode_cmd_t':       'preset_mode_command_topic',
    'pr_mode_cmd_tpl':     'preset_mode_command_template',
    'pr_mode_stat_t':      'preset_mode_state_topic',
    'pr_mode_val_tpl':     'preset_mode_value_template',
    'pr_modes':            'preset_modes',
    'r_tpl':               'red_template',
    'rel_s':               'release_summary',
    'rel_u':               'release_url',
    'ret':                 'retain',
    'rgb_cmd_t':           'rgb_command_topic',
    'rgb_cmd_tpl':         'rgb_command_template',
    'rgb_stat_t':          'rgb_state_topic',
    'rgb_val_tpl':         'rgb_value_template',
    'rgbw_cmd_t':          'rgbw_command_topic',
    'rgbw_cmd_tpl':        'rgbw_command_template',
    'rgbw_stat_t':         'rgbw_state_topic',
    'rgbw_val_tpl':        'rgbw_value_template',
    'rgbww_cmd_t':         'rgbww_command_topic',
    'rgbww_cmd_tpl':       'rgbww_command_template',
    'rgbww_stat_t':        'rgbww_state_topic',
    'rgbww_val_tpl':       'rgbww_value_template',
    'send_cmd_t':          'send_command_topic',
    'send_if_off':         'send_if_off',
    'set_fan_spd_t':       'set_fan_speed_topic',
    'set_pos_tpl':         'set_position_template',
    'set_pos_t':           'set_position_topic',
    'pos_t':               'position_topic',
    'pos_tpl':             'position_template',
    'spd_rng_min':         'speed_range_min',
    'spd_rng_max':         'speed_range_max',
    'src_type':            'source_type',
    'stat_cla':            'state_class',
    'stat_clsd':           'state_closed',
    'stat_closing':        'state_closing',
    'stat_jam':            'state_jammed',
    'stat_off':            'state_off',
    'stat_on':             'state_on',
    'stat_open':           'state_open',
    'stat_opening':        'state_opening',
    'stat_stopped':        'state_stopped',
    'stat_locked':         'state_locked',
    'stat_locking':         'state_locking',
    'stat_unlocked':       'state_unlocked',
    'stat_unlocking':       'state_unlocking',
    'stat_t':              'state_topic',
    'stat_tpl':            'state_template',
    'stat_val_tpl':        'state_value_template',
    'step':                'step',
    'stype':               'subtype',
    'sug_dsp_prc':         'suggested_display_precision',
    'sup_clrm':            'supported_color_modes',
    'sup_dur':             'support_duration',
    'sup_vol':             'support_volume_set',
    'sup_feat':            'supported_features',
    'swing_mode_cmd_tpl':  'swing_mode_command_template',
    'swing_mode_cmd_t':    'swing_mode_command_topic',
    'swing_mode_stat_tpl': 'swing_mode_state_template',
    'swing_mode_stat_t':   'swing_mode_state_topic',
    'temp_cmd_tpl':        'temperature_command_template',
    'temp_cmd_t':          'temperature_command_topic',
    'temp_hi_cmd_tpl':     'temperature_high_command_template',
    'temp_hi_cmd_t':       'temperature_high_command_topic',
    'temp_hi_stat_tpl':    'temperature_high_state_template',
    'temp_hi_stat_t':      'temperature_high_state_topic',
    'temp_lo_cmd_tpl':     'temperature_low_command_template',
    'temp_lo_cmd_t':       'temperature_low_command_topic',
    'temp_lo_stat_tpl':    'temperature_low_state_template',
    'temp_lo_stat_t':      'temperature_low_state_topic',
    'temp_stat_tpl':       'temperature_state_template',
    'temp_stat_t':         'temperature_state_topic',
    'temp_unit':           'temperature_unit',
    'tilt_clsd_val':       'tilt_closed_value',
    'tilt_cmd_t':          'tilt_command_topic',
    'tilt_cmd_tpl':        'tilt_command_template',
    'tilt_max':            'tilt_max',
    'tilt_min':            'tilt_min',
    'tilt_opnd_val':       'tilt_opened_value',
    'tilt_opt':            'tilt_optimistic',
    'tilt_status_t':       'tilt_status_topic',
    'tilt_status_tpl':     'tilt_status_template',
    'tit':                 'title',
    't':                   'topic',
    'uniq_id':             'unique_id',
    'unit_of_meas':        'unit_of_measurement',
    'url_t':               'url_topic',
    'url_tpl':             'url_template',
    'val_tpl':             'value_template',
    'whit_cmd_t':          'white_command_topic',
    'whit_scl':            'white_scale',
    'xy_cmd_t':            'xy_command_topic',
    'xy_cmd_tpl':          'xy_command_template',
    'xy_stat_t':           'xy_state_topic',
    'xy_val_tpl':          'xy_value_template',
```

{% enddetails %}
{% details "Supported abbreviations for device registry configuration" %}

```txt
    'cu':                  'configuration_url',
    'cns':                 'connections',
    'ids':                 'identifiers',
    'name':                'name',
    'mf':                  'manufacturer',
    'mdl':                 'model',
    'hw':                  'hw_version',
    'sw':                  'sw_version',
    'sa':                  'suggested_area',
```
{% enddetails %}
{% details "Supported abbreviations for origin info" %}

```txt
    'name':                'name',
    'sw':                  'sw_version',
    'url':                 'support_url',
```
{% enddetails %}

### Support by third-party tools

The following software has built-in support for MQTT discovery:

- [ArduinoHA](https://github.com/dawidchyrzynski/arduino-home-assistant)
- [Arilux AL-LC0X LED controllers](https://github.com/smrtnt/Arilux_AL-LC0X)
- [ebusd](https://github.com/john30/ebusd)
- [ecowitt2mqtt](https://github.com/bachya/ecowitt2mqtt)
- [EMS-ESP32 (and EMS-ESP)](https://github.com/emsesp/EMS-ESP32)
- [ESPHome](https://esphome.io)
- [ESPurna](https://github.com/xoseperez/espurna)
- [HASS.Agent](https://github.com/LAB02-Research/HASS.Agent)
- [IOTLink](https://iotlink.gitlab.io) (starting with 2.0.0)
- [MiFlora MQTT Daemon](https://github.com/ThomDietrich/miflora-mqtt-daemon)
- [MyElectricalData](https://github.com/MyElectricalData/myelectricaldata#english)
- [Nuki Hub](https://github.com/technyon/nuki_hub)
- [Nuki Smart Lock 3.0 Pro](https://support.nuki.io/hc/articles/12947926779409-MQTT-support), [more info](https://developer.nuki.io/t/mqtt-api-specification-v1-3/17626)
- [OpenMQTTGateway](https://github.com/1technophile/OpenMQTTGateway)
- [room-assistant](https://github.com/mKeRix/room-assistant) (starting with 1.1.0)
- [SmartHome](https://github.com/roncoa/SmartHome)
- [SpeedTest-CLI MQTT](https://github.com/adorobis/speedtest-CLI2mqtt)
- [SwitchBot-MQTT-BLE-ESP32](https://github.com/devWaves/SwitchBot-MQTT-BLE-ESP32)
- [Tasmota](https://github.com/arendst/Tasmota) (starting with 5.11.1e, development halted)
- [TeddyCloud](https://github.com/toniebox-reverse-engineering/teddycloud)
- [Teleinfo MQTT](https://fmartinou.github.io/teleinfo2mqtt) (starting with 3.0.0)
- [Tydom2MQTT](https://fmartinou.github.io/tydom2mqtt/)
- [What's up Docker?](https://fmartinou.github.io/whats-up-docker/) (starting with 3.5.0)
- [WyzeSense2MQTT](https://github.com/raetha/wyzesense2mqtt)
- [Xiaomi DaFang Hacks](https://github.com/EliasKotlyar/Xiaomi-Dafang-Hacks)
- [Zehnder Comfoair RS232 MQTT](https://github.com/adorobis/hacomfoairmqtt)
- [Zigbee2MQTT](https://github.com/koenkk/zigbee2mqtt)
- [Zwave2Mqtt](https://github.com/OpenZWave/Zwave2Mqtt) (starting with 2.0.1)

### Discovery examples

#### Motion detection (binary sensor)

A motion detection device which can be represented by a [binary sensor](/integrations/binary_sensor.mqtt/) for your garden would send its configuration as JSON payload to the Configuration topic. After the first message to `config`, then the MQTT messages sent to the state topic will update the state in Home Assistant.

- Configuration topic: `homeassistant/binary_sensor/garden/config`
- State topic: `homeassistant/binary_sensor/garden/state`
- Configuration payload with derived device name:  `{"name": null, "device_class": "motion", "state_topic": "homeassistant/binary_sensor/garden/state", "unique_id": "motion01ad", "device": {"identifiers": ["01ad"], "name": "Garden" }}`
- Retain: The -r switch is added to retain the configuration topic in the broker. Without this, the sensor will not be available after Home Assistant restarts.

It is also a good idea to add a `unique_id` to allow changes to the entity and a `device` mapping so we can group all sensors of a device together. We can set "name" to `null` if we want to inherit the device name for the entity. If we set an entity name, the `friendly_name` will be a combination of the device and entity name. If `name` is left away and a `device_class` is set, the entity name part will be derived from the `device_class`.

- Example configuration payload with no name set and derived `device_class` name: `{"device_class": "motion", "state_topic": "homeassistant/binary_sensor/garden/state", "unique_id": "motion01ad", "device": {"identifiers": ["01ad"], "name": "Garden" }}`

If no name is set, a default name will be set by MQTT ([see the MQTT platform documentation](#mqtt-discovery)).

To create a new sensor manually and with the name set to `null` to derive the device name "Garden":

```bash
mosquitto_pub -r -h 127.0.0.1 -p 1883 -t "homeassistant/binary_sensor/garden/config" -m '{"name": null, "device_class": "motion", "state_topic": "homeassistant/binary_sensor/garden/state", "unique_id": "motion01ad", "device": {"identifiers": ["01ad"], "name": "Garden" }}'
```

Update the state:

```bash
mosquitto_pub -h 127.0.0.1 -p 1883 -t "homeassistant/binary_sensor/garden/state" -m ON
```

Delete the sensor by sending an empty message.

 ```bash
mosquitto_pub -h 127.0.0.1 -p 1883 -t "homeassistant/binary_sensor/garden/config" -m ''
```

For more details please refer to the [MQTT testing section](/integrations/mqtt/#testing-your-setup).

#### Sensors

Setting up a sensor with multiple measurement values requires multiple consecutive configuration topic submissions.

- Configuration topic no1: `homeassistant/sensor/sensorBedroomT/config`
- Configuration payload no1: `{"device_class": "temperature", "state_topic": "homeassistant/sensor/sensorBedroom/state", "unit_of_measurement": "°C", "value_template": "{% raw %}{{ value_json.temperature}}{% endraw %}","unique_id": "temp01ae",  "device": {"identifiers": ["bedroom01ae"], "name": "Bedroom" }}`
- Configuration topic no2: `homeassistant/sensor/sensorBedroomH/config`
- Configuration payload no2: `{"device_class": "humidity", "state_topic": "homeassistant/sensor/sensorBedroom/state", "unit_of_measurement": "%", "value_template": "{% raw %}{{ value_json.humidity}}{% endraw %}","unique_id": "hum01ae", "device": {"identifiers": ["bedroom01ae"], "name": "Bedroom" } }`
- Common state payload: `{ "temperature": 23.20, "humidity": 43.70 }`

#### Entities with command topics

Setting up a light, switch etc. is similar but requires a `command_topic` as mentioned in the [MQTT switch documentation](/integrations/switch.mqtt/).

- Configuration topic: `homeassistant/switch/irrigation/config`
- State topic: `homeassistant/switch/irrigation/state`
- Command topic: `homeassistant/switch/irrigation/set`
- Payload:  `{"name": "Irrigation", "command_topic": "homeassistant/switch/irrigation/set", "state_topic": "homeassistant/switch/irrigation/state", "unique_id": "irr01ad", "device": {"identifiers": ["garden01ad"], "name": "Garden" }}`
- Retain: The -r switch is added to retain the configuration topic in the broker. Without this, the sensor will not be available after Home Assistant restarts.

```bash
mosquitto_pub -r -h 127.0.0.1 -p 1883 -t "homeassistant/switch/irrigation/config" \
  -m '{"name": "Irrigation", "command_topic": "homeassistant/switch/irrigation/set", "state_topic": "homeassistant/switch/irrigation/state", "unique_id": "irr01ad", "device": {"identifiers": ["garden01ad"], "name": "Garden" }}}'
```

Set the state:

```bash
mosquitto_pub -h 127.0.0.1 -p 1883 -t "homeassistant/switch/irrigation/set" -m ON
```

#### Using abbreviations and base topic

Setting up a switch using topic prefix and abbreviated configuration variable names to reduce payload length.

- Configuration topic: `homeassistant/switch/irrigation/config`
- Command topic: `homeassistant/switch/irrigation/set`
- State topic: `homeassistant/switch/irrigation/state`
- Configuration payload: `{"~": "homeassistant/switch/irrigation", "name": "garden", "cmd_t": "~/set", "stat_t": "~/state"}`

#### Another example using abbreviations topic name and base topic

Setting up a [light that takes JSON payloads](/integrations/light.mqtt/#json-schema), with abbreviated configuration variable names:

- Configuration topic: `homeassistant/light/kitchen/config`
- Command topic: `homeassistant/light/kitchen/set`
- State topic: `homeassistant/light/kitchen/state`
- Example state payload: `{"state": "ON", "brightness": 255}`
- Configuration payload:

  ```json
  {
    "~": "homeassistant/light/kitchen",
    "name": "Kitchen",
    "uniq_id": "kitchen_light",
    "cmd_t": "~/set",
    "stat_t": "~/state",
    "schema": "json",
    "brightness": true
  }
  ```

#### Example with using abbreviated Device and Origin info in discovery messages

  ```json
  {
    "~": "homeassistant/light/kitchen",
    "name": null,
    "uniq_id": "kitchen_light",
    "cmd_t": "~/set",
    "stat_t": "~/state",
    "schema": "json",
    "dev": {
      "ids": "ea334450945afc",
      "name": "Kitchen",
      "mf": "Bla electronics",
      "mdl": "xya",
      "sw": "1.0",
      "hw": "1.0rev2",
    },
    "o": {
      "name":"bla2mqtt",
      "sw": "2.1",
      "url": "https://bla2mqtt.example.com/support",
    }
  }
  ```

#### Use object_id to influence the entity id

The entity id is automatically generated from the entity's name. All MQTT integrations optionally support providing an `object_id` which will be used instead if provided.

- Configuration topic: `homeassistant/sensor/device1/config`
- Example configuration payload:

```json
{
  "name":"My Super Device",
  "object_id":"my_super_device",
  "state_topic": "homeassistant/sensor/device1/state"
 }
```

In the example above, the entity_id will be `sensor.my_super_device` instead of `sensor.device1`.

## Manual configured MQTT items

For most integrations, it is also possible to manually set up MQTT items in `configuration.yaml`. Read more [about configuration in YAML](/docs/configuration/yaml).

MQTT supports two styles for configuring items in YAML. All configuration items are placed directly under the `mqtt` integration key. Note that you cannot mix these styles. Use the *YAML configuration listed per item* style when in doubt.

### YAML configuration listed per item

This method expects all items to be in a YAML list. Each item has a `{domain}` key and the item config is placed directly under the domain key. This method is considered as best practice. In all the examples we use this format.

```yaml
mqtt:
  - {domain}:
      name: ""
      ...
  - {domain}:
      name: ""
      ...
```

### YAML configuration keyed and bundled by `{domain}`

All items are grouped per `{domain}` and where all configs are listed.

```yaml
mqtt:
  {domain}:
    - name: ""
      ...
    - name: ""
      ...
```

{% details "MQTT components that support setup via YAML" %}

- [Alarm control panel](/integrations/alarm_control_panel.mqtt/)
- [Binary sensor](/integrations/binary_sensor.mqtt/)
- [Button](/integrations/button.mqtt/)
- [Camera](/integrations/camera.mqtt/)
- [Cover](/integrations/cover.mqtt/)
- [Device Tracker](/integrations/device_tracker.mqtt/)
- [Event](/integrations/event.mqtt/)
- [Fan](/integrations/fan.mqtt/)
- [Humidifier](/integrations/humidifier.mqtt/)
- [Image](/integrations/image.mqtt/)
- [Climate/HVACs](/integrations/climate.mqtt/)
- [Lawn Mower](/integrations/lawn_mower.mqtt/)
- [Light](/integrations/light.mqtt/)
- [Lock](/integrations/lock.mqtt/)
- [Number](/integrations/number.mqtt/)
- [Scene](/integrations/scene.mqtt/)
- [Select](/integrations/select.mqtt/)
- [Sensor](/integrations/sensor.mqtt/)
- [Siren](/integrations/siren.mqtt/)
- [Switch](/integrations/switch.mqtt/)
- [Text](/integrations/text.mqtt/)
- [Update](/integrations/update.mqtt/)
- [Vacuum](/integrations/vacuum.mqtt/)
- [Water Heater](/integrations/water_heater.mqtt/)

{% enddetails %}

If you have a lot of manual configured items you might want to consider [splitting up the configuration](/docs/configuration/splitting_configuration/).

## Using Templates

The MQTT integration supports templating. Read more [about using templates with the MQTT integration](/docs/configuration/templating/#using-templates-with-the-mqtt-integration).

## MQTT Notifications

The MQTT notification support is different than for the other [notification](/integrations/notify/) integrations. It is a service. This means you need to provide more details when calling the service.

**Call Service** section from **Developer Tools** -> **Services** allows you to send MQTT messages. Choose *mqtt.publish*  from the list of **Available services:** and enter something like the sample below into the **Service Data** field and hit **CALL SERVICE**.

```json
{"payload": "Test message from HA", "topic": "home/notification", "qos": 0, "retain": 0}
```

<p class='img'>
  <img src='/images/screenshots/mqtt-notify.png' />
</p>

The same will work for automations.

<p class='img'>
  <img src='/images/screenshots/mqtt-notify-action.png' />
</p>


### Examples

#### REST API

Using the [REST API](https://developers.home-assistant.io/docs/api/rest/) to send a message to a given topic.

```bash
$ curl -X POST \
    -H "Authorization: Bearer ABCDEFGH" \
    -H "Content-Type: application/json" \
    -d '{"payload": "Test message from HA", "topic": "home/notification"}' \
    http://IP_ADDRESS:8123/api/services/mqtt/publish
```

#### Automations

Use as [`script`](/integrations/script/) in automations.

{% raw %}

```yaml
automation:
  alias: "Send me a message when I get home"
  trigger:
    platform: state
    entity_id: device_tracker.me
    to: "home"
  action:
    service: script.notify_mqtt
    data:
      target: "me"
      message: "I'm home"

script:
  notify_mqtt:
    sequence:
      - service: mqtt.publish
        data:
          payload: "{{ message }}"
          topic: home/"{{ target }}"
          retain: true
```

{% endraw %}

## Publish & Dump services

The MQTT integration will register the service `mqtt.publish` which allows publishing messages to MQTT topics. There are two ways of specifying your payload. You can either use `payload` to hard-code a payload or use `payload_template` to specify a [template](/docs/configuration/templating/#using-templates-with-the-mqtt-integration) that will be rendered to generate the payload.

### Service `mqtt.publish`

| Service data attribute | Optional | Description                                                  |
| ---------------------- | -------- | ------------------------------------------------------------ |
| `topic`                | no       | Topic to publish payload to.                                 |
| `topic_template`       | no       | Template to render as topic to publish payload to.           |
| `payload`              | yes      | Payload to publish.                                          |
| `payload_template`     | yes      | Template to render as payload value.                         |
| `qos`                  | yes      | Quality of Service to use. (default: 0)                      |
| `retain`               | yes      | If message should have the retain flag set. (default: false) |

<p class='note'>
You must include either `topic` or `topic_template`, but not both. If providing a payload, you need to include either `payload` or `payload_template`, but not both.
</p>

```yaml
topic: homeassistant/light/1/command
payload: on
```

{% raw %}

```yaml
topic: homeassistant/light/1/state
payload_template: "{{ states('device_tracker.paulus') }}"
```

{% endraw %}

{% raw %}

```yaml
topic_template: "homeassistant/light/{{ states('sensor.light_active') }}/state"
payload_template: "{{ states('device_tracker.paulus') }}"
```

{% endraw %}

`payload` must be a string.
If you want to send JSON using the YAML editor then you need to format/escape
it properly. Like:

```yaml
topic: homeassistant/light/1/state
payload: "{\"Status\":\"off\", \"Data\":\"something\"}"`
```

When using Home Assistant's YAML editor for formatting JSON
you should take special care if `payload` contains template content.
Home Assistant will force you in to the YAML editor and will treat your
definition as a template. Make sure you escape the template blocks as like
in the example below. Home Assistant will convert the result to a string
and will pass it to the MQTT publish service.

The example below shows how to publish a temperature sensor 'Bathroom Temperature'.
The `device_class` is set, so it is not needed to set the "name" option. The entity
will inherit the name from the `device_class` set and also support translations.
If you set "name" in the payload the entity name will start with the device name.

{% raw %}

```yaml
service: mqtt.publish
data:
  topic: homeassistant/sensor/Acurite-986-1R-51778/config
  payload: >-
    {"device_class": "temperature",
    "unit_of_measurement": "\u00b0C",
    "value_template": "{{ value|float }}",
    "state_topic": "rtl_433/rtl433/devices/Acurite-986/1R/51778/temperature_C",
    "unique_id": "Acurite-986-1R-51778-T",
    "device": {
    "identifiers": "Acurite-986-1R-51778",
    "name": "Bathroom",
    "model": "Acurite-986",
    "manufacturer": "rtl_433" }
    }
```

{% endraw %}

Example of how to use `qos` and `retain`:

```yaml
topic: homeassistant/light/1/command
payload: on
qos: 2
retain: true
```

### Service `mqtt.dump`

Listen to the specified topic matcher and dumps all received messages within a specific duration into the file `mqtt_dump.txt` in your configuration folder. This is useful when debugging a problem.

| Service data attribute | Optional | Description                                                                 |
| ---------------------- | -------- | --------------------------------------------------------------------------- |
| `topic`                | no       | Topic to dump. Can contain a wildcard (`#` or `+`).                         |
| `duration`             | yes      | Duration in seconds that we will listen for messages. Default is 5 seconds. |

```yaml
topic: openzwave/#
```

## Logging

The [logger](/integrations/logger/) integration allows the logging of received MQTT messages.

```yaml
# Example configuration.yaml entry
logger:
  default: warning
  logs:
    homeassistant.components.mqtt: debug
```

## Event `event_mqtt_reloaded`

Event `event_mqtt_reloaded` is fired when Manually configured MQTT entities have been reloaded and entities thus might have changed.

This event has no additional data.
