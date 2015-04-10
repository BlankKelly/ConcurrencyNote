###What Every Programmer Should know about Time

Some notes about time:

- **UTC:** The time at zero degrees longitude (the Prime Meridian) is called Coordinated Universal Time (UTC is a compromise between the French and English initialisms)
- **GMT: **UTC used to be called Greenwich Mean Time (GMT) because the Prime Meridian was (arbitrarily) chosen to pass through the Royal Observatory in Greenwich.
- Other timezones can be written as an offset from UTC. Australian Eastern Standard Time is UTC+1000. e.g. 10:00 UTC is 20:00 EST on the same day.
- **Daylight saving** does not affect UTC. It's just a polity deciding to change its timezone (offset from UTC). For example, GMT is still used: it's the British national timezone in winter. In summer it becomes BST.
- **Leap Seconds: **By international convention, UTC (which is an arbitrary human invention) is kept within 0.9 seconds of physical reality (UT1, which is a measure of solar time) by introducing a "leap second" in the last minute of the UTC year, or in the last minute of June.
- Leap seconds don't have to be announced much more than six months before they happen. This is a problem if you need second-accurate planning beyond six months.
- **Unix time: **Measured as the number of seconds since epoch (the beginning of 1970 in UTC). Unix time is not affected by time zones or daylight saving.
- According to POSIX.1, Unix time is supposed to handle a leap second by replaying the previous second. e.g.:
<code>
59.00
59.25
59.50
59.75
59.00 ← replay
59.25
59.50
59.75
00.00 ← increment
00.25</code>

This is a trade-off: you can't represent a leap second, and your time is guaranteed to go backwards. On the other hand, every day has exactly 86,400 seconds, and you don't need a table of all previous and future leap seconds in order to format Unix time as human-preferred hours-minutes-seconds.

- See also: my write-up on the 2012 leap second, along with logs from several systems showing the replay behavior as well as Google's "leap smear."
- <code>ntpd</code> can be configured with the leapfile directive to announce an upcoming leap second. Most leaf installations don't bother with this, and rely on enough of their upstreams getting it right.

Some time-related considerations when programming:

- **Timezone are a presentation-layer problem!** Most of your code shouldn't be dealing with timezones or local time, it should be passing Unix time around.
- <code>libc</code> and/or your language's runtime has code to do timezone conversions and formatting. This stuff is tricky, avoid re-implementing it yourself.
- When measuring an interval of time, use a monotonic (always increasing) clock. Read the <code>clock_gettime()</code> manpage.
- When recording a point in time, measure Unix time. It's UTC. It's easy to obtain. It doesn't have timezone offsets or daylight saving (or leap seconds).
- When storing a timestamp, store Unix time. It's a single number.
- If you want to store a humanly-readable time (e.g. logs), consider storing it along with Unix time, not instead of Unix time.
- When displaying time, always include the timezone offset. A time format without an offset is useless.
- The system clock is inaccurate.
- You're on a network? Every other system's clock is differently inaccurate.
- The system clock can, and will, jump backwards and forwards in time due to things outside of your control. Your program should be designed to survive this.
- The number of [clock] seconds per [real] second is both inaccurate and variable. It mostly varies with temperature.
- <code>ntpd</code> can change the system time in two ways:
	- Step: making the clock jump backwards or forwards to the correct time instantaneously.
	- Slew: changing the frequency of the clock so that it slowly drifts toward the correct time.
- Slew is preferred because it's less disruptive, but it's only useful for correcting small offsets.

Special mentions:

- Leap seconds happen more often than leap years.
- Time passes at a rate of one second per second for every observer. The frequency of a remote clock relative to an observer is affected by velocity and gravity. The clocks inside GPS satellites are adjusted for relativistic effects. (A clock moving faster than you appears to tick more slowly.)
- MySQL (at least 4.x and 5.x) stores the <code>DATETIME</code> type as a binary encoding of its <code>"YYYY-MM-DD HH:MM:SS"</code> string. It does not store an offset, and interprets times according to <code>@@session.time_zone</code>:

		mysql> insert into times values(now());
		mysql> select unix_timestamp(t) from times;
		1310128044
		mysql> SET SESSION time_zone='+0:00';
		mysql> select unix_timestamp(t) from times;
		1310164044
	
	
There's also a TIMESTAMP type, which is stored as UNIX time, but has other magic.

In conclusion, if you care at all about storing timestamps in MySQL, store them as integers and use the <code>UNIX_TIMESTAMP()</code> and <code>FROM_UNIXTIME() </code>functions.
