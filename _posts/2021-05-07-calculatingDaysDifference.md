---
layout: post
title: Lagging Around - Getting Previous Date Value and Getting the Days Difference
tags: [SAS, Dates, LAG, INTCK]
thumbnail-img: /assets/img/DaysDifferenceUsingSAS.png
comments: true
---

We often need to calculate how many days have passed from an index date or prior date. This is a common procedure required with any data containing dates, and it is a frequently asked question on the SAS forums.

One way to accomplish this is through the `LAG` function in SAS. `LAG` grabs the previous record's value and stores it in a new field.

Let's first create a fake data set that mimics medical claim records. Do not be afraid to run the code below.

```sas
/* The DATA step below creates patient ID's and dates. */

data have;
	do patid = 1 to 10000;
		do date = "01JAN2000"d to "31DEC2020"d;
			output;
		end;
	end;
	format date mmddyy10.;
run;

/* The PROC below selects a sample of 5000 people and outputs a data set called have_sampled. */
/* Note the seed option is set to 1225 (numbers of UF COP address). This will create a consistent data set. */

proc surveyselect data = have 
	out = have_sampled
		seed = 1225
		sampsize = 5000;
run;
```

To show what went on, let's run a `PROC PRINT`.

```sas
/* NOOBS suppresses default printing of OBS column. */

proc print data = have_sampled (obs = 10) noobs;
run;

/* Data below */

patid 	date
3 	08/24/2012
6 	04/18/2020
7 	12/06/2014
7 	02/16/2017
8 	01/18/2003
9 	04/13/2004
10 	01/30/2010
11 	03/13/2000
14 	05/30/2012
15 	06/22/2011
```

You can see that patient 7 has two records. This is an instance where we would want to calculate how many days there are between two dates for patient 7.

Before you can do that, though, your data need to be sorted by 'patid' and 'date' in ascending order to get the earliest record on top for a given patient.

Use `PROC SORT` to sort your data.

```sas
proc sort data = have_sampled;
	by patid date;
run;
```

Now that our data are sorted, we need to invoke by-group processing to make SAS recognize that we want our data cased by 'patid'.

By-group processing is invoked via the `BY` statement. A corollary of the `BY` statement is the `FIRST.` and `LAST.` statements which SAS internally marks as the first and last records for a given patient. I use it below to clear out the lagged date value from other records.

*NOTE* Even though you have invoked by-group processing, it doesn't necessarily mean that SAS automatically recognizes that previous values ***could*** be for another patient.

Accordingly, you must invoke the `MISSING` function to clear these result if it's the first record for a given patient.

```sas
data have_sampled_days_diff;
	set     have_sampled;
	format  lag_date mmddyy10.;
	by      patid; /* Here is the BY statement - it tells SAS we want results cased by 'patid'. */
	
			lag_date = lag(date); /* Invoking the LAG function to take previous record REGARDLESS if it's the same patient. */
			if first.patid then call missing(lag_date); /* If it's the first record for a given patient , clear out the value for 'lag_date'--they cannot have a lag value if it's the first record. */
			if not missing(lag_date) then do; /* This IF-THEN-DO prevents a NOTE in the log showing that missing values were generated. */
				difference = intck("days", lag_date, date, "c"); /* The INTCK function can compute days, years, etc. You first call the time-unit you want, followed by the earlier date or lagged date, and then the current date value. The "c" option requests SAS to compute continuous days to get the complete number of days between the two dates. */
			end; /* You MUST end the IF-THEN-DO statement. */
run;
```

Again, let's run a small `PROC PRINT` to see exactly what happened.

```sas
proc print data = have_sampled_days_diff (obs = 10) noobs;
run;

patid 	date 	    lag_date 	difference
3 	08/24/2012 	. 	.
6 	04/18/2020 	. 	.
7 	12/06/2014 	. 	.
7 	02/16/2017  12/06/2014  803
8 	01/18/2003 	. 	.
9 	04/13/2004 	. 	.
10 	01/30/2010 	. 	.
11 	03/13/2000 	. 	.
14 	05/30/2012 	. 	.
15 	06/22/2011 	. 	.
```

We see that patient 7 had 803 days between their visit in 2014 and their visit in 2017. What about if a patient has multiple records?

I'm going to write a quick `PROC SQL` query since it's relatively easy to get this information. You don't need to know anything about `PROC SQL`, this just pulls individuals with more than two records.

```sas
/* This pulls all records for a given patient where they have more than two records. */

proc sql outobs = 19;
	select		*
	from		have_sampled_days_diff
	group by 	patid
	having		count(patid) > 2
	order by	patid, date;
quit;

/* The data */

patid 	date 	        lag_date        difference
158 	02/20/2003 	. 	        .
158 	03/03/2015 	02/20/2003 	4394
158 	11/06/2020 	03/03/2015 	2075
178 	10/07/2007 	. 	        .
178 	03/08/2008 	10/07/2007 	153
178 	04/28/2010 	03/08/2008 	781
325 	02/16/2000 	. 	        .
325 	01/09/2001 	02/16/2000 	328
325 	03/03/2009 	01/09/2001 	2975
385 	02/04/2006 	. 	        .
385 	08/18/2016 	02/04/2006 	3848
385 	01/19/2018 	08/18/2016 	519
501 	03/14/2017 	. 	        .
501 	03/25/2018 	03/14/2017 	376
501 	01/23/2019 	03/25/2018 	304
504 	06/22/2011 	. 	        .
504 	03/02/2012 	06/22/2011 	254
504 	06/14/2014 	03/02/2012 	834
504 	12/09/2015 	06/14/2014 	543
```

As you can see, we are consistently calculating the days difference between the current record and the prior record for a given patient. Adding one to the total is common here, but should be discussed or determined on a case-by-case basis.

**TLDR:**
- Sort your data by ID and your date.
- Create a new data set in a `DATA` step. `SET` the sorted data set.
- Use `BY` statement to case on 'patid'.
- Use `LAG` to get previous value. This gets the last record no matter the 'patid'!
- Use `FIRST` and `MISSING` to clear out the value obtained from the previous step for the patient's first record. 
- Use `IF-THEN` processing to minimize notes in the log and only compute the days difference metric on those records that have values.
- Use `INTCK` to calculate the number of days between the patient's current record's date and the last date.
- Make your decision as to what you need to do!

Do you not have a SAS license? No problem! SAS is freely available [here](https://welcome.oda.sas.com/home). You will need to create an account.

Also, here are some additional resources that may be helpful if you want to truly understand what is going on underneath the hood.

- The program data vector (PDV) 
  - [Processing a DATA Step: A Walkthrough](https://v8doc.sas.com/sashtml/lrcon/z0961108.htm)
- By-group processing
  - [By-Group Processing](https://v8doc.sas.com/sashtml/lrcon/z1330158.htm)
- Fun little blog on INTCK and its counterpart, INTNX.
  - [INTCK and INTNX: Two essential functions for computing intervals between dates in SAS](https://blogs.sas.com/content/iml/2017/05/15/intck-intnx-intervals-sas.html)
- Some alright SAS documentation on `LAG`
  - [LAG Function](https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/lefunctionsref/n0l66p5oqex1f2n1quuopdvtcjqb.htm)
