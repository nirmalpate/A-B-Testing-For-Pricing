# The Project: "Simulating the EV Upsell Experiment"

Hypothesis Testing For Pricing (Rental Car)

The Project: "Simulating the EV Upsell Experiment"

-- Synthetic DataSet ----

import pandas as pd
import numpy as np
import scipy.stats as stats

# 1. Set the "Truth" (The hidden parameters of real life)
# We assume Group B performs slightly better (14% vs 12%)
N_A = 5000
N_B = 5000
p_A = 0.12  # 12% conversion rate
p_B = 0.14  # 14% conversion rate

# 2. Generate Synthetic Data
# Create Group A
group_a = pd.DataFrame({
    'user_id': range(1, N_A + 1),
    'group': 'A',
    'converted': np.random.binomial(n=1, p=p_A, size=N_A), # 0 = No Book, 1 = Booked
    'revenue': 0
})

# Create Group B
group_b = pd.DataFrame({
    'user_id': range(N_A + 1, N_A + N_B + 1),
    'group': 'B',
    'converted': np.random.binomial(n=1, p=p_B, size=N_B),
    'revenue': 0
})

# 3. Add Revenue Data (Only for those who converted)
# Let's say EV bookings are worth £150 on average, with some standard deviation
def generate_revenue(row):
    if row['converted'] == 1:
        return np.round(np.random.normal(150, 20), 2) # Avg £150, std dev £20
    return 0

# Combine both groups into one "Raw Data Log"
df = pd.concat([group_a, group_b])
df['revenue'] = df.apply(generate_revenue, axis=1)

# Shuffle rows to make it look like real logs
df = df.sample(frac=1).reset_index(drop=True)

# Show the first 5 rows
print(df.head())


Step 2: The Analysis

# Analyze the results
results = df.groupby('group').agg({
    'user_id': 'count',           # Total Visitors
    'converted': 'sum',           # Total Conversions
    'revenue': 'sum'              # Total Revenue
}).reset_index()

# Calculate Metrics
results['conversion_rate'] = results['converted'] / results['user_id']
results['arpu'] = results['revenue'] / results['user_id'] # Average Revenue Per User

print(results)

The Statistical Test (

# Create a Contingency Table
#          | Converted | Not Converted
# Group A  |    ?      |      ?
# Group B  |    ?      |      ?

contingency_table = pd.crosstab(df['group'], df['converted'])

# Run Chi-Square Test
chi2, p_value, dof, expected = stats.chi2_contingency(contingency_table)

print(f"P-Value: {p_value:.5f}")

# Interpretation Logic
alpha = 0.05
if p_value < alpha:
    print("Result is Statistically Significant! Reject Null Hypothesis.")
    print("The 'Fuel Savings' badge works.")
else:
    print("Result is NOT Significant. Differences might be due to chance.")


  Don't just stop at the code. Add the business interpretation:
"If the test shows that Variant B increased EV bookings by 10%, but decreased Total Revenue by 5% (because users switched from expensive SUVs to cheaper EVs), I would interpret this as a Strategic Trade-off.
If our main company goal is 'Green Fleet Utilization' (perhaps for tax credits or ESG goals), I would recommend rolling it out. But if our goal is purely 'Short-term EBITDA,' I would recommend iterating on the test to perhaps only show the badge on premium EVs to protect our margins."
