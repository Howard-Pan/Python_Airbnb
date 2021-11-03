# Python_Airbnb
Using the dataset from Airbnb to know more about Taipei
# Import the packages that we need.
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

import plotly.offline as py
py.init_notebook_mode(connected = True)
import plotly.graph_objs as go
import plotly.tools as tls
import plotly.figure_factory as ff

import cufflinks

# Import csv file as calendar and take a look at the first 5 rows in the dataframe.
calendar = pd.read_csv('/Users/Howard/Desktop/Airbnb_Taiwan_2021/calendar.csv')
print('我們還有',calendar.date.nunique() ,'天，還有', calendar.listing_id.nunique() ,\
      '不同的清單在我們的calendar中')
print(calendar.date.min(), calendar.date.max())
print(calendar.head().to_string())

# Check the availability of the rooms and make a bar plot.
calendar.available.value_counts().plot(kind = 'bar')
plt.title('Number of available rooms')
plt.xlabel('T = Available and F = Not Available')
plt.ylabel('Numbers')
plt.show()

# If there are not many rooms to book, then we would define this at the hot-season for tourism.
# If available = False, meaning it's the hot-season.
hot_calendar = calendar[['date','available']]
hot_calendar['busy'] = hot_calendar['available'].map(lambda x:0 if x == 't' else 1)

# Calculate average level of availability and the line plot.
hot_calendar = hot_calendar.groupby('date')['busy'].mean().reset_index()

hot_calendar['date'] = pd.to_datetime(hot_calendar['date'])

# plt.figure is used to set the width and the height of the plot.
plt.figure(figsize = (10 , 5))
plt.plot(hot_calendar.date,hot_calendar.busy)
plt.title('Airbnb Taipei Calendar')
plt.show()

# Let's see how the price changes with the date.
calendar['date'] = pd.to_datetime(calendar['date'])
# Use str.replace('a', 'b') to change 'a' to 'b'.
calendar['price'] = calendar['price'].str.replace(',','').str.replace('$','').astype(float)

mean_of_month = calendar.groupby(calendar['date'].dt.strftime('%B'), sort = False)['price'].sum()

mean_of_month.plot(kind = 'barh', figsize= (12,7))
plt.xlabel('average monthly price')
plt.ylabel('Month')
plt.show()

# Let's check the weekday, and show it as weekday name. For example, such as Monday, Tuesday.....
# So we use df['date'].dt.day_name()
calendar['day_of_week'] = calendar['date'].dt.day_name()
cats = calendar['day_of_week'].unique().tolist()
price_week = calendar.groupby('day_of_week')['price'].mean().reindex(cats)
price_week.plot(kind = 'line')
plt.title('More people on the weekend')
plt.xlabel('Weekday')
plt.ylabel('Price')
plt.show()



# Let's check the listing dataset.
# Import the packages that we need.
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

import plotly.offline as py
py.init_notebook_mode(connected = True)
import plotly.graph_objs as go
import plotly.tools as tls
import plotly.figure_factory as ff

import cufflinks

# Read csv file 'listing' and see which region has the highest number of rooms.
listing = pd.read_csv('/Users/Howard/Desktop/Airbnb_Taiwan_2021/listings.csv')
groupby_neighborhood = listing.groupby('host_neighbourhood')['id'].count().sort_values(ascending = False)
groupby_neighborhood.plot(kind = 'bar', figsize = (12,6), title = 'Number of room')
plt.xlabel('Region')
plt.ylabel('Number')
plt.show()

# We will draw a distribution plot to see the distribution of the review scores.
sns.displot(listing['review_scores_rating'].dropna(), kde = True)
plt.title('Most people give a relatively high score')
plt.show()

# Let's check the distribution of the room price.
listing['price'] = listing['price'].str.replace(',','').str.replace('$','').astype(float)
print(listing['price'].describe())
sns.displot(listing['price'].dropna(), kde = True, rug= True)
plt. title('The price is relatively low')
plt.show()

# By looking at the listing['price'].describe(), we discovered that the maximum price is 300245.
# We will draw the distribution plot for those rooms which have the price lower than 50000.
sns.displot(listing[listing['price'] < 50000]['price'].dropna(), kde = True, rug = True)
plt.title('For those price below 50000')
sns.despine()
plt.show()

# We think 50000 is still to high, so we would look at the price between 0 and 10000. Then we will draw a histogram.
price_range = listing[(listing['price'] > 0) & (listing['price'] < 10000)]['price']
price_range.hist(bins = 20)
plt.title('The price of the rooms are mostly below 2000')
plt.ylabel('Count')
plt.xlabel('Price')
plt.show()

# We will be using boxplot to see the price range in different region.
price_delete_outlier = listing[listing['price'] < 10000]
sns.boxplot(x = 'host_neighbourhood', y = 'price', data = price_delete_outlier)
plt.title('It seems most of the price is exactly the same', fontsize = 15)
plt.xticks(rotation = 90) # Rotate the words by 90 degree.
plt.show()

# Then we could see how the room category and property type would affect the price.
sns.boxplot(x= 'room_type', y = 'price', data = price_delete_outlier)
plt.show()

sns.boxplot(x= 'property_type', y = 'price', data = price_delete_outlier)
plt.xticks(rotation = 90)
plt.show()

# Use the pivot table and draw boxplots
price_delete_outlier.pivot(values = 'price', columns = 'property_type').plot(kind = 'box')
plt.xticks(rotation = 90)
plt.show()

grouped_df = price_delete_outlier.pivot(columns = 'room_type' , values ='price')
grouped_df.plot(kind = 'hist', bins = 100)
plt.title('Private rooms are more expensive')
plt.show()

# What about the amenity? Does this affect the price?
listing['amenities'] = listing['amenities'].str.replace('[{}]','').str.replace('"','')
split_amenities = listing['amenities'].str.split(',')
all_item_ls = np.concatenate(split_amenities)
top_20 = pd.Series(all_item_ls).value_counts().head(20)
top_20.plot(kind = 'bar', figsize= (18,6))
plt.xticks(rotation = 90)
plt.title('Top 20 amenities')
plt.show()

# Numbers of beds and price in a boxplot.
price_delete_outlier = listing[listing['price'] < 10000]
sns.boxplot(x = 'beds', y = 'price', data = price_delete_outlier)
plt.xticks(rotation = 45)
plt.show()
