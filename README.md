# Data Analysis for AirBNB New York
#studying the format of the overall dataset, types of all variables, checking and cleaning the dataset.

import numpy as np 
import pandas as pd 
import matplotlib.pyplot as plt 
import seaborn as sns 
import folium
from folium import plugins
from folium.plugins import HeatMap

data = pd.read_csv('../Source/AB_NYC_2019.csv')
data.info()
#studying the format of the overall dataset, types of all variables, checking and cleaning the dataset.
​
import numpy as np 
import pandas as pd 
import matplotlib.pyplot as plt 
import seaborn as sns 
import folium
from folium import plugins
from folium.plugins import HeatMap
​
data = pd.read_csv('../Source/AB_NYC_2019.csv')
data.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 48895 entries, 0 to 48894
Data columns (total 16 columns):
id                                48895 non-null int64
name                              48879 non-null object
host_id                           48895 non-null int64
host_name                         48874 non-null object
neighbourhood_group               48895 non-null object
neighbourhood                     48895 non-null object
latitude                          48895 non-null float64
longitude                         48895 non-null float64
room_type                         48895 non-null object
price                             48895 non-null int64
minimum_nights                    48895 non-null int64
number_of_reviews                 48895 non-null int64
last_review                       38843 non-null object
reviews_per_month                 38843 non-null float64
calculated_host_listings_count    48895 non-null int64
availability_365                  48895 non-null int64
dtypes: float64(3), int64(7), object(6)
memory usage: 6.0+ MB
data.head()
id	name	host_id	host_name	neighbourhood_group	neighbourhood	latitude	longitude	room_type	price	minimum_nights	number_of_reviews	last_review	reviews_per_month	calculated_host_listings_count	availability_365
0	2539	Clean & quiet apt home by the park	2787	John	Brooklyn	Kensington	40.64749	-73.97237	Private room	149	1	9	2018-10-19	0.21	6	365
1	2595	Skylit Midtown Castle	2845	Jennifer	Manhattan	Midtown	40.75362	-73.98377	Entire home/apt	225	1	45	2019-05-21	0.38	2	355
2	3647	THE VILLAGE OF HARLEM....NEW YORK !	4632	Elisabeth	Manhattan	Harlem	40.80902	-73.94190	Private room	150	3	0	NaN	NaN	1	365
3	3831	Cozy Entire Floor of Brownstone	4869	LisaRoxanne	Brooklyn	Clinton Hill	40.68514	-73.95976	Entire home/apt	89	1	270	2019-07-05	4.64	1	194
4	5022	Entire Apt: Spacious Studio/Loft by central park	7192	Laura	Manhattan	East Harlem	40.79851	-73.94399	Entire home/apt	80	10	9	2018-11-19	0.10	1	0
# some data having null values that we need to clean up
data.isnull().sum()
id                                    0
name                                 16
host_id                               0
host_name                            21
neighbourhood_group                   0
neighbourhood                         0
latitude                              0
longitude                             0
room_type                             0
price                                 0
minimum_nights                        0
number_of_reviews                     0
last_review                       10052
reviews_per_month                     0
calculated_host_listings_count        0
availability_365                      0
dtype: int64
#Both last_review and reviews_per_month have over 10k missing values in the dataset
(data['last_review'].isnull()==data['reviews_per_month'].isnull()).all()
True
#for missing values in reviews_per_month, I will assign them as zero.
data.loc[data['reviews_per_month'].isnull(),'reviews_per_month']=0
​
len(set(data['id']))
48895
​
len(set(data['host_id']))
37457
​
data.drop(['id','name','host_name','last_review'], axis=1, inplace=True)
#checking numeric variables
data.describe()
host_id	latitude	longitude	price	minimum_nights	number_of_reviews	reviews_per_month	calculated_host_listings_count	availability_365
count	4.889500e+04	48895.000000	48895.000000	48895.000000	48895.000000	48895.000000	38843.000000	48895.000000	48895.000000
mean	6.762001e+07	40.728949	-73.952170	152.720687	7.029962	23.274466	1.373221	7.143982	112.781327
std	7.861097e+07	0.054530	0.046157	240.154170	20.510550	44.550582	1.680442	32.952519	131.622289
min	2.438000e+03	40.499790	-74.244420	0.000000	1.000000	0.000000	0.010000	1.000000	0.000000
25%	7.822033e+06	40.690100	-73.983070	69.000000	1.000000	1.000000	0.190000	1.000000	0.000000
50%	3.079382e+07	40.723070	-73.955680	106.000000	3.000000	5.000000	0.720000	1.000000	45.000000
75%	1.074344e+08	40.763115	-73.936275	175.000000	5.000000	24.000000	2.020000	2.000000	227.000000
max	2.743213e+08	40.913060	-73.712990	10000.000000	1250.000000	629.000000	58.500000	327.000000	365.000000
data=data[data['price']>0]
data=data[data['minimum_nights']<=365]
​
data.reset_index(drop=True,inplace=True)
#Using describe() can help me understand the range of possible values for each variable
​
#Analysis:
​
#in this dataset, there are some abnormalities. The first one is that there are records with “price” = 0 which is 
#unrealistic as this means no need to pay to stay. So these records should be removed.
​
#Any records with “minimum_nights” larger than 365 will also be removed.
​
data.info()
​
#data after cleannig below
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 48870 entries, 0 to 48869
Data columns (total 12 columns):
host_id                           48870 non-null int64
neighbourhood_group               48870 non-null object
neighbourhood                     48870 non-null object
latitude                          48870 non-null float64
longitude                         48870 non-null float64
room_type                         48870 non-null object
price                             48870 non-null int64
minimum_nights                    48870 non-null int64
number_of_reviews                 48870 non-null int64
reviews_per_month                 38827 non-null float64
calculated_host_listings_count    48870 non-null int64
availability_365                  48870 non-null int64
dtypes: float64(3), int64(6), object(3)
memory usage: 4.5+ MB
data['neighbourhood_group'].unique()
array(['Brooklyn', 'Manhattan', 'Queens', 'Staten Island', 'Bronx'],
      dtype=object)
