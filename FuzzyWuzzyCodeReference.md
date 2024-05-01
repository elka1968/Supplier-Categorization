# Supplier Categorization 'FuzzyWuzzy' Code Reference
```
import pandas as pd
from rapidfuzz import process, fuzz
from alive_progress import alive_bar
import time
import re

def remove_trailing_numbers(name):
    if pd.isna(name):
        return name
    name_without_digits = ''.join(filter(lambda x: not x.isdigit(), name))
    return re.sub(r'\s*\d+\s*$', '', name_without_digits)

def remove_special_characters(name):
    if pd.isna(name):
        return name
    return re.sub(r'[^a-zA-Z0-9\s]', '', name)

def extract_store_numbers(name):
    if pd.isna(name):
        return ''
    match = re.search(r'\d+$', str(name))
    if match:
        return match.group()
    else:
        return ''

def remove_commas_and_ellipses(name):
    if pd.isna(name):
        return name
    return name.replace(',', '').replace('...', '').replace('..', '').replace('.', '')

def replace_supplier_names(df_u, df_d):
    supp_join_to_supplier = {}
    for index, row in df_d.iterrows():
        supp_join1 = row['Supp_Join1']
        supp_join2 = row['Supp_Join2']
        supplier_name = row['SUPPLIER_NAME']
        if not pd.isna(supp_join1):
            supp_join_to_supplier[supp_join1] = supplier_name
        if not pd.isna(supp_join2):
            supp_join_to_supplier[supp_join2] = supplier_name

    print(f"Building Supplier Dictionary...")
    with alive_bar(len(df_u)) as bar:
        for index, row in df_u.iterrows():
            supp_join = row['supp_join']
            if supp_join in supp_join_to_supplier:
                supplier_name = supp_join_to_supplier[supp_join]
                df_u.at[index, 'Supplier_Normalized'] = supplier_name
                df_u.at[index, 'Supplier_Type'] = 'Old'
            else:
                df_u.at[index, 'Supplier_Type'] = 'New'
            bar()

    return df_u


def detect_and_add_duplicates_info(input_file, name_column, city_column, similarity_threshold=95):
    df = pd.read_csv(input_file)

    # Read Default_Supplier_Category CSV file
    default_supplier_df = pd.read_csv(Default_Supplier_Category)

    # Convert all columns to strings to handle NaN values
    df = df.astype(str)
    default_supplier_df = default_supplier_df.astype(str)

    # Remove trailing numbers from 'Supplier_Normalized'
    df[name_column] = df[name_column].apply(remove_trailing_numbers)

    # Remove special characters from 'Supplier_Normalized'
    df[name_column] = df[name_column].apply(remove_special_characters)

    # Remove commas and ellipses from 'Supplier_Normalized'
    df[name_column] = df[name_column].apply(remove_commas_and_ellipses)

    # Create a new column for normalized supplier names and capitalize all entries
    df['Supplier_Normalized'] = df[name_column].str.upper()

    # Replace supplier names based on supp_join and classify suppliers as old or new
    df = replace_supplier_names(df, default_supplier_df)

    # Extract store numbers from 'Invoice_Supplier_Name'
    df['Store_Numbers'] = df['Invoice_Supplier_Name'].apply(extract_store_numbers)

    # Sort DataFrame by the name column
    df.sort_values(by=name_column, inplace=True)

    # Identify similar supplier names using RapidFuzz
    print(f"Identifying duplicate supplier names using RapidFuzz...")
    similar_names_mapping = {}
    unique_supplier_names = df['Supplier_Normalized'].dropna().unique()
    with alive_bar(len(unique_supplier_names)) as bar:
        for name in unique_supplier_names:
            # Exclude NaN values only when applying fuzzy matching
            matches = process.extract(name, unique_supplier_names, scorer=fuzz.ratio,
                                      limit=None, score_cutoff=similarity_threshold)
            for match in matches:
                matched_name = match[0]
                score = match[1]
                if pd.notna(matched_name) and pd.notna(name):
                    # Add a more specific condition to control the matching
                    if score > similarity_threshold:  # You can adjust the threshold as needed
                        similar_names_mapping[matched_name] = (name, score)  # Include the score in the mapping
            time.sleep(0.01)
            bar()

    # Remove commas and ellipses from 'Supplier_Normalized' again
    df[name_column] = df[name_column].apply(remove_commas_and_ellipses)

    # Create a new column for normalized supplier names and capitalize all entries again
    df['Supplier_Normalized'] = df[name_column].str.upper()

    # Map similar names to create a column Unique_Identifier
    df['Unique_Identifier'] = df.apply(lambda
                                        row: f"{row['Supplier_Normalized']}_{row['Store_Numbers']}_{row[city_column]}_{row['Invoice_Supplier_Country']}" if pd.notna(row[city_column]) else f"{row['Supplier_Normalized']}_{row['Store_Numbers']}_{row['Invoice_Supplier_Country']}",
                                        axis=1)

    # Remove double underscores and replace them with single underscores
    df['Unique_Identifier'] = df['Unique_Identifier'].str.replace('__', '_')

    # Create a column for the probability score
    df['Probability_Score'] = df['Supplier_Normalized'].map(
        lambda x: similar_names_mapping.get(x, (None, 0))[1])  # Set default score to 0 if no match found

    # Count the amount of duplicate instances based on Unique_Identifier
    df['Duplicates_Count'] = df.groupby('Unique_Identifier')['Unique_Identifier'].transform('size')

    # Save duplicates information to Excel file
    output_file_duplicates = "/Users/milan/OneDrive/Desktop/duplicates_info_CUB/duplicates_info_CUB_full[RapidFuzz].xlsx"
    df.to_excel(output_file_duplicates, index=False)
    print(f"Duplicates information has been saved to '{output_file_duplicates}'.")

    # Filter entries with probability score less than 100%
    df_below_threshold = df[df['Probability_Score'] < 100]

    # Save filtered entries to another Excel file
    output_file_below_threshold = "/Users/milan/OneDrive/Desktop/duplicates_info_CUB/duplicates_info_manualReview[RapidFuzz].xlsx"
    df_below_threshold.to_excel(output_file_below_threshold, index=False)
    print(f"Entries with probability score less than 100% have been saved to '{output_file_below_threshold}'.")

# Example usage:
input_file = '/Users/milan/OneDrive/Desktop/duplicates_info_CUB/CUB_Tiny_Test.csv'
name_column = 'Supplier_Normalized'
Default_Supplier_Category = '/Users/milan/OneDrive/Desktop/duplicates_info_CUB/Default_Supplier_Category_fix.csv'
city_column = 'Invoice_Supplier_City'
detect_and_add_duplicates_info(input_file, name_column, city_column)


```
# Code line 132 should be changed appropriatley to your desktops file
# Code line 140 should be changed appropriatley to your desktops file
# Code line 145 should be changed appropriatley to your desktops file/matching directory of your desktop
# Code line 147 should be changed appropriatley to your desktops file/matching directory of your desktop

