# ha-battery-inventory
Home-Assistant Battery Inventory Management

![Battery Inventory Management][report]


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

For the `battery_type` attribute you can use the any text representation of the battery type.

Example:

- CR123A
- LR03/AAA
- LR06/AA
- CR2450

The `quantity` attribute is optional, the template will assume 1 if not specified. The value must be a number.

## UI Card

Just copy the [yaml](MarkdownCard.yam) code and paste it in `Manual card` inside of your Dashboard.

<!-- MARKDOWN-AUTO-DOCS:START (CODE:src=./config/MarkdownCard.yaml) -->
<!-- The below code snippet is automatically added from ./config/MarkdownCard.yaml -->
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
  <td VALIGN=TOP>{{ state.battery_type }}</td><td VALIGN=TOP><b>{{ state.quantity }}</b></td> <td VALIGN=TOP> {%- for info in state.info -%} {{ info.quantity }}x {{ info.name }} <font color={{ (info.level < states.input_number.low_battery_alert_threshold.state | int) | iif("OrangeRed", "White") }}><b>{{ info.level }}%</b></font><br> {%- endfor -%}<br></td>
  </tr>
  {%- endfor -%}
  </tbody>
  </table>
title: Battery Inventory Management
```
<!-- MARKDOWN-AUTO-DOCS:END -->

## License

[MIT](LICENSE)

[report]: images/report-card.png