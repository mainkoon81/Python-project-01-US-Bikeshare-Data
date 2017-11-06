# Python-project-01-US-Bikeshare-Data

## Multivariate Data Analysis / Machine Learning Playground with Python
----------------------------------------------------------------------
__Data:__ US Bikeshare data (https://www.motivateco.com/use-our-data/)

__Story:__ Comparing the system usage between three large cities: New York City, Chicago, and Washington, DC. What kinds of information would we want to know about in order to make smarter business decisions? If we were a user of the bike-share service, what factors might influence how we would want to use the service? 

__Investigation:__ Any differences within each system by those users that are registered, regular users VS short-term, casual users? It is possible that users who are subscribed to a bike-share system will have different patterns of use compared to users who only have temporary passes.

(__Any other relationships?__ Where do bikers ride? When do they ride? How far do they go? Which stations are most popular? What days of the week are most rides taken on? Where are the most commonly used docks? What are the most common routes? Weather has potential to have a large impact on daily ridership? How much is ridership impacted when there is rain or snow? Are subscribers or customers affected more by changes in weather?)

#### *|Data Wrangling|*
- __package:__ csv, datetime, pprint
- __func:__ `datetime.date()`, `csv.DictReader()`, `csv.DictWriter()`, `.next()`, `.whiteheader()`, `.whiterow()`, `.strftime(%A)`, `.replace(':','')`
- __issue:__ We obtain 3 csv files, but (a)The size of each dataset is too huge to get to the intuitive exploration.(b)There are inconsistencies. Each file has a different way of delivering its data. Even where the information is the same, the column names and formats are sometimes different (Chicago updates with new data twice a year, Washington DC is quarterly, and New York City is monthly). 
- __approach:__ (a)Using DictReader object, starting off by looking at one entry from each of the dataset.(b)Making sure that the data formats across the cities are consistent, and trimming focusing only on the parts of the data we are most interested in - **1)trip duration, 2)starting month, 3)starting hour, 4)day of the week, and 5)user type.** 
- __columns to extract from :__ 
  - **Duration:**  Given to us in seconds (New York, Chicago) or milliseconds (Washington). Need to be given in terms of minutes. 
  - **Month, Hour, Day of Week:**  Ridership volume is likely to change based on the season, time of day, and whether it is a weekday or weekend. We use the start time of the trip to obtain these values. The New York City data includes the seconds in their timestamps, while Washington and Chicago do not. 
  - **User Type:**  Washington divides its users into two types: 'Registered'(annual, monthly) and 'Casual'(24-hour, 3-day, other short-term passes). The New York and Chicago data uses 'Subscriber' and 'Customer' for these groups, respectively. For consistency, we convert the Washington labels to match the other two. There are some missing 'user-type' values in the New York city dataset. Since we don't have enough information to fill these values in, we leave them as-is. 

- **[CODE]**
> **Planning_01.(Data Collecting)**: Define a function that prints the name of the city and returns(extracts) the 'first data point'(second row) from the large csv files (DictReader object) that includes a 'header row'. It prints the city names and the first data point to check now so that the first trip is parsed in the form of a dictionary. And it also returns those values to use when we create a 'dictionary-nesting dictionary' later on....which is important to deal with some iterable situations..
  - DictReader object: When we set up a DictReader object, the first row of the data file is normally interpreted as column names. Every other row in the data file will use those column names as keys, as a dictionary is generated for each row.
  - next(): Return the next row of the reader’s iterable object as a list
  - pprint(): 
```
def print_first_point(filename):
    city = filename.split('-')[2].split('/')[-1]
    print('\nCity: {}'.format(city))
    
    with open(filename, 'r') as f_in:
        trip_reader = csv.DictReader(f_in) 
        first_trip = next(trip_reader) 
        pprint(first_trip)
    return (city, first_trip)

#testing------------------------------------------------------------------------------------------------------------
data_files = ['C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L2/bike-share-analysis/data/NYC-CitiBike-2016.csv',
              'C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L2/bike-share-analysis/data/Chicago-Divvy-2016.csv',
              'C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L2/bike-share-analysis/data/Washington-CapitalBikeshare-2016.csv',]

example_trips = {} ###there we go!###
for i in data_files:
    ct, ftrip = print_first_point(i)
    example_trips[ct] = ftrip 

example_trips['NYC']['tripduration']
example_trips['Washington']['user_type']
example_trips['Chicago']['starttime']
```
<img src="https://user-images.githubusercontent.com/31917400/32451453-ba628d9a-c30e-11e7-8354-5bf0e6351074.jpg" width="250" height="200" />
<img src="https://user-images.githubusercontent.com/31917400/32451451-b9b56e30-c30e-11e7-92d2-bb606920c73b.jpg" width="250" height="290" />

