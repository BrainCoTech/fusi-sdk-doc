# Fusi cross-platform SDK

## Introduction

Fusi Software Development Kit is a developer toolkit for Focus headband. Developers can use the SDK to develop applications that are suitable for multi-platform applications. You can easily get the user's EEG raw data, brain wave data, user attention data, etc.

The SDK supports a wide range of platforms with high-performance AI algorithms implemented in low-level language, and provides some basic computations while implementing multiple EEG data monitoring functions.

We have designed our technology to enable new forthcoming algorithms to work with our existing Focus headband hardware. Begin your development and familiarity with our API today so that you can take full advantage of the next series of algorithms.

###Features

* Scan available Wi-Fi hotspots
* Configure headband Wi-Fi connection to specific Wi-Fi
* Automatic scanning to find the headband in a Wi-Fi network
* Query the amount of battery power in the headband
* Query the alpha spike frequency of the user's brain power (only when available)
* Monitor the connection status of the application with the headband
* Monitor the rotating posture of the application and the headband
* Monitor user EEG raw voltage data
* Monitor user EEG frequency band energy (Delta, Theta, Alpha, LowBeta, HighBeta, Gamma) 
* Monitor user's current attention rating (BrainCo private AI algorithm)
* Monitor user's blink events

### Core function

* **Monitor user EEG raw voltage data**

	When the user wears the headband and the headband is well connected, the developer will get the EEG data(`EEGData` object) every half second, including the sampling rate and an array of raw EEG data.

	Focus headband has the precision equivalent to medical-standard EEG equipment, and its signal has a strong correlation(97.5%) with Gtek EEG equipment. Developers can collect raw data for scientific research and customized analysis.

* **Monitor user EEG frequency band energy (Delta, Theta, Alpha, LowBeta, HighBeta, Gamma)**

	The developer gets a summary of the frequency domain energy distribution every half second, including the Delta, Theta, Alpha, Low-Beta, High-Beta and Gamma bands.
		
	Developers can use data to make applications related to the EEG frequency domain, such as neurofeedback training applications. 
	
* **Monitor user's current attention rating (BrainCo private AI algorithm)**

	The developer gets an estimated value of the user's current attention every half second, and the APP related to the force, for example, when learning A warning box pops up when the attention is low.

### Highlight features
* Extremely developer-friendly API design

	* adhere to developer-friendly principles
	* better support for new API openness
	* reduce developer learning costs
	* easy to integrate it into your own application projects

* High-performance data processing technology 
	* implemented by C language language 
	* low-level language writing
	* fast running speed
	* high-performance AI algorithm
	* can run in various systems.

* Consistence among all platforms
	* greatly reduce developer learning and switching costs
	* facilitate code building
	* easy for developers to develop applications on different platforms
	* result in greater business value.

### Platform support

Fusi cross-platform SDK is written in C with wrapper libraries. It supports multiple platforms:

* Android AAR: API 14+
* iOS Framework: 9.0+
* C static library: for embedded system and operating system level APP
* NodeJS AddOn: for cross-platform HTML/JS APP
* Python 3.5 binding Python 3.5 binding
* C# .NET standard 2.0: For Windows WPF/Form/UWP APP
* MacOS Framework: 10.0+ (Ver 1.3 in development)
* C# Unity (developing Ver 2.0)

### Data

For the developer who is interested in developing applications that interact with Focus headband, you can acquire sufficient brainwave data using Fusi SDK. Currently, the following outputs are available:

* Raw EEG
* Brainwave bands:
	* delta
	* theta
	* alpha
	* low beta
	* high beta
	* gamma
* Attention
* Meditation
* Eyeblinks

## Python
###1. Installation

Python version: 3.5+

Install requirements:

`pip install -r requirements.txt`

###2. APIs reference

#### Listener
##### class FusiHeadbandListener
The FusiHeadbandListener class is an abstract base class for any objects which wish to register to receive notifications of new messages. Listeners are registered with `Notifier` object(s) which ensure they are notified whenever a new message is received.

