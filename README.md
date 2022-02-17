
# HA UStvgo sensor
[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://github.com/hacs/integration)

## description:
This custom component for watching UStvgo channels via home assistant, broadcasting to google chromecast. 
This component will create senosr for every channel from UStvgo

------------
## instaltion:
Using HACS (add [this repo](https://github.com/yohaybn/HA_ustvgo_sensor ) as custome repository) 

Or copy [ustvgo folder](https://github.com/yohaybn/HA_ustvgo_sensor/tree/main/custom-components/ustvgo ) to your custom component folder and restart your HA.  

add sensor to your configuration.yaml
<pre><code>-  sensor:
	- platform: ustvgo
	  name: ustvgo
	  scan_interval: 3600
	  hide_vpn_required: true # optional- default true
</code></pre>
this will create sensor for each channel from UStvgo
## Create Lovelace card
![enter image description here](https://github.com/vlad36N/HA_ustvgo_sensor/blob/main/images/lovelace1.png?raw=true)

To create such card that cast the selected channel to selected Chromecast device you need to add the following

**Lovelace card:** (using [auto-entities](https://github.com/thomasloven/lovelace-auto-entities) awesome card )
<pre><code>
type: vertical-stack
cards:
  - type: entities
    entities:
      - entity: input_select.media_tv
  - type: custom:auto-entities
    card:
      type: glance
      show_name: true
      show_icon: false
      show_state: false
      title: USA TV
    filter:
      include:
        - state: ustvgo_*
          options:
            tap_action:
              action: call-service
              service: script.cast_ustvgo
              service_data:
                sensor: this.entity_id
      exclude: []
    sort:
      method: none
    show_empty: true
</code></pre>
**Script:**
<pre><code>
'cast_ustvgo':
  sequence:
    - service: media_player.play_media
      data_template:
        media_content_id: >-
          {{ state_attr(sensor, 'm3u') }}
        media_content_type: media
      data:
        entity_id: "{{ states('input_text.media_tv_to_play') }}"

</code></pre>

add to your input_select.yaml
<pre><code>
    media_tv:
      name: 'TV Players:'
      options:
        - Living
        - Bedroom
        - Office
        - Kitchen
</code></pre>

add to your input_text.yaml
<pre><code>
    media_tv_to_play:
      name: 'TV Devices'
</code></pre>

add to your automation.yaml
<pre><code>
  - alias: "Cast - Selected Name to Device"
    trigger:
      - platform: homeassistant
        event: start
      - platform: state
        entity_id: input_select.media_tv
    action:
      - service: input_text.set_value
        data:
          entity_id: input_text.media_tv_to_play
          value: >-
            {% if is_state("input_select.media_tv", "Living") -%}
              media_player.xbr_75x800g #your device
            {% elif is_state("input_select.media_tv", "Bedroom") -%} 
              media_player.chromecast9410 #your device
            {% elif is_state("input_select.media_tv", "Office") -%}
              media_player.chromecast8612 #your device
            {% elif is_state("input_select.media_tv", "Kitchen") -%}
              media_player.chromecast0280 #your device
            {% endif %}
</code></pre>

restart HA


## Note
I able to cast to LG TV (with Chromecast connected to HDMI) and 2 Sony Bravia (one with cast build in and second one with Chromecast connected to HDMI) and Insignia (with Chromecast connected to HDMI)


## Thanks

to everyone @[Home Assistant](https://www.home-assistant.io/ "Home Assistant")
 for creating the amazing opensource platform for smart home integrations! 🙏🏼 

Thanks to @[benmoose39](https://github.com/benmoose39 "benmoose39") for writing the automation on m3u fetching.
Thanks to @[yishait](https://github.com/yishait "yishait") for the idea.
