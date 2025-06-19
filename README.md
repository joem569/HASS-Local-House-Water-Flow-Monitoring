# HASS-Local-House-Water-Flow-Monitoring

This setup works for me to provide the sensors and entities into Home Assistant that let you monitor your whole house water usage using ESPHome and Node-RED. It is HEAVILY based on J-Pipe's Whole-House-Water-Monitoring setup. It was broken recently, and I couldn't update the WIFI credentials, so I had to try to figure out how to update it. I'm still using the unchanged hardware that he cited. I'll link the Amazon listings for what I bought below.


# Parts List

[ESP Board - $14.19 USD](https://www.amazon.com/dp/B081PX9YFV?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_1&th=1)

[Water Meter - $76.00 USD](https://www.amazon.com/dp/B01DAZOQO2?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_17&th=1)


# Hardware Setup

The water meter pulses when water flows through. So it was very simple to hook it up to the ESP board to get it to read. I attached the water meter between pins D3 and GND. I used a connector to make it more modular, but you can solder it directly on.

![Picture](https://github.com/user-attachments/assets/ca90dd0a-937d-47ac-80f3-f1b67b773c0d)

Otherwise, it's just a matter of plumbing as you'd normally do. My motivator for this was to monitor how much water flowed through my whole house water filter, so I plumbed it after the PRV, but before the filter setup.

I'm powering my ESP with a USB and wall plug. It wasn't worth it to me to come up with something better.


# Software Setup

So I'm going to assume that you already have a running Home Assistant setup, with the ESPHome Builder, Node-RED, and Studio Code Server (or any code editor) addons already setup. 

This is how I have my cards set up, Lovelace code included:

![Screen Shot 2025-06-19 at 2 36 23 PM](https://github.com/user-attachments/assets/6b02f86f-da57-4b71-b9f0-e52bbb586bde)

## ESPHome

I'm not going to lie and say that I DIDN'T just set up my secrets properly in ESPHome right before typing this writeup lol, but you should be using the secrets if you're not already. My code uses it, so you'll have to figure that out. You probably have to name your ESP "water-meter" to make my code copy-paste, but if you name it something else, you'll have to figure out what to change. 


Make sure that you update the pin in the code to be the pin that you actually used.


And if you're using a different water meter than I used, you have to find out one important piece of information for the meter: the pulses per gallon. In the code, you have to update the multiply line to reflect the pulses per gallon. The meter I used, it was 1 pulse per gallon. Easy.


When done, this should expose a few things to the ESPHome addon (and HASS). It should be a "Water Flow Rate," and "Total Water Used," sensor, and a "Water Running," binary sensor.

## Home Assistant

There's a bit more to the HASS setup. You'll need something that you can edit files in HASS installed. I'm using the Studio Code Server addon, but I'm sure that there are others.

You need to copy-paste the HASS Config Code into your main Configuration.yaml file. 

I have my Home Assistant configuration [split](https://www.home-assistant.io/docs/configuration/splitting_configuration/), I just find it easier to work with this way. If yours is not split, maybe have something like ChatGPT help you integrate this code.

You'll need to copy-paste the HASS Template Code into your templates.yaml file.

When that's all done, go to the Developer Tools and check your configuration. Hopefully we didn't mess things up and you get the green check. Restart HASS, a full restart is required for the input_numbers and sensors to load properly.

## Node-RED

Open Node-RED, go to your hamburger menu on the top right, click Import, then paste in the Node-RED.json code. You'll PROBABLY have to update things like the server, any changed entity names, and the device(s) you want to notify. This might be the most finicky part for you. I love Node-RED, but I spend a ton of time troubleshooting my stuff lol.

![Screen Shot 2025-06-19 at 2 38 53 PM](https://github.com/user-attachments/assets/c12d44c7-bf1e-4b1c-9e94-fcd235fd7538)

When it looks like there aren't any more issues, deploy it. Hopefully it worked.

## Lovelace Card

Pick your dashboard, make a new card, any card, then go to the code editor for that card and paste the code from the HASS Card Setup. Make sure the entity names are right. 


# Pulling it all Together

So what you now have is a card that monitors the water measured by the water meter. The Node-RED stuff basically makes sure that:
- If the ESP power cycles, HASS remembers and reloads the previous meter readings.
- Gives you a box and to put in your current meter reading. Type that in and hit the "Update Total," button. That does a little math behind the scenes and makes the values read right in HASS.
- Sends alerts when the water is running for too long (left over code from J-Pipe, thanks!)
- Sends alerts when the water is running when it's late at night (left over code from J-Pipe, thanks!)
- Sends alerts when nobody is home (left over code from J-Pipe, thanks!)
- Sends an alert when the water used goes over the defined threshold, I used this for the water filter setup.


If you want, you can disable the alerts in Node-RED, or go into the "When sleeping, or no one is home, Send notification," node and change the times and notification destinations.
