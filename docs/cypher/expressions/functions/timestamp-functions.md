---
layout: default
title: Timestamp function
parent: Expressions
grand_parent: Cypher
---

# Timestamp operators

| Operator | Description | Example | Result |
| ----------- | ----------- |  ----------- |  ----------- |
| + | addition of INTERVAL | TIMESTAMP('2022-11-12 13:22:17') + INTERVAL('4 minutes, 3 hours, 2 days') | 2022-11-14 17:22:21 (TIMESTAMP) | 
| - | subtraction of TIMESTAMP | TIMESTAMP('2022-11-22 15:12:22') - TIMESTAMP('2011-10-09 13:00:21') | 4062 days 02:12:01 (INTERVAL)|
| - | subtraction of INTERVAL | TIMESTAMP('2011-10-21 14:25:13') - INTERVAL('35 days 2 years 3 hours') | 2009-09-16 11:25:13 (TIMESTAMP) |

# Timestamp functions

| Function | Description | Example | Result |
| ----------- | ----------- |  ----------- |  ----------- |
| century(timestamp) | returns the century of the timestamp | century(TIMESTAMP('2013-12-11 14:22:13')) | 21 (INT64) | 
| date_part(part, timestamp) | returns the subfield of the timestamp | date_part('second', TIMESTAMP('1995-11-02 12:05:21')) | 21 (INT64) |
| datepart(part, timestamp) | alias of date_part | datepart('month', TIMESTAMP('1926-11-21 13:22:19')) | 11 (INT64) |
| date_trunc(part, timestamp) | returns the given timestamp with specified precision | date_trunc('month', TIMESTAMP('2002-10-21 13:51:21')) | 2002-10-01 00:00:00 (TIMESTAMP) |
| datetrunc(part, timestamp) | alias of date_trunc | datetrunc('year', TIMESTAMP('2005-12-11 11:21:31')) | 2005-01-01 00:00:00 (TIMESTAMP) |
| dayname(timestamp) | returns the english name of the day of the timestamp | dayname(TIMESTAMP('2022-11-08 11:12:20')) | Tuesday (STRING) | 
| epoch_ms(ms) | converts the ms to timestamp | epoch_ms(INT64(701222402100)) | 1992-03-22 00:00:02.1 (TIMESTAMP) |
| greatest(timestamp, timestamp) | returns the later of the two timestamps | greatest(TIMESTAMP('2013-12-11 10:22:11'), TIMESTAMP('2011-05-13 13:22:11')) | 2013-12-11 10:22:11 (TIMESTAMP) |
| least(timestamp, timestamp) | returns the earlier of the two timestamps | least(TIMESTAMP('1966-12-21 15:22:11'), TIMESTAMP('2005-11-12')) | 1966-12-21 15:22:11 (TIMESTAMP) |
| last_day(timestamp)	| returns the last day of the month of the timestamp | last_day(TIMESTAMP('2022-11-08 05:11:31')) | 2022-11-30 (DATE) |
| monthname(timestamp) | returns the english name of the month of the date | monthname(TIMESTAMP('2022-04-21 06:11:22')) | April (STRING) |
| to_timestamp(sec)	| converts sec to timestamp | to_timestamp(701222723) | 1992-03-22 00:05:23 (TIMESTAMP) |