Subclasses of Listener that do not override methods in FusiHeadbandListener class will cause NotImplementedError to be thrown when a message is received.


`on_eeg_data(self, eeg_data)`

This method is called to handle EEG data in the receive thread.

`on_attention(self, attention)`

This method is called to handle attention in the receive thread.

`on_brain_wave(self, brain_wave)`

This method is called to handle brave waves in the receive thread.

`on_orientation_change(self, orientation)`

This method is called to handle orientation changes in the receive thread.

`on_error(self, error)`

This method is called to handle errors in the receive thread.

`on_connection_change(self, connection_state)`

This method is called to handle connection changes in the receive thread.

`on_contact_state_change(self, contact_state)`

This method is called to handle contact state changes in the receive thread.

`on_blink(self)`

This method is called to handle blink in the receive thread.

`on_meditation(self, meditation)`

#### Firmware update listener callbacks
`on_firmware_update_status_change(self, status)`

This method is called to handle firmware update status changes in the receive thread.

`on_firmware_update_error(self, error)`

This method is called to handle firmware update errors in the receive thread.


#### Wrapper objects
The classes below create wrapper objects from C library.

##### class FusiHeadbandError

`code`		error code

`message`		error message

##### class EEGData

`sample_rate` 	sample rate

`data`		eeg data

`pga`		pga

##### class Wifi
`ssid`		ssid 

`mac`		mac address
    
`encryption`		encryption methods

##### class BrainWave
`delta`	delta waves

`theta`	theta waves

`alpha`	alpha waves

`low_beta`	low beta waves

`high_beta`	high beta waves

`gamma`	gamma waves

#### Firmware Update Status

##### class FirmwareUpdateStatus
`stage`

`message`

"Firmware update begin"

"Firmware update entering OTA mode"

"Validating new firmware information"

"Transmitting firmware data"

"Unknown OTA status"

`progress`

`error(code)`

"Firmware data error"

"Firmware data unknown error"

"Firmware information error"

"Firmware information unknown error"

"OTA packet parsing error"

"OTA crc validation error"

"OTA parameter error"

"OTA command error"

"OTA mode is disabled"

"OTA update is not allowed under AP mode"

"Battery is too low"

"OTA update connection timeout"

#### class FusiHeadband

FusiHeadbandListener

##### class element:

```
_device_pointer_map = {}
    _device_map = {}
    __connection_state = ConnectionState.disconnected
    __mac = None
    __name = None
    __ip = None
    __impedance_test_enabled = 1
``` 

###### Callback references

`_on_scan_wifi_finished`

`_on_scan_wifi_error = None`

`_on_switch_wifi_finished = None`

###### methods
`get_name(self)`
get name

`get_mac(self)`
get mac address

`get_ip(self)`
get ip

`get_connection_state(self)`
get connectionstate

`get_battery_level(self)`
get battery level

`def get_firmware_info`
get firmware information

`get_firmware_version(self)`
get firmware version

`get_alpha_peak(self)`
get alpha peak value

`get_device_mode(self)`
get device_mode

`is_impedance_test_enabled(self)`
whether impedance test is enabled

`set_impedance_test_enabled(self, enabled)`
set impedance test enabled
           

`is_charging(self):
whether the device is charging

`get_device_orientation(self)`
get device orientation

`get_contact_state(self)`
get contact state

`set_forehead_led_color(self, rgb_color)`
set forehead LED color

`set_listener(self, listener)`

* set attention callback;

* set eeg\_data callback;

* set eeg\_stats callback;

* set device\_oriention\_callback;

* set\_error\_callback;                

* set\_device\_contact\_state\_callback

* set\_blink\_callback

* set\_meditation\_callback


`unset_listener(self)`
Unset listener = `None`

`__get_c_info(self)`
get `mac_fld`, `name_fld`, `ip_fld`

`connect(self)`
connect to the device

`disconnect(self)`
disconnect to the device

`scan_wifi(self, on_finish, on_error)`
scan Wi-Fi

`switch_wifi(self, wifi, password, on_finish)`
switch wifi

`set_station_mode(self, mode, on_finish)`
not implemented yet

`restart(self)`
not implemented yet

`update_firmware(self, data)`
update the firmware

##### class methods
`create_fusi_headband_with_c_info(cls, c_device_info)`
return cls.create_fusi_headband(mac, name, ip)

`create_fusi_headband(cls, mac, name, ip)`
return `device` object

#### override FusiHeadbandListener.on_connection_change
`on_connection_change(self, connection_state)
                
