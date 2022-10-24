# Module-6-Challenge-Visualisation-Real-Estate
Creating Graphs and Plotting Data on Maps 
# Import the required libraries and dependencies
import pandas as pd
import hvplot.pandas
from pathlib import Path
import os
from cartopy import crs
import geoviews.feature as gf

# Using the read_csv function and Path module, create a DataFrame 
# by importing the sfo_neighborhoods_census_data.csv file from the Resources folder
sfo_data_df = pd.read_csv(Path('.\Resources\sfo_neighborhoods_census_data.csv'), header =0)

# Review the first and last five rows of the DataFrame
display(sfo_data_df.head(5))
display(sfo_data_df.tail(5))

# Create a numerical aggregation that groups the data by the year and then averages the results.
housing_units_by_year = sfo_data_df.groupby('year').mean()

# Review the DataFrame
housing_units_by_year

# Ploting the Data frame in hvplot adjusting the scle for professional look
housing_units_by_year['housing_units'].hvplot.bar(label = 'Houseing Units in San Francisco from 2010 to 2016',
    xlabel = 'year',
    ylabel = 'housing_units',
    rot =10).opts(yformatter = '%.0f',ylim=(365000, 385000)) 

# Plot prices_square_foot_by_year. 
# Inclued labels for the x- and y-axes, and a title.
Sales_price_graph = housing_units_by_year['sale_price_sqr_foot'].hvplot.line(
    label = 'Aveerage Price per Square Foot ',
    xlabel = 'year',
    ylabel = 'sale_price_sqr_foot',
    rot =10
   
)
Sales_price_graph

# Plot prices_square_foot_by_year. 
# Inclued labels for the x- and y-axes, and a title.
gross_rent_graph = housing_units_by_year['gross_rent'].hvplot.line(
     label = 'Gross Rent By Year',
    xlabel = 'year',
    ylabel = 'gross_rent',
    rot =10
)
gross_rent_graph

# combined the two graphs together
Sales_price_graph*gross_rent_graph

# Group by year and neighborhood and then create a new dataframe of the mean values
sfo_data_df = sfo_data_df.set_index('year')

prices_by_year_by_neighborhood = sfo_data_df.groupby(['year', 'neighborhood']).mean()
# Review the DataFrame
display(prices_by_year_by_neighborhood.head(2))
display(prices_by_year_by_neighborhood.tail(2))

# Filter out the housing_units
prices_by_year_by_neighborhood = prices_by_year_by_neighborhood[[
     "sale_price_sqr_foot",
     "gross_rent"]]

# Review the first and last five rows of the DataFrame
display(prices_by_year_by_neighborhood.head(5))
#display(prices_by_year_by_neighborhood.tail(5))


# Calculate the mean values for each neighborhood
all_neighborhood_info_df = sfo_data_df.groupby('neighborhood').mean()
all_neighborhood_info_df =pd.DataFrame(all_neighborhood_info_df,index=None)
# Review the resulting DataFrame
all_neighborhood_info_df.head(2)

# Load neighborhoods coordinates data
neighborhood_locations_df = pd.read_csv(Path('./Resources/neighborhoods_coordinates.csv'),index_col ='Neighborhood')
# Review the DataFrame
neighborhood_locations_df.head()

# Using the Pandas `concat` function, joined the 
# neighborhood_locations_df and the all_neighborhood_info_df DataFrame
# The axis of the concatenation is "columns".
# The concat function will automatially combine columns with
# identical information, while keeping the additional columns.
all_neighborhoods_df = pd.concat(
    [neighborhood_locations_df, all_neighborhood_info_df], 
    axis='columns',
    sort=False,
)

# Review the resulting DataFrame
display(all_neighborhoods_df.head(3))
display(all_neighborhoods_df.tail(3))

# Call the dropna function to remove any neighborhoods that do not have data
all_neighborhoods_df = all_neighborhoods_df.reset_index().dropna()

# Rename the "index" column as "Neighborhood" for use in the Visualization
all_neighborhoods_df = all_neighborhoods_df.rename(columns={"index": "Neighborhood"})
# Review the resulting DataFrame
display(all_neighborhoods_df.head())
display(all_neighborhoods_df.tail())

# Create a plot to analyze neighborhood info
map_plot = all_neighborhoods_df.hvplot.points(
    'Lon', 
    'Lat', 
    geo=True, 
    size = 'sale_price_sqr_foot',
    color='gross_rent',
    alpha=0.8,
    tiles='OSM',
    frame_width = 700,
    frame_height = 500
    )

map_plot
