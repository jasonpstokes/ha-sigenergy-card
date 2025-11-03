# ha-sigenergy-card

![solar-card](https://github.com/user-attachments/assets/e986a16b-4170-42cc-bec2-7ad6de78afbf)

Based on the amazing work by [fbradyirl](https://gist.github.com/fbradyirl/08fef90bd11d7bdddf588a56e668d879) and others [here](https://github.com/TypQxQ/Sigenergy-Local-Modbus/discussions/184), this card mirrors the functionality (with numerous changes/fixes) but removes the house (that looks nothing like mine!).

## 1. Uses the [Sigenergy Local Modbus integration](https://github.com/TypQxQ/Sigenergy-Local-Modbus) default entities  
Plus two (that I remember) extras enabled:
- Sigen Plant > Controls > Remote EMS Control Mode
- Sigen Plant > Diagnostic > Rated Energy Capacity

## 2. Create Battery Usable Capacity template helper:
  
*Name*: Sigen Plant Battery Usable Capacity    (for entity_id sensor.sigen_plant_battery_usable_capacity)  
*State*:
```yaml
{{ (states('sensor.sigen_plant_battery_state_of_charge')|float(0) / 100 * states('sensor.sigen_plant_rated_energy_capacity')|float(0))|round(2) }}
```  
*Unit*: kWh  
*Device Class*: Energy  
*State Class*: Total  
*Device*: (optional) Sigen Plant  

## 3. Solar (PV) forecst 
I use [Solcast](https://github.com/BJReplay/ha-solcast-solar) - you'll likely need to modify the entity_id to your solar forecast entity.

## 4. Images  
Save/copy the images into your Home Assistant /config/www/  

- I've created the gateway image, but the  battery one is a vector converstion that needs to be fixed! (I might, it only annoys me a little though)

## 5. In the UI, add a new *manual* card and paste the yaml from *ha-sigenergy-card.yaml*  

</hr>

> [!NOTE]
> - Reminaing time (to charged/empty) only displays if time < 24 hours
> - the card width is auto (not defined) to fill the UI space, but the elements are arranged on the left to fit my smartphone screen width.

Please let me know if something isn't working.
