# How to run pipeline
The actual “pipeline” is essentially the Pipeline notebook that triggers the individual pieces. Run the “pipeline” notebook to get all the notebook outputs. Open + run the individual notebooks to see how they are ran. All dependencies are included in the requirements.txt.
<p align="center">
  <img src="ETL_Design.jpeg" alt="Pipeline Diagram" width="500"/>
  <br>
  <em>Pipeline Diagram: End-to-End Data Flow
  <br>This is how I would set this pipeline up in a real data orchestrating tool</em>
</p>

# Example outputs for analysis queries
##### Please see ./analysis questions.ipynb for derivation of answers!
1. Basic: Which top 5 clients have the largest total invoice amounts outstanding?
    - [Show Answer 1 CSV](output_tables/answer1.csv)

2. Intermediate: Show the month-over-month invoice growth per client for 2024–2025.
    - [Show Answer 2 CSV](output_tables/answer2.csv)
3. Discount Scenario: Show total costs for each client, if discounts were applied:
    - 20% off GROUND, then, who are the new top 5 spenders?
      - [Show Answer 3 part A CSV](output_tables/answer3parta.csv) 
    - 30% off FREIGHT, then, who are the new top 5 spenders?
      - [Show Answer 3 part B CSV](output_tables/answer3partb.csv) 
    - 50% off 2 DAY, then, who are the new top 5 spenders?
      - [Show Answer 3 part C CSV](output_tables/answer3partc.csv) 
4. Reclassification Scenario: Suppose all “EXPRESS” shipments were instead billed as “GROUND” (lower cost).
    - What is the total cost savings opportunity per client?
      - [Show Answer 4 part A CSV](output_tables/answer4parta.csv) 
    - Which clients have >50% savings?
      - [Show Answer 4 part B CSV](output_tables/answer4partb.csv)
      - *No clients have saved more than 50%*
    - Which clients have >$500k savings?
      - [Show Answer 4 part C CSV](output_tables/answer4partc.csv) 

# What I would do differently for production code
1. I would investigate and test tabula-py more before using it in production code and consider other options
    - It did give me early issues and was crashing my kernel
2. I would definitely add a json file with configs for each table so I can specify which tables I want as CAPS or not
3. I would not use Jupyter Notebooks to orchestrate this dataflow. It would be on something like Dagster

# Assumptions made & Design Choices
- Originally I wanted to do the following to the clients table:
  - I found that there was rows in the joined clients.csv table where it had the same company_name but a variation where one row had “INACTIVE” and the other had “ACTIVE” status. I wanted to keep the ACTIVE row and drop the INACTIVE one as I was originally assuming that the INACTIVE entry is "expired".
  - I found that some companies had multiple active rows where one tier was higher than the other. I wanted to take the higher tier and drop the lower tier and ranked the tiers in the following: GOLD > SILVER > BRONZE > None
  - I found that some companies had active rows with the same tier, I wanted to take the most recent entry. I assumed that the company bought the same tier again but the system failed to remove the older entry
  - I wanted to drop rows that are inactive companies with NULL values in tier AND invalid active_flag value (not Y or N). As they may create data issues down the pipeline. I considered these companies as invalid and I wanted to enforce that all companies to have Y or N in the active_flag column via a validation check       I created in validate_save_df().
- All these ideas were scrapped after I found entries in the invoice table where orders were made in 2024-01-01 through 2025-12-31 from all the client_ids, **including those that I was targetting to drop.** I really thought about how is it possible that a company made an order while they are in inactive. The most logical way for me to reason this is that the company made all these orders before they became inactive. I thought it was best to not drop any client_ids to honor those orders that were made.

- If a company has a client_id entry as two different tiers, then I am going to assume they signed up for the higher available tier available. Meaning that I do not need to keep records of the lower tier as long as the client_id is the same for both entries
- For missing tier: If a company does not have a value for tier, but are labeled as ACTIVE, then I am going to assume that BRONZE is the entry level tier and those active companies are BRONZE tier
- For missing status and active_flag: Im going to assume client_id is INACTIVE and active_flag is N as long as tier is NULL because if the company were to be active, it would of had a tier
- I did think about forward filling these missing values but there wasnt a clear pattern to follow where I was able to assume a story to the reason of these missing values. Forward filling would guarantee inaccuracy in this case
- For the analysis queries, I am assuming every client is represented per client_id value and not by company_name. I assume that the reason these companies have multiple client_ids is because there may be different business needs required by the company.

- create_rate_sheet notebook
  - I decided to make this notebook to allow the edits of the rate sheet table values. Maybe the company will come up with a new shipment type. Maybe the prices will need to be updated. At the end of the day, it is better to provide an easy option to update the values instead of hard coding them into the notebooks.

