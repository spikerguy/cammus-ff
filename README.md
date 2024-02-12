# Force feedback support for Cammus C5 steering wheel

Linux module driver for Cammus C5 steering wheel.

Cammus C5 wheel is basically DirectInput wheel without 0xa7 descriptor (effect delay). Current in-tree driver does not take into account the lack of descriptor and fully rejects FFB support.
In that repository - copy of pidff driver from 6.3 kernel with some patches (removed 0xa7 support), and some changes around infinite length effects (like that https://github.com/berarma/ffbtools/issues/26)

And that's basically it

## What devices are supported?
### Bases:
1. Cammus C5 Base

Just because they have identical VendorID/ProductID (`0x3416` `0x0301`)

### Wheel rims:
Tested on C5, and all buttons works perfectly. I suspect that all rims will work, except of FX-Pro display.

## What works?
1. FFB (all effects from device descriptor, with [caveats](#known-issues-with-the-firmware)) only tested it on Truck Sims so far
2. All inputs (wheel axis, buttons)
3. Setup through proprietary Simagic soft (with [some tweaking](#how-to-set-up-a-base-parameters))

## What does not work?
1. Telemetry functions (Shift lights, FX-Pro display), mostly because telemetry works only with proprietary soft, which can't get access to shared memory chunks from games.
2. `Firmware Update` function. Use Windows PC or Windows VM at the moment.
3. Driver only detects wheels in "Normal mode". Compatibility mode changes VID/PID of the wheelbase to vJoy defaults (`0x1234` `0xbead`)

## How to use that driver?
You can install it through DKMS or manually.
### DKMS
1. Install `dkms`
2. Clone repository to `/usr/src/simagic-ff`
3. Install the module: 
`sudo dkms install /usr/src/simagic-ff`
4. Update initramfs:
`sudo update-initramfs -u`
5. Reboot

To remove module:
`sudo dkms remove simagic-ff/<version> --all`
### Manually 

1. Install `linux-headers-$(uname -r)`
2. Clone repository
3. `make`
4. `sudo insmod hid-simagic-ff.ko`

To unload module:
`sudo rmmod hid_simagic_ff`


## How to set up a base parameters?

You can do it through Android app of Cammus.

I have not used the windows app yet, as it is the only way to update wheels firmware.


## Known issues with the driver

1. ~~If you use `WheelCheck.exe` (iRacing .exe to check FFB on the wheel), it sends some bizare parameters (like 2147483647 in saturation coefficient) to driver - and hid-core rejects it with kernel warning. It does not happening in games, and it's unclear if it is a bug in firmware, or we need to fix pidff driver for it.~~ Fixed - it was unclamped PID_LOOP_COUNT, bug in pidff. 
2. Force Feedback clipping. Output queue could become full, and your kernel log will fill up with `output queue full` messages. I could reproduce it with spamming `WheelCheck.exe` parameters, i did not see it in games at all (maybe time will tell).
3. Firmware update does not work. Please use Windows machine or Windows VM for any firmware updates
4. Wheel firmware update does not work. Please use Windows machine or Windows VM for any firmware updates

## Known issues with the firmware 
Here is some issues, which is also a case for Windows
1. Cannot use FFB on AM,AM2, AC, ACC.

## DISCLAIMER
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Credits
To all those who worked on [simagic-ff](https://github.com/JacKeTUs/simagic-ff) and its core developer [JacKeTUs](https://github.com/JacKeTUs)
