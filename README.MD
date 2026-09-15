# Demystifying Postgres

In this experiment we're going to demystify Postgres behavior by hammering
a bulky 30 million row table with no indexes and seeing what actually
happens under real concurrent load. No pg_sleep, no guesswork just raw
seq scans, real CPU contention, and pool sizes fighting it out. we will see real production workloads

Should be fun to see what the numbers actually conclude.
