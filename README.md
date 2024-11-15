# PLUTO-GPS-SIM

__Project closed!__ Development continues on [multi-sdr-gps-sim](https://github.com/Mictronics/multi-sdr-gps-sim).

PLUTO-GPS-SIM generates a GPS baseband signal IQ data stream, which is then transmitted by the
software-defined radio (SDR) platform [ADALM-Pluto](https://wiki.analog.com/university/tools/pluto).

Project based on [gps-sdr-sim](https://github.com/osqzss/gps-sdr-sim). Credits to Takuji Ebinuma.

### Generating the GPS signal

The user is able to assign a static location directly through the command line.

The user also specifies the GPS satellite constellation through a GPS broadcast
ephemeris file. The daily GPS broadcast ephemeris file (brdc) is a merge of the
individual site navigation files into one.

These files are then used to generate the simulated pseudorange and
Doppler for the GPS satellites in view. This simulated range data is
then used to generate the digitized I/Q samples for the GPS signal
which are then feed into ADALM-Pluto SDR to transmit the GPS L1 frequency.

The simulation start time can be specified if the corresponding set of ephemerides
is available. Otherwise the first time of ephemeris in the RINEX navigation file
is selected.

__Note__

The ADLAM-Pluto SDR internal oscillator is not precise enough (frequency offset) and stable (temperature drift)
for GPS signal generation.

Do not run this code without Pluto oscillator modification, either by TCXO or more expensive (but highest precision) OCXO.
Search for [adlam pluto tcxo](https://duckduckgo.com/?q=adlam+pluto+tcxo)

### Ephemeris files

NASA CDDIS anonymous ftp service has been discontinued on October 31, 2020.

The new way to get ephemeris data is as follows:

Register an account with https://urs.earthdata.nasa.gov/ (There is no way around this step).

Create a file called .netrc with the following format.
machine urs.earthdata.nasa.gov login <username> password <password> where <username> and <password> are the values you set when you created your Earthdata login account. Don't forget to sudo chmod 400 .netrc so that no one can read your file and get your username and password.

You can automate with the process with following script.

```
#!/bin/sh
day=$(date +%j)
year=$(date +%Y)
yr=$(date +%y)

echo $day
echo $year
echo $yr

curl -c /tmp/cookie -n -L -o "brdc""$day""0.$yr""n.gz" "https://cddis.nasa.gov/archive/gnss/data/daily/$year""/brdc/brdc""$day""0.$yr""n.gz"

uncompress "brdc""$day""0.$yr""n.gz"
echo "brdc""$day""0.$yr""n.gz"
```

The .netrc file along with the -n flag allows curl to automatically authenticate when attempting to retrieve the file.

### Build instructions

#### Dependencies

Build depends on libm, libpthread and libcurl4-openssl-dev.

You will also need the latest build and install of libad9361-dev and libiio-dev. The Debian packages
libad9361-dev that is available up to Debian 9 (stretch) is outdated and missing a required function.
So you have to build packages from source:

The first step is to fetch the dependencies, which as of now is only libxml2. On a Debian-flavoured GNU/Linux distribution, like Ubuntu for instance:

```
$ sudo apt-get install libxml2 libxml2-dev bison flex libcdk5-dev cmake
```

Depending on the backend (how you want to attach the IIO device), you may need at least one of:

```
$ sudo apt-get install libaio-dev libusb-1.0-0-dev libserialport-dev libxml2-dev libavahi-client-dev doxygen graphviz
```

Then build in this order:

```
 sudo apt install -y cmake libusb-1.0-0-dev libavahi-client-dev libxml2-dev libaio-dev flex bison
 cd
 git clone https://github.com/analogdevicesinc/libiio.git
 cd libiio
 cmake ./
 make
 sudo make install
```

```
 cd
 git clone https://github.com/analogdevicesinc/libad9361-iio.git
 cd libad9361-iio
 cmake ./
 make
 sudo make install
```

#### Building under Linux with GCC

```
$ git clone https://github.com/mictronics/pluto-gps-sim
$ cd pluto-gps-sim
$ make all
```

### Usage

````
pluto-gps-sim [options]
Options:
  -e <file name>   RINEX navigation file for GPS ephemerides (required)
  -u <file name>   User motion file (dynamic mode) 10Hz, Max 3000 points
  -3               Use RINEX version 3 format
  -f               Pull actual RINEX navigation file from NASA FTP server
  -c <location>    ECEF X,Y,Z in meters (static mode) e.g. 3967283.15,1022538.18,4872414.48
  -l <location>    Lat,Lon,Hgt (static mode) e.g. 30.286502,120.032669,100
  -t <date,time>   Scenario start time YYYY/MM/DD,hh:mm:ss
  -T <date,time>   Overwrite TOC and TOE to scenario start time (use ```now``` for actual time)
  -s <frequency>   Sampling frequency [Hz] (default: 2600000)
  -i               Disable ionospheric delay for spacecraft scenario
  -v               Show details about simulated channels
  -A <attenuation> Set TX attenuation [dB] (default -20.0)
  -B <bw>          Set RF bandwidth [MHz] (default 5.0)
  -U <uri>         ADALM-Pluto URI (eg. usb:1.2.5)
  -N <network>     ADALM-Pluto network IP or hostname (default pluto.local)
````

Set static mode location:

```
> pluto-gps-sim -e brdc3540.14n -l 30.286502,120.032669,100
```

Set user motion mode:

```
> pluto-gps-sim -e brdc3540.14n -u circle.csv
```

Set TX attenuation:

```
> pluto-gps-sim -e brdc3540.14n -A -30.0
```

Default -20.0dB. Applicable range 0.0dB to -80.0dB in 0.25dB steps.

Set RF bandwidth:

```
> pluto-gps-sim -e brdc3540.14n -B 3.0
```

Default 3.0MHz. Applicable range 1.0MHz to 5.0MHz

### Transmitting the samples

The TX port of a particular SDR platform is connected to the GPS receiver
under test through a DC block and a fixed 50-60dB attenuator.

### License

Copyright &copy; 2018 Mictronics
Distributed under the [MIT License](http://www.opensource.org/licenses/mit-license.php).
