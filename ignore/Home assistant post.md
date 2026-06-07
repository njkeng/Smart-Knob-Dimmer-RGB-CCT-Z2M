
I have been installing lots of RGB+CCT lights at my place.  I wanted a way to control the lights without having to open the Home Assistant app. My requirements were
- Full control over brightness, color temperature, hue and saturation
- Control all of the lights in a room simultaneously
- No use of scenes as my lighting requirements vary from day to day
- Use Zigbee, which is my protocol of choice

I couldn't find a blueprint that does all of these things. I purchased two cheap smart knobs from AliExpress to experiment with.  It turns out that they are performing well and I have no plans to replace them.

So without further ado:

___

# Smart knob light dimmer blueprint

## Introduction

This blueprint turns your Z2M smart knob into a full-control dimmer for RGB CCT lights. 

- With the dimmer in *CCT* (white color temperature) mode, you can adjust brightness and color temperature.  

- With the dimmer in RGB mode, you can adjust hue, saturation and brightness.

To use this blueprint, you need the following devices:
- A smart knob communicating with Home Assistant using Z2M (Zigbee2MQTT). I have developed and tested this blueprint using one specific device. However, I am confident that any Z2M Smart Knob device can be used. I have read as many brand-specific blueprints as I can find and the output from the knobs is almost identical across all of them. (Maybe it's in a Zigbee spec somewhere?) If you happen to have a Smart Knob that is different, the blueprint has a simple procedure for adapting the blueprint settings for your device.

- One or more light devices configured in Home Assistant.  This blueprint is best used with lights that have both RGB and CCT modes.  If your light has RGB only, CCT only, brightness only or is simply an on/off device, this blueprint will work without errors.  The light will simply ignore any commands that it can't use.

    *Note:* If you are going to mix devices with different capabilities, put the one with the most functions first.  Otherwise the blueprint may do something like trying to read a hue value from a lamp that is CCT only. This will not produce errors, but it will give you confusing results. The blueprint uses the first device in the list as the reference source for all of the RGB+CCT values.  With a simpler device, these fields still exist but their values will not change. The effect on the blueprint will be that it is unable to alter that value for any of the lights in the list.

## Development devices

This blueprint was developed using generic brand Smart Knob TS004F. Zigbee2MQTT automatically recognized the Smart Knob as Moes ZG-101ZD. Link to the device:

https://www.aliexpress.com/item/1005008917041951.html

The blueprint has been tested on these lights:
- Arlec Grid Connect GLD322HA RGB+CCT on Tuya WiFi
- AliExpress Zigbee RGB+CCT light model TS0505B_1
- Three different generic brand AliExpress RGB+CCT Tuya wifi lights
- Ikea STOFTMOLN ceiling/wall lamp WW24. This lamp has brightness adjustment only. No color temp. No RGB.

## Operation

The actions performed by the dimmer are:
- Short press: Toggle light On/Off
- Long press (3 seconds): Change between CCT mode and RGB mode
- Double press: Change the parameter to adjust
    - In CCT mode, toggles between adjustment of brightness and color temperature parameters
    - In RGB mode, cycles between adjustment of hue, saturation and brightness parameters
- Turn left/right: Adjust the value of the active parameter. The 
adjustment knob works best when turned slowly.

When the dimmer changes between RGB and CCT modes, the blueprint stores the current parameter values prior to changing modes. The next time the mode changes, the previous values are restored.
- CCT to RGB: Stores current values for brightness and color temperature
- RGB to CCT: Stores current values for hue, saturation and brightness

- When changing from CCT mode to RGB mode, the blueprint selects the RGB hue parameter for adjustment, regardless of the previous parameter selected.
- When changing from RGB mode to CCT mode, the blueprint selects the CCT brightness parameter for adjustment, regardless of the previous parameter selected.


## Installation

### Step 1 - The blueprint
The simplest way to import the blueprint is to click this button:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fnjkeng%2FSmart-Knob-Dimmer-RGB-CCT-Z2M%2Fblob%2F5fabf4777775226bd4c90bb7873543e147bf60a1%2Fdimmer_RGB_CCT.yaml)


Alternatively, to import the Blueprint manually:

1. Copy this URL        
https://github.com/njkeng/Smart-Knob-Dimmer-RGB-CCT-Z2M/blob/5fabf4777775226bd4c90bb7873543e147bf60a1/dimmer_RGB_CCT.yaml
1.  Go to Home Assistant → Configuration → Automations & Scenes → Blueprints and click on "Import Blueprint" button in the bottom right.
1.  Paste the URL and click on "Preview Blueprint"
1.  Click on "Import Blueprint"

### Step 2 - The helpers

This blueprint relies on helpers to store the current state of the dimmer control and to retain RGB and CCT setting values.  These helpers need to be defined in your Home Assistant configuration section. 

We are going to install a helper definition file as a package in Home Assistant.  This can't be done automatically.  Here are the manual steps required to create the helper definition file.

