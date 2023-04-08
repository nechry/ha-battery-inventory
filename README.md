# ha-battery-inventory
Home-Assistant Battery Inventory Management

I wanted to have a report of all my battery devices, by battery type with there quantity.

![Battery Inventory Management][report]

With this report, I can easily see if I need to buy more batteries.
Unfortunately, at the moment, in Home-Assistant there is no way to know the battery type and the quantity used by a device.

But Home-Assistant has a way to customize entities, so I can add the battery type and the quantity used by a device.

## Installation

I use customize.yaml to store all my customizations. If you don't have a customize.yaml file, you can create one.

In your `configuration.yaml` set customize with a file named `customize.yaml` like this:

```yaml
homeassistant:
  customize: !include customize.yaml # customize Home Assistant
```

Inside the `customize.yaml` file use the following code as base template:
<!-- MARKDOWN-AUTO-DOCS:START (CODE:src=./config/customize.yaml) -->
<!-- The below code snippet is automatically added from ./config/customize.yaml -->
```yaml
sensor.first_floor_landing_motion_sensor_battery_level:
  battery_type: CR123A
sensor.first_floor_landing_smoke_sensor_battery_level:
  battery_type: CR123A
sensor.first_floor_landing_lcd_battery:
  battery_type: LR03/AAA
  quantity: 2
sensor.aqara_opple_switch_2x2_battery:
  battery_type: CR2032
sensor.anemometer_battery:
  battery_type: LR06/AA
sensor.attic_staircase_motion_sensor_battery:
  battery_type: CR2450
sensor.garden_south_irrigation_controller_battery:
  battery_type: LR06/AA
  quantity: 4
```
<!-- MARKDOWN-AUTO-DOCS:END -->

You need to adapt the `entity_id` to your needs. Is the entity_id of your battery sensor.
The mandatory attributes is `battery_type`.

For the `battery_type` attribute you can use any text representation of the battery type.

Example:

- CR123A
- LR03/AAA
- LR06/AA
- CR2450

The `quantity` attribute is optional, the template will assume a value of `1` if is not specified. The `quantity` must be a valid number.

## UI Card

Just copy the [yaml](markdown-card.yam) code and paste it into a empty `Manual card` inside of your Dashboard.

<!-- MARKDOWN-AUTO-DOCS:START (CODE:src=./config/markdown-card.yaml) -->
<!-- The below code snippet is automatically added from ./config/markdown-card.yaml -->
```yaml
type: markdown
content: >-
  {%- set ns = namespace(sensors=[], battery_type = '', quantity = 0, report=[], info=[]) -%}
  {%- for state in states.sensor 
      | selectattr('attributes.device_class', 'defined')
      | selectattr('attributes.device_class', 'eq', 'battery')
      | selectattr('attributes.battery_type', 'defined') -%}
    {%- if not state_attr(state.entity_id, 'battery_type') in ['SolarPanel','DC', 'Battery', 'APCRBC110'] -%}
      {% set ns.sensors = ns.sensors + [dict(name= state.name | lower | replace(':', '') | replace(' battery level', '') | replace(' battery', '') | capitalize() , battery_type=state_attr(state.entity_id, 'battery_type'), quantity=state_attr(state.entity_id, 'quantity') | int(1), level=state.state |int(0) )] %}
    {%- endif -%}
  {%- endfor -%} {%- set battery_list = ns.sensors | sort(attribute='battery_type') %} {%- for battery in battery_list -%}
    {% if ns.battery_type == battery.battery_type %}
      {% set ns.quantity=ns.quantity + battery.quantity %}
      {% set ns.info=ns.info+[dict(name=battery.name,quantity=battery.quantity,level=battery.level)] %}
    {% else %}
      {% if ns.battery_type != '' %}      
        {% set ns.report = ns.report+[dict(battery_type=ns.battery_type, quantity=ns.quantity, info=ns.info | sort(attribute='level')) ] %}
      {% endif %}
      {% set ns.battery_type= battery.battery_type %}
      {% set ns.quantity=battery.quantity %}
      {% set ns.info=[dict(name=battery.name,quantity=battery.quantity,level=battery.level)] %}
    {% endif %}
  {%- endfor -%}
  <table>
  <thead>
  <tr><th>Type</th><th>Quantity</th><th>Details</th></tr>
  </thead>
  <tbody>
  {%- for state in ns.report -%}
  <tr>
  <td VALIGN=TOP>{{ state.battery_type }}</td><td VALIGN=TOP><b>{{ state.quantity }}</b></td> <td VALIGN=TOP> {%- for info in state.info -%} {{ info.quantity }}x {{ info.name }} <font color={{ (info.level < states.input_number.low_battery_report_threshold.state | int) | iif("OrangeRed", "White") }}><b>{{ info.level }}%</b></font><br> {%- endfor -%}<br></td>
  </tr>
  {%- endfor -%}
  </tbody>
  </table>
title: Battery Inventory Management
```
<!-- MARKDOWN-AUTO-DOCS:END -->