#### class FusiSDK

`dispose()`
dispose fusi sdk
    
```
search_devices(cls, on_finish, on_error)
```
*classmethod* search for devices     
 
        
#### FFT function

`fft_transform(signal)` Calculate the FFT for the input signals


## Examples
###1. FFT (example_fft.py)

Input data:

```
array = [2,1,0,-1,-2,-1,0,1,2,1,0,-1,-2,-1,0,1,2,1,0,-1,-2,-1,0,1,2]
```

Call method **fft_transform** in FusiSDK:

```
fft = fft_transform(array)
```

Output:

```
[0.11821638087705445, 0.1232341319368554, 0.8035190188423144, 0.3271568980997121, 0.046972693121087046, 0.01477784276777906, 0.007021158065102148, 0.017265417598034606, 0.054940709751649296, 0.10673422824584916, 0.061657848075272956, 0.018784028675231573, 0.007262166541151867]
```

###2.OTA (example_ota.py)  

**Read OTA firmware file**

```dir_path = os.path.abspath(os.path.dirname(__file__))
firmware_data_path = os.path.join(lib_path, "firmware.ota")
with open(firmware_data_path, mode='rb') as fw_file:
    fw_data = fw_file.read()
```

**Set up an event listener for OTA life-cycles and events**

 Override method `on_firmware_update_status_change` and `on_firmware_update_error`
 
``` python
class OTAEventListener(FusiHeadbandListener):
	def on_firmware_update_status_change(self, status):
        print("Firmware update status changed:%s:%s" % (status.stage.name, status.message))
        if status.stage == FirmwareUpdateStage.complete:
            global ota_finished
            ota_finished = True

	def on_firmware_update_error(self, error):
        print("Firmware update error:%i:%s" % (error.code, error.message))
        global ota_finished
        ota_finished = True
```

**Look for target device from devices found**

``` python
def on_found_devices(devices):
    global headband
    for device in devices:
        print("Found device: %s" % device.get_mac())
        if device.get_mac() == _TARGET_MAC:
            headband = device
    if headband is None:
        print("Target device not found")
        global ota_finished
        ota_finished = True
    else:
        headband.set_listener(OTAEventListener())
        headband.update_firmware(fw_data)
```

**Specify what to do when search device failed**

``` python
def on_search_error(error):
    print(error)
    global ota_finished
    ota_finished = True
```

**Start searching for device**

``` 
FusiSDK.search_devices(on_found_devices, on_search_error)
```
###3.device (example.py)  

(1) Initialization

```
_TOTAL_RUN_TIME = 360 # seconds
_TARGET_MAC = "d8b04cc83dd8"
headband = None
```

(2) Build a class `MyListener`

```
class MyListener(FusiHeadbandListener):
    def on_connection_change(self, connection_state):
        print("Connection state changed:%s" % connection_state.name)
        if connection_state == ConnectionState.connected:
            print("Headband connected")
        elif connection_state == ConnectionState.interrupted:
            print("Headband connection interrupted")
        elif connection_state == ConnectionState.disconnected:
            print("Headband disconnected")
    def on_attention(self, attention):
        print("Attention: %.3f" % attention)
```

(3) Find devices, set listeners and connect

```
def on_found_devices(devices):
    global headband
    for device in devices:
        if device.get_mac() == _TARGET_MAC:
            headband = device

    if headband is None:
        print("No device found")
    else:
        headband.set_listener(MyListener())
        headband.connect()
```

(4) Handle errors

```
def on_search_error(error):
    print(error)
```

(5) Wait until total run time

`time.sleep(_TOTAL_RUN_TIME)`

(6) Dispose

`FusiSDK.dispose()`