​
set(data['neighbourhood_group'])
{'Bronx', 'Brooklyn', 'Manhattan', 'Queens', 'Staten Island'}
#distribution of price in each neighbourhood_group and across all.
data.groupby('neighbourhood_group')['price'].describe()
count	mean	std	min	25%	50%	75%	max
neighbourhood_group								
Bronx	1090.0	87.577064	106.725371	10.0	45.0	65.0	99.0	2500.0
Brooklyn	20089.0	124.452238	186.922112	10.0	60.0	90.0	150.0	10000.0
Manhattan	21654.0	196.888011	291.421157	10.0	95.0	150.0	220.0	10000.0
Queens	5664.0	99.493997	167.125802	10.0	50.0	75.0	110.0	10000.0
Staten Island	373.0	114.812332	277.620403	13.0	50.0	75.0	110.0	5000.0
#Analysis:
#The numbers of Airbnb is not even
#Most Airbnb houses are in either Manhattan or Brooklyn.
#However, these two regions are also the highest prices among the five regions
#A possible reason is that because the demands in these regions are high, 
#causing more hosts to rent out their rooms or apartments.
​
len(set(data['neighbourhood']))
221
#Analysis:
#Most Airbnb houses are in either Manhattan or Brooklyn.
#However, these two regions are also the highest prices among the five regions
#A possible reason is that because the demands in these regions are high, 
#causing more hosts to rent out their rooms or apartments.
#The following breakdown show more accurately on the price
​
data.pivot_table(index='neighbourhood_group',columns='room_type',values='price',aggfunc='mean')
room_type	Entire home/apt	Private room	Shared room
neighbourhood_group			
Bronx	127.506596	66.890937	59.800000
Brooklyn	178.356844	76.553547	50.773723
Manhattan	249.276359	116.776622	88.933194
Queens	147.031996	71.762456	69.020202
Staten Island	173.846591	62.292553	57.444444
#Analysis:
#In average, you can get a private room in Brooklyn but you can’t even afford a shared room in Manhattan.
#Considerations:
​
# I Decided to use the HeatMap because I want to see which intersections of the categorical values
#have higher concentration of the data compared to the others. (As a preset before starting the jupyter user should run the following 
# commnad cmd: pip install folium)
ny_map = folium.Map(location=[40.7, -74],zoom_start =10)
data_loc= data[['latitude','longitude']].values
data_loc =data_loc.tolist()
hm = plugins.HeatMap(data_loc)
hm.add_to(ny_map)
ny_map

#MapAnalysis:
#There is a large red area in Manhattan and Brooklyn
#Performing small breakdown per each neibourhood
​
for gp in set(data['neighbourhood_group']):
    print(data.loc[data['neighbourhood_group']==gp,].groupby(['neighbourhood_group','neighbourhood']).agg({'price':['count','mean']}).sort_values(by=('price', 'mean'),ascending=False).head())
    print()
                                      price            
                                      count        mean