> **Planning_02.(Data Condensing)**: It is observable from the above printout that each city provides different information. We want a bundle of functions that generate new data files with the five columns of interest for each trip - 1)trip duration, 2)starting month, starting hour, day of the week, 3)user type - and a single function that produces a final dictionary that consists of the extracted new data files.
 - func_01. duration_in_min(datum, city) :Takes as input a "dictionary" containing info about a single trip (datum) and its origin city (city) and returns the trip duration in units of minutes.
 - func_02. time_of_trip(datum, city) :Takes as input a dictionary containing info about a single trip (datum) and its origin city (city) and returns the month, hour, and day of the week in which the trip was made.
 - func_03. type_of_user(datum, city) :Takes as input a "dictionary" containing info about a single trip (datum) and its origin city (city) and returns the type of system user that made the trip.
 - func_04. condense_data(in_file, out_file, city) :Takes full data from the specified input file and writes the condensed data to a specified output file. The city argument determines how the input file will be parsed.
 - The csv module reads in all of the data as strings, including numeric values. We will need a function to convert the strings into an appropriate numeric type when making transformations.
```
def duration_in_mins(datum, city):
    if city=='Washington':
        dura_value = float(datum['Duration (ms)'])/60000
    else: 
        dura_value = float(datum['tripduration'])/60
    return dura_value


def time_of_trip(datum, city):
    if city=='Washington': 
        month,day,year = (int(x.replace(':','')) for x in datum['Start date'].split()[0].split('/')) 
        hour = int(datum['Start date'].split()[-1][:2].replace(':',''))  
        ans = datetime.date(year, month, day) 
        day_of_week = ans.strftime('%A') 
    else:
        month,day,year = (int(x.replace(':','')) for x in datum['starttime'].split()[0].split('/'))
        hour = int(datum['starttime'].split()[-1][:2].replace(':','')) 
        ans = datetime.date(year, month, day)
        day_of_week = ans.strftime('%A')        
    return (month, hour, day_of_week)


def type_of_user(datum, city):
    if city == 'Washington':
        if datum['Member Type']=='Registered':
            user_type = 'Subscriber'
        else:
            user_type = 'Customer'
    else:
        user_type = datum['usertype']
    return user_type


def condense_data(in_file, out_file, city): 
    with open(out_file, 'w') as f_out, open(in_file, 'r') as f_in:
        trip_writer = csv.DictWriter(f_out, fieldnames = ['duration', 'month', 'hour', 'day_of_week', 'user_type'])
        trip_writer.writeheader() 
        trip_reader = csv.DictReader(f_in)

        for row in trip_reader:
            new_point = {}
            new_point['duration'] = duration_in_mins(row, city)
            new_point['month'] = time_of_trip(row, city)[0]
            new_point['hour'] = time_of_trip(row, city)[1]
            new_point['day_of_week'] = time_of_trip(row, city)[2]
            new_point['user_type'] = type_of_user(row, city)
            trip_writer.writerow(new_point)
            
city_info = {'Washington': {'in_file': 'C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L2/bike-share-analysis/data/Washington-CapitalBikeshare-2016.csv',
                            'out_file': 'C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L2/bike-share-analysis/data/Washington-2016-Summary.csv'},
             'Chicago': {'in_file': 'C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L2/bike-share-analysis/data/Chicago-Divvy-2016.csv',
                         'out_file': 'C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L2/bike-share-analysis/data/Chicago-2016-Summary.csv'},
             'NYC': {'in_file': 'C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L2/bike-share-analysis/data/NYC-CitiBike-2016.csv',
                     'out_file': 'C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L2/bike-share-analysis/data/NYC-2016-Summary.csv'}}

for city, filenames in city_info.items():
    condense_data(filenames['in_file'], filenames['out_file'], city) 
    print_first_point(filenames['out_file'])            
```
<img src="https://user-images.githubusercontent.com/31917400/32448619-5084d146-c307-11e7-80e3-f58778900775.jpg" width="160" height="190" />

#### *|Data Exploration|*
- __package:__ matplotlib
- __func:__ `plt.hist(data, bins = range(,,))`, `plt.title('str: {}'.format(i))`,`plt.xlabel('str')`,`plt.show()`
- __issue:__  















