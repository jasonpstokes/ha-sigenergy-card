# ha-sigenergy-card

<img width=50% src="https://github.com/user-attachments/assets/fd67844e-4caa-41a0-8236-24834ff41fb3"/>

Based on the amazing work by [fbradyirl](https://gist.github.com/fbradyirl/08fef90bd11d7bdddf588a56e668d879) and others [here](https://github.com/TypQxQ/Sigenergy-Local-Modbus/discussions/184) with numerous changes/fixes.  
</br>  

> [!IMPORTANT]
> This card assumes the following are installed (e.g. via HACS):  
> 1. [Sigenergy Local Modbus integration](https://github.com/TypQxQ/Sigenergy-Local-Modbus)  
> 2. [Button Card](https://github.com/custom-cards/button-card)  
> 3. A Solar Forecast Integration like [Solcast](https://github.com/BJReplay/ha-solcast-solar) or [Forecast.solar](https://www.home-assistant.io/integrations/forecast_solar/)  
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

ii. **Sigen Plant Battery time remaining**
> Name: `Sigen Plant Battery time remaining`  
> State:  
> ```yaml
> {% set capacity = states('sensor.sigen_plant_rated_energy_capacity') | float(0) %}
> {% set power = states('sensor.sigen_plant_battery_power') | float(0) %}
> {% set usable = states('sensor.sigen_plant_battery_usable_capacity') | float(0) %}
> {% set remaining = capacity - usable if power >= 0 else usable %}
> {% if remaining > 0 and power != 0 %}
>   {% set t = remaining / power | abs %}
>   {% set h = t | int %}
>   {% set m = ((t - h) * 60) | int %}
>   {{ '24h+' if h >= 24 else (h ~ 'h ' if h > 0 else '') ~ (m ~ 'm' if m > 0) }}
>   until {{ 'charged' if power > 0 else 'empty' }}
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
Note: Yes it would be a lot simpler with templates, but left it as one file for easy copy and paste.

## 5. Adjust Solar forecst entity  
THis will work for [Solcast](https://github.com/BJReplay/ha-solcast-solar); modify the entity_id to match a different solar forecast entity. (Search the card for solcast)  
</br>  
</hr>  

> [!NOTE]
> - Reminaing time (to charged/empty) only displays if time < 24 hours
> - the card width is auto (not defined) to fill the UI space, but the elements are arranged on the left to fit my smartphone screen width.

Please let me know if something isn't working.
