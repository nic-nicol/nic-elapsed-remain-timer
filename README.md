# nic-elapsed-remain-timer
HASS timer: enter start date, set runtime in weeks. Sensors output: elapsed time, % of total time, remaining time in weeks &amp; days. 

Suitable for school/uni term, project management, grow room

![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/ed4e0f50-be5f-4245-bae1-a8297161bed6)

You require 2 helpers for inputing start date and runtime:  
Go to settings > devices & services > helpers > create helper

Choose 'Date and/or time' and select 'date only'
Name it 'demo.set.start.date' (or name it Fred, or anything you like, just substitute it in the code later on)

![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/7dcc7747-635e-4f11-9e36-fee8c3d64331)

Create another: choose 'number' and name it 'demo.set.runtime' 
Set the 'step size' as 1, input field (I'm sure it'll work with slider if you prefer) and 'unit of measurement' to week
    
![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/0ca3184a-cd70-4548-a3e8-08a5d57bd80d)
![image](https://github.com/nic-nicol/nic-elapsed-remain-timer/assets/98160640/20f3029f-dce7-4390-8724-dcfe22ba972c)

Next up is adding this code to your config.yaml, or wherever makes you happy.