1. Open your *configuration.yaml* file for editing. I strongly recommend using the editor *Studio Code Server*.  If you don't already have this installed, install it from the Home Assistant app store.  Go to Home Assistant → Settings → Apps and click on Install App.  Find *Studio Code Server* and follow the directions to install it.

1. Locate the root folder named *CONFIG* which contains your *configuration.yaml* file. Create a new folder in the root folder and name it *packages*. 
1. Create a new file in the *packages* folder.  Name the file *dimmer_helpers.yaml*.  

1. Open *dimmer_helpers.yaml* for editing. Copy the contents of the helpers file from Github. Open the file using this link
https://github.com/njkeng/Smart-Knob-Dimmer-RGB-CCT-Z2M/blob/5fabf4777775226bd4c90bb7873543e147bf60a1/dimmer_helpers.yaml


    Use the *Copy raw file* ⧉ function to copy the contents of the file. Paste the contents into your *dimmer_helpers.yaml*. The file is saved automatically.

1. The next step involves editing your *configuration.yaml* so that your installation can use package.  This file is critical to your whole Home Assistant installation.  It is always a good idea to make a backup of your *configuration.yaml* before you edit the active file. Right-click *configuration.yaml* and select Copy.  Press Ctrl-V to paste.  You will now have a backup named *configuration copy.yaml*.

1. Edit your *configuration.yaml* to allow use of package files.  Skip this step if you already have Home Assistant configured to use packages. Package files are dynamically loaded into *configuration.yaml* whenever home assistant reads *configuration.yaml*. Copy and paste the following lines into your *configuration.yaml*

```yaml
# Custom packages for my stuff
homeassistant:
    packages:
        dimmer_helpers: !include packages/dimmer_helpers.yaml
```
6. Reload all home assistant configuration.  Two steps are needed.
    1. Check for errors. Go to settings → developer tools →  YAML tab and click *check configuration*.
    1. If errors are found, fix them.  Issues with YAML files tend to be to do with indentation.
    1. If there are no errors, reload all YAML.  Go to settings → developer tools → YAML tab and click *all YAML configuration*.  Alternatively, restart *Home Assistant* if you wish.

## Configuration

For most installations, configuration is fairly basic.

### Essential configuration
1. The dimmer

    Your dimmer needs to be a *Zigbee* type and be configured in Home Assistant through *Zigbee2MQTT*. Enter the parent MQTT topic of the smart knob. Typically the last part will be the friendly name of your device. To get the MQTT topic for your device, go to *Zigbee2MQTT* and open the details page for your Smart Knob.  Copy the text for MQTT and paste it in here. It should look something like *zigbee2mqtt/My Smart Knob*.

2. Lights

    Select the light or lights that you wish to control with this dimmer. The type of light doesn't matter.  This blueprint fully controls RGB CCT lights, but it will control any light and make use of whatever functionality the light has. RGB alone is fine.  CCT alone is fine.  Simple ON / OFF is fine.  The light will simply ignore commands that it can't use.

    The dimmer can control multiple lights simultaneously.  This is intended for rooms with more than one light installed.

3. Helper

    Select a different helper number for each dimmer that is configured using this blueprint. Each dimmer needs a unique number to avoid unintentionally altering the parameters of some other dimmer. The *dimmer_helpers.yaml* file provided with this blueprint defines helpers for up to 5 dimmers.  If more than 5 dimmers are needed, the *dimmer_helpers.yaml* file needs to be edited to add more helpers.  Detailed instructions on how to do this are contained with the file.

### Optional configuration

The blueprint will typically operate satisfactorily using the default settings.  Check the settings in this section if your dimmer is not operating correctly or if you want to adjust the responsiveness of the controls.

1. Tuning

    Settings for how quickly knob rotations change the light's parameters.

1. Action text

    When any controls are operated on the smart knob it sends *"action"* text to Home Assistant.  This section defines what operation the dimmer should perform when some *"action"* text is received.  

    If your smart knob doesn't work with the
    default values, it is probably because the output text from your smart knob doesn't
    fully match what is defined in the default values. If this is the case,
    you can add your own custom values to the list.
    
    To work out what "action" test is produced by your device
    - Open Zigbee2MQTT in Home Assistant
    - Open the details page for your smart knob
    - Change to the "State" tab
    - Operate the button in each of the ways listed in the blueprint. Single click, double click etc.
    - Take note of the text that appears next to the "action" parameter
    - Enter the text into the blueprint, choosing the section to match the action you want the dimmer to take

    *Important:* All the smart knobs that I have used have at least two modes.  The "action" text changes depending on which mode is active.  That is why there are two entries for most of the actions.  You must record the text for all available modes and enter them in the template.  

    Check the documentation for your smart knob to learn how to change modes.
    For instance, for the smart knob used for development, triple-clicking the button toggles it's modes. So, while you are recording text, triple-click your button or otherwise change modes and run through all of the actions again.  Take note of any new text and add it to the desired field.

    If you make additions, there is no need to delete redundant text entries.  The blueprint simply runs through all action text and checks for a match. No match, no action.

