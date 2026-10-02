# UCI-Texera-2025
Student Social Media Mental Health Affects and Severity dependent on electronic use. 

Access the Workbook [Here](https://hub.texera.io/user/workflow/2110#PythonUDFV2-operator-616252ce-9660-430e-98ee-8e6dc9193427)

Access the Presentation [Here](https://docs.google.com/presentation/d/1gSYxaiWgZdeshYqwQENjhWHaHtVi9zqw02XdMiWfMTk/edit?usp=sharing)

## Tech Stack 
- Python
- Texera
- CSV File (Kaggle)

## Python 

## Moderation Analysis- Calculate Correlation by Mental Health - Analyzed relationships between social media usage and addiction scores across mental health categories using Python and Pandas.

            #start of python code
            from pytexera import *
            import pandas as pd

            class ProcessTableOperator(UDFTableOperator):
            def process_table(self, table: Table, port: int) -> Iterator[Optional[TableLike]]:
            df: pd.DataFrame = table
        
        # Calculate correlation for each mental health category
        results = []
        
        for category in df["Mental_Health_Category"].unique():
            if pd.isna(category):
                continue
                
            subset = df[df["Mental_Health_Category"] == category]
            
            if len(subset) > 1:
                correlation = subset[["Avg_Daily_Usage_Hours", "Addicted_Score"]].corr().iloc[0, 1]
                
                results.append({
                    "Mental_Health_Category": category,
                    "Correlation_Usage_to_Addiction": correlation,
                    "Sample_Size": len(subset),
                    "Avg_Usage": subset["Avg_Daily_Usage_Hours"].mean(),
                    "Avg_Addiction": subset["Addicted_Score"].mean()
                })
        
        result_df = pd.DataFrame(results)
        yield result_df

## Moderation Analysis - Mental Health Effect: Used Python and statistical modeling to standardize social media usage and mental health variables, create interaction terms, and evaluate relationships with addiction scores using regression analysis

    #start of python code 
    from pytexera import *
    import pandas as pd
    from sklearn.preprocessing import StandardScaler

    class ProcessTableOperator(UDFTableOperator):
    def process_table(self, table: Table, port: int) -> Iterator[Optional[TableLike]]:
        df: pd.DataFrame = table
        
        # Standardize variables for easier interpretation
        scaler = StandardScaler()
        df['Usage_Std'] = scaler.fit_transform(df[['Avg_Daily_Usage_Hours']])
        df['MentalHealth_Std'] = scaler.fit_transform(df[['Mental_Health_Score']])
        
        # Create interaction term
        df['Interaction_UsagexMentalHealth'] = df['Usage_Std'] * df['MentalHealth_Std']
        
        # Additional analysis: effect size by category
        results = []
        for category in df['Mental_Health_Category'].unique():
            if pd.isna(category):
                continue
            subset = df[df['Mental_Health_Category'] == category]
            if len(subset) > 1:
                # Calculate effect size (slope of usage on addiction)
                from scipy import stats
                slope, intercept, r_value, p_value, std_err = stats.linregress(
                    subset['Avg_Daily_Usage_Hours'], 
                    subset['Addicted_Score']
                )
                results.append({
                    'Mental_Health_Category': category,
                    'Effect_Size_Slope': slope,
                    'R_Squared': r_value ** 2,
                    'P_Value': p_value,
                    'Effect_Strength': 'Strong' if abs(slope) > 5 else ('Moderate' if abs(slope) > 2 else 'Weak')
                })
        
        result_df = pd.DataFrame(results)
        yield result_df


