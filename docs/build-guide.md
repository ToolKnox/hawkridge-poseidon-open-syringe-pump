# Poseidon Open Syringe Pump - Arduino CNC Shield Lab Pump - Build Guide

https://github.com/ToolKnox/hawkridge-poseidon-open-syringe-pump

Remix of poseidon by the Pachter Lab, Caltech (https://github.com/pachterlab/poseidon), updated/refurbished for modern use. Print files unchanged.

## What you are building

A 3D printed syringe pump. A NEMA 17 stepper turns an M5 lead screw; the screw pushes a carriage sled that runs on two 6 mm rods and drives the syringe plunger. An Arduino Uno with a CNC Shield runs up to three pumps; a desktop app (Python + Qt) sets syringe size, flow units, speed and acceleration and sends the moves over USB. An optional microscope station (Raspberry Pi + 7 inch touchscreen + USB microscope) can run the same app next to the pumps.

The Bill of Material PDF lists the parts for **one pump plus the controller**. For two or three pumps, multiply the pump rows (motor, coupler, rods, bearings, threaded rod, screws, driver, jumpers, syringe); one Arduino, shield and power supply drive all three. The optional microscope rows are marked as such.

Safety: this is a research/teaching tool, not a medical device. Test it dry on the bench first. The upstream team reports that, once a pump has been used in a wet lab, it should stay there. PLA can be attacked by organic solvents; keep spills off the printed parts.

## 1. Firmware first (Arduino Uno)

1. Install the Arduino IDE and, in the Library Manager, the **AccelStepper** library.
2. Download the upstream project at the pinned commit: https://github.com/pachterlab/poseidon/tree/5a139fed350bbf5d775ffa9650f465e557b6ccb0
3. Open `SOFTWARE/arduino_serialCOM_v0.1/arduino_serialCOM_v0.1.ino`, select **Arduino Uno** and the board's port, and upload.
4. The sketch talks to the app at **230400 baud**. It drives three steppers with step/direction pins X = 2/5, Y = 3/6, Z = 4/7, which are the X, Y and Z sockets of the CNC Shield. Pin 8 is the shield's enable line; the sketch does not switch it, so the shield must enable the drivers with a jumper (step 2).

## 2. CNC Shield and drivers

1. Plug the CNC Shield onto the Uno.
2. Under each driver socket you use, fit **three jumpers on M0, M1 and M2**. With a DRV8825 that selects 1/32 microstepping, which is what the upstream setup uses (200 steps x 32 = 6400 microsteps per motor turn). An A4988 driver tops out at 1/16, so use DRV8825 drivers.
3. Fit a jumper between **EN and GND** (next to the reset button on the V3 shield) so the drivers are always enabled.
4. Insert the DRV8825 (heatsink on the chip) in the **X socket for pump 1** (Y = pump 2, Z = pump 3). Match the driver's EN pin to the EN mark on the shield; a reversed driver is destroyed at power-up.
5. Set the current limit before connecting a motor: on the DRV8825 the current limit is about **2 x Vref** (Vref measured between the trim pot and GND). Start around Vref = 0.5 V (about 1 A) and stay at or below the motor's rated current. The upstream team set the current so the motor stalls before a 1 mL syringe reaches about 4 MPa; a lower current gives a lower stall pressure.
6. Power: connect the 12 V adapter through the DC jack screw-terminal adapter to the shield's power terminal (+ to +, - to -). Never plug or unplug a motor while the shield is powered.

## 3. Print the parts

| File (HARDWARE/pump/stl) | Qty per pump |
|---|---|
| Pump Base v51.stl | 1 |
| Pump Carriage Sled v51.stl | 1 |
| Pump Cover Slide v51.stl | 1 |
| Pump Syringe Brace v51.stl | 1 |

STEP files of the same parts are included for editing. Any FFF printer works; PLA is what upstream used. Upstream added set screws and a thicker base to stop the body flexing, so print the base with solid walls (4 or more perimeters).

## 4. Assemble one pump

Watch the upstream build video alongside these steps: https://www.youtube.com/watch?v=7YSiO6usR1M

1. Mount the NEMA 17 to the motor end of the base with four **M3 x 20** screws.
2. Cut the M5 threaded rod to **170 mm** and deburr the end.
3. Press the two **LM6UU** bearings into the carriage sled and drop an **M5 nut** into the sled's nut pocket.
4. Slide the two **6 mm x 200 mm** rods through the base and the sled bearings, then clamp the rods with **M3 x 10** screws and **M3 nuts** (two per pump).
5. Thread the M5 rod through the sled nut and join it to the motor shaft with the **5 mm to 5 mm** flexible coupler. Turn the coupler by hand: the sled must travel the full length without binding. If it binds, loosen the motor screws, align, and tighten again.
6. Fit the cover slide, seat the syringe barrel in the cradle at the far end with the finger flange in its slot, put the plunger end against the sled, and clamp the syringe with the **M5 knob** and the syringe brace, as shown in the video.
7. Plug the motor into the X-socket header on the shield.

## 5. Install and run the control app

The app is `SOFTWARE/poseidon_main.py` (with `poseidon_controller_gui.py` next to it). It needs Python 3 with PyQt5, pyserial and OpenCV.

- Raspberry Pi OS / Debian / Ubuntu: `sudo apt install python3-pyqt5 python3-serial python3-opencv`
- Windows / macOS: `pip install pyqt5 pyserial opencv-python`

The upstream README's Python 3.5 and Qt 4 package lines date from 2019; use the commands above on current systems. The old pre-built executables on the upstream Releases page are optional.

Run `python3 poseidon_main.py` from the `SOFTWARE` folder, then:

1. Under **Select port**, pick the Arduino's port (press **Refresh ports** if it is missing).
2. Under **Set Microstepping**, choose **32** to match the jumpers. The app starts at 1; a mismatch makes every move 32 times too short.
3. Pick the syringe (BD 1, 3, 5, 10, 20, 30 or 60 mL), units, speed and acceleration.
4. Press **Connect to Controller**, then **Send all settings**.
5. On the controller tab, tick the pump and press **Jog +**. If the sled moves the wrong way, power down and turn that motor's plug round by 180 degrees.

## 6. Calibration and first run

- The app converts volumes using the BD barrel cross-section areas built into it, so use BD syringes or measure your barrel's inner diameter and expect a volume error otherwise.
- The M5 rod advances 0.8 mm per turn, so at 1/32 microstepping 1 mm of plunger travel is 8000 microsteps (6400 / 0.8). (A comment in the firmware says "4000 steps == 1 mm"; that does not match the app's maths.)
- Check: mark the sled, command 10 mm, and measure the travel with calipers. Then pump water into a container on a scale (1 g = 1 mL) to check the volume.
- Upstream tip: speeds and flow rates are limited by the microstepping and the syringe size; small syringes give finer flow, large syringes need more force.

## 7. Optional: microscope station

The repository also has a stand for a Raspberry Pi with the official 7 inch touchscreen and a USB microscope. It is optional; the pump works from any computer.

| File (HARDWARE/microscope/stl) | Qty |
|---|---|
| Scope Base v33.stl | 1 |
| Scope Screen Leg Mount v33.stl | 1 |
| Scope Top Stage v33.stl | 1 |

- Hardware from the optional rows: Raspberry Pi 4, official 7 inch touchscreen, Pi 4 USB-C supply, microSD card, USB microscope, eight **M5 x 14** screws with M5 nuts, and four **M3 x 10** screws.
- Install Raspberry Pi OS with desktop, install the packages from step 5, and run `poseidon_main.py` on the Pi. The app opens the USB microscope as a normal camera (OpenCV) and can save snapshots.
- The upstream parts list names a generic Raspberry Pi board with a 5 V 1.5 A supply and a 16 GB card; the list here uses a Raspberry Pi 4 with its 5.1 V 3 A USB-C supply as the current choice. This combination has not been tested by us. Add a USB keyboard and mouse for the first setup.

## Credits

Original design, firmware and software: Pachter Lab, Caltech - A. Sina Booeshaghi, Eduardo da Veiga Beltrame, Dylan Bannon, Jase Gehring and Lior Pachter. Paper: "Principles of open source bioinstrumentation applied to the poseidon syringe pump system", Scientific Reports 9, 12385 (2019). The pump builds on the Pearce Research Group open-source syringe pump; the microscope on a Satija Lab design. poseidon is distributed under the BSD 2-Clause License; the upstream notice is reproduced in full below.

```
BSD 2-Clause License

Copyright (c) 2018, Pachter Lab
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

Source at the pinned commit: https://github.com/pachterlab/poseidon/tree/5a139fed350bbf5d775ffa9650f465e557b6ccb0