## Thanks

This blueprint is inspired by smirnowegor's blueprint for CCT control 

https://community.home-assistant.io/t/zigbee2mqtt-control-light-entity-including-press-turn-with-tuya-moes-smart-knob-ers-10tzbvk-aa-v1-1/787779/33

## YAML configuration

As per the directions on Blueprints Exchange, here is the full blueprint and helper YAML config

### The blueprint

```yaml
blueprint:
  homeassistant:
    min_version: 2024.6.0
  name: "Control RGB CCT light with Zigbee Smart Knob"
  description: >
    Blueprint to easily configure a **Zigbee Smart Knob** to control one or several light entities when 
    integrated into Home Assistant using **Zigbee2MQTT**.


    You can configure any light to be controlled by this blueprint.  If a light doesn't support a
    particular control function, the command is simply ignored by the light.

    
    Helpers must be created for this blueprint to operate.  You must follow the installation procedure detailed in the Github repository

    https://github.com/njkeng/Smart-Knob-Dimmer-RGB-CCT-Z2M


    This blueprint is inspired by smirnowegor's blueprint for CCT control 

    https://community.home-assistant.io/t/zigbee2mqtt-control-light-entity-including-press-turn-with-tuya-moes-smart-knob-ers-10tzbvk-aa-v1-1/787779/33


    Version 2006.06.06


    <b>Dimmer operating instructions</b>

      - Short press: Toggle light On/Off

      - Long press: Change between CCT mode and RGB mode
      
      - Double press: Change the parameter to adjust
        - In CCT mode, toggles between brightness and color temperature
        - In RGB mode, cycles between hue, saturation and brightness

      - Turn left/right: Adjust the value of the active parameter. The 
      adjustment knob works best when turned slowly.



    <details>
    <summary><b>Note about common smart bulb terminology</b></summary>
      
      - When the bulb is in RGB mode, it is a multi-color device and the brightness, hue and 
      saturation can be adjusted using the knob. 

      - When the bulb is in CCT mode, it is a single-color white light and the brightness 
      and warmth can be adjusted using the knob.
    </details>


    <details>
    <summary><b>Devices used for development</b></summary>

    Developed using generic brand Smart Knob TS004F.  
    Zigbee2MQTT automatically selected the device protocol as Moes ZG-101ZD.

    https://www.aliexpress.com/item/1005008917041951.html



    Tested on these bulbs:
      - Arlec Grid Connect GLD322HA on Tuya WiFi
      - AliExpress Zigbee RGB+CCT light model TS0505B_1
      - Three different generic brand AliExpress RGB CCT Tuya wifi bulbs
      - STOFTMOLN ceiling/wall lamp WW24. This lamp has brightness adjustment only.
    </details>

  source_url: https://github.com/njkeng/Smart-Knob-Dimmer-RGB-CCT-Z2M/blob/6e7191cc1fcf452b0e97b4cc9a1c0991e6692752/dimmer_RGB_CCT.yaml
  domain: automation
  input:
    Essentials:
      description: Physical devices
      input:
        mqtt_topic:
          name: MQTT Topic
          description: >
            The parent MQTT topic of the smart knob (e.g. 'zigbee2mqtt/Tuya Smart Knob)'. 
            Typically the last part will be the friendly name of your device.


            To get the MQTT topic for your device, go to Zigbee2MQTT and open the details page for
            your Smart Knob.  Copy the text for MQTT and paste it in here.

          selector:
            text:
        light_entities:
          name: Light Entity List
          description: "The light or lights to be controlled."
          selector:
            entity:
              multiple: true
              filter:
                - domain: light
    Data:
      description: Data storage
      collapsed: false
      input:
        helper_name:
          name: Dimmer number
          description: |
            Each dimmer blueprint you configure needs to use a different number.


            <details>
            <summary><b>Configuration help</b></summary>
            Home Assistant Helpers are needed to store the current state of 
            the dimmer controls. The supplied file contains all of the helpers 
            needed to configure 5 dimmers.  If your installation needs more 
            than 5 dimmers, you must extend the Helpers configuration file.  
            Simply copy and paste more helpers and update the numerical suffix 
            on every helper.
            </details>
          default: 1
          selector:
            # number:
            #   min: 1
            #   max: 5
            #   mode: slider
            state:
              entity_id: input_select.dimmer_number
              hide_states:
                - unavailable
                - unknown
    Tuning:
      description: Adjust responsiveness of the dimmer
      collapsed: true
      input:
        brightness_step_pct:
          name: Brightness Step Size
          description: "Brightness change per knob click in percent. Higher values = faster changes."
          default: 15
          selector:
            number:
              min: 1
              max: 20
              mode: slider
        min_brightness_pct:
          name: Minimum Brightness (%)
          description: "The minimum brightness you want the light to dim. Set 0 to be able to turn off the light with the dimmer. (Default 1)"
          default: 1
          selector:
            number:
              min: 0
              max: 20
        cct_step:
          name: Color Temperature Step Size
          description: "Color temperature change per knob click in percent. Higher values = faster changes."
          default: 15
          selector:
            number:
              min: 1
              max: 20
              mode: slider
        hue_step_pct:
          name: Hue Step Size
          description: "Hue change per knob click in percent. Higher values = faster changes."
          default: 4
          selector:
            number:
              min: 1
              max: 20
              mode: slider
        saturation_step:
          name: Saturation Step Size
          description: "Saturation change per knob click. Higher values = faster changes."
          default: 10
          selector:
            number:
              min: 1
              max: 20
              mode: slider
    Action text:
      description: |
        "action" text received from the smart knob

        <details>
        <summary><b>Configuration help</b></summary>

        The blueprint looks for the "action" text that the dimmer returns when the user
        operates something on the smart knob. If your smart knob doesn't work with the
        default values, it is probably because the output text from your smart knob doesn't
        fully match what is defined in the default list. If this is the case,
        you can add your own custom values to the list.



        To work out what "action" test is produced by your device
            - Open Zigbee2MQTT in Home Assistant
            - Open the details page for your smart knob
            - Change to the "State" tab
            - Operate the button in each of the ways listed in the template below. Single click, double click etc.
            - Take note of the text that appears next to the "action" parameter
            - Enter the text into the blueprint, choosing the section to match the action you want the dimmer to take



        Important:
        All the smart knobs that I have used have at least two modes.  The "action" text 
        changes depending on which mode
        is active.  That is why there are two entries for most of the actions.  You must record the text
        for all available modes and enter them in the template.  



        Check the documentation for your smart knob to learn how to change modes.
        For instance, for my smart knobs you simply
        triple-click the button to toggle modes. So, while you are recording text, triple-click your
        button or otherwise change modes and run through all of the actions again.  
        Take note of any new text and add it to the template sensor.
        There is no need to delete unused text entries.  The blueprint simply runs through 
        all actions and checks for a match. No match, no action.

      collapsed: true
      input:
        single_click:
          name: Single click
          description: "Output text items for single click action"
          default: "toggle\nsingle"
          selector:
            text:
              multiline: true
        double_click:
          name: Double click
          description: "Output text items for single click action"
          default: "double"
          selector:
            text:
              multiline: true
        rotate_left:
          name: Rotate left
          description: "Output text items for rotating left action"
          default: "brightness_step_down\nrotate_left"
          selector:
            text:
              multiline: true
        rotate_right:
          name: Rotate right
          description: "Output text items for rotating right action"
          default: "brightness_step_up\nrotate_right"
          selector:
            text:
              multiline: true
        long_press:
          name: Long press
          description: "Output text items for long press action"
          default: "hue_move\nhold"
          selector:
            text:
              multiline: true
    Automation:
      description: Blueprint automation mode (advanced)
      collapsed: true
      input:
        automation_mode:
          name: Automation Mode
          description: "Experiment with the different modes if your experience does not feel smooth enough. Normally single should work alright. (Also see https://www.home-assistant.io/docs/automation/modes/ )"
          default: single
          selector:
            select:
              mode: dropdown
              options:
                - single
                - restart
                - queued
                - parallel
trigger:
  - platform: mqtt
    topic: !input mqtt_topic

condition: []

action:
  - variables:
      command: "{{ trigger.payload_json.action }}"
      brightness_step_pct: !input brightness_step_pct
      brightness_step: "{{ (brightness_step_pct * 255 / 100) | int }}"
      cct_step: !input cct_step
      hue_step_pct: !input hue_step_pct
      hue_step: "{{ (hue_step_pct * 360 / 100) | int }}"
      saturation_step: !input saturation_step
      min_brightness_pct: !input min_brightness_pct
      min_brightness: "{{ (min_brightness_pct * 255 / 100) | int }}"
      mqtt_topic: !input mqtt_topic
      light_entities: !input light_entities
      helper_name: !input helper_name
      color_mode: "{{ 'input_select.dimmer_blueprint_mode_' + helper_name | string }}"
      color_mode_txt: "{{ states(color_mode) }}"
      rgb_mode: "{{ 'input_select.dimmer_blueprint_rgb_' + helper_name | string }}"
      rgb_mode_txt: "{{ states(rgb_mode) }}"
      cct_mode: "{{ 'input_select.dimmer_blueprint_cct_' + helper_name | string }}"
      cct_mode_txt: "{{ states(cct_mode) }}"
      cct_brightness: "{{ 'input_text.dimmer_blueprint_cct_brightness_' + helper_name | string }}"
      cct_temperature: "{{ 'input_text.dimmer_blueprint_cct_temperature_' + helper_name | string }}"
      rgb_hue: "{{ 'input_text.dimmer_blueprint_rgb_hue_' + helper_name | string }}"
      rgb_saturation: "{{ 'input_text.dimmer_blueprint_rgb_saturation_' + helper_name | string }}"
      rgb_brightness: "{{ 'input_text.dimmer_blueprint_rgb_brightness_' + helper_name | string }}"
      single_click_txt: !input single_click
      single_click_list: "{{ single_click_txt.split('\n') }}"
      double_click_txt: !input double_click
      double_click_list: "{{ double_click_txt.split('\n') }}"
      rotate_left_txt: !input rotate_left
      rotate_left_list: "{{ rotate_left_txt.split('\n') }}"
      rotate_right_txt: !input rotate_right
      rotate_right_list: "{{ rotate_right_txt.split('\n') }}"
      long_press_txt: !input long_press
      long_press_list: "{{ long_press_txt.split('\n') }}"

  - choose:
      - conditions: # Change color mode to RGB
          - condition: template
            value_template: >-
              {% if color_mode_txt == "cct" %}
                {% for action in long_press_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          - alias: "Update tracking parameters"
            sequence:
              - service: input_select.select_option
                target:
                  entity_id: "{{ color_mode }}"
                data:
                  option: "rgb"
              - service: input_select.select_option
                target:
                  entity_id: "{{ rgb_mode }}"
                data:
                  option: "hue"
              - service: input_text.set_value
                target:
                  entity_id: "{{ cct_brightness }}"
                data:
                  value: "{{ state_attr(light_entities | first, 'brightness') | int(80) }}"
              - service: input_text.set_value
                target:
                  entity_id: "{{ cct_temperature }}"
                data:
                  value: "{{ state_attr(light_entities | first, 'color_temp_kelvin') | int(4000) }}"
          - alias: "Turn on devices"
            repeat:
              for_each: "{{ light_entities }}"
              sequence:
                - service: light.turn_on
                  target:
                    entity_id: "{{ repeat.item }}"
                  data:
                    brightness: "{{ states(rgb_brightness) | int(80) }}"
                    hs_color:
                      - "{{ states(rgb_hue) | float(80) }}"
                      - "{{ states(rgb_saturation) | float(80) }}"
          - delay: 2

      - conditions: # Change color mode to CCT
          - condition: template
            value_template: >-
              {% if color_mode_txt == "rgb" %}
                {% for action in long_press_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          - alias: "Update tracking parameters"
            sequence:
              - service: input_select.select_option
                target:
                  entity_id: "{{ color_mode }}"
                data:
                  option: "cct"
              - service: input_select.select_option
                target:
                  entity_id: "{{ cct_mode }}"
                data:
                  option: "brightness"
              - service: input_text.set_value
                target:
                  entity_id: "{{ rgb_hue }}"
                data:
                  value: "{{ state_attr(light_entities | first, 'hs_color')[0] | float(80) }}"
              - service: input_text.set_value
                target:
                  entity_id: "{{ rgb_saturation }}"
                data:
                  value: "{{ state_attr(light_entities | first, 'hs_color')[1] | float(80) }}"
              - service: input_text.set_value
                target:
                  entity_id: "{{ rgb_brightness }}"
                data:
                  value: "{{ state_attr(light_entities | first, 'brightness') | int(80) }}"
          - alias: "Turn on devices"
            repeat:
              for_each: "{{ light_entities }}"
              sequence:
                - service: light.turn_on
                  target:
                    entity_id: "{{ repeat.item }}"
                  data:
                    brightness: "{{ states(cct_brightness) | int(80) }}"
                    color_temp_kelvin: "{{ states(cct_temperature) | int(4000) }}"
          - delay: 2

      - conditions: # Change RGB parameter
          - condition: template
            value_template: >-
              {% if color_mode_txt == "rgb" %}
                {% for action in double_click_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          - service: input_select.select_next
            target:
              entity_id: "{{ rgb_mode }}"

      - conditions: # Change CCT parameter
          - condition: template
            value_template: >-
              {% if color_mode_txt == "cct" %}
                {% for action in double_click_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          - service: input_select.select_next
            target:
              entity_id: "{{ cct_mode }}"

      - conditions:
          - condition: template # Toggle light ON or OFF
            value_template: >-
              {% for action in single_click_list %}
                {% if command == action %}
                  true
                {% endif %}
              {% endfor %}
        sequence:
          repeat:
            for_each: "{{ light_entities }}"
            sequence:
              - service: light.toggle
                target:
                  entity_id: "{{ repeat.item }}"

      - conditions: # Turn brightness down
          - condition: template
            value_template: >-
              {% if (color_mode_txt == 'rgb' and rgb_mode_txt == 'brightness') or (color_mode_txt == 'cct' and cct_mode_txt == 'brightness') %}
                {% for action in rotate_left_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          repeat:
            for_each: "{{ light_entities }}"
            sequence:
              - service: light.turn_on
                target:
                  entity_id: "{{ repeat.item }}"
                data:
                  brightness: >-
                    {% set current_brightness = state_attr(repeat.item, 'brightness') | int %} 
                    {% set new_brightness = current_brightness - brightness_step %}
                    {{ [new_brightness, min_brightness] | max }}

      - conditions: # Turn brightness up
          - condition: template
            value_template: >-
              {% if (color_mode_txt == 'rgb' and rgb_mode_txt == 'brightness') or (color_mode_txt == 'cct' and cct_mode_txt == 'brightness') %}
                {% for action in rotate_right_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          repeat:
            for_each: "{{ light_entities }}"
            sequence:
              - service: light.turn_on
                target:
                  entity_id: "{{ repeat.item }}"
                data:
                  brightness: >-
                    {% set current_brightness = state_attr(repeat.item, 'brightness') | int %} 
                    {% set new_brightness = current_brightness + brightness_step %}
                    {{ [new_brightness, 255] | min }}

      - conditions: # Turn color temperature down
          - condition: template
            value_template: >-
              {% if color_mode_txt == 'cct' and cct_mode_txt == 'temperature' %}
                {% for action in rotate_left_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          repeat:
            for_each: "{{ light_entities }}"
            sequence:
              - service: light.turn_on
                target:
                  entity_id: "{{ repeat.item }}"
                data:
                  color_temp_kelvin: >-
                    {% set current_kelvin = state_attr(repeat.item, 'color_temp_kelvin') | int(0) %}
                    {% set kelvin_min = state_attr(repeat.item, 'min_color_temp_kelvin') | int(0) %}
                    {% set kelvin_max = state_attr(repeat.item, 'max_color_temp_kelvin') | int(0) %}
                    {% set step_scaled = (kelvin_max - kelvin_min) / 100 | int(0) %}
                    {% if current_kelvin is not none %}
                      {% set new_kelvin = [current_kelvin - (step_scaled * cct_step), kelvin_min] | max %}
                      {{ new_kelvin }}
                    {% else %}
                      {{ kelvin_min }}
                    {% endif %}

      - conditions: # Turn color temperature up
          - condition: template
            value_template: >-
              {% if color_mode_txt == 'cct' and cct_mode_txt == 'temperature' %}
                {% for action in rotate_right_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          repeat:
            for_each: "{{ light_entities }}"
            sequence:
              - service: light.turn_on
                target:
                  entity_id: "{{ repeat.item }}"
                data:
                  color_temp_kelvin: >-
                    {% set current_kelvin = state_attr(repeat.item, 'color_temp_kelvin') | int(0) %}
                    {% set kelvin_min = state_attr(repeat.item, 'min_color_temp_kelvin') | int(0) %}
                    {% set kelvin_max = state_attr(repeat.item, 'max_color_temp_kelvin') | int(0) %}
                    {% set step_scaled = (kelvin_max - kelvin_min) / 100 | int(0) %}
                    {% if current_kelvin is not none %}
                      {% set new_kelvin = [current_kelvin + (step_scaled * cct_step), kelvin_max] | min %}
                      {{ new_kelvin }}
                    {% else %}
                      {{ kelvin_min }}
                    {% endif %}

      - conditions: # Turn hue down
          - condition: template
            value_template: >-
              {% if color_mode_txt == 'rgb' and rgb_mode_txt == 'hue' %}
                {% for action in rotate_left_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          repeat:
            for_each: "{{ light_entities }}"
            sequence:
              - service: light.turn_on
                target:
                  entity_id: "{{ repeat.item }}"
                data:
                  hs_color:
                    - >-
                      {% set current_hue = state_attr(repeat.item, 'hs_color')[0] | float %}
                      {% set new_hue = current_hue - hue_step | float %}
                      {% if new_hue < 0 %}
                        {% set new_hue = 360 | float %}
                      {% endif %}
                      {{ new_hue }}

                    - >-
                      {{ state_attr(repeat.item, 'hs_color')[1] }}

      - conditions: # Turn hue up
          - condition: template
            value_template: >-
              {% if color_mode_txt == 'rgb' and rgb_mode_txt == 'hue' %}
                {% for action in rotate_right_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          repeat:
            for_each: "{{ light_entities }}"
            sequence:
              - service: light.turn_on
                target:
                  entity_id: "{{ repeat.item }}"
                data:
                  hs_color:
                    - >-
                      {% set current_hue = state_attr(repeat.item, 'hs_color')[0] | float %}
                      {% set new_hue = current_hue + hue_step | float %}
                      {% if new_hue > 360 %}
                        {% set new_hue = 0 | float %}
                      {% endif %}
                      {{ new_hue }}

                    - >-
                      {{ state_attr(repeat.item, 'hs_color')[1] }}

      - conditions: # Turn saturation down
          - condition: template
            value_template: >-
              {% if color_mode_txt == 'rgb' and rgb_mode_txt == 'saturation' %}
                {% for action in rotate_left_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          repeat:
            for_each: "{{ light_entities }}"
            sequence:
              - service: light.turn_on
                target:
                  entity_id: "{{ repeat.item }}"
                data:
                  hs_color:
                    - >-
                      {{ state_attr(repeat.item, 'hs_color')[0] }}

                    - >-
                      {% set current_saturation = state_attr(repeat.item, 'hs_color')[1] | float %}
                      {% set new_saturation = current_saturation - saturation_step | float %}
                      {% if new_saturation < 0 %}
                        {% set new_saturation = 0 | float %}
                      {% endif %}
                      {{ new_saturation }}

      - conditions: # Turn saturation up
          - condition: template
            value_template: >-
              {% if color_mode_txt == 'rgb' and rgb_mode_txt == 'saturation' %}
                {% for action in rotate_right_list %}
                  {% if command == action %}
                    true
                  {% endif %}
                {% endfor %}
              {% endif %}
        sequence:
          repeat:
            for_each: "{{ light_entities }}"
            sequence:
              - service: light.turn_on
                target:
                  entity_id: "{{ repeat.item }}"
                data:
                  hs_color:
                    - >-
                      {{ state_attr(repeat.item, 'hs_color')[0] }}

                    - >-
                      {% set current_saturation = state_attr(repeat.item, 'hs_color')[1] | float %}
                      {% set new_saturation = current_saturation + saturation_step | float %}
                      {% if new_saturation > 100 %}
                        {% set new_saturation = 100 | float %}
                      {% endif %}
                      {{ new_saturation }}

mode: !input automation_mode
max_exceeded: silent
```
### The helpers package

