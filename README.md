# Zendure Home Assistant Integration — Omakase Edition

> **This is an opinionated, agent-accelerated fork of
> [Gielz1986/Zendure-HA-zenSDK](https://github.com/Gielz1986/Zendure-HA-zenSDK).**
> All credit for the original design, the zenSDK reverse engineering, the automation logic and the
> dashboard goes to **Michiel Hofker (Gielz1986)**. Without his work this repository would not exist.
> If you want the canonical, community-supported version, use the original.

![Preview](Images/Global-Dashboard-290526.gif)

<a href="https://github.com/Gielz1986/Zendure-HA-zenSDK/wiki/Global-%E2%80%90-Available-entities">
Explanation of all entities and the dashboard (upstream wiki)
</a>

<br>

## 🍣 Omakase

Omakase in the [Rails](https://rubyonrails.org/doctrine#omakase) and
[Omarchy](https://omarchy.org) sense: a curated setup where the choices have already been made for you.
The upstream project is built to work for everyone. This fork is built to work for one house, mine.

- **English only.** The Dutch variant is gone.
- **My defaults.** Settings, entity names and dashboard layout follow my own installation. Anything that
  looks like a choice was made on purpose for my setup, not for the general case.
- **Agent-accelerated.** Changes are made together with [Claude Code](https://claude.com/claude-code).
  The agent reads, refactors and documents; I decide what goes in. Review before you copy anything into a
  production Home Assistant.
- **Public, not general-purpose.** The repo is public so others can borrow ideas. It is not a drop-in
  replacement for upstream and I make no promises about compatibility with it.

## 📝 Modifications from upstream

The upstream license requires that modifications are clearly stated. This list is kept current.

| Area | Change | Status |
|-|-|-|
| Documentation | README rewritten for this fork; credits and license pointer added; donation links removed | ✅ Done |
| Languages | Dutch (NL) integration, Dutch README and Dutch screenshots removed | ✅ Done |
| Repository links | Update-check sensor, dashboard badges, pull-request link and device-image CDN point to this repository; the wiki links still go to the upstream wiki | ✅ Done |
| Naming | Package file renamed to `zendure_martienz_global.yaml`; the automation alias and the `gielz` entities now use `martienz` (`input_datetime.zendure_martienz_automation_last_time_executed`, `sensor.github_zendure_martienz`, `sensor.github_zendure_martienz_releasecheck`) | ✅ Done |
| Modes | Added **Smart Matching (Price Aware)** and **Dynamic Smart Matching (Price Aware)**, plus `sensor.dynamic_solar_charging_allowed` and a Solar → Grid badge. See [Price Aware modes](#-price-aware-modes) | ✅ Done |
| Preferences | Further personal defaults and dashboard layout | ⏳ Planned |

Versions of this fork are tagged `vYYYYMMDD` like upstream; the update check in the dashboard compares against the releases of this repository. Everything not listed here is unchanged from upstream
[v20261007](https://github.com/Gielz1986/Zendure-HA-zenSDK/releases).

---

**Get your battery running locally in Home Assistant in just 2️⃣ simple steps.**

Based on the zenSDK RESTful API for Home Assistant. This package connects locally to one
**Zendure Solarflow 2400 (AC, AC+ or AC Pro) / Zendure Solarflow 1600 AC+ / Zendure Solarflow 800
(Pro(2) or Plus) / Zendure Solarflow 3000 Mix AC+ / Zendure Solarflow 4000 Mix (AC+ or Pro)**.
Perfect for anyone who wants to run their battery **100% local** in Home Assistant. There are
**13 preconfigured modes**, from relaxed solar-based usage to acting like an energy trader with dynamic
pricing for a few extra cents.

Several integration add-ons are also available, including support for multiple inverters and older
models such as the SolarFlow Hyper 2000. See [Integration add-ons](#-optional-integration-add-ons).

## 1️⃣ Making Entities Available

#### ℹ️ Required Hardware

- Homewizard P1 (or another home energy meter that provides per-second data (+watt import / -watt export)).
- One Solarflow 2400 (AC, AC+ or AC Pro) / Solarflow 1600 AC+ / Solarflow 800 (Pro(2) or Plus) / Solarflow 3000 Mix AC+ / Solarflow 4000 Mix (AC+ or Pro).
- Or multiple identical inverters combined with the [Node-RED proxy by Gast777](https://github.com/gast777/Zendure-zenSDK-proxy)

---

### #️⃣ Configuration and Restart

1. Make sure **HEMS is disabled** in the Zendure app.
2. Place [zendure_martienz_global.yaml](./Global%20(EN)%20Integration/packages/zendure_martienz_global.yaml) from the packages folder into your Home Assistant `packages` folder. If it does not exist, create it.
3. Create a **backup** of your `configuration.yaml`.
4. Then edit your `configuration.yaml` and add the following:

```
homeassistant:
  packages: !include_dir_named packages
```

| ![Preview](Images/packages2.png) |
|-----------------------------------|

5. Restart Home Assistant.
6. Optionally create the plug-n-play dashboard [Go to Plug-N-Play Dashboard](#-optional-plug-n-play-dashboard). Or fill in the entities below in Home Assistant and restart again.
7. Go to [part 2](#2️⃣-zendure-zensdk-automation) to not only read data but also control the battery with a single automation.

---

<img alt="image" src="https://github.com/user-attachments/assets/c3dc867f-6022-4d12-8926-0e593f411856" />
*plug-n-play dashboard

<br>
<br>

| Explanation per configuration item | |
|-|-|
| **Configuration (Basic)** | **Information** |
| `zendure_setting_ip_address` | **e.g. 192.168.0.172** – Found in the Zendure app under device information. |
| `homewizard_setting_p1_ip_address` | **e.g. 192.168.0.192** – Enable local API in the Homewizard app |
| `zendure_setting_standby_delay` | **(Recommended: 15 minutes) 5–30 minutes** – Defines how quickly the inverter goes into full standby at 0 activity. Prevents ~19W idle consumption. |
| `zendure_setting_set_default_settings` | Once the battery is running, you can use this to apply the recommended settings below. |
| **Configuration (Charging)** | **Information** |
| `zendure_setting_max_charge_power` | **400–2400W** – Maximum charging power (up to 7200W with multiple inverters via Node-RED). |
| `zendure_setting_start_charging_at` | **(Recommended: -300W) -1000 to -80W** – Defines when charging starts. |
| `zendure_setting_charge_buffer` | **(Recommended: 50W) 0–250W** – Determines how much less to include during charging. In summer you can increase this (e.g. 200W) to prevent daytime grid import. |
| **Configuration (Discharging)** | **Information** |
| `zendure_setting_max_discharge_power` | **400–2400W** – Maximum discharge power (up to 7200W with multiple inverters via Node-RED). |
| `zendure_setting_start_discharging_at` | **(Recommended: 100W) 80–500W** – Defines when discharging starts. |
| `zendure_setting_discharge_buffer` | **(Recommended: 5W) 0–250W** – Extra margin for discharging. |
| **Configuration (State Of Charge)** |**Information**|
| `zendure_setting_soc_protection_disabled`    | Check this to disable the dual SOC protection. When the battery drops below the minimum allowed charge percentage, it will no longer automatically recharge. It will wait until the BMS (Battery Management System) takes action. |
| `zendure_setting_minimum_allowed_state_of_charge` | **(Recommended: 10%) 5% to 50%** – Set the minimum allowed state of charge percentage here. |
| `zendure_setting_maximum_allowed_state_of_charge` | **(Recommended: 100%) 70% to 100%** – Set the maximum allowed state of charge percentage here. At 100%, an SOC calibration is performed to accurately estimate the state of charge level. |
| **Configuration (PV)** |**Information**|
| `zendure_setting_pv_export_disabled`    | Check this to disable the pv export. When the battery is full it will no longer export the energy of connected solarpanels. |
| **Configuration (Optional)** | **Information** |
| `home_energy_setting_meter_sensor` | **e.g. sensor.custom_energy** – Add your own home energy sensor (+watt import / -watt export). This will take priority. [Go to the upstream wiki](https://github.com/Gielz1986/Zendure-HA-zenSDK/wiki/Global-and-NL-%E2%80%90-P1-CT-meters-(API's)) for P1/CT API's. |
| `zendure_setting_battery_order` | **e.g. 1;5;3;4;2** – Manually define battery order based on serial numbers and physical stacking. |
| **Configuration (Dynamic)** | **Information** |
| `dynamic_setting_nordpool_sensor` | **e.g. sensor.nordpool_kwh_nl_eur_3_09_0** – Your Nordpool (HACS) sensor. |
| `dynamic_export_correction` | Specify how many cents should be subtracted during export. This value is used in the spread calculation. |
| `dynamic_setting_minimal_spread` | **e.g. 25%** – Minimum price spread before dynamic charging/discharging activates. |
| `dynamic_setting_15_minute_interval` | Enable this if you want to use 15-minute intervals. |
| **Configuration (Dashboard)** | **Information** |
| `dashboard_setting_show_help` | Enable to show help text on the dashboard. |
| `dashboard_setting_show_pv` | Enable to show connected (offgrid/mppt) PV on the dashboard. |
| `dashboard_setting_show_dynamic` | Enable to show dynamic control on the dashboard. |

---

## 2️⃣ Zendure zenSDK Automation

The engine of everything: it charges smartly, discharges smartly, and ensures everything works together. Choose from 13 different modes to make it behave exactly how you want.

If you didn't change any entity names above, you can simply create this automation:

1. Create a new automation.
2. Click **Edit in YAML**.
3. Paste the YAML code from [automation_global.yaml](./Global%20(EN)%20Integration/automation_global.yaml).
4. Save and start the automation.

![Preview](Images/Global-Automation-300326.gif)
![Preview](Images/Global-Automation2-300326.gif)

### 🍣 Price Aware modes

Two modes that exist only in this fork. They are twins of **Smart Matching** and **Dynamic Smart Matching** with one extra rule for solar surplus: sell it while the export price is still high, store it once the cheap part of the day has started.

- **Before the first cheap slot of the day**, solar surplus is exported to the grid. The battery still discharges to cover consumption, exactly like the twin mode.
- **From the start of the first cheap slot**, surplus is charged into the battery for the rest of the day, like the twin mode. The cheap slots are the ones already selected by **Lowest Price X Period** or the manual period, so no new settings are needed.
- **A zero or negative export price** (Nordpool price minus **Export Correction**) charges immediately, because exporting would cost money.
- **No price data** means charging is always allowed, so the mode degrades to its twin.

The decision is visible in `sensor.dynamic_solar_charging_allowed` (`Yes`/`No`, with the attributes `first_cheap_slot`, `export_price` and `reason`), and the dashboard shows a red **Solar → Grid** badge in the Dynamic Today section while the mode is waiting. **Dynamic Smart Matching (Price Aware)** additionally keeps the grid charging of its twin during cheap slots.

---

## ✅ Battery Ready to Go

The moment has arrived: time for your battery to prove it's more than just an expensive decoration with cables.

1. Open the plug-n-play dashboard or add the entity **Zendure Operation Mode**.
2. The mode will be set to **Standby**.
3. Select your desired mode to activate the automation.
4. The battery will run in the desired operation mode.

![Preview](Images/Mode-290326.gif)
<a href="https://github.com/Gielz1986/Zendure-HA-zenSDK/wiki/Global-%E2%80%90-Operation-modes">
Explanation of all modes (upstream wiki)
</a>

---

## 🔃 (Optional) Plug-N-Play Dashboard

You can also directly use a fully plug-n-play dashboard:

1. Requires [Apexcharts HACS](https://github.com/RomRider/apexcharts-card) and (optionally) [Graphite HACS](https://github.com/TilmanGriesel/graphite).
2. Create a new empty dashboard via ⚙️ and go to **Dashboards**.
3. Click **Add Dashboard** and choose **New dashboard from scratch**
4. Open the new dashboard.
5. Click on the 3 dots → **Edit Dashboard**.
6. Click on the 3 dots → **Raw configuration editor**.
7. Paste the YAML from [dashboard_global.yaml](./Global%20(EN)%20Integration/dashboard_global.yaml).
8. Save, the dashboard is ready to use.
9. [Go to the upstream wiki](https://github.com/Gielz1986/Zendure-HA-zenSDK/wiki/Global-%E2%80%90-Available-entities) for an explanation of all entities.

![Preview](Images/Global-Plug-N-Play-Dashboard.gif)

---

## 🔃 (Optional) Integration add-ons

Several users have created additional code to integrate directly with the upstream integration. They work with this fork as long as the entity names are unchanged.

| Extra add-ons | Use case | Information |
|-|-|-|
| [Node-RED proxy](https://github.com/gast777/Zendure-zenSDK-proxy) from [@gast777](https://github.com/gast777) | Multiple inverters in use | Then you can expand this with this Node-RED proxy. This proxy ensures everything in this automation works seamlessly together, allowing multiple identical inverters to be controlled intelligently with optimal power distribution.  |
| [Node-RED proxy (Hyper 2000)](https://github.com/twoenter/Zendure-zenSDK-Hyper-proxy) from [@twoenter](https://github.com/twoenter) | SolarFlow Hyper 2000 user | Then you can use this add-on to connect Zendure Home Assistant (MQTT/Cloud) through a Node-RED proxy and gain access to all the features of this integration. |

---

## 🙏 Credits

- **[Michiel Hofker (Gielz1986)](https://github.com/Gielz1986)**: author of the original
  [Zendure-HA-zenSDK](https://github.com/Gielz1986/Zendure-HA-zenSDK). The package, automation, dashboard,
  wiki and device images are his work. If this fork is useful to you, star and support the original project.
- **[@gast777](https://github.com/gast777)** and **[@twoenter](https://github.com/twoenter)**: Node-RED proxies for multi-inverter and Hyper 2000 setups.
- **[Martien de Kleijn (iMartien)](https://github.com/iMartien)**: this fork, its preferences and the agent-assisted maintenance.

Issues and pull requests for this fork go to
[iMartien/MartienZ-Zendure-HA-zenSDK](https://github.com/iMartien/MartienZ-Zendure-HA-zenSDK/issues).
Bugs in the underlying logic that also exist upstream are better reported to the
[original repository](https://github.com/Gielz1986/Zendure-HA-zenSDK/issues).

## 📄 License

The original software is licensed for **personal, non-commercial use only** by its author. This fork is
distributed under the same terms. See [LICENSE](./LICENSE) for the full text. The
[Modifications from upstream](#-modifications-from-upstream) section above is the statement of changes
that license requires.
