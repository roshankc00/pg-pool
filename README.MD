# Demystifying Postgres

In this experiment we're going to demystify Postgres behavior by hammering
a bulky 30 million row table with no indexes and seeing what actually
happens under real concurrent load. No pg_sleep, no guesswork just raw
seq scans, real CPU contention, and pool sizes fighting it out. we will see real production workloads

Should be fun to see what the numbers actually conclude.

## My machine

- CPU: Apple M4
- Physical cores: 10
- Logical cores: 10
- RAM: 16 GB
- Disk: SSD
- macOS: 15.4.1
- Postgres: 18.0 (Postgres.app)

## Disclaimer

These numbers came off my own machine — 10 Mac cores, this RAM, this disk
speed, this OS scheduler, all of it baked in. Run this on a different
machine and you'll get different curves, maybe even a different "optimal"
pool size. Treat this as one data point showing the shape of the behavior,
not a universal number to copy into your own config.

- CPU: Apple M4
- Physical cores: 10
- Logical cores: 10
- RAM: 16 GB
- Disk: SSD
- macOS: 15.4.1
- Postgres: 18.0 (Postgres.app)

## What the hell am I actually trying to do

Basically everyone says "set your pool size to your core count" like it's
gospel, but nobody shows the actual behavior behind that advice. I want to
watch it happen throw real concurrent load at Postgres with zero indexes,
crank the pool size up from 1 to 200, and see with my own eyes where more
connections stop helping and start just getting in each other's way.

No pg_sleep faking it. Real CPU, real contention, real numbers.

## Setup

We spun up a `bench_data` table and dumped 30 million dummy rows into it
random category, random value, a chunky text payload. No index on anything
we'll actually query, on purpose. That's the whole point every query has
to do a real seq scan + aggregate, not a free index lookup.

Loaded it in 1M row batches into an UNLOGGED table just to make the _dumping_
fast doesn't affect how queries read the data later. Took about 55s,
landed at ~4GB on disk.
