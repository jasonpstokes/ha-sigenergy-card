# ha-sigenergy-card

<img width=50% src="https://github.com/user-attachments/assets/fd67844e-4caa-41a0-8236-24834ff41fb3"/>

Based on the amazing work by [fbradyirl](https://gist.github.com/fbradyirl/08fef90bd11d7bdddf588a56e668d879) and others [here](https://github.com/TypQxQ/Sigenergy-Local-Modbus/discussions/184) with numerous changes/fixes - including sole use of custom-button-card (no mushroom legacy template).  
</br>  

> [!IMPORTANT]
> This card assumes the following are installed (e.g. via HACS):  
> 1. [Sigenergy Local Modbus integration](https://github.com/TypQxQ/Sigenergy-Local-Modbus)  
> 2. [Button Card](https://github.com/custom-cards/button-card)
> 3. [Card-mod](https://github.com/thomasloven/lovelace-card-mod)
> 4. A solar forecast integration like [Solcast](https://github.com/BJReplay/ha-solcast-solar) or [Forecast.solar](https://www.home-assistant.io/integrations/forecast_solar/)  
</br>

# Instructions

## 1. Enable additional Sigen integration entities  
- Sigen Plant > Controls > Remote EMS Control Mode  
- Sigen Inverter > Diagnostic > Available Battery Charge Energy  

## 2. Create two Helpers: Template > Sensor:

i. **Sigen Plant Battery usable capacity**
> Name: `Sigen Plant Battery usable capacity`  
> State:  
> ```yaml
> {{ (states('sensor.sigen_plant_rated_energy_capacity')|float(0) - states('sensor.sigen_inverter_available_battery_charge_energy')|float(0))|round(2) }}
> ```
> Unit: `kWh`  
> Device Class: `Energy`  
> State Class: `Total`  
> Device: `Sigen Plant` (optional)  
> Availability template:
> ```yaml
> {{ has_value('sensor.sigen_plant_rated_energy_capacity') and has_value('sensor.sigen_inverter_available_battery_charge_energy') }}
> ```
</br>  

ii. **Sigen Plant Battery time remaining (until backup/reserve reached)**
> Name: `Sigen Plant Battery time remaining`  
> State:  
> ```yaml
> {% set capacity = states('sensor.sigen_plant_rated_energy_capacity')|float(0) %}
> {% set power = states('sensor.sigen_plant_battery_power')|float(0) %}
> {% set usable = states('sensor.sigen_plant_battery_usable_capacity')|float(0) %}
> {% set is_on_grid = is_state('sensor.sigen_plant_grid_connection_status', 'On Grid') %}
>   
> {# Set reserve energy to 0 if off-grid, otherwise use the backup SOC #}
> {% if is_on_grid %}
> >   {% set reserve_pct = states('number.sigen_plant_ess_backup_state_of_charge')|float(0) %}
>   {% set reserve_energy = capacity * (reserve_pct / 100) %}
> {% else %}
>   {% set reserve_energy = 0 %}
> {% endif %}
> 
> {# Adjust usable energy based on discharging vs charging #}
> {% set remaining = (capacity - usable) if power >= 0 else (usable - reserve_energy) %}
> 
> {% if remaining > 0 and power != 0 %}
>   {% set t = remaining / power|abs %}
>   {% set h = t|int %}
>   {% set m = ((t - h) * 60)|int %}
>   {{ '24h+' if h >= 24 else (h ~ 'h ' if h > 0 else '') ~ (m ~ 'm' if m > 0) }}
>   until {{ 'charged' if power > 0 else ('reserve' if is_on_grid else 'empty') }}
> {% endif %}
> ```  
> Device: `Sigen Plant` (optional)  
> Availability template:
> ```yaml
> {{ has_value('sensor.sigen_plant_rated_energy_capacity') and has_value('sensor.sigen_plant_battery_usable_capacity') and has_value('sensor.sigen_plant_battery_power') }}
> ```

## 3. Copy Images  
Copy the [images](https://github.com/jasonpstokes/ha-sigenergy-card/tree/main/www) into your Home Assistant /config/www/  

## 4. Add the card
In your Dashboard, add a new *manual* card and paste the yaml from [ha-sigenergy-card.yaml](https://github.com/jasonpstokes/ha-sigenergy-card/blob/main/ha-sigenergy-card.yaml)  
Note: Yep, it would be simpler to use HA templates for the data, but left it as one file for easy copy and paste.

## 5. Adjust Solar forecst entity  
THis will work for [Solcast](https://github.com/BJReplay/ha-solcast-solar); modify the entity_id to match a different solar forecast entity. (Search the card for solcast)  
</br>  
</hr>  

> [!NOTE]
> - Reminaing time (to charged/empty) only displays if time < 24 hours
> - the card width is auto (not defined) to fill the UI space, but the elements are arranged on the left to fit my smartphone screen width.

Please let me know if something isn't working.
