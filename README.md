# Snapchat-Ad-Targeting-Analysis
Digital advocacy often operates in a "black box." I wanted to deconstruct how granular personal data—age, interests, and location—is leveraged in social media ad targeting to influence global audiences.

📌 Project Overview
This project investigates the "black box" of digital advocacy by deconstructing how granular personal data is leveraged to influence global audiences. Using Snapchat’s political transparency data, I engineered a data pipeline to analyze 1.7 Billion+ impressions and over $3.8M in ad spend, uncovering the specific algorithmic criteria used to segment and target users.

Interactive Dashboard: Designed a multi-panel Tableau dashboard that allows users to drill down from high-level global spend to specific organizational targeting strategies.(https://public.tableau.com/app/profile/tyrese.dieudonne/viz/SnapchatAdTargetingAnalysis/SnapshotAdsDashboard)

<img width="1920" height="1080" alt="Screenshot 2026-04-27 at 11 34 07 AM" src="https://github.com/user-attachments/assets/0b0160dd-a6b2-41a1-b8c5-af8d1e12f34e" />

📊 The Dataset
The analysis is based on a multi-source dataset comprising over 25,000 records of global political advertising metadata.

Source: Snapchat Political Ads Transparency Report.

Key Attributes: * Financials: Spend (USD), Impressions.

Targeting Metadata: Age brackets, Gender, Language, Interests, and Advanced Demographics.

Geospatial Data: Campaign reach across 29 countries.

Temporal Data: Start/End dates and seasonal spikes correlated with global election cycles.<img width="1920" height="877" alt="Screenshot 2026-04-27 at 11 30 54 AM" src="https://github.com/user-attachments/assets/eb70602f-feba-455f-9f71-796850834232" />

Tech Stack
Data Processing: Python (Pandas) & Excel (Cleaning, handling null values, and normalizing currency).

Data Visualization: Tableau Public (Interactive Storyboarding).

Analysis: Exploratory Data Analysis focused on targeting density and ethical benchmarking.


 Targeting Density: Discovered that 90% of advocacy ads utilize at least two granular targeting criteria (e.g., combining "Interests" with specific "Age Brackets").

Youth Engagement & Ethics: Uncovered that 75% of targeted advocacy spending is directed toward younger demographics, with some campaigns reaching individuals as young as 14.

Geospatial Comparison: Built a cross-market analysis comparing the US and UK, highlighting how "Spending Spikes" align perfectly with major regional political milestones.

The Consumer Sentiment Gap
My analysis indicates a growing friction point: while 75% of advocacy spend is directed at younger demographics, consumer trust in algorithmic transparency is at an all-time low. Users are increasingly concerned about:

Data Sovereignty: How their "Interests" (e.g., political leanings, language) are being harvested without explicit, granular consent.

The "Filter Bubble" Effect: Hyper-targeting creates echo chambers, where users are only exposed to information that reinforces existing biases.

Youth Vulnerability: The high exposure levels of 14–17 year-olds to political messaging necessitate higher ethical standards than standard commercial retail ads.

2. Strategic Recommendations
For Platforms (Snapchat/Social Media):
Implement "Targeting Labels": Much like "Paid for by" labels, platforms should include a "Why am I seeing this?" tooltip that explicitly lists the targeting criteria used (e.g., "You are seeing this because you are interested in 'Environmentalism' and live in 'New Jersey'").

Establish "Youth Protected Zones": Restrict the use of advanced algorithmic segments (behavioral tracking) for users under 18, defaulting instead to broad, non-personal demographic reach.

For Advertisers (Organizations):
Balance Personalization with Privacy: Adopt a "Contextual-First" strategy. Instead of tracking a user's private history, target ads based on the content they are currently viewing to respect privacy while maintaining relevance.

Transparent Retargeting: Disclose when a user is part of a "lookalike" audience, reducing the "creepy factor" of highly specific advocacy ads.

For Regulatory Bodies:
Standardized Disclosure Metadata: Require political advertisers to provide a standardized, machine-readable "Targeting Log" (similar to the CSVs used in this study) to allow for independent third-party auditing of potential bias.

3. The "Bottom Line" Impact
Privacy is now a competitive advantage. Organizations that proactively adopt transparent data practices will see higher long-term engagement and brand loyalty. My analysis proves that we currently have the data to track these trends—the next step is using that data to build a more equitable and transparent digital town square.

Adding a "Lessons Learned" section is a professional touch—it shows you are reflective and capable of identifying the technical "gotchas" that occur in real-world data work.

Here is a breakdown of the lessons learned from this specific Snapchat project:

🧠 Lessons Learned & Technical Retrospective
Handling Sparsity in Metadata: One of the biggest challenges was managing the high percentage of null values in fields like Advanced Demographics and Interests. I learned that in advocacy data, a "null" isn't always missing data; it often signifies Broad Targeting, which is a strategic choice by the advertiser. Differentiating between "missing" and "intentional" was key to accurate ethical reporting.

The Complexity of Multi-Source Joins: Combining four distinct CSV files (Overtime data, Whole data, and Multiple Connections) required a deep understanding of Key Constraints. I had to ensure that joining on Ad ID didn't result in data duplication, which would have artificially inflated the "Total Spend" and "Impressions" metrics.

Visualizing Ethics with Intent: I realized that standard bar charts weren't enough to convey the "Trajectory of Targeting." I had to experiment with Donut Charts and Side-by-Side Comparisons to make the "Targeting Density" (the number of criteria used per ad) visually striking and easy for a non-technical audience to digest.

Data Normalization for Global Insights: Working with 29 different countries meant dealing with varied date formats and regional naming conventions. I learned the importance of Standardizing Start/End Dates in the cleaning phase to ensure that "Seasonal Spikes" correctly aligned with global political events across different time zones.
