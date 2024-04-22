# Supplier Categorization Project - Arrow2

This project represents a strategic initiative to streamline Arrow Electronics' supply chain operations. With the Arrow2 project, the team set out to significantly enhance the efficiency of supplier management by leveraging advanced data analysis techniques and automation. Arrow, a Fortune 500 leader in the distribution of electronic components with a diverse product portfolio, has seen the critical importance of an efficient supply chain through its remarkable $37 billion sales achievement in 2022. This project is designed to classify suppliers accurately and remove duplicate entries from a substantial database, ensuring integrity and optimization of the procurement process.

# **Deliverables:**

- An Automated Identification Tool/Script for pinpointing duplicate suppliers.
- A Cleaned and Consolidated Supplier Database that is free of inaccuracies and redundant entries.
- A Comprehensive Report detailing methodologies, findings, and actionable insights, to be shared via GitHub and presentations.
- A series of Visualizations and Dashboards providing clear insights into supplier data, facilitating easier decision-making and reporting.
- Implementation of a Three-tier Data Validation and Cleaning System:

Tier 1: Tackles the US supplier subset with a focus on removing duplicates and resolving clear data issues.

Tier 2: Catches and rectifies anomalies unaddressed by Tier 1 using a refined algorithm for data validation.

Tier 3: Delves into data that remains unresolved post Tiers 1 and 2, earmarked for further research and external resolution efforts.

# **Project Highlights:**

• Development of a data validation algorithm to ensure accuracy.

• Conversion of character encodings to maintain data quality (Corrupt characters are translated correctlty to UTF-8 output).

• Structuring of data outputs for clear and actionable reporting such as duplicate counts, probability scores, and unique identifiers.

• Efficient handling of the automation code, it can be utilized not only for the U.S. supplier subset but for the whole dataset of 117k entries.

# Trials To Reference:**

Trials   

# **Failures to Reference

• Uncompleted Task (Spell checking automation for the "Invoice Supplier City" column)

Packages That Didn't Work: 

-pyspellchecker
-SymsSpell
-RapidFuzz

Other Methods That Didn't Work:

- Similarity scoring by matches a dictionary from a source of U.S. cities pulling from Excel, CSV, and Textfile.
- Implementation of a Binpackage utilizing LargeLanguageModels of data relating to U.S. cities to use as a reference.
