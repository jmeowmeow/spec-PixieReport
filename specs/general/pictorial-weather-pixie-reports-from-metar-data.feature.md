# Pictorial weather pixie reports from METAR data

_Extracted from `.`._

PixieReport turns the latest METAR weather report for an airport station into a small 125×175 pixel-doll picture. The layers, back to front, are a day/night background, clouds, lightning, haze, wind flags, a doll dressed for the temperature, precipitation, and a frame. The picture comes with station and weather text and generated alt text. Weather reports and rendered pixies are cached for five minutes. Temperature and wind are shown in metric units unless the station or the request calls for Fahrenheit.

## S1: Cached values are returned until they expire after five minutes @v1 [proposed]

- Given: an empty cache whose duration is 300000 milliseconds
- When: the value 'world' is put under the key 'hello' at the current time
- Then: getting 'hello' at that same time returns 'world'
- and: getting 'hello' more than 300000 milliseconds later returns undefined

_Source: `spec/cacheSpec.js`._

## S2: Expiring the cache removes only stale entries @v1 [proposed]

- Given: 'hello' is cached now and 'goodnight' is cached at a time more than five minutes later
- When: the cache is expired at that later time
- Then: the cache size goes from 2 to 1
- and: 'hello' is no longer returned at any time
- and: 'goodnight' still returns 'Irene'

_Source: `spec/cacheSpec.js`._

## S3: Units default to metric and follow the station or the requested units @v1 [proposed]

- Given: pixie parameters with no units requested
- When: the units are worked out
- Then: a US station such as KSEA uses Fahrenheit, while LIMC, an unknown station '....' or no station at all uses metric
- and: units 'C' always gives metric and units 'F' always gives Fahrenheit, whatever the station

_Source: `spec/imageTextSpec.js`._

## S4: Decoded METAR reports are parsed into weather parameters @v1 [proposed]

- Given: sample decoded METAR reports for stations such as KBLI, KFNT, NZSP, PANU and EHJA
- When: each report is parsed
- Then: the stationCode matches the station and the stationDesc contains the station code
- and: degreesC, zuluTime, zuluDayOfMonth, windSpeedMph, windSpeedKph and windDir match the report, for example NZSP at -72 °C at 23:50Z and KFNT with a gusty wind of '15-25' mph

_Source: `spec/parserSpec.js`._

## S5: An empty report falls back to an unknown or requested station code @v1 [proposed]

- Given: an empty METAR text
- When: it is parsed with no station requested
- Then: the stationCode is '????'
- and: when it is parsed for the requested station 'KYYX', the stationCode is 'KYYX'

_Source: `spec/parserSpec.js`._

## S6: The background follows the sun's position, or shows static when there is no report @v1 [proposed]

- Given: parsed parameters for a station
- When: the pixie is composed
- Then: an empty METAR gets the 'video static' no-report background
- and: otherwise a solar zenith angle below 89° gives day, below 96° gives dusk, below 102° gives gray twilight, and anything higher gives night
- and: when the sun position is unknown, the background is gray twilight

_Source: `compose-async.js`, `preloads.js`._

## S7: The doll is dressed for the temperature level @v1 [proposed]

- Given: a doll set and a temperature in degreesC
- When: the pixie is composed
- Then: below -9 °C the icy doll is shown, below 5 °C the cold doll, below 19 °C the cool doll, below 30 °C the warm doll, and otherwise the hot doll
- and: when the temperature is not a finite number, no pixel doll is shown

_Source: `compose-async.js`, `preloads.js`, `spec/pixieJsIncludeSpec.js`._

## S8: A requested doll set is used, otherwise one is picked at random @v1 [proposed]

- Given: a request that may carry a set parameter, either a set name or a set number
- When: the pixie is composed
- Then: a known set name or set number picks that doll set
- and: without a set, a doll set is picked at random and recorded in the parameters, so the pixie can be cached under that set

_Source: `compose-async.js`, `preloads.js`, `express-progress/cache.md`._

## S9: High winds raise warning flags @v1 [proposed]

- Given: a parsed wind speed in mph
- When: the pixie is composed
- Then: above 24 mph a red high wind warning pennant is shown, above 38 mph two red gale warning pennants, above 54 mph a red and black storm warning flag, and above 73 mph two red and black hurricane warning flags
- and: at 24 mph or less no flag is shown

_Source: `compose-async.js`, `preloads.js`._

## S10: Weather conditions map to overlay layers, and unknown conditions are left out @v1 [proposed]

- Given: parsed weather conditions such as 'light rain with thunder', 'heavy snow', 'patches of fog' or 'Lightning observed'
- When: the pixie is composed
- Then: lightning and haze are drawn behind the doll, while precipitation, mist, fog and smoke are drawn over it in the order reported
- and: aliases share one layer, so 'light rain with thunder' shows light rain and 'patches of fog' shows fog
- and: a condition with no layer, such as ice crystals, is not drawn and is recorded as unrendered

_Source: `compose-async.js`, `preloads.js`._

## S11: Reports older than two hours look faded @v1 [proposed]

- Given: a report observed more than 2 hours ago
- When: the pixie is composed
- Then: a yellowish fade is laid over the picture and a curled corner frame is added on top
- and: neither of these layers appears in the scene description

_Source: `compose-async.js`, `preloads.js`._

## S12: Alt text describes the scene, the weather and the doll @v1 [proposed]

- Given: parsed sample reports drawn with the bunny doll set
- When: the alt text is computed
- Then: it names the time of day, the sky cover and any weather, for example KFNT includes 'night', 'overcast', 'lightning' and 'light rain'
- and: it ends with the doll description, here one beginning "with Beatrix Potter's"
- and: for empty parameters it reads 'Weatherpixie image for ICAO METAR ????, showing undefined'

_Source: `spec/parserSpec.js`, `compose-async.js`, `spec/resources/layerDefsTest.js`._
