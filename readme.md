# nvidia-co2

Show gCO2eq emissions information with nvidia-smi, at the top right corner. For example: **79.2gCO2eq/h** or **23.76mm^2/h sea ice**.

Copies code from [experiment-impact-tracker](https://github.com/Breakend/experiment-impact-tracker) for mapping geolocations to energy usage, which can be used to monitor and report on longer-running experiments.

This script doesn't take into account:

- Carbon intensity changes with time of day.
- Datacenters often have unique energy sources. `experiment-impact-tracker` tracks this information, and it can be accessed with their `scripts/lookup-cloud-region-info`. I would be happy to add this info if the script can automatically detect the provider and region, possibly from the IP address.
- The state of California has more detailed information available via [California ISO](http://www.caiso.com/Pages/default.aspx) and this script does not use that data.
- CPU usage is only monitored if it is tracked at `/sys/class/powercap/intel-rapl`. Doing this in a hardware-independent way requires a lot more code, with some first steps in `experiment-impact-tracker`.

When running the first time at an IP address, the script will geolocate your IP address and estimate the local carbon intensity. This information will be cached between runs in `/tmp/nvidia-co2-cache.(dir|bak|dat)`. The first run might take 1 second, additional runs should take 200ms.

This script won't work by default on Google Cloud because I'm using `dig` to quickly get a public IP address. Permissions are also set up in a way where you would need to install it to `--user` and call `python -m nvidia-co2` or similar. But with a little work it could be done :)

## Install

`pip install git+https://github.com/kylemcdonald/nvidia-co2.git`

## Example

```
$ nvidia-co2 -m ice
Sun Feb 16 14:44:50 2020                                    23.76mm^2/h sea ice
+-----------------------------------------------------------------------------+
| NVIDIA-CO2 435.21       Driver Version: 435.21       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce RTX 208...  Off  | 00000000:05:00.0  On |                  N/A |
| 45%   59C    P2   206W / 260W |  10975MiB / 11016MiB |     89%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce RTX 208...  Off  | 00000000:09:00.0 Off |                  N/A |
| 26%   34C    P8    19W / 260W |    166MiB / 11019MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1149      G   /usr/lib/xorg/Xorg                            85MiB |
|    0      1359      G   /usr/bin/gnome-shell                          91MiB |
|    0     21752      C   ...e/kyle/anaconda3/envs/tf2gpu/bin/python 10787MiB |
|    1     21752      C   ...e/kyle/anaconda3/envs/tf2gpu/bin/python   155MiB |
+-----------------------------------------------------------------------------+
```

## Command-line options

```
$ nvidia-co2 --help
usage: nvidia-co2 [-h] [--mode MODE]

Show gCO2eq emissions information with nvidia-smi. Combines CPU and GPU usage.
Emissions are corrected for location using IP address geolocation.

optional arguments:
  -h, --help            show this help message and exit
  --mode MODE, -m MODE  [ice|beef|tofu|car-mph|car-kph|bulb|cfl|watt|gco2eqph]
                        `ice` shows how much sea ice is lost per hour due to
                        your emissions. `beef` and `tofu` shows how many grams
                        of each it takes to produce the same emissions. `car-
                        mph` and `car-kph` show how fast a car would have to
                        drive to produce the same emissions. `bulb` and `cfl`
                        show how many incandescent lightbulbs or CFLs are
                        required to use the same power. `watt` shows how many
                        watts used, and `gco2eqph` shows gCOeq/hour used.
                        (default: gco2eqph)
```