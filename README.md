# How to run pipeline
The actual “pipeline” is essentially the Pipeline notebook that triggers the individual pieces. Run the “pipeline” notebook to get all the notebook outputs. Open + run the individual notebooks to see how they are ran. All dependencies are included in the requirements.txt. See Reveel Project Process.jpeg for a visual look into how I would set this pipeline up in a real data orchestrating tool

# Assumptions made
If a company has a client_id entry as two different tiers, then I am going to assume they signed up for the higher available tier available. Meaning that I do not need to keep records of the lower tier as long as the client_id is the same for both entries
For missing tier: If a company does not have a value for tier, but are labeled as ACTIVE, then I am going to assume that BRONZE is the entry level tier and those active companies are BRONZE tier
For missing status and active_flag: Im going to assume client_id is INACTIVE and active_flag is N as long as tier is NULL because if the company were to be active, it would of had a tier
What is really throwing me off is how INACTIVE client_id made orders AFTER the cli_join_dt. I really thought about how is it possible that a company made an order while they are in inactive. The most logical way for me to reason this is that the company made all these orders before they became inactive
Min invoice_date: 2024-01-01
I did think about forward filling these missing values but there wasnt a clear pattern to follow where I was able to assume a story to the reason of these missing values. Forward filling would guarantee inaccuracy in this case

# Example outputs for analysis queries
##### For a summarized view of analysis queries outputs, please look into "./notebooks/analysis questions.ipynb" notebook. <br>
##### For a detailed view of analysis queries outputs, please look into "./output_tables/answer*.csv" files
 - 1. Basic: Which top 5 clients have the largest total invoice amounts outstanding?
   - 'Stark Partners', 'Red Logistics', 'Wayne Group', 'Umbrella Industries', 'Nimbus Holdings'

 - 2. Intermediate: Show the month-over-month invoice growth per client for 2024–2025.
   - 

- 3. Discount Scenario: Show total costs for each client, if discounts were applied:
  - 20% off GROUND
     - Then, who are the new top 5 spenders?
  - 30% off FREIGHT
     - Then, who are the new top 5 spenders?
  - 50% off 2 DAY
     - Then, who are the new top 5 spenders?
- 4. Reclassification Scenario: Suppose all “EXPRESS” shipments were instead billed as “GROUND” (lower cost).
  - What is the total cost savings opportunity per client?
    - 
  - Which clients have >50% savings?
    - 
  - Which clients have >$500k savings?
    - 



# What I would do differently for production code
1.) I would investigate and test tabula-py more before using it in production code and consider other options
 - It did give me early issues and was crashing my kernel
2.) I would definitely add a json file with configs for each table so I can specify which tables I want as CAPS or not
3.) I would not use Jupyter Notebooks to orchestrate this dataflow. It would be on something like Dagster
