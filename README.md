# Zooz Z-Wave Blueprints for Home Assistant

Home Assistant automation blueprints for Zooz Z-Wave switches and remotes using the Z-Wave JS integration.

These are updated versions of blueprints from [gregmac/HomeAssistant-ZoozBlueprints](https://github.com/gregmac/HomeAssistant-ZoozBlueprints), updated to work with Home Assistant 2026.11 and later.

## Why this fork exists

Home Assistant 2026.11 requires zwave-js-server 3.11.0 or newer. With that version, Central Scene events from Zooz devices carry new label text. For example, a ZEN76 down-paddle press used to report `label: "Scene 002"` and now reports `label: "Lower Paddle"`.

Blueprints that matched on the label text stopped working: the automation triggers, the `choose` block finds no match, and the trace ends with no error.

These blueprints match on the numeric fields in the event instead, which don't depend on label text:

| Field | Meaning |
|---|---|
| `property_key` | `001` = up paddle, `002` = down paddle |
| `value_raw` | `0` pressed, `1` released, `2` held, `3` 2x, `4` 3x, `5` 4x, `6` 5x |

## Blueprints

### ZEN76 S2 Switch

Actions for 1x–5x presses, hold, and release on both paddles. On automation reload and on Home Assistant start, it also:

- enables scene control (parameter 10)
- disables paddle programming mode (parameter 17, firmware 10.0+), which is needed for 3x presses
- optionally sets local paddle control (parameter 12, "Smart Bulb Mode")

Supports ZEN76 and ZEN76 800LR.

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fwotelljm%2FHomeAssistantZoozSwitchBlueprints%2Fblob%2Fmain%2Fzooz-zen76-s2-zwavejs.yaml)

**Changes from the original:**
- Matches on `property_key` / `value_raw` instead of label text
- Current automation syntax (`triggers:` / `actions:` / `action:`)
- Device selector uses the current `filter:` format and includes the 800LR
- Device settings are also applied on Home Assistant start, not only on reload
- Fixed: the "Do not modify configuration" option for local control was ignored, and parameter 12 was always written

### ZEN34 Remote

Actions for 1x–5x taps and hold on both paddles.

Based on the blueprint in the Home Assistant community thread [Zooz ZEN34 Z-Wave JS](https://community.home-assistant.io/t/zooz-zen34-zwave-js/290561).

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fwotelljm%2FHomeAssistantZoozSwitchBlueprints%2Fblob%2Fmain%2Fzooz-zen34-zwave-js.yaml)

**Changes from the original:**
- Matches on `property_key` / `value_raw` instead of label text
- Fixed: hold actions never fired (the original checked for `KeyHeld`; Z-Wave JS sends `KeyHeldDown`)
- Triggers only on the selected remote instead of every Z-Wave scene event in the house
- Runs in `queued` mode so presses aren't dropped when other devices send events at the same time
- Fixed the fallback debug log message

## Updating an existing automation

If you already use the original blueprint:

1. Replace the blueprint file in `config/blueprints/automation/...` with the version from this repo, or import it with the button above.
2. Go to Developer Tools → YAML → **Reload automations**.

Input names are unchanged, so your configured actions carry over.

## Troubleshooting

If presses don't trigger anything, watch the raw event:

- **Developer Tools → Events:** listen for `zwave_js_value_notification`, then press the paddle.
- **Node-RED:** an *events: all* node for `zwave_js_value_notification` wired to a debug node.

If no event appears, scene control is probably off on the device. Reload automations to send the setup parameters again (ZEN76), or enable scene control in the device's configuration.

## Credits

- Original ZEN blueprints: [gregmac](https://github.com/gregmac/HomeAssistant-ZoozBlueprints)
- Original ZEN34 blueprint: Home Assistant community, [thread 290561](https://community.home-assistant.io/t/zooz-zen34-zwave-js/290561)
- 2026.11 updates: [wotelljm](https://github.com/wotelljm)
