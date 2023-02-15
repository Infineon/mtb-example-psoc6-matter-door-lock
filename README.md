# Matter PSoC&trade; 6 door dock example

This code example makes use of Matter on an Infineon PSoC&trade; 6 board to demonstrate its functionality as a Wi-Fi door lock device.

<a name="requirements"></a>

## Requirements

- ModusToolbox&trade; software(https://www.infineon.com/modustoolbox) v3.0 or later (tested with v3.0)
- PSoC&trade; 6 board with 2 user buttons, 2 MB of Flash, and two user LEDs
- PSoC&trade; 6 board support package (BSP) minimum required version: 4.0.0
- Programming language: C/C++
- Associated parts: All PSoC&trade; 6 MCU(https://www.infineon.com/PSoC6) parts

<a name="supported toolchains"></a>

## Supported toolchains (make variable 'TOOLCHAIN')

- GNU Arm&reg; embedded compiler v10.3.1 (`GCC_ARM`) - Default value of `TOOLCHAIN`

<a name="supported kits"></a>

## Supported kits (make variable 'TARGET')

[CY8CKIT-062S2-43012](https://www.infineon.com/CY8CKIT-062S2-43012)

<a name="software setup"></a>

## Software setup

-   [ModusToolbox&trade; tools package]( https://www.infineon.com/cms/en/design-support/tools/sdk/modustoolbox-software/)

<a name="hardware setup"></a>

## Hardware setup

-   The board should be in KITPROG3 mode.
-   Check for the LED in the ON state (not blinking)
-   Connect the USB device to the host.

<a name="Using the CE"></a>

## Using the CE

The Project can be created using any one of the following ways:

### Using Eclipse IDE for ModusToolbox&trade; Follow the steps given here:
1. **Quick Panel** --> **New Application**

2. In the Project Creator Tool, choose a kit that is supported by the code example  **Project Creator** --> **Board Support Package (BSP)** dialog.

3. Click on the check box containing the example under **Select Application**

4. Click **Create**.

### Using the CLI

   1. Open the  Project Creator from the Windows Start menu

   2. You could also choose  *{ModusToolbox&trade; software install directory}/tools_{version}/project-creator/project-creator.exe*.

   3. Choose the required BSP, and click **Next**.

   3. Select the appropriate IDE from the **Target IDE** drop-down menu under the **Select Application**

   4. Click **Create**

   5. Export the application to the desired IDE. Refer to the ModusToolbox&trade; tools package user guide, "Using applications with third-party tools" chapter for details.
<a name="Setting up chip-tool"></a>

## Setting up chip-tool

Once the application is running, set up chip-tool on Raspberry Pi 4 to perform commissioning and cluster control.

-   Set up python controller.

           $ cd {path-to-connectedhomeip}
           $ ./scripts/examples/gn_build_example.sh examples/chip-tool out/debug

-   Execute the controller.

           $ ./out/debug/chip-tool

## Commissioning over Bluetooth&reg; LE

Run the built executable and pass it the discriminator and pairing code of the remote device, as well as the network credentials to use.

         $ ./out/debug/chip-tool pairing ble-wifi 1234 ${SSID} ${PASSWORD} 20202021 3840

         Parameters:
         1. Discriminator: 3840
         2. Setup-pin-code: 20202021
         3. Node ID: 1234 (you can assign any node id)
         4. SSID : Wi-Fi SSID
         5. PASSWORD : Wi-Fi Password

<a name="Notes"></a>

## Notes

Raspberry Pi 4 Bluetooth&reg; LE connection issues can be avoided by running the following commands. These power cycle the Bluetooth&reg; hardware and disable BR/EDR mode.

          $ sudo btmgmt -i hci0 power off
          $ sudo btmgmt -i hci0 bredr off
          $ sudo btmgmt -i hci0 power on

<a name="Cluster control"></a>

## Cluster control

-   After successful commissioning, use the Lock Unlock cluster command to toggle     device between Lock or Unlock states.

    `$ ./out/debug/chip-tool door-lock lock 1234 1`

    `$ ./out/debug/chip-tool door-lock unlock 1234 1`

-   Cluster Lock Unlock can also be done using the `USER_BTN1` button on the board.
    This button is configured with `APP_LOCK_BUTTON` in `include/AppConfig.h`.
    Press `USER_BTN1` on the board to toggle between lock and unlock states. The     Lock/Unlock status of the door lock can be observed with 'Red LED' on the board. This     LED is configured with `LOCK_STATE_LED` in `include/AppConfig.h`.

<a name="OTA Software Update"></a>

## OTA software update
-  Over The Air (OTA) support can be included in the Wi-Fi door lock application by following either of these routes:

### Using the Infineon OTA middleware library
The OTA library provides support for Over-The-Air update of the application code running on an Infineon device. The ModusToolbox OTA code examples import the ota-update library automatically. This application is built in ModusToolbox&trade; software, integrated with the OTA library.

For more information refer - [OTA_using_ModusToolbox] https://github.com/Infineon/ota-update

### Using MATTER (project-chip repository)

Matter provides a system for OTA to be run as a provider requestor.

For further information, head to [Matter] https://github.com/project-chip/

<a name="PSoC&trade; 6 Deep Sleep"></a>

## PSoC&trade; 6 Deep Sleep

- PSoC&trade; 6 can put to Deep Sleep for longer period of time by suspending Network stack and enabling WLAN Offload features using [LPA](https://github.com/Infineon/lpa).

- Refer to LPA Quick Start Guide - [Code Snippets](https://infineon.github.io/lpa/lpa_api_reference_manual/html/group__lpautilities.html) for more information on APIs to be used.

- Network stack suspend/resume logic should be added after Wi-Fi Connection is successfully established. This can be done in `AppTask::Init` function after starting `DnssdServer`.

```
    if (event->InternetConnectivityChange.IPv4 == kConnectivity_Established ||
        event->InternetConnectivityChange.IPv6 == kConnectivity_Established)
    {
        chip::app::DnssdServer::Instance().StartServer();
        //Create new task which calls wait_net_suspend API implemented in LPA
    }
```

- Configure Various WLAN offloads using the ModusToolbox&trade; Device Configurator. Refer to LPA [Quick Start Guide](https://infineon.github.io/lpa/lpa_api_reference_manual/html/index.html#section_lpa_getting_started)
## Design and Implementation

**Table 1. Application resources**

 Resource  |  Alias/object     |    Purpose
 :-------- | :-------------    | :------------
 UART (HAL)    |cy_retarget_io_uart_obj| UART HAL object used by retarget-io for the Debug UART port
 GPIO (HAL)    | CYBSP_USER_LED        | User LED
 GPIO (HAL)    | CYBSP_USER_BTN        | User button

<br />

## Related resources

Resources  | Links
-----------|----------------------------------
Application notes  | [AN228571](https://www.infineon.com/AN228571) - Getting started with PSoC&trade; 6 MCU on ModusToolbox&trade; software <br />  [AN215656](https://www.infineon.com/AN215656) - PSoC&trade; 6 MCU: Dual-CPU system design <br />  [AN234334](https://www.infineon.com/AN234334) - Getting started with XMC7000 MCU on ModusToolbox&trade; software
Code examples  | [Using ModusToolbox&trade; software](https://github.com/Infineon/Code-Examples-for-ModusToolbox-Software) on GitHub
Device documentation | [PSoC&trade; 6 MCU device website]( https://www.infineon.com/cms/en/product/microcontroller/32-bit-psoc-arm-cortex-microcontroller/psoc-6-32-bit-arm-cortex-m4-mcu/?term=PSoC%206&view=kwr&intc=searchkwr#!documents)
Development kits | Visithttps://www.infineon.com/cms/en/design-support/tools/sdk/modustoolbox-software/#!designsupport.
Libraries on GitHub  | [mtb-pdl-cat1](https://github.com/Infineon/mtb-pdl-cat1) - Peripheral driver library (PDL)  <br> [mtb-hal-cat1](https://github.com/Infineon/mtb-hal-cat1) - Hardware abstraction layer (HAL) library <br> [retarget-io](https://github.com/Infineon/retarget-io) - Utility library to retarget STDIO messages to a UART port
Middleware on GitHub  | [capsense](https://github.com/Infineon/capsense) - CAPSENSE&trade; library and documents <br> [psoc6-middleware](https://github.com/Infineon/modustoolbox-software) - Links to all ModusToolbox&trade; middleware
Tools  | [ModusToolbox&trade; tools package]( https://www.infineon.com/cms/en/design-support/tools/sdk/modustoolbox-software/) - ModusToolbox&trade; software is a collection of easy-to-use software and tools enabling rapid development with Infineon MCUs, covering applications from embedded sense and control to wireless and cloud-connected systems using AIROC&trade; Wi-Fi and Bluetooth&reg; connectivity devices.
<br>

## Other resources


Infineon provides a wealth of data at www.infineon.com to help you select the right device, and quickly and effectively integrate it into your design.

For PSoC&trade; 6 MCU devices, see [How to design with PSoC&trade; 6 MCU - KBA223067](https://community.infineon.com/t5/Knowledge-Base-Articles/How-to-Design-with-PSoC-6-MCU-KBA223067/ta-p/248857) in the Cypress community.


## Document history


Document title: *CE236478* - *Wi-Fi door lock*

| Version | Description of change |
| ------- | --------------------- |
| 1.0.0   | New code example for Wi-Fi door lock|


------
<br />

© Cypress Semiconductor Corporation, 2020-2023. This document is the property of Cypress Semiconductor Corporation, an Infineon Technologies company, and its affiliates ("Cypress").  This document, including any software or firmware included or referenced in this document ("Software"), is owned by Cypress under the intellectual property laws and treaties of the United States and other countries worldwide.  Cypress reserves all rights under such laws and treaties and does not, except as specifically stated in this paragraph, grant any license under its patents, copyrights, trademarks, or other intellectual property rights.  If the Software is not accompanied by a license agreement and you do not otherwise have a written agreement with Cypress governing the use of the Software, then Cypress hereby grants you a personal, non-exclusive, nontransferable license (without the right to sublicense) (1) under its copyright rights in the Software (a) for Software provided in source code form, to modify and reproduce the Software solely for use with Cypress hardware products, only internally within your organization, and (b) to distribute the Software in binary code form externally to end users (either directly or indirectly through resellers and distributors), solely for use on Cypress hardware product units, and (2) under those claims of Cypress’s patents that are infringed by the Software (as provided by Cypress, unmodified) to make, use, distribute, and import the Software solely for use with Cypress hardware products.  Any other use, reproduction, modification, translation, or compilation of the Software is prohibited.
<br />
TO THE EXTENT PERMITTED BY APPLICABLE LAW, CYPRESS MAKES NO WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, WITH REGARD TO THIS DOCUMENT OR ANY SOFTWARE OR ACCOMPANYING HARDWARE, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.  No computing device can be absolutely secure.  Therefore, despite security measures implemented in Cypress hardware or software products, Cypress shall have no liability arising out of any security breach, such as unauthorized access to or use of a Cypress product. CYPRESS DOES NOT REPRESENT, WARRANT, OR GUARANTEE THAT CYPRESS PRODUCTS, OR SYSTEMS CREATED USING CYPRESS PRODUCTS, WILL BE FREE FROM CORRUPTION, ATTACK, VIRUSES, INTERFERENCE, HACKING, DATA LOSS OR THEFT, OR OTHER SECURITY INTRUSION (collectively, "Security Breach").  Cypress disclaims any liability relating to any Security Breach, and you shall and hereby do release Cypress from any claim, damage, or other liability arising from any Security Breach.  In addition, the products described in these materials may contain design defects or errors known as errata which may cause the product to deviate from published specifications. To the extent permitted by applicable law, Cypress reserves the right to make changes to this document without further notice. Cypress does not assume any liability arising out of the application or use of any product or circuit described in this document. Any information provided in this document, including any sample design information or programming code, is provided only for reference purposes.  It is the responsibility of the user of this document to properly design, program, and test the functionality and safety of any application made of this information and any resulting product.  "High-Risk Device" means any device or system whose failure could cause personal injury, death, or property damage.  Examples of High-Risk Devices are weapons, nuclear installations, surgical implants, and other medical devices.  "Critical Component" means any component of a High-Risk Device whose failure to perform can be reasonably expected to cause, directly or indirectly, the failure of the High-Risk Device, or to affect its safety or effectiveness.  Cypress is not liable, in whole or in part, and you shall and hereby do release Cypress from any claim, damage, or other liability arising from any use of a Cypress product as a Critical Component in a High-Risk Device. You shall indemnify and hold Cypress, including its affiliates, and its directors, officers, employees, agents, distributors, and assigns harmless from and against all claims, costs, damages, and expenses, arising out of any claim, including claims for product liability, personal injury or death, or property damage arising from any use of a Cypress product as a Critical Component in a High-Risk Device. Cypress products are not intended or authorized for use as a Critical Component in any High-Risk Device except to the limited extent that (i) Cypress’s published data sheet for the product explicitly states Cypress has qualified the product for use in a specific High-Risk Device, or (ii) Cypress has given you advance written authorization to use the product as a Critical Component in the specific High-Risk Device and you have signed a separate indemnification agreement.
<br />
Cypress, the Cypress logo, and combinations thereof, WICED, ModusToolbox, PSoC, CapSense, EZ-USB, F-RAM, and Traveo are trademarks or registered trademarks of Cypress or a subsidiary of Cypress in the United States or in other countries. For a more complete list of Cypress trademarks, visit www.infineon.com. Other names and brands may be claimed as property of their respective owners.
