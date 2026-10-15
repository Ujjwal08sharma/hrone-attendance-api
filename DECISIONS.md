# DECISIONS.md  (about 150-200 words in total - plain language, your own words)

1. **Indexes.** Which indexes did you create, and which query or guarantee does each one serve? Which one did you consider and reject?
Indexes. Planned: a unique compound index on attendance\_logs (emp\_code, date) so one record per employee per day is guaranteed by the database and lookups by employee and date are fast. A second index on (date, status) for the date and status filters in the list endpoint. On employees, a unique index on emp\_code. I considered a separate single-field index on date and rejected it, because the compound index already covers date-prefix queries. Note: I ran out of time, so I have not verified which of these are created at startup.

2. **Punch-in race.** Two identical punch-in requests arrive at the same instant. Walk through exactly what happens in your code.
Punch-in race. The correct design is to not check first and insert later, because two requests can both pass the check. Instead, both requests try insert\_one directly. The unique index on (emp\_code, date) lets MongoDB accept exactly one. The other gets a DuplicateKeyError, which I catch and return as 409. So exactly one 201 and the rest 409. Note: I have not confirmed that my submitted code does exactly this.

3. **Ties.** Two employees are tied on late minutes at the cutoff of `limit`. What does your leaderboard return, and why?
Ties. Not implemented in my submission. My plan: group late minutes per employee, then use $setWindowFields with $rank (not sort-and-index), so ties share a rank and the next rank is skipped (1, 2, 2, 4). Apply $limit after ranking, so both tied employees at the cutoff are kept if their rank is within the limit, ordered by emp\_code.

4. **Headcount.** How does your department summary make sure employees with zero logs are counted?
Headcount. Not implemented in my submission. My plan: start from the employees collection, filtered by joined\_on on or before the end of the month, then $lookup the logs. Employees with no logs still stay in the pipeline, so they are counted in headcount, with zero present days.

5. **One thing you would change** if this had to serve 100x the data.

100x data. I would add proper pre-aggregated monthly summaries (computed once and stored) instead of aggregating raw logs on every request, and shard or partition attendance\_logs by date or emp\_code.

