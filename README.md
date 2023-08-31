# nic-elapsed-remain-timer

**HASS timer: **   
**enter start date on calendar**  
**set runtime in weeks** 

Sensors will output:  
**elapsed time**  
**% of total time**   
**remaining time in weeks and days**   


**Use case: school/uni term, project management, grow room, etc**

![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/ed4e0f50-be5f-4245-bae1-a8297161bed6)

First up, you require 2 helpers for inputing start date and runtime:  
Go to settings > devices & services > helpers > create helper

Choose 'Date and/or time' and select 'date only'

Name it 'demo.set.start.date' (or name it Fred, or anything you like, just substitute it in the code later on)

![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/7dcc7747-635e-4f11-9e36-fee8c3d64331)

Create another: choose 'number' and name it 'demo.set.runtime' 

Set the 'step size' as 1, input field (I'm sure it'll work with slider if you prefer) and 'unit of measurement' to week
    
![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/0ca3184a-cd70-4548-a3e8-08a5d57bd80d)
![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/20f3029f-dce7-4390-8724-dcfe22ba972c)


Next up is adding this code to your config.yaml, or wherever makes you happy (such as an !include file to keep your config tidy)

However, I'd suggest testing it in Dev tools first (see below) 


```
          - platform: template
    sensors:
      time_elapsed:
        friendly_name: "demo Time Elapsed 01"
        value_template: >-
          {% set start_date = strptime(states('input_datetime.demo_set_start_date'), '%Y-%m-%d') %}  
          {% set current_date = now().replace(microsecond=0, tzinfo=None) %}
          {% set start_timestamp = as_timestamp(start_date) %}
          {% set current_timestamp = as_timestamp(current_date) %}
          {% set elapsed_seconds = current_timestamp - start_timestamp %}
          {% set elapsed_days = elapsed_seconds // 86400 %}
          {% set elapsed_weeks = elapsed_days // 7 %}
          {{ " " ~ int(elapsed_weeks) ~ " Weeks  " ~ int(elapsed_days % 7) }} Days

      time_remaining:
        friendly_name: "demo Time Remaining 01"
        value_template: >-
          {% set start_date = strptime(states('input_datetime.demo_set_start_date'), '%Y-%m-%d') %} 
          {% set end_date = start_date + timedelta(weeks=states('input_number.demo_set_runtime')|int) %}
          {% set current_date = now().replace(microsecond=0, tzinfo=None) %}
          {% set start_timestamp = as_timestamp(start_date) %}
          {% set end_timestamp = as_timestamp(end_date) %}
          {% set current_timestamp = as_timestamp(current_date) %}
          {% set remaining_seconds = end_timestamp - current_timestamp if current_timestamp < end_timestamp else 0 %}
          {% set remaining_days = remaining_seconds // 86400 %}
          {% set remaining_weeks = remaining_days // 7 %}
          {{ " " ~ int(remaining_weeks) ~ " weeks " ~ int(remaining_days % 7) }} days

      demo_time_elapsed_percent_entity:
        friendly_name: "Demo Time Elapsed Percent Entity"
        unit_of_measurement: "%"
        value_template: >-
          {% set start_date = strptime(states('input_datetime.demo_set_start_date'), '%Y-%m-%d') %}  
          {% set end_date = start_date + timedelta(weeks=states('input_number.demo_set_runtime')|int) %}
          {% set current_date = now().replace(microsecond=0, tzinfo=None) %}
          {% set start_timestamp = as_timestamp(start_date) %}
          {% set end_timestamp = as_timestamp(end_date) %}
          {% set current_timestamp = as_timestamp(current_date) %}
          {% set elapsed_seconds = current_timestamp - start_timestamp %}
          {% set total_seconds = end_timestamp - start_timestamp %}
          {% set progress_percentage = (elapsed_seconds / total_seconds) * 100 %}
          {{ min(progress_percentage, 100) | round(0) }}
```

To test it, go to 'dev tools' > 'template'

In the left hand test window, select all the demo code and replace with the code above.

You should now see the results displayed on the right hand window.

