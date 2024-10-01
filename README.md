# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
Import data and perform basic cleaning before begining to manipulate the data, going back to add more cleaning steps
as needed. 

## Process
Identified data types in order to load csv into pgadmin; used pivot tables in excel to show all values in mostly 
empty columns

Looked for duplicate rows I might need to account for while working with the data and discovered the mess that is
the analytics table, which needed dealing with.

Split off secondary analytics table for active use, establish primary key for every table

Perform price adjustment step as instructed, normalizing all prices at the same time
 
Spent probably too long messing around with the questions proposed by the assigment.md under Part 4

Worked through Part 3: Starting With Questions

Had to decide what from my messing around with Part 4 I was going to keep

Kept a version of the 'disinct IDs' question, came up with two others.

Worked through Part 4: Starting With Data

Kept each notepad updated as I worked; there was no way I was going to be able to accurately remember what I had
done in various places at the end

Had a side notepad with a list of every column in each table, as well as a list of all useful (non-empty) columns

Went back at the end to standardize my formatting

## Results
This data set is a mess; two of the tables are essentially completely redundant, analytics is 60% duplicates, there are
multiple empty columns. There is also a lot of missing data, making it very difficult to use the data that exists, and
where there is potentially useable and useful data, its quality is questionable. Some of this could potentially be 
helped by more extensive cleaning, but a lot of it probably isn't. 

You could, with more work that I could get done, get a better idea of information relating to popular catagories, but
right now, there is no quick way to sort those. They seem to have been generated based on navigartion within the website
to reach the page, rather than the item's actual designation.

The most useable data that came out was that most of the confirmed transactions were from the United States, by a huge 
margin, and that the most ordered items were, overall, men's shirts, although it's hard to say where those were being 
purchased from. 

## Challenges 
My computer had a permissions issue that forced me to use the GUI to import the CSVs. 

Trying to get the price values into XX.XX format resulted in having to remake the table 3 times because the calculation 
broke somehow and totally changed the values inside

Made a mistake correcting the city-country mismatch and had to reload the all_sessions table again. 

Got very confused about what exactly P4 wanted from me in terms of which questions to ask and answer, and how many. 
Lost a lot of time trying to figure that out and putting in unnecessary work.

## Future Goals

With more time I would look into cleaning up the category and name values between the various tables, as well as 
confirming the apparently redundant sales_by_sku and products tables could be dropped. I would also actually delete the
empty tables, as well as the one in all_sessions where all the values are identical. 


