# SerialEM Scripts & Documentation

[SerialEM](https://bio3d.colorado.edu/SerialEM/) enables robust acquisition of tomographic tilt series, large area montages, single-particle and images for single‑particle reconstruction of macromolecules. Its flexible automation framework, driven by predictive specimen‑positioning algorithms, supports high‑throughput workflows across diverse TEM applications, offering both speed and reliability in demanding data‑collection environments.

This repository contains a collection of scripts, code snippets, and documentation to support and enhance the integration of [Direct Electron](https://directelectron.com/) (DE) cameras with SerialEM. These resources are intended to help users streamline setup, automate workflows, and customize their data acquisition processes. Contributions, feedback, and improvements from the community are welcome.

This README.md file contains information about:

* [Setup](#setup)
  * [Installation and calibration](#installation-and-calibration)
  * [SerialEMproperties camera entries](#serialemproperties-camera-entries)
* [Script commands unique to DE cameras](#script-commands-unique-to-de-cameras)
  * [Saving acquisitions](#saving-acquisitions)
  * [Frame rate](#frame-rate)
  * [Dark reference](#dark-reference)
  * [Getting DE-MC properties](#getting-de-mc-properties)
  * [Setting DE-MC properties](#setting-de-mc-properties)
  * [Sensor maintenance cycle](#sensor-maintenance-cycle)
* [SerialEM scripts](#serialem-scripts)
* [Additional resouces](#additional-resources)

## Setup

Unless otherwise noted, the features described in this repository require **DE Mission Control (DE-MC) 2.7.4.11762** and **SerialEM 4.3.0beta8 or later**.

If you need to update DE-MC, please [contact us](https://directelectron.com/support/).

### Installation and calibration

For the best performance, SerialEM should be installed on the Direct Electron computer.

To install and setup SerialEM:
1. Download and extract a framework (containing SerialEMproperties.txt and SerialEMsettings.txt, specific for the particular microscope you are using) into C:\ProgramData\SerialEM. See the first part of the [SerialEM Download & Installation Page](https://bio3d.colorado.edu/SerialEM/download.html) for information about how to obtain a suitable framework.
2. Download SerialEM from the [SerialEM Download & Installation Page](https://bio3d.colorado.edu/SerialEM/download.html).
3. Right-click the downloaded file and choose "Run as administator". Use the default location for extracting the archive, so that it will extract to C:\Program Files\SerialEM\SerialEM_x-x-x.
4. Open a Command Prompt as an administrator. Change the directory to the newly extracted folder (i.e., `cd C:\Program Files\SerialEM\SerialEM_x-x-x`). Then run `INSTALL.bat`. Ensure no errors are shown on the screen.
5. Follow any instructions shown on the screen related to copying files to other computers (e.g., copying FEI-SEMserver.exe and SEM-AutoIT.exe to a Thermo Fisher microscope computer).
6. Edit the C:\ProgramData\SerialEM\SerialEMproperties.txt file to set the microscope IP address (e.g., `FEISEMServerIP` for Thermo Fisher microscopes or `SocketServerIPif64` for JEOL microscopes) and to add camera specific information (see below).
7. Open SerialEM, test, and perform calibrations.

Chen Xu has a helpful [Step-by-Step Guide for Installation and Calibration](https://sphinx-emdocs.readthedocs.io/en/latest/serialEM-note-install-and-calib.html) that provides more details on the basic steps outlined above.

For detailed information about SerialEM setup and calibration, see the detailed [Setting Up SerialEM](https://bio3d.colorado.edu/SerialEM/hlp/html/setting_up_serialem.htm) page.

Another helpful resource is the detailed list of [all SerialEMproperties.txt file entries](https://bio3d.colorado.edu/SerialEM/hlp/html/about_properties.htm).

### SerialEMproperties camera entries

Near the top of the SerialEMproperties.txt file (outside of a CameraProperties section), include `UseAPI2ForDE 1`.

If you want to prevent auto-retraction of the DE camera when SerialEM closes, include `DebugOutput W`.

Copy the CameraProperties section for the appropriate DE camera. If you have a DE camera model not listed below, please [contact us](https://directelectron.com/support/).

- [Apollo, ApolloXS, or Artemis](CameraProperties/CameraProperties_Apollo.txt)
- [Celeritas or CeleritasXS](CameraProperties/CameraProperties_Celeritas.txt)
- [Centuri or CenturiPlus](CameraProperties/CameraProperties_Centuri.txt)
- [DE-16](CameraProperties/CameraProperties_DE16.txt)
- [DE-64](CameraProperties/CameraProperties_DE64.txt)

## Script commands unique to DE cameras

For more information, see the full list of [SerialEM script commands](https://bio3d.colorado.edu/SerialEM/hlp/html/script_commands.htm).

### Saving acquisitions

To control whether acquisitions are saved by DE-MC, use:

`SetDoseFracParams set 1 #S`

where `#S` is the sum of 4 to save final images, 16 to save summed frames, and 32 to save single (unsummed) frames.

To set the path for saving acquisitions with DE-MC, use:

`SetFolderForFrames dir`

where `dir` is an absolute path on the DE computer (assuming that SerialEM is installed on the DE computer). This should be on the D: drive, which is designed to be fast enough for real-time storage during acquisition.

To get the path for saving acquisitions with DE-MC, use:

`ReportFrameSavingPath`

which sets `reportedValue1` to the path, or to NONE if it is not defined. Note that this only works correctly if SerialEM is running on the DE computer.

### Frame rate

To set the hardware frame rate of the camera, use:

`SetDECamFrameRate #`

where `#` is the desired frame rate of the camera. For most situations, we recommend setting the frame rate in the Camera Properties dialog box and NOT controlling this in a script command.

### Dark reference

To acquire a new dark reference, use:

`NewDEserverDarkRef #P #H`

where `#P` is 0 for linear mode, 1 for counting mode, or -1 for the mode selected in the current Record parameters, and `#H` is the number of hours to wait between dark reference acquisitions. If `#H` is positive, a new dark reference is only acquired if the number of hours since the last dark reference acquisition exceeds `#H`. If `#H` is 0, the command executes immediately and unconditionally. If `#H` is negative (e.g., -1), the command simply records the current time as the time of the last dark reference.

Note that dark references are NOT applicable for Direct Electron's event-based counting cameras, including Apollo, ApolloXS, Artemis, Centuri, and CenturiPlus.

### Getting DE-MC properties

To get the value of a DE-MC property, use:

`GetDEServerProperty text`

where `text` is the property name from DE-MC. This command sets `reportedValue1` to 0 on success or 1 on failure or for no returned value, and `reportedValue2` to the property value.

For example, to get the value of the "Sensor Threshold" property, you may use:

```
propName @= Sensor Threshold
GetDEServerProperty $propName
propValue = $reportedValue2
if $reportedValue1 > 0
    echo $propName is not valid
else
    echo $propName value is $propValue
endif
```

### Setting DE-MC properties

To set the value of a DE-MC property, use:

`SetDEServerProperty var text`

where `var` is the name of the SerialEM variable containing the property name from DE-MC, and `text` is new value to set this property to. Sets `reportedValue1` to 0 on success or 1 on failure, and `reportedValue2` to 0.

For example, to set the value of the "Sensor Threshold" property, you may use:

```
propName @= Sensor Threshold
propValueToSet = 120
SetDEServerProperty propName $propValueToSet
if $reportedValue1 > 0
    echo $propName is not valid
else
    GetDEServerProperty $propName
    echo $propName changed to $reportedValue2
endif
```

### Sensor maintenance cycle

To perform a sensor maintenance cycle (heating the sensor for a period of time in order reduce noise during long automated data acquisition sessions), use:

`LongOperation SM #H`

where `#H` is the number of hours to wait between sensor maintenance cycles. If `#H` is positive, the sensor maintenance cycle is only executed if the number of hours since the last sensor maintenance cycle exceeds `#H`. If `#H` is 0, the command executes immediately and unconditionally. If `#H` is negative (e.g., -1), the command simply records the current time as the time of the last sensor maintenance cycle.

SerialEM's maximum waiting time for completion of the sensor maintenance cycle is controlled by the value of the "Sensor - Maintenance Cycle Duration (s)" property in DE-MC when SerialEM starts. IMPORTANT: If you modify this property value after SerialEM is already running, it should only be changed to a LOWER value that its value at SerialEM start-up.

## SerialEM scripts

* [DE-Continuous-Rotation-Tomography](Scripts/DE_Continuous_Rotation_Tomography.txt): Bi-directional continuous rotation tomography acquisition, with an optional zero-degree tilt image acquired first.

## Additional resources

The following additional resources may be helpful for SerialEM setup and use:

* The SerialEM Notes at the [UMASS Cryo-EM Docs](https://sphinx-emdocs.readthedocs.io/en/latest/index.html) contain a variety of information, procedures, tips, and scripts.
* Many SerialEM scripts are available at the [SerialEM Script Repository](https://serialemscripts.nexperion.net/) managed by [Nexperion](https://www.nexperion.net/).