```yaml
# This file contains the definitions for helpers required by the automation blueprint
# Control RGB CCT light with Zigbee Smart Knob
#
# To make use of this file:
# 1/ Create a new folder in the root folder that contains your config.yaml
# 2/ Copy this file into you new packages folder
# 3/ Edit your config.yaml so that you can add package definition files.  These
#   files are added into config.yaml whenever home assistant loads config.yaml
#   Copy and paste the following lines into your config.yaml
#   (Make sure you remove the leading indentation after you paste these lines in.
#   The first two lines need to be up against the left.)
#
# # Custom packages for my stuff
# homeassistant:
#   packages:
#     dimmer_helpers: !include packages/dimmer_helpers.yaml
#
# 3/ Reload all home assistant configuration.  Two steps are needed.
#   The first step checks for errors. 
#     Go to settings -> developer tools -> check configuration
#   And if there are no errors, reload all YAML
#     Go to settings -> developer tools -> all YAML configuration
#

input_select:
  # This helper allows selection of a unique number for each dimmer configured using the 
  # blueprint. If you extend the number of dimmers by adding more helpers, make sure you 
  # also add more options to this list so that you can select your new dimmer numbers.
  # Initially, this helper defines enough helpers for five dimmers.
  #
  dimmer_number:
    name: Dimmer number
    options:
      - 1
      - 2
      - 3
      - 4
      - 5
    initial: 1
    icon: mdi:knob

  # The following sets of 3 helpers are needed for the Dimmer RGB CCT blueprint
  # One set of thee helpers is required for each instance of the dimmer
  # blueprint that you configure. There are 5 sets of helpers defined here.
  # If you are using more than 5 instances, copy and paste a new set of these 
  # helpers and change the number suffix at the end. Keep using the same number 
  # suffix for each of the 3 helpers in each set.
  #
  # Dimmer 1
  #
  dimmer_blueprint_mode_1:
    name: Mode dimmer 1
    options:
      - cct
      - rgb
    initial: cct
  dimmer_blueprint_cct_1:
    name: CCT parameter dimmer 1
    options:
      - brightness
      - temperature
    initial: brightness
  dimmer_blueprint_rgb_1:
    name: RGB parameter dimmer 1
    options:
      - hue
      - saturation
      - brightness
    initial: hue
  #
  # Dimmer 2
  #
  dimmer_blueprint_mode_2:
    name: Mode dimmer 2
    options:
      - cct
      - rgb
    initial: cct
  dimmer_blueprint_cct_2:
    name: CCT parameter dimmer 2
    options:
      - brightness
      - temperature
    initial: brightness
  dimmer_blueprint_rgb_2:
    name: RGB parameter dimmer 2
    options:
      - hue
      - saturation
      - brightness
    initial: hue
  #
  # Dimmer 3
  #
  dimmer_blueprint_mode_3:
    name: Mode dimmer 3
    options:
      - cct
      - rgb
    initial: cct
  dimmer_blueprint_cct_3:
    name: CCT parameter dimmer 3
    options:
      - brightness
      - temperature
    initial: brightness
  dimmer_blueprint_rgb_3:
    name: RGB parameter dimmer 3
    options:
      - hue
      - saturation
      - brightness
    initial: hue

  #
  # Dimmer 4
  #
  dimmer_blueprint_mode_4:
    name: Mode dimmer 4
    options:
      - cct
      - rgb
    initial: cct
  dimmer_blueprint_cct_4:
    name: CCT parameter dimmer 4
    options:
      - brightness
      - temperature
    initial: brightness
  dimmer_blueprint_rgb_4:
    name: RGB parameter dimmer 4
    options:
      - hue
      - saturation
      - brightness
    initial: hue
  #
  # Dimmer 5
  #
  dimmer_blueprint_mode_5:
    name: Mode dimmer 5
    options:
      - cct
      - rgb
    initial: cct
  dimmer_blueprint_cct_5:
    name: CCT parameter dimmer 5
    options:
      - brightness
      - temperature
    initial: brightness
  dimmer_blueprint_rgb_5:
    name: RGB parameter dimmer 5
    options:
      - hue
      - saturation
      - brightness
    initial: hue

input_text:

  # The following sets of 5 helpers are needed for the Dimmer RGB CCT blueprint
  # One set of these helpers is required for each instance of the dimmer
  # blueprint that you configure. There are 5 sets of helpers defined here.
  # If you are using more than 5 instances, copy and paste a new set of these 
  # helpers and change the number suffix at the end. Keep using the same number 
  # suffix for each of the 5 helpers in each set.
  #
  # Dimmer 1
  #
  dimmer_blueprint_cct_brightness_1:
    name: CCT last brightness value dimmer 1
    min: 1
    max: 10
  dimmer_blueprint_cct_temperature_1:
    name: CCT last temperature value dimmer 1
    min: 1
    max: 10
  dimmer_blueprint_rgb_hue_1:
    name: RGB last hue value dimmer 1
    min: 1
    max: 10
  dimmer_blueprint_rgb_saturation_1:
    name: RGB last saturation value dimmer 1
    min: 1
    max: 10
  dimmer_blueprint_rgb_brightness_1:
    name: RGB last brightness value dimmer 1
    min: 1
    max: 10
  #
  # Dimmer 2
  #
  dimmer_blueprint_cct_brightness_2:
    name: CCT last brightness value dimmer 2
    min: 1
    max: 10
  dimmer_blueprint_cct_temperature_2:
    name: CCT last temperature value dimmer 2
    min: 1
    max: 10
  dimmer_blueprint_rgb_hue_2:
    name: RGB last hue value dimmer 2
    min: 1
    max: 10
  dimmer_blueprint_rgb_saturation_2:
    name: RGB last saturation value dimmer 2
    min: 1
    max: 10
  dimmer_blueprint_rgb_brightness_2:
    name: RGB last brightness value dimmer 2
    min: 1
    max: 10
  #
  # Dimmer 3
  #
  dimmer_blueprint_cct_brightness_3:
    name: CCT last brightness value dimmer 3
    min: 1
    max: 10
  dimmer_blueprint_cct_temperature_3:
    name: CCT last temperature value dimmer 3
    min: 1
    max: 10
  dimmer_blueprint_rgb_hue_3:
    name: RGB last hue value dimmer 3
    min: 1
    max: 10
  dimmer_blueprint_rgb_saturation_3:
    name: RGB last saturation value dimmer 3
    min: 1
    max: 10
  dimmer_blueprint_rgb_brightness_3:
    name: RGB last brightness value dimmer 3
    min: 1
    max: 10
  #
  # Dimmer 4
  #
  dimmer_blueprint_cct_brightness_4:
    name: CCT last brightness value dimmer 4
    min: 1
    max: 10
  dimmer_blueprint_cct_temperature_4:
    name: CCT last temperature value dimmer 4
    min: 1
    max: 10
  dimmer_blueprint_rgb_hue_4:
    name: RGB last hue value dimmer 4
    min: 1
    max: 10
  dimmer_blueprint_rgb_saturation_4:
    name: RGB last saturation value dimmer 4
    min: 1
    max: 10
  dimmer_blueprint_rgb_brightness_4:
    name: RGB last brightness value dimmer 4
    min: 1
    max: 10
  #
  # Dimmer 5
  #
  dimmer_blueprint_cct_brightness_5:
    name: CCT last brightness value dimmer 5
    min: 1
    max: 10
  dimmer_blueprint_cct_temperature_5:
    name: CCT last temperature value dimmer 5
    min: 1
    max: 10
  dimmer_blueprint_rgb_hue_5:
    name: RGB last hue value dimmer 5
    min: 1
    max: 10
  dimmer_blueprint_rgb_saturation_5:
    name: RGB last saturation value dimmer 5
    min: 1
    max: 10
  dimmer_blueprint_rgb_brightness_5:
    name: RGB last brightness value dimmer 5
    min: 1
    max: 10
```
