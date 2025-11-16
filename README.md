# ha-sigenergy-card

![solar-card](https://github.com/user-attachments/assets/e986a16b-4170-42cc-bec2-7ad6de78afbf)

Based on the amazing work by [fbradyirl](https://gist.github.com/fbradyirl/08fef90bd11d7bdddf588a56e668d879) and others [here](https://github.com/TypQxQ/Sigenergy-Local-Modbus/discussions/184) with numerous changes/fixes.

> [!NOTE]
> This card needs/assumes the following are installed:
> 1. [Sigenergy Local Modbus integration](https://github.com/TypQxQ/Sigenergy-Local-Modbus)
> 2. [Card Mod](https://github.com/thomasloven/lovelace-card-mod)
> 3. [Button Card](https://github.com/custom-cards/button-card)
> 4. [Mushroom](https://github.com/piitaya/lovelace-mushroom)
> 5. A Solar Forecast Integration like [Solcast](https://github.com/BJReplay/ha-solcast-solar) or [Forecast.solar](https://www.home-assistant.io/integrations/forecast_solar/)

# Instructions

## 1. Enable additional Sigen Plant entities  
- Sigen Plant > Controls > Remote EMS Control Mode
- Sigen Plant > Diagnostic > Rated Energy Capacity

## 2. Create two template helpers:

i. **Sigen Plant Battery usable capacity**
> Name: `Sigen Plant Battery usable capacity`  
> State:  
> ```yaml
> {{ (states('sensor.sigen_plant_battery_state_of_charge')|float(0) / 100 * states('sensor.sigen_plant_rated_energy_capacity')|float(0))|round(2) }}
> ```
> Unit: `kWh`  
> Device Class: `Energy`  
> State Class: `Total`  
> Device: `Sigen Plant` (optional)
> Availability template:
> ```yaml
> {{ has_value('sensor.sigen_plant_battery_state_of_charge') and has_value('sensor.sigen_plant_rated_energy_capacity') }}
> ```

ii. **Sigen Plant Battery time remaining**
> Name: `Sigen Plant Battery time remaining`  
> State:  
> ```yaml
> {% set capacity = states('sensor.sigen_plant_rated_energy_capacity')|float(0) %}
> {% set power = states('sensor.sigen_plant_battery_power')|float(0) + 0.001 %}
> {% set remaining_kwh = states('sensor.sigen_plant_battery_usable_capacity')|float(0) %}
> {% if power > 0 %}
>   {% set remaining_kwh = capacity - remaining_kwh %}
> {% endif %}
> {% set hours = (remaining_kwh / (power|abs)) %}
> {% set h = (hours // 1)|int %}
> {% set m = (hours%1 * 60)|int %}
> {% if h < 24 and (h != 0 or (h >= 0 and m > 0)) %}
>   {{ h ~ 'h ' if h > 0 }}{{ m ~ 'm ' if m >= 1 }}until {{ 'charged' if (power > 0) else 'empty' }}
> {% endif %}`
> ```  
> Device: `Sigen Plant` (optional)  
> Availability template:
> ```yaml
> {{ has_value('sensor.solcast_pv_forecast_forecast_today') and has_value('sensor.sigen_plant_daily_load_consumption') and has_value('sensor.sigen_plant_daily_grid_import_energy') }}
> ```

## 3. Adjust Solar forecst entity  
I use [Solcast](https://github.com/BJReplay/ha-solcast-solar); you may need to modify the entity_id to match your solar forecast entity. (Search the card for solcast)

## 4. Copy Images  
Copy the [images](https://github.com/jasonpstokes/ha-sigenergy-card/tree/main/images) into your Home Assistant /config/www/  

## 5. Add the card
In your Dashboard, add a new *manual* card and paste the yaml from [ha-sigenergy-card.yaml](https://github.com/jasonpstokes/ha-sigenergy-card/blob/main/ha-sigenergy-card.yaml)  

</br>
</hr>

> [!NOTE]
> - Reminaing time (to charged/empty) only displays if time < 24 hours
> - the card width is auto (not defined) to fill the UI space, but the elements are arranged on the left to fit my smartphone screen width.

Please let me know if something isn't working.
