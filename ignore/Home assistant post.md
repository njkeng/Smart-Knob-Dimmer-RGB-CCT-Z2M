
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

## YAML configuration

The directions on Blueprints Exchange say to include the full YAML configuration here.  Including the config makes the post too long and the forum refuses to create the psot.  So instad, please get the configuration from Github here:

https://github.com/njkeng/Smart-Knob-Dimmer-RGB-CCT-Z2M

## Thanks

This blueprint is inspired by smirnowegor's blueprint for CCT control 

https://community.home-assistant.io/t/zigbee2mqtt-control-light-entity-including-press-turn-with-tuya-moes-smart-knob-ers-10tzbvk-aa-v1-1/787779/33


As per the directions on Blueprints Exchange, here is the full blueprint and helper YAML config

