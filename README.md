# Musik-Display

A 4" touch display for Home Assistant that shows what's playing on a
Bang & Olufsen MasterLink system, and the weather when nothing is playing.
Built with [ESPHome](https://esphome.io) and LVGL on a Guition
ESP32-S3-4848S040 (480 × 480).

It's made for the [mdtv2-tools](https://github.com/tb59427/mdtv2-tools)
MasterLink bridge, but the music page works with any Home Assistant
`media_player` plus a few template sensors.

The on-screen texts and the comments in the YAML are German.

## What it shows

- **Music page:** cover (or station logo), title, artist, and a header line
  with source and album. Sources:
  - N.MUSIC via Music Assistant: title, artist, cover or station logo from
    the MA player;
  - N.MUSIC from other providers (AirPlay, Spotify Connect, …), the
    turntable, and other MasterLink sources such as the CD player:
    recognized title, artist, album and cover from mdtv2-tools. Without a
    recognized cover it shows a built-in record or CD picture.
- **Weather page:** current condition and temperature, 4-day forecast,
  background picture matching the weather.
- **Room awareness:** the display only switches to music when one of the
  B&O players in *its* room plays. The list of players is an `input_text`
  helper in Home Assistant, one per display.
- **Touch:** while music plays, a tap toggles music ↔ weather; the weather
  page returns to music after `wetter_anzeigedauer`.
- **Brightness** follows a light sensor in the room (or the sun's
  elevation); the display is off during the night hours (`nacht_von` …
  `nacht_bis`).

## Requirements

- Guition ESP32-S3-4848S040 (ESP32-S3, 8 MB PSRAM, 16 MB flash, GT911
  touch). Tested with ESPHome 2026.9.
- Home Assistant with the ESPHome add-on/dashboard.
- For the music page with mdtv2-tools: its Home Assistant setup
  (`sensor.mdt_n_music`, `sensor.mdt_phono`, `sensor.mdt_ml_listen`,
  `sensor.n_music_quelle`, `sensor.n_music_inhalt`, `sensor.n_music_logo`);
  see [docs/home-assistant.md](https://github.com/tb59427/mdtv2-tools/blob/master/docs/home-assistant.md).
- A `weather` entity (any weather integration).

## Files

| File | What |
|---|---|
| `musik-display.yaml` | ESPHome configuration of the display |
| `secrets.yaml.example` | Template for the secrets it uses |
| `wetter/` | Background pictures (not included, see [wetter/README.md](wetter/README.md)) |
| `ha/musik-display-sensor.yaml` | `sensor.musik_display`: what to show (phono / bus / ext / nmusic / aus), header, title, artist, picture |
| `ha/raum-aktiv.yaml` | `input_text.musik_display_raumplayer` (the room's players) and `binary_sensor.musik_display_raum_aktiv`: is one of them on? |
| `ha/wetter-daily.yaml` | `sensor.wetter_daily`: daily/hourly forecast of a weather entity as a sensor |
| `ha/wetter-vorhersage.yaml` | `sensor.wetter_vorhersage_display`: current weather and 4-day forecast, prepared for the display; uses a METAR sensor for "now" if you have one |
| `ha/display-helligkeit.yaml` | `sensor.display_helligkeit`: target brightness from a light sensor or the sun |

## Setup

1. **Home Assistant:** add the snippets from `ha/` to your
   `configuration.yaml` (each is a `template:` block; if you already have
   `template:`, append only the list items). Adjust the entity IDs marked in
   the comments: your weather entity (`weather.home`), light sensor, optional
   METAR sensor. Reload template entities.
2. **ESPHome:** copy `musik-display.yaml` into the `esphome/` folder of your
   HA config, the background pictures into `esphome/wetter/`, and add the
   keys from `secrets.yaml.example` to `esphome/secrets.yaml`.
3. Edit the `substitutions` at the top of `musik-display.yaml`:
   - `name` / `friendly_name`, one per display;
   - `ma_player`: your Music Assistant player;
   - `raum_sensor`: the binary sensor from `ha/raum-aktiv.yaml` for this display;
   - `ha_url`: Home Assistant **by IP address** (`http://<ip>:8123`). The
     ESP32 can't resolve `.local` names here. External covers from Music
     Assistant are loaded through HA's image proxy at this address.
   - brightness, weather page timeout and night hours.
4. Install from the ESPHome dashboard (first time via USB, then OTA), and
   adopt the device in Home Assistant.
5. In Home Assistant, enter the room's players in the helper
   **Musik-Display Raumplayer**, comma-separated, e.g.
   `media_player.beovision_10, media_player.beolab_3500`. Empty means: react
   to any music.

## Covers and memory

The ESP32-S3 has little internal RAM. HTTPS downloads need about 30 KB of
it in one block, and a 100 KB cover from the internet can take several
seconds to download and decode. So the display:

- loads covers over plain HTTP from the LAN where it can: mdtv2-tools'
  ha-notifier fetches recognized covers and serves them scaled to 480 px
  (`cover_base_url` must be the Pi's IP), and Music Assistant covers come
  through HA's image proxy (`ha_url`);
- waits for enough free memory only before HTTPS downloads;
- downloads one picture at a time and hides the image while its buffer is
  replaced.

## License

MIT, see [LICENSE](LICENSE).
