# Satellites-Traking-Discosat
https://discosat.dk/index.php/2022/01/groundstation-workshop/

## Information:

The DISCO project will launch 3 satellites into Low Earth Orbit over the next three years. To communicate with the satellites we need to have ground stations that manage the radio communications links for telemetry and data transfer.

The DISCO project is building both fixed ground stations at the participating universities, as well as mobile ground stations that can be loaned to schools in the program.

In this workshop we cover the basics of ground station operation and building including demos and a chance to get hands on experience with a range of equipment, both mobile and fixed equipment that will be later installed at ITU.

## Workshop: Satellites Traking with Open Source Platforms:

This workshop is oriented to build an Open Source satellite tracking. First, we're going to use Python with [Skyflied library](https://rhodesmill.org/skyfield/) and [Celestrak](http://celestrak.com/) to receive the orbit from many satellites. Second, we will use Arduino, one 180 servo motor, and a 360 servo motor to point them. 

### Python and Arduino Installation:

* You can use Python in your web navigator and gmail account: [Google Colab] https://colab.research.google.com/ or install it in your computer: [Anaconda](https://www.anaconda.com/products/individual).
*  With Arduino at the moment, the best option is installing on the computer (there is a beta online version), [Arduino](https://www.arduino.cc/en/software)

### Python Code: 

You can run the code in the selected environment without issues. 

Install the library:

* Google colab: pip install skyfield
* Computer:  Windows/Mac/Linux Search -> Anconda pront -> pip install skyfield

Test the library:
``` python
#Library
from skyfield.api import load, wgs84

import serial

# Load the JPL ephemeris DE421 (covers 1900-2050).
planets = load('de421.bsp')
earth, mars = planets['earth'], planets['mars']

# What's the position of Mars, viewed from Earth?
astrometric = earth.at(t).observe(mars)
ra, dec, distance = astrometric.radec()
print(ra)
print(dec)
print(distance)

```
Get the satellites information:
``` python
#print satellites
stations_url = 'http://celestrak.com/NORAD/elements/stations.txt'
satellites = load.tle_file(stations_url)
print('Loaded', len(satellites), 'satellites')
#loop satellites
by_name = {sat.name: sat for sat in satellites}
for i in by_name:
    print(i)
```
When was the last connection to Earth from each satellite:
``` python
t = ts.now()
for i in range(0,len(satellites)):
    satellite=satellites[i]
    days = t - satellite.epoch
    print('Satellite: ',satellite.name)
    print('{:.3f} days away from epoch'.format(days))
  ```
Find the time that we can track satellites in our specific location

``` python
#ITU LATITUDE AND LOGITUDE
bluffton = wgs84.latlon(+55.65, +12.59)
#days that we like to track satellites

t0 = ts.utc(2022, 3, 14)
t1 = ts.utc(2022, 3, 15)
for i in range(0,len(satellites)):
    satellite=satellites[i]
    t, events = satellite.find_events(bluffton, t0, t1, altitude_degrees=30.0)   
    for ti, event in zip(t, events):
        name = ('rise above 30°', 'culminate', 'set below 30°')[event]
        times=ti.utc_strftime('%Y %b %d %H:%M:%S')
        aux=list(times)
        hour_max=int(aux[12])
        hour_min=int(aux[13])
        hour=hour_max+hour_min
        if(hour>8 and hour < 18):
            print('Satellite: ',satellite.name)
            print(ti.utc_strftime('%Y %b %d %H:%M:%S'), name)
            print(ti.utc_strftime('%Y %b %d %H:%M:%S'), name)
  ```
  
  Select the satellite and receive his coordinates
 ``` python 
  # You can instead use ts.now() for the current time
t = ts.now()
satellite = by_name['AEROCUBE 12A']
geocentric = satellite.at(t)
print(geocentric.position.km)

#location
lat, lon = wgs84.latlon_of(geocentric)
print('Latitude:', lat)
print('Longitude:', lon)
difference = satellite - bluffton
topocentric = difference.at(t)
alt, az, distance = topocentric.altaz()
print('Altitude:',alt.degrees*-1)
print('Azimuth:', az.degrees)
print('Distance: {:.1f} km'.format(distance.km))
```
To check latitude and longitud, you can check: [n2yo](https://www.n2yo.com/satellite/).

To determine the COM communication, check this: [Find Arduino port](https://se.mathworks.com/help/supportpkg/arduinoio/ug/find-arduino-port-on-windows-mac-and-linux.html) or [Arduino support](https://support.arduino.cc/hc/en-us/articles/4406856349970-Find-the-port-your-board-is-connected-to)
 Send the azimuth and altitude to Arduino board:
  ``` python
  import serial,time
if alt.degrees<0:
    alti=int(alt.degrees*-1)
else:
    alti=int(alt.degrees*-1)
    
azi=int(round(az.degrees,0))
message=str(azi)+":"+str(alti)
 ```
Run this code just one time
``` python
ser = serial.Serial('COM3', 9600)
 ```
 Run this section each time that you what to track another satellite.
``` python
ser.open()
ser.write(message.encode())
ser.close()
 ```
 If you're working on GoogleColab:
 
  ``` python
 if alt.degrees<0:
    alti=int(alt.degrees*-1)
else:
    alti=int(alt.degrees*-1)
    
azi=int(round(az.degrees,0))
message=str(azi)+":"+str(alti)
print(message)
```
Copy the message and put it in Arduino Serial monitor
```
After you have uploaded this sketch onto your Arduino, click on the right-most button on the toolbar in the Arduino IDE. The button is circled below. The following window will open. This window is called the Serial Monitor and it is part of the Arduino IDE software.
```
