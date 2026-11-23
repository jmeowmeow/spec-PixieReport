# Time-limited pixie cache, recent-client tallies and runtime counters

_Extracted from `webapp`._

The webapp keeps a pixie cache whose entries stay valid for 5 minutes. It also keeps recent-activity tallies for clients, robots and pages, with entries that expire on the same 5-minute window and are reported in descending order of count. Named runtime counters track things like pixies rendered and are reported sorted by name.

## S1: A cached value is returned within the five-minute window @v1 [proposed]

- Given: a value is put in the cache under a key at time T
- When: the key is read at a time no more than 5 minutes after T
- Then: the stored value is returned

_Source: `webapp/cache.js`._

## S2: A cached value is not returned after five minutes @v1 [proposed]

- Given: a value is put in the cache under a key at time T
- When: the key is read at a time more than 5 minutes after T
- Then: no value is returned

_Source: `webapp/cache.js`._

## S3: Expiring the cache removes stale entries @v1 [proposed]

- Given: the cache holds some entries older than 5 minutes and some newer ones
- When: the cache is expired at the current time
- Then: the stale entries are deleted
- and: the fresh entries remain and the cache size shows how many remain

_Source: `webapp/cache.js`._

## S4: Deleting and clearing entries updates the size @v1 [proposed]

- Given: the cache holds entries
- When: one key is deleted, or the whole cache is cleared
- Then: that key, or every key, is gone
- and: the size shows the number of entries left

_Source: `webapp/cache.js`._

## S5: Incrementing a client adds one to its recent count @v1 [proposed]

- Given: a client has a recent count, or no count at all
- When: the client is incremented at a time
- Then: its count goes up by one, starting from zero if it had no fresh count
- and: the count is stamped with that time

_Source: `webapp/cache.js`._

## S6: A client's count starts again after its entry goes stale @v1 [proposed]

- Given: a client was last incremented more than 5 minutes ago
- When: the client is incremented again
- Then: its count is 1

_Source: `webapp/cache.js`._

## S7: The client report lists recent clients by count, highest first @v1 [proposed]

- Given: several clients have been incremented within the last 5 minutes
- When: the clients are shown
- Then: stale entries are expired first
- and: each remaining client is listed with its count, in descending order of count

_Source: `webapp/cache.js`._

## S8: Robots and pages are tallied the same way as clients @v1 [proposed]

- Given: the robots and pages tallies
- When: a robot or a page is incremented and then shown
- Then: it is counted, expired and reported in the same way as clients

_Source: `webapp/cache.js`._

## S9: The client and page tallies are seeded at start-up @v1 [proposed]

- Given: the webapp starts
- When: the clients and pages are shown straight afterwards
- Then: the clients list shows 255.255.255.255 with 3, 192.168.0.1 with 2 and 10.0.0.1 with 1
- and: the pages list shows /dev/null with 1

_Source: `webapp/cache.js`._

## S10: Incrementing named counters adds one to each @v1 [proposed]

- Given: some named counters, which may not exist yet
- When: an increment is made with one or more counter names
- Then: each named counter goes up by one, starting from zero if it was new

_Source: `webapp/counters.js`._

## S11: Clearing out counters sets them to zero @v1 [proposed]

- Given: some named counters with tallies, or not yet defined
- When: those names are cleared out
- Then: each named counter is set to zero, so it shows up in the report before anything is tallied

_Source: `webapp/counters.js`._

## S12: Counters are reported sorted by name @v1 [proposed]

- Given: several named counters with tallies
- When: the counters are shown
- Then: each counter is listed with its tally, in order of name

_Source: `webapp/counters.js`._
