# Pixie image text and decoded METAR parsing

_Extracted from `pixifier`._

The Pixifier reads a decoded METAR weather report and pulls out the station, time, temperature, humidity, wind and pressure. It then builds the text lines written on a pixie image: a temperature/humidity line, a wind/pressure line, the zulu time and a shortened location label. Metric or US units are picked from an explicit units choice or from the station code. Each doll set gives a text description of its doll for each of five weather levels, from 0 (very cold) to 4 (hot).

## S1: An explicit units choice decides between metric and US units @v1 [proposed]

- Given: params with units 'C' and station code 'KSEA'
- When: the unit system is chosen
- Then: metric is used
- and: with units 'F' and any station code, US units are used

_Source: `pixifier/compute-image-text.js`._

## S2: Station codes for US regions use US units @v1 [proposed]

- Given: no units choice
- When: the station code starts with 'K', 'TJ', 'PH' or 'PA'
- Then: US units are used
- and: a station code starting with 'P' with a latitude above 50 degrees also uses US units
- and: other station codes, or a missing station code, use metric

_Source: `pixifier/compute-image-text.js`._

## S3: Metric image text shows Celsius, km-h and hectopascals @v1 [proposed]

- Given: metric units, degreesCstr '12', humidity '80%', windDir 'SW', windSpeedKph '19' and hectoPressure '1016'
- When: the image text values are computed
- Then: temphum is '12C 80%'
- and: windbar is 'SW 19kph 1016hPa'

_Source: `pixifier/compute-image-text.js`._

## S4: US-unit image text shows Fahrenheit, mph and the inHg value @v1 [proposed]

- Given: US units, degreesFstr '54', humidity '80%', windDir 'N', windSpeedMph '12' and inHgPressure '30.10'
- When: the image text values are computed
- Then: temphum is '54F  80%'
- and: windbar is 'N 12mph 30.10mmHg'

_Source: `pixifier/compute-image-text.js`._

## S5: Calm or missing wind and missing values become placeholders @v1 [proposed]

- Given: windDir 'Calm'
- When: the image text values are computed
- Then: windbar starts with 'Calm  ' and has no wind speed
- and: when windDir is missing, windbar starts with blank spaces
- and: when the temperature is missing, temphum starts with five spaces
- and: an empty pressure string leaves the pressure off windbar

_Source: `pixifier/compute-image-text.js`._

## S6: The location label is a shortened station name @v1 [proposed]

- Given: stationDesc 'Los Angeles, Los Angeles International Airport, CA, United States (KLAX) ...'
- When: the image text values are computed
- Then: locationLabel keeps only the text before the last ' (', drops the repeated city name and shortens words, e.g. 'Los Angeles Intl Apt, CA, USA'
- and: when stationDesc has no ' (', locationLabel is the station code

_Source: `pixifier/compute-image-text.js`._

## S7: Wind direction is turned into a compass point @v1 [proposed]

- Given: a METAR observation with wind group '23019KT'
- When: the wind direction is read
- Then: the direction is 'SW'
- and: 'VRB' gives 'Vrbl', '///' gives 'Wind', a speed of '00' gives 'Calm', and directions below 11 or above 349 degrees give 'N'

_Source: `pixifier/decoded-metar-parser.js`._

## S8: Wind speed and gusts are converted to mph and km-h @v1 [proposed]

- Given: a METAR wind group '26017G28KT'
- When: the wind speed is read
- Then: the mph value is '20-32' and the km/h value is '32-52'
- and: speeds given in MPS are converted at 3.6 km/h and 2.23 mph per metre per second
- and: a speed of '//' gives an empty value

_Source: `pixifier/decoded-metar-parser.js`._

## S9: Pressure is read and converted between hPa and inHg @v1 [proposed]

- Given: an observation with 'A3010'
- When: the pressure is read
- Then: inHg pressure is '30.10' and the hPa pressure is worked out from it
- and: an observation with 'Q1016' gives hPa pressure '1016' and an inHg value worked out from it
- and: with no pressure group, both are empty

_Source: `pixifier/decoded-metar-parser.js`._

## S10: Temperature is read from the temperature-dew point group @v1 [proposed]

- Given: an observation containing 'M05/M10'
- When: the temperature is read
- Then: the Celsius temperature is -5 and the Fahrenheit temperature is 23
- and: a group with no dew point, such as '31///', still gives 31 C
- and: visibility text like '1 3/4SM' is not taken as a temperature

_Source: `pixifier/decoded-metar-parser.js`._

## S11: Station, time and location are read from the decoded report @v1 [proposed]

- Given: a decoded report whose 'ob:' line is 'EGSC 171550Z 23019KT CAVOK 32/14 Q1016'
- When: the report is parsed
- Then: the station code is 'EGSC', zuluTime is '15:50' and zuluDayOfMonth is '17'
- and: an observation with no time gives zuluTime '(no time)', and an observation with no station code gives '????'
- and: the station description comes from the report's first line if that line holds the code, otherwise from the ICAO station list, otherwise it is the code itself
- and: latitude and longitude such as '35-40-05N 095-56-55W' become signed decimal degrees, with S and W negative

_Source: `pixifier/decoded-metar-parser.js`._

## S12: Sky, humidity and weather conditions are read from the decoded lines @v1 [proposed]

- Given: a decoded report with 'Weather: light rain; mist'
- When: the report is parsed
- Then: the weather conditions are 'light rain' and 'mist'
- and: the 'Relative Humidity:' value is used as humidity, or empty if that line is missing
- and: the 'Sky conditions:' value is used as sky cover, or 'clear' if that line is missing

_Source: `pixifier/decoded-metar-parser.js`._
