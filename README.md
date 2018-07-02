# Home Automation 
#### ([Project Video](https://youtu.be/rRmA0EZAl3k))

A system to control and monitor home appliances was designed. Appliances can be controlled with Google Assistant and status of appliances can be viewed on [Ubidots](https://app.ubidots.com/accounts/signin/). Hardware: Raspberry Pi 3 Model B. Software: Particle Core for Raspberry Pi, Ubidots and IFTTT.

## System Overview
![](https://github.com/GomesLuis479/HomeAutomation/blob/master/readMe_pics/overview.JPG)

###### Cloud Integration:

Particle cloud service along with Particle core on Raspberry Pi offer the required functionality. Firmware code for appliance control is fed to the Raspberry Pi through Particle core. The firmware code contains functions registered to Particle cloud. These functions control appliances based on arguments passed to it. Since this is a cloud function, it can be called by external sources with appropriate authentication and arguments. The system developed uses IFTTT to link to Google Assistant on a user’s device. Thus, IFTTT allows for commands received from Google Assistant to be passed to the required function on Particle Cloud linked firmware code enabling appliance control.

###### Appliance Status Visualization: 

Ubidots Cloud service is used for status visualization. The firmware code uses Webhooks on Particle Cloud to make REST API calls to Ubidots. Variables are created on the Ubidots Cloud to log appliance statuses. REST calls are made with appropriate authentication tokens, device labels and variable labels. A REST call is made to the required variable every time a device status is changed. The call also contains the current status of the device in a JSON object which is used by Ubidots to update the variable(device status) value. 

## Setup and Usage

### Particle Setup

 - Create an account on [Particle](https://login.particle.io/login) and login
 - [Install](https://docs.particle.io/guide/getting-started/start/raspberry-pi/#install-particle-pi-software) Particle Core on Raspberry Pi
 
 For instance, during installation of Particle on Raspberry Pi, we have given our Raspberry Pi the **device label** T2.
 
 The installed `particle-agent` should start automatically on Raspberry Pi boot. If not, `particle-agent` status can be viewed by running the following command in terminal:
 ```
 particle-agent status
 ```
 If the above command does not show a running instance of `particle-agent` run:
 ```
 particle-agent start
 ```
Login to Particle and navigate to console. The Raspberry Pi should now show up along with the device label provided during installation.
 
 ![](https://github.com/GomesLuis479/HomeAutomation/blob/master/readMe_pics/particle_device.JPG)

### Ubidots Setup
Ubidots cloud service allows for the creation of **Devices** and **Variables**. *Variables* are entities that hold data in a particular *Device*. We will use values of these variables to indicate appliance status.

- Sign-in to [Ubidots](https://app.ubidots.com/accounts/signin/)
- Navigate to Devices tab
- Create a Device
- Create Variables in the device you just created for each appliance

For example, we have created a Device *particledata* that has two Variables *pump* and *fan*.

![](https://github.com/GomesLuis479/HomeAutomation/blob/master/readMe_pics/device_variable.JPG)

### Particle Code
- Login to your particle console
- Open the build environment. This is present at the bottom-left corner of the console window
- The root folder of this repository contains a file named `particle-source-code` 
- Copy this code into the build environment and save it
- Now select the `Flash` option to transfer code to Raspberry Pi

### Code Analysis

###### Physical Pin Connections

```
int fanPin = D2;
int pumpPin = D5;
```
These declare the GPIO pins that control the appliances. Relays are used to drive high power loads. Here is a reference of the Particle pin-out for Raspberry Pi 3 Model B:

<img src="https://github.com/GomesLuis479/HomeAutomation/blob/master/readMe_pics/particle_pin_out.png" alt="drawing" width="500px"/>

###### Controller Function

The code is used to control and view status of a pump and fan. A function `int controller(String command)` is created to offer the required functionalities. This function needs to be registered with Particle so that it may be called by external agents. The following registers the function:
```
 Particle.function("controller", controller);
```
In this case we will use IFTTT to call this function with the appropriate arguments via Google Assistant. More about IFTTT later.

###### WebHooks

WebHooks serve as an interface between Particle and Ubidots. The given code contains two WebHooks: `pumpWebHook` and `fanWebHook`.
These will send data to the Ubidots variables (*pump* and *fan*) on the device (*particledata*) we created earlier. Given below is the code executed when a command to turn ON the fan is issued:
```
void fanOn() {
    digitalWrite(fanPin, HIGH);
    Particle.publish("fanWebHook", "1", PRIVATE);
}
```
This function thus sends data ("1") to Ubidots to indicate that the fan has been turned ON.

### Creating WebHooks

Given below are steps required to setup `fanWebHook`. `pumpWebHook` may be setup similarly with appropriate changes.

- Login to Particle console
- Navigate to *Integrations* in the left pane
- Select new Integration and select Webhook

Enter the following fields:
```
Event Name: fanWebHook

URL: https://things.ubidots.com/api/v1.6/devices/DEVICE_NAME/VARIABLE_NAME/values/?token=TOKEN

Request Format: JSON

DEVICE: T2 (select your device)
```
Under Advanced Settings -> Custom enter:
```
{
  "value": "{{{PARTICLE_EVENT_VALUE}}}"
}
```
Leave the remaining fields blank and select *Create Webhook*.

### URL Details
The URL for Webhooks above has three fields.
https://things.ubidots.com/api/v1.6/devices/DEVICE_NAME/VARIABLE_NAME/values/?token=TOKEN

- DEVICE_NAME
- VARIABLE_NAME
- TOKEN

Here are examples to get the above three fields:

For Example, consider a Device named **lamp_not** and a variable **lamp** in it as shown in the screenshots below:

![fig1](https://github.com/GomesLuis479/LampNot/blob/master/readMe_pics/device.JPG)

---

![fig2](https://github.com/GomesLuis479/LampNot/blob/master/readMe_pics/variable.JPG)

---

**Token** can be found by clicking your username in the upper-right corner and selecting API Credentials option.

![fig3](https://github.com/GomesLuis479/LampNot/blob/master/readMe_pics/token.JPG)

### Linking IFTTT

```
int controller(String command) {
    
    if(command == "fan on") {
        fanOn();
    }
       
    if(command == "fan off") {
        fanOff();
    }
    
    if(command == "pump on") {
        pumpOn();
    }
    
    if(command == "pump off") {
        pumpOff();
    }
    
    return 0;
}
```

The above function has already been registered with Particle. We will call this function over Google Assistant to trigger events.

###### Applet to turn ON the fan:
- Login to [IFTTT](https://ifttt.com/)
- Navigate to *My Applets*
- Select *New Applet*
- Select *+this*
- Search for Google Assistant
- Select *Say a simple phrase*

Enter the following fields:
```
What do you want to say?:
turn on the fan

What do you want the Assistant to say in response?:
ok, fan turned on
```
- Select *Create Trigger*
- On the next screen select *+that*
- Search for Particle
- Select *Call a Function*

Enter the following fields:
```
Then call (Function Name):
controller on "T2" (select your device)

with input (Function Input):
fan on
```
- Select *Finish*

##### Applet to turn OFF the fan:

The procedure is almost identical as above.

In *+if*, enter the following fields:
```
What do you want to say?:
turn off the fan

What do you want the Assistant to say in response?:
ok, fan turned off
```

In *+that*, enter the following fields:
```
Then call (Function Name):
controller on "T2" (select your device)

with input (Function Input):
fan off
```
Follow the same procedure to trigger pump (other appliances) events.

### Creating Visualizations on Ubidots

- Login to Ubidots
- In the Dashboard tab, select *New Widget*
- Select *Indicator*
- Select *On/Off*
- Add the required Variable from the appropriate device to monitor status of

### Finished

That's it. You can now turn on the fan by saying.

*Hey Google, turn ON the fan.*