neighbourhood_group neighbourhood                      
Manhattan           Tribeca             177  490.638418
                    Battery Park City    69  367.086957
                    Flatiron District    80  341.925000
                    NoHo                 78  295.717949
                    SoHo                358  287.103352

                                    price            
                                    count        mean
neighbourhood_group neighbourhood                    
Queens              Neponsit            3  274.666667
                    Breezy Point        3  213.333333
                    Jamaica Estates    19  182.947368
                    Arverne            77  171.779221
                    Belle Harbor        8  171.500000

                                   price       
                                   count   mean
neighbourhood_group neighbourhood              
Staten Island       Fort Wadsworth     1  800.0
                    Woodrow            1  700.0
                    Prince's Bay       4  409.5
                    Randall Manor     19  336.0
                    Willowbrook        1  249.0

                                     price            
                                     count        mean
neighbourhood_group neighbourhood                     
Brooklyn            Sea Gate             7  487.857143
                    Cobble Hill         99  211.929293
                    Brooklyn Heights   154  209.064935
                    DUMBO               36  196.305556
                    Vinegar Hill        34  187.176471

                                   price            
                                   count        mean
neighbourhood_group neighbourhood                   
Bronx               Riverdale         11  442.090909
                    City Island       18  173.000000
                    Spuyten Duyvil     4  154.750000
                    Eastchester       13  141.692308
                    Unionport          7  137.142857

#Analysis:
# The above showing the top 5 most expensive neighborhoods in each group.
#Next variable to be studied is “minimum_nights”
data.groupby('neighbourhood_group')['minimum_nights'].describe()
count	mean	std	min	25%	50%	75%	max
neighbourhood_group								
Bronx	1090.0	4.563303	15.638775	1.0	1.0	2.0	3.0	365.0
Brooklyn	20089.0	5.894569	14.533385	1.0	2.0	3.0	5.0	365.0
Manhattan	21654.0	8.345617	18.824170	1.0	1.0	3.0	6.0	365.0
Queens	5664.0	5.010240	11.952636	1.0	1.0	2.0	3.0	365.0
Staten Island	373.0	4.831099	19.727605	1.0	1.0	2.0	3.0	365.0
#Analysis:
#As all records with ‘minimum_nights’ larger than 365 were removed already, the max is only 365.
#As shown, over 25% of Airbnb place only require 1 night and over half only 
#require 2 or 3 nights which fits in the original principle of Airbnb service, a short term accommodation.
#Next i will analyse reviews_per_month since this can eliminate the effect of duration listing on Airbnb.
for gp in set(data['neighbourhood_group']):
    print(gp)
    print(data[data['neighbourhood_group']==gp][['price','reviews_per_month']].corr())
    print()
Manhattan
                      price  reviews_per_month
price              1.000000          -0.011292
reviews_per_month -0.011292           1.000000

Queens
                      price  reviews_per_month
price              1.000000          -0.039539
reviews_per_month -0.039539           1.000000

Staten Island
                      price  reviews_per_month
price              1.000000          -0.102423
reviews_per_month -0.102423           1.000000

Brooklyn
                      price  reviews_per_month
price              1.000000          -0.015322
reviews_per_month -0.015322           1.000000

Bronx
                     price  reviews_per_month
price              1.00000           -0.00954
reviews_per_month -0.00954            1.00000

sns.distplot(data['reviews_per_month'],kde=False).astype(int)
plt.ylabel('Number of records').astype(int)
plt.show()
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-24-bb32c621d74f> in <module>
----> 1 sns.distplot(data['reviews_per_month'],kde=False).astype(int)
      2 plt.ylabel('Number of records').astype(int)
      3 plt.show()

//anaconda3/lib/python3.7/site-packages/seaborn/distributions.py in distplot(a, bins, hist, kde, rug, fit, hist_kws, kde_kws, rug_kws, fit_kws, color, vertical, norm_hist, axlabel, label, ax)
    213     if hist:
    214         if bins is None:
--> 215             bins = min(_freedman_diaconis_bins(a), 50)
    216         hist_kws.setdefault("alpha", 0.4)
    217         if LooseVersion(mpl.__version__) < LooseVersion("2.2"):

//anaconda3/lib/python3.7/site-packages/seaborn/distributions.py in _freedman_diaconis_bins(a)
     37         return int(np.sqrt(a.size))
     38     else:
---> 39         return int(np.ceil((a.max() - a.min()) / h))
     40 
     41 

ValueError: cannot convert float NaN to integer


​