![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/74dc3c0f-6c43-4afe-8750-f84de20735c5)
![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/42065df8-6ed5-4378-98ef-6e69b78dbe8e)


Now, Im not 100% sure what your test results will look like just now. You can see mine show the friendly name of each of the 3 sensors and their values, but you haven't inputted a start date or runtime yet, so don't panic! (yet!)

Go back to your two helpers and imput a date and runtime and back in Dev tools you should see a result now.

the three friendly names are:
"demo Time Elapsed 01"
"demo Time Remaining 01"
"Demo Time Elapsed Percent Entity"

Last up, you want to create a nice frontend. 

I'm not great at frontend - I'm not actually any good at coding either! I wrote this with the help of ChatGPT (but with hours of failures I might add!!) and my new pal Neil Englefield from the FaceBook community (thanks a million dude!) who showed me how to integrate the helper inputs: the first itteration of this involved going right into the yaml to set the peramiters every time.

So, here's a few custon cards I've made, you can pop them into whatever dash works for you.

But really, from here you can use whatever cards suit your style.

You'll need HACS and 'custom button card' installed - there's hundreds of guides out there how to instal those. You also need mushroom cards installed (you should get them anyway, they're the best cards going!) 


Click add card, select 'custom button card', select all and paste this into the card config (overwritting what's there):

ELAPSED:

```type: custom:button-card
entity: sensor.time_elapsed
show_name: true
show_icon: false
name: elapsed
tap_action: none
show_state: true
styles:
  card:
    - filter: opacity(100%)
    - height: 70px
    - font-size: 22px
  name:
    - font-size: 18px
    - color: white
    - theme: slate
```

REMAINING:

```type: custom:button-card
entity: sensor.time_remaining
name: remaining
show_state: true
show_name: true
show_icon: false
tap_action: none
styles:
  card:
    - filter: opacity(100%)
    - height: 74px
    - font-size: 22px
  name:
    - font-size: 18px
    - color: white
    - theme: slate
```

PERCENTAGE (I used a gauge card because I couldn't get a bar card to work the way I liked)
```
type: gauge
entity: sensor.demo_time_elapsed_percent_entity
name: progress
needle: true
severity:
  green: 0
  yellow: 28
  red: 88
```

SET START DATE (clicking this will open the helper dialogue where you can set a date on a calendar)

For these two you need 'custom mushroom entity' card, but if you don't have mushroom, then a plain old button card will do: just set the entity to 'input_datetime.demo_set_start_date' (or Fred, if that's what you called it)


```type: custom:mushroom-entity-card
name: set start date
icon: mdi:calendar-cursor-outline
icon_color: light-blue
entity: input_datetime.demo_set_start_date
hold_action:
  action: none
double_tap_action:
  action: none
  card_mod:
    style: |
      ha-card {
                      
      height: 50px !important;
      width: auto;
      border: 2.5px outset blue
                    }  
```

SET RUNTIME (clicking this will open the helper dialogue where you can set a time period in weeks)

```
type: custom:mushroom-entity-card
                name: set runtime
                icon: mdi:update
                icon_color: light-blue
                entity: input_number.demo_set_runtime
                hold_action:
                  action: none
                double_tap_action:
                  action: none
                card_mod:
                  style: |
                    ha-card {
                      
                      height: 50px !important;
                      width: auto;
                      border: 2.5px outset blue
                    }
```


LIMITATIONS:

You can only set whole weeks.

I did try adding a 'days helper', but got an error and didn't continue, because, to be honest, this is all I need for my use case.
I dunno if using a helper with the 'step' set to 0.1 instead of 1 might give you that functionality?

Anyone is welcome to adapt and improve.

FUTURE DEVELOPMENT:

I'm unlikely to do much more in the short term, I've got what I need, but if I do:
add a 'days helper'

Add an automation/script to copy the start and run times into local calendar, creating an event

Roll it all into one automation blueprint?

I hope this works for you, appologies if it doesn't, this is literally my first post on github and I only learned how to do this yesterday!

Comments are most welcome, I'd love to hear it's helped someone out...

Cheers
Nic 
'