You will also need to create a Numeric helper to set the `low_battery_alert_threshold` value as show:

![Numeric low_battery_report_threshold Helper][low_battery-helper]

### Other Battery level reports

Base on [Lovelace: Battery state card](https://community.home-assistant.io/t/lovelace-battery-state-card/100115)

You can build this battery level reports:

![Battery level][battery_level]

You need to install the Lovelace: Battery state card, check official documentation [Lovelace: Battery state card](https://community.home-assistant.io/t/lovelace-battery-state-card/100115)

Then you can use the following code to build the report:

<!-- MARKDOWN-AUTO-DOCS:START (CODE:src=./config/lovelace-battery.yaml) -->
<!-- The below code snippet is automatically added from ./config/lovelace-battery.yaml -->
```yaml
type: custom:battery-state-card
title: Battery levels
sort_by_level: asc
tap_action: more-info
filter:
  include:
    - name: attributes.device_class
      value: battery
  exclude:
    - name: state
      value: Unknown
      operator: '='
    - name: state
      value: 'On'
      operator: '='
    - name: state
      value: 'Off'
      operator: '='
    - name: state
      value: 60
      operator: '>'
    - name: state
      value: Unavailable
      operator: '='
    - name: attributes.attribution
      value: 'Data provided by Apple iCloud'
    - name: entity_id
      value: sensor.*apple_watch*  
    - name: entity_id
      value: sensor.*iphone*    
    - name: entity_id
      value: sensor.oregon_cm180i*
secondary_info: battery_type
round: 0
bulk_rename:
  - from: Battery Level
    to: ''
  - from: ': Battery level'
    to: ''
  - from: sensor_battery
    to: ''
  - from: Battery level
    to: ''
  - from: battery level
    to: ''
  - from: Battery
    to: ''
  - from: battery
    to: ''
```
<!-- MARKDOWN-AUTO-DOCS:END -->

### Battery stop reported level

Is my last battery report.
Is helpful to discover if a device stop reporting the battery level. The battery level can be good, but the device didn't send anymore report. Can be a problem with the device or the battery is dead!

![Battery report date][battery_report_date]

Also base on [Lovelace: Battery state card](https://community.home-assistant.io/t/lovelace-battery-state-card/100115)

You can use the following code to build the report:

<!-- MARKDOWN-AUTO-DOCS:START (CODE:src=./config/lovelace-battery-date.yaml) -->
<!-- The below code snippet is automatically added from ./config/lovelace-battery-date.yaml -->
```yaml
type: custom:battery-state-card
title: Battery levels date report
sort_by_level: asc
tap_action: more-info
collapse: 10
filter:
  include:
    - name: entity_id
      value: '*_battery_level'
    - name: attributes.device_class
      value: battery
  exclude:
    - name: state
      value: 'On'
      operator: '='
    - name: attributes.attribution
      value: Data provided by Apple iCloud
    - name: entity_id
      value: sensor.*apple_watch*
    - name: entity_id
      value: sensor.*iphone*
    - name: state
      value: 'Off'
      operator: '='
secondary_info: last_updated
bulk_rename:
  - from: Battery Level
    to: ''
  - from: ': Battery level'
    to: ''
  - from: sensor_battery
    to: ''
  - from: Battery level
    to: ''
  - from: battery level
    to: ''
  - from: Battery
    to: ''
  - from: battery
    to: ''
round: 0
```
<!-- MARKDOWN-AUTO-DOCS:END -->

## License

[MIT](LICENSE)

[report]: images/report-card.png
[low_battery-helper]: images/low_battery_report_threshold.png
[battery_level]: images/battery_level.png
[low_battery-alert-helper]: images/low_battery_alert_threshold.png
[battery_report_date]: images/battery_report_date.png
