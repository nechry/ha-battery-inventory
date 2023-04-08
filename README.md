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
<!-- MARKDOWN-AUTO-DOCS:END -->

## License

[MIT](LICENSE)

[report]: images/report-card.png