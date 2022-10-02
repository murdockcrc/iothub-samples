# Notes

# IoT Edge architecture

https://learn.microsoft.com/en-us/training/modules/examine-azure-iot-edge-environment/3-examine-iot-edge-runtime

## Edge hub

Acts as a proxy for IoT Hub. For clients, this service looks like it's a real IoT Hub.

Supports only clients on AMQP or MQTT.

Some operations are proxied to the real IoTH and they are not possible when offline. Example: on a device's first connection, the authenticatoin is delegated to the cloud. Only when this first authentication is successful does the Edge hub cache these credentials and allows connectivity for that device when offline.

## Edge agent

Manages the modules to instantiate and monitor. Executes configuration based on the device twin and its deployment manifest.

It is possible to deploy the same module multiple times to the same device, they need different names. Example:

```
Device: /devices/Car1

Modules: /devices/Car1/modules/filter1
Modules: /devices/Car1/modules/filter2
```

### Desired properties

The following can be configured for the edge agent:
https://learn.microsoft.com/en-us/training/modules/examine-azure-iot-edge-environment/5-module-twin-properties-of-edge-runtime-modules

### Reported properties

These can be queries using the IoT Hub query language.

### Manifest

Possible configuration values are:

* settings.image – The container image that the IoT Edge agent uses to start the module. The IoT Edge agent must be configured with credentials for the container registry if the image is protected by a password. Credentials for the container registry can be configured remotely using the deployment manifest, or on the IoT Edge device itself by updating the config.yaml file in the IoT Edge program folder.
* settings.createOptions – A string that is passed directly to the Moby container daemon when starting a module’s container. Adding options in this property allows for advanced configurations like port forwarding or mounting volumes into a module’s container.
* status – The state in which the IoT Edge agent places the module. Usually, this value is set to running as most people want the IoT Edge agent to immediately start all modules on the device. However, you could specify the initial state of a module to be stopped and wait for a future time to tell the IoT Edge agent to start a module. The IoT Edge agent reports the status of each module back to the cloud in the reported properties. A difference between the desired property and the reported property is an indicator of a misbehaving device. The supported statuses are:
  * Downloading
  * Running
  * Unhealthy
  * Failed
  * Stopped
* restartPolicy – How the IoT Edge agent restarts a module. Possible values include:
  * Never – The IoT Edge agent never restarts the module.
  * On-failure - If the module crashes, the IoT Edge agent restarts it. If the module shuts down cleanly, the IoT Edge agent does not restart it.
  * On-unhealthy - If the module crashes or is considered unhealthy, the IoT Edge agent restarts it.
  * Always - If the module crashes, is considered unhealthy, or shuts down in any way, the IoT Edge agent restarts it.
* imagePullPolicy - Whether the IoT Edge agent attempts to pull the latest image for a module automatically or not. If you don't specify a value, the default is onCreate. Possible values include:
  * On-create - When starting a module or updating a module based on a new deployment manifest, the IoT Edge agent will attempt to pull the module image from the container registry.
  * Never - The IoT Edge agent will never attempt to pull the module image from the container registry. The expectation is that the module image is cached on the device, and any module image updates are made manually or managed by a third-party solution.

## Troubleshooting

  https://learn.microsoft.com/en-us/training/modules/examine-azure-iot-edge-environment/3-examine-iot-edge-runtime

# Tooling

## Azcli extension

`az extension add --name azure-iot`

## Azure IoT Tools packs for vscode

https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools

# Automatic device management

Requires Standard SKU.

Uses a document class called _Configuration_ which defines:

* The target condition which defines which devices in the fleet to manage. Specified on device twin tags or reported props.
* Target content defined the desired props to be added or updated.
* Metrics which define the counts for various config states, such as Success, In Prgress, Error.

Configurations may conflict with each other if they target the same device and properties. The configuration with the highest priority wins.
