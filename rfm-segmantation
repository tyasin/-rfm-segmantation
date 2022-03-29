import datetime as dt
import pandas as pd
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)
pd.set_option('display.float_format', lambda x: '%.5f' % x)
pd.options.mode.chained_assignment = None


#############################################
# 1.Understanding and Preparing Data
#############################################

# Read the 2010-2011 data in the Online Retail II excel
# Make a copy of the dataframe you created
df_ = pd.read_excel("datasets/online_retail_II.xlsx", sheet_name="Year 2010-2011")
df = df_.copy()

# Examine the descriptive statistics of the dataset
df.head()
df.shape
df.info()
df.describe().T

# Are there any missing observations in the dataset?
# If yes, how many missing observations in which variable?
df.isnull().sum()

# Remove the missing observations from the dataset
df.dropna(inplace=True)

# How many unique items are there?
df["Description"].nunique()

# How mant of each product are there?
df["Description"].value_counts()

# Rank the 5 most ordered products from most to least
df.groupby("Description").agg({"Quantity":"sum"}).sort_values("Quantity", ascending=False).head(5)

# The 'C' in the invoices shows the canceled transactions
# Remove the canceled transactions from the dataset
df = df[~df["Invoice"].str.contains("C", na=False)]

# Create a variable named 'TotalPrice' that represents the total earnings per invoice
df["TotalPrice"] = df["Quantity"] * df["Price"]
df.head()

#############################################
# 2.Calculating RFM Metrics
#############################################

# Recency: Time from customer's last purchase to date
# Frequency: Total number of purchases
# Monetary: Total spend by the customer


df["InvoiceDate"].max()
today_date = dt.datetime(2011, 12, 11)

rfm = df.groupby('Customer ID').agg({'InvoiceDate': lambda date: (today_date - date.max()).days,
                                     'Invoice': lambda num: num.nunique(),
                                     'TotalPrice': lambda TotalPrice: TotalPrice.sum()})
rfm.head()

rfm.columns = ['recency', 'frequency', 'monetary']

rfm = rfm[rfm["monetary"] > 0]
#############################################
# 3.Generating and Converting RFM Scores to Single Variable
#############################################

# Latest date score. Here 1 is the closest date and 5 is the farthest date.
rfm["recency_score"] = pd.qcut(rfm['recency'], 5, labels=[5, 4, 3, 2, 1])

# Shopping frequency score. Here 1 represents the least frequency, 5 the most frequent.
rfm["frequency_score"] = pd.qcut(rfm['frequency'].rank(method="first"), 5, labels=[1, 2, 3, 4, 5])

# The amount of money the customer leaves with the company.
# Here 1 represents the least amount of money and 5 represents the most money.
rfm["monetary_score"] = pd.qcut(rfm["monetary"], 5, labels=[1,2,3,4,5])

rfm["RFM_SCORE"] = (rfm["recency_score"].astype(str) + rfm["frequency_score"].astype(str))

#############################################
# 4.Defining RFM Scores As Segments
#############################################
seg_map = {
    r'[1-2][1-2]': 'hibernating',
    r'[1-2][3-4]': 'at_Risk',
    r'[1-2]5': 'cant_loose',
    r'3[1-2]': 'about_to_sleep',
    r'33': 'need_attention',
    r'[3-4][4-5]': 'loyal_customers',
    r'41': 'promising',
    r'51': 'new_customers',
    r'[4-5][2-3]': 'potential_loyalists',
    r'5[4-5]': 'champions'
}

rfm["segment"] = rfm["RFM_SCORE"].replace(seg_map, regex=True)

#############################################
# 5.Select The Customer IDs Of The "Loyal Customers" Segment and Get The Excel Output
#############################################
rfm[["segment", "recency", "frequency", "monetary"]].groupby("segment").agg(["mean", "count"])

new_df = pd.DataFrame()
new_df["loyal_customer_id"] = rfm[rfm["segment"] == "loyal_customers"].index
new_df.head()

new_df.to_excel("loyal_customers.xls")
