# Your Tasks

![Databaseball](../../.gitbook/assets/databaseball%20%282%29%20%283%29%20%283%29%20%283%29.jpg)

In this project we will be working with the commonly-used [Lahman baseball statistics database](http://www.seanlahman.com/baseball-archive/statistics/) \(our friends at the San Francisco Giants tell us they use it!\) The database contains pitching, hitting, and fielding statistics for Major League Baseball from 1871 through 2019. It includes data from the two current leagues \(American and National\), four other "major" leagues \(American Association, Union Association, Players League, and Federal League\), and the National Association of 1871-1875.

At this point you should be able to run SQLite and view the database using either `./sqlite3 -header lahman.db` \(if in the previous section you downloaded a precompiled binary\) or `sqlite3 -header lahman.db` otherwise.

```text
$ sqlite3 lahman.db
SQLite version 3.33.0 2020-08-14 13:23:32
Enter ".help" for usage hints.
sqlite> .tables
```

Try running a few sample commands in the SQLite console and see what they do:

```text
sqlite> .schema people
```

```text
sqlite>  SELECT playerid, namefirst, namelast FROM people;
```

```text
sqlite> SELECT COUNT(*) FROM fielding;
```

## Understanding the Schema

The database is comprised of the following main tables:

```text
People - Player names, date of birth (DOB), and biographical info
Batting - batting statistics
Pitching - pitching statistics
Fielding - fielding statistics
```

It is supplemented by these tables:

```text
  AllStarFull - All-Star appearances
  HallofFame - Hall of Fame voting data
  Managers - managerial statistics
  Teams - yearly stats and standings
  BattingPost - post-season batting statistics
  PitchingPost - post-season pitching statistics
  TeamFranchises - franchise information
  FieldingOF - outfield position data
  FieldingPost- post-season fielding data
  FieldingOFsplit - LF/CF/RF splits
  ManagersHalf - split season data for managers
  TeamsHalf - split season data for teams
  Salaries - player salary data
  SeriesPost - post-season series information
  AwardsManagers - awards won by managers
  AwardsPlayers - awards won by players
  AwardsShareManagers - award voting for manager awards
  AwardsSharePlayers - award voting for player awards
  Appearances - details on the positions a player appeared at
  Schools - list of colleges that players attended
  CollegePlaying - list of players and the colleges they attended
  Parks - list of major league ballparls
  HomeGames - Number of homegames played by each team in each ballpark
```

For more detailed information, see the [docs online](http://www.seanlahman.com/files/database/readme2019.txt).

## Writing Queries

We've provided a skeleton solution file, `proj1.sql`, to help you get started. In the file, you'll find a `CREATE VIEW` statement for each part of the first 4 questions below, specifying a particular view name \(like `q2i`\) and list of column names \(like `playerid`, `lastname`\). The view name and column names constitute the interface against which we will grade this assignment. In other words, _don't change or remove these names_. Your job is to fill out the view definitions in a way that populates the views with the right tuples.

For example, consider Question 0: "What is the highest `era` \([earned run average](https://en.wikipedia.org/wiki/Earned_run_average)\) recorded in baseball history?".

In the `proj1.sql` file we provide:

```sql
CREATE VIEW q0(era) AS
    SELECT 1 -- replace this line
;
```

You would edit this with your answer, keeping the schema the same:

```sql
-- solution you provide
CREATE VIEW q0(era) AS
 SELECT MAX(era)
 FROM pitching
;
```

To complete the project, create a view for `q0` as above \(via copy-paste\), and for all of the following queries, which you will need to write yourself.

You can confirm the test is now passing by running `python3 test.py -q 0`

```text
> python3 test.py -q 0
PASS q0
```

More details on testing can be found in the [Testing](testing.md) section.

### Changes from Lecture

SQLite doesn't support every SQL feature covered in lecture, specifically:

* There is support for `LEFT OUTER JOIN` but not `RIGHT OUTER` or `FULL OUTER`.
  * To get equivalent output to `RIGHT OUTER` you can reverse the order of the tables \(i.e. `A RIGHT JOIN B` is the same as `B LEFT JOIN A`.
  * While it isn't required to complete this assignment, the equivalent to `FULL OUTER JOIN` can be done by `UNION`ing `RIGHT OUTER` and `LEFT OUTER`
* There is no regex match \(`~`\) tilde operator. You can use `LIKE` instead.
* There is no `ANY` or `ALL` operator.

## Your Tasks

### Task 1: **Basics**

**i.** In the `people` table, find the `namefirst`, `namelast` and `birthyear` for all players with weight greater than 300 pounds.

**ii.** Find the `namefirst`, `namelast` and `birthyear` of all players whose `namefirst` field contains a space. Order the results by `namefirst`, breaking ties with `namelast` both in ascending order

**iii.** From the `people` table, group together players with the same `birthyear`, and report the `birthyear`, average `height`, and number of players for each `birthyear`. Order the results by `birthyear` in _ascending_ order.

Note: Some birth years have no players; your answer can simply skip those years. In some other years, you may find that all the players have a `NULL` height value in the dataset \(i.e. `height IS NULL`\); your query should return `NULL` for the height in those years.

**iv.** Following the results of part iii, now only include groups with an average height &gt; `70`. Again order the results by `birthyear` in _ascending_ order.

### Task 2: **Hall of Fame Schools**

**i.** Find the `namefirst`, `namelast`, `playerid` and `yearid` of all people who were successfully inducted into the Hall of Fame in _descending_ order of `yearid`. Break ties on `yearid` by `playerid` \(ascending\).

**ii.** Find the people who were successfully inducted into the Hall of Fame and played in college at a school located in the state of California. For each person, return their `namefirst`, `namelast`, `playerid`, `schoolid`, and `yearid` in _descending_ order of `yearid`. Break ties on `yearid` by `schoolid, playerid` \(ascending\). For this question, `yearid` refers to the year of induction into the Hall of Fame.

* Note: a player may appear in the results multiple times \(once per year in a college in California\).

**iii.** Find the `playerid`, `namefirst`, `namelast` and `schoolid` of all people who were successfully inducted into the Hall of Fame -- whether or not they played in college. Return people in _descending_ order of `playerid`. Break ties on `playerid` by `schoolid` \(ascending\). \(Note: `schoolid` will be `NULL` if they did not play in college.\)

### Task 3: [**SaberMetrics**](https://en.wikipedia.org/wiki/Sabermetrics)

**i.** Find the `playerid`, `namefirst`, `namelast`, `yearid` and single-year `slg` \(Slugging Percentage\) of the players with the 10 best annual Slugging Percentage recorded over all time. For statistical significance, only include players with more than 50 at-bats in the season. Order the results by `slg` descending, and break ties by `yearid, playerid` \(ascending\).

* Baseball note: Slugging Percentage is not provided in the database; it is computed according to a [simple formula](https://en.wikipedia.org/wiki/Slugging_percentage) you can calculate from the data in the database.
* SQL note: You should compute `slg` properly as a floating point number---you'll need to figure out how to convince SQL to do this!
* Data set note: The online documentation `batting` mentions two columns `2B` and `3B`. On your local copy of the data set these have been renamed `H2B` and `H3B` respectively \(columns starting with numbers are tedious to write queries on\).
* Data set note: The column `H` o f the `batting` table represents all hits = \(\# singles\) + \(\# doubles\) + \(\# triples\) + \(\# home runs\), not just \(\# singles\) so you’ll need to account for some double-counting
* If a player played on multiple teams during the same season \(for example `anderma02` in 2006\) treat their time on each team separately for this calculation

**ii.** Following the results from Part i, find the `playerid`, `namefirst`, `namelast` and `lslg` \(Lifetime Slugging Percentage\) for the players with the top 10 Lifetime Slugging Percentage. Lifetime Slugging Percentage \(LSLG\) uses the same formula as Slugging Percentage \(SLG\), but it uses the number of singles, doubles, triples, home runs, and at bats each player has over their entire career, rather than just over a single season.

Note that the database only gives batting information broken down by year; you will need to convert to total information across all time \(from the earliest date recorded up to the last date recorded\) to compute `lslg`. Order the results by `lslg` \(descending\) and break ties by `playerid` \(ascending\)

* Note: Make sure that you only include players with more than 50 at-bats across their lifetime.

**iii.** Find the `namefirst`, `namelast` and Lifetime Slugging Percentage \(`lslg`\) of batters whose lifetime slugging percentage is higher than that of San Francisco favorite Willie Mays.

You may include Willie Mays' `playerid` in your query \(`mayswi01`\), but you _may not_ include his slugging percentage -- you should calculate that as part of the query. \(Test your query by replacing `mayswi01` with the playerid of another player -- it should work for that player as well! We may do the same in the autograder.\)

* Note: Make sure that you still only include players with more than 50 at-bats across their lifetime.

_Just for fun_: For those of you who are baseball buffs, variants of the above queries can be used to find other more detailed SaberMetrics, like [Runs Created](https://en.wikipedia.org/wiki/Runs_created) or [Value Over Replacement Player](https://en.wikipedia.org/wiki/Value_over_replacement_player). Wikipedia has a nice page on [baseball statistics](https://en.wikipedia.org/wiki/Baseball_statistics); most of these can be computed fairly directly in SQL.

_Also just for fun_: SF Giants VP of Baseball Operations, [Yeshayah Goldfarb](https://www.mlb.com/giants/team/front-office/yeshayah-goldfarb), suggested the following:

> Using the Lahman database as your guide, make an argument for when MLBs “Steroid Era” started and ended. There are a number of different ways to explore this question using the data.

\(Please do not include your "just for fun" answers in your solution file! They will break the autograder.\)

### Task 4: **Salaries**

**i.** Find the `yearid`, min, max and average of all player salaries for each year recorded, ordered by `yearid` in _ascending_ order.

**ii.** For salaries in 2016, compute a [histogram](https://en.wikipedia.org/wiki/Histogram). Divide the salary range into 10 equal bins from min to max, with `binid`s 0 through 9, and count the salaries in each bin. Return the `binid`, `low` and `high` boundaries for each bin, as well as the number of salaries in each bin, with results sorted from smallest bin to largest.

* Note: `binid` 0 corresponds to the lowest salaries, and `binid` 9 corresponds to the highest. The ranges are left-inclusive \(i.e. `[low, high)`\) -- so the `high` value is excluded. For example, if bin 2 has a `high` value of 100000, salaries of 100000 belong in bin 3, and bin 3 should have a `low` value of 100000.
* Note: The `high` value for bin 9 may be inclusive\).
* Note: The test for this question is broken into two parts. Use `python3 test.py -q 4ii_bins_0_to_8` and `python3 test.py -q 4ii_bin_9` to run the tests
* Hidden testing advice: we will be testing the case where a bin has zero player salaries in it. The correct behavior in this case is to display the correct `binid`, `low` and `high` with a `count` of zero, NOT just excluding the bin altogether.

Some useful information:

* You may find it helpful to use the provided helper table containing all the possible `binid`s. We'll only be testing with these possible binid's \(there aren't any hidden tests using say, 100 bins\) so using the hardcoded table is fine
* If you want to take the [floor ](https://en.wikipedia.org/wiki/Floor_and_ceiling_functions)of a positive float value you can do `CAST (some_value AS INT)`

**iii.** Now let's compute the Year-over-Year change in min, max and average player salary. For each year with recorded salaries after the first, return the `yearid`, `mindiff`, `maxdiff`, and `avgdiff` with respect to the previous year. Order the output by `yearid` in _ascending_ order. \(You should omit the very first year of recorded salaries from the result.\)

**iv.** In 2001, the max salary went up by over $6 million. Write a query to find the players that had the max salary in 2000 and 2001. Return the `playerid`, `namefirst`, `namelast`, `salary` and `yearid` for those two years. If multiple players tied for the max salary in a year, return all of them.

* Note on notation: you are computing a relational variant of the [argmax](https://en.wikipedia.org/wiki/Arg_max) for each of those two years.

**v.** Each team has at least 1 All Star and may have multiple. For each team in the year 2016, give the `teamid` and `diffAvg` \(the difference between the team's highest paid all-star's salary and the team's lowest paid all-star's salary\).

* Note: Due to some discrepancies in the database, please draw your team names from the All-Star table \(so use `allstarfull.teamid` in the SELECT statement for this\).

## You're done!

Rerun `python3 test.py` to see if you're passing tests. If so, follow the instructions in the next section to submit your work.

