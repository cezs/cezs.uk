---
title: Instrumentation with jtx1inst C Library
date: 2017-09-13 14:07:10
tags:
---

<a id="org1930f7f"></a>

# Instrumentation with *jtx1inst* C Library

<a id="orgec10f4a"></a>

Both Jetson TX1 module and carrier board are each equipped with INA3221 current and voltage monitors. The Jetson TX1 has also various temperature sensors placed in total of eight thermal zones of module and SoC. In order to leverage the information that is output by these sensors, we have developed custom API for accessing the two INA3221 on-board and on-module monitors as well as reading the values of eight thermal zones.

Using the sysfs file `/sys/class/thermal/thermal_zone*/type`, we are able to retrieve information about the following zones: `AO-therm`, `CPU-therm`, `GPU-therm`, `PLL-therm`, `PMIC-Die`, `Tdiode_tegra`, `Tboard_tegra` and `thermal-fan-est.46`. Each zone ending in `-therm` provides information from sensors located on SoC, while those ending in `_tegra` are located on module. Throughout the experiments we will only concentrate temperature values available from debugfs files located at `/sys/kernel/debug/tegra_soctherm/`, namely CPU, GPU, memory I/O, and PLL temperatures.

We also use sysfs files, in order to take voltage, current or power measurements from carrier board's 3 channel INA3221 monitor. We access rails of main carrier board power input `VDD_MUX`,f main carrier board 5V supply `VDD_5V_IO_SYS`, main carrier board 3.3V supply `VDD_3V3_SYS` through the I2C address of `0x42`. At the `0x43` address, there are rails for carrier board 3.3V sleep supply `VDD_3V3_IO`, main carrier board 1.8V supply `VDD_1V8_IO` and 3.3V supply for M.2 Key E connector `VDD_M2_IN`. 
At the time of writing, there are no sysfs files for module's INA3221 monitor and we access the information located at the I2C address of `0x40`, namely main module power input `VDD_IN`, GPU Power rail `VDD_GPU` and CPU Power rail `VDD_CPU` through custom userspace functions. 

The sysfs files can be also employed for the system-level control of Jetson TX1 performance. In order to control CPU performance we can manually change pattern of each of four cores utilisation by either enabling or disabling each of the cores or by changing their operating frequency. We can also control GPU system-level performance  by either changing it's operating frequency or changing the operating rate of it's memory. We encapsulate the aforementioned operations in open-sourced C API and provide on-line documentation . 

The current version of the API supplies among the others the following functions:

-   `jtx1_get_temp` for reading on-chip and on-module temperature. It takes two arguments, first argument is one of the zones which are indexed with `jtx1_tzone` enumeration (see table [1](#org6d2193b)), and second is reference to a variable that is going to store the actual value of temperature read from sensor specified in first of the arguments. The temperature value is output in millidegree Celsius.

| thermal zone   | description            |
|----------------|:-----------------------|
| `A0`           | on-chip thermal zone   |
| `CPU`          | on-chip thermal zone   |
| `GPU`          | on-chip thermal zone   |
| `PLL`          | on-chip thermal zone   |
| `PMIC`         | on-chip thermal zone   |
| `TDIODE`       | on-module thermal zone |
| `TBOARD`       | on-module thermal zone |
| `FAN`          | on-chip thermal zone   |

-   `jtx1_get_ina3221` for reading on-board and on-module INA3221's values. This function currently uses sysf files to access on-board INA3221 sensor and userspace I2C to access on-module INA3221 sensor and read power, current, and voltage information. It takes three arguments: rail which is indexed by `jtx1_rail` enumeration, second parameter specifies the type of measurement which can be either `VOLTAGE`, `POWER` or `CURRENT` value from `jtx1_rail_type` enumeration (see table [2](#org3530ef2), and third is the actual output's reference where value is given either in millivolts, milliwatts or milliamps depending on the setting of the second argument.

| rail            | description                       |
|-----------------|-------------------------------------|
| `VDD_IN`        | main module power input             |
| `VDD_GPU`       | GPU Power rail                      |
| `VDD_CPU`       | CPU Power rail                      |
| `VDD_MUX`       | main carrier board power input      |
| `VDD_5V_IO_SYS` | main carrier board 5V supply        |
| `VDD_3V3_SYS`   | main carrier board 3.3V supply      |
| `VDD_3V3_IO`    | carrier board 3.3V Sleep supply     |
| `VDD_1V8_IO`    | main carrier board 1.8V supply      |
| `VDD_M2_IN`     | 3.3V supply for M.2 Key E connector |

-   `jtx1_get_rate` and `jtx1_set_rate` allowing to either set or get value of the frequency of either the external memory controller (EMC), the graphics processing unit (GPU) or one of the four available CPU cores. As the first argument both functions take one of the available choices specified in `jtx1_unit` enumeration (see table [3](#org71a0ab9)).

| unit        | definition                                |
|-------------|---------------------------------------------|
| `EMC_RATE`  | external memory controller (EMC)            |
| `GPU_RATE`  | graphics processing unit (GPU)              |
| `CPU0_RATE` | first core of central processing unit (CPU) |
| `CPU1_RATE` | second core of CPU                          |
| `CPU2_RATE` | third core of CPU                           |
| `CPU3_RATE` | fourth core of CPU                          |

The API is provided in the form of C library while it's sources are stored in project's repository. In order to use them, one follows standard build and installation process described below in listing [1](#org3d2397f).

    mkdir build
    cd build
    cmake ..
    make
    sudo make install
    sudo ldconfig

The *jtx1inst* library is released under public domain type license, thus it can be copied and modified to satisfy any specific requirements of a custom project. 
