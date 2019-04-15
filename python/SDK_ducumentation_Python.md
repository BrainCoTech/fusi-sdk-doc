# Fusi cross-platform SDK draft

        
#### FFT function

`fft_transform(signal)` Calculate the FFT for the input signals


## Examples
### 1. FFT (example_fft.py)

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

### 2.OTA (example_ota.py)  

**Read OTA firmware file**

```
dir_path = os.path.abspath(os.path.dirname(__file__))
firmware_data_path = os.path.join(lib_path, "firmware.ota")
with open(firmware_data_path, mode='rb') as fw_file:
    fw_data = fw_file.read()
```

**Set up an event listener for OTA life-cycles and events**

 Override method `on_firmware_update_status_change` and `on_firmware_update_error`
 
```
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

```
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

```
def on_search_error(error):
    print(error)
    global ota_finished
    ota_finished = True
```

**Start searching for device**

``` 
FusiSDK.search_devices(on_found_devices, on_search_error)
```
### 3.device (example.py)  

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









