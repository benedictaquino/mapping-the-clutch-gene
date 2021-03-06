# Technical Report

## Data

I scraped the relevant data from [RotoGuru] and the [NFL Fantasy API] and stored
it in a PostgreSQL database. Details can be seen in the [data_pipeline.py] 
script. The functions in data_pipeline.py scrape the data to pandas DataFrames,
then using SQLAlchemy the data is inserted into Postgres. [Here] is the SQL 
dump file. I primarily worked with the data from the [NFL Fantasy API].

Table 1 describes the features I utilized throughout my project.

##### Table 1.

|Feature|Description|
|---|---|
|`id`|Unique identifier|
|`name`|Player's name|
|`pos`|Player's position|
|`weekpoints`| Fantasy Points |
|`passing_attempts`| Passing Attempts |
|`passing_completions`| Passing Completions | 
|`incomplete_passes`| Incomplete Passes | 
|`passing_yards`| Passing Yards|  
|`passing_touchdowns`| Passing Touchdowns | 
|`interceptions_thrown`| Interceptions Thrown | 
|`every_time_sacked`| Count of times Sacked |
|`rushing_attempts`| Rushing Attempts |
|`rushing_yards`| Rushing Yards |
|`rushing_touchdowns`| Rushing Touchdowns |
|`receptions`| Receptions | 
|`receiving_yards`| Receiving Yards | 
|`receiving_touchdowns`| Receiving Touchdowns |
|`kickoff_and_punt_return_yards`| Kickoff and Punt Return Yards |
|`kickoff_and_punt_return_touchdowns`| Kickoff and Punt Return Touchdowns |
|`fumble_recovered_for_td`| Fumbles Recovered for a Touchdown |
|`fumbles_lost`| Fumbles Lost |
|`fumble`| Fumble |
|`2_point_conversions`| Successful 2-Point Conversions |
|`pat_made`| Point After Touchdown Made |
|`pat_missed`| Point Afte Touchdown Missed |
|`fg_made_0_19`| Field Goal Made between 0 and 19 yards |
|`fg_made_20_29`| Field Goal Made between 20 and 29 yards |
|`fg_made_30_39`| Field Goal Made between 30 and 39 yards |
|`fg_made_40_49`| Field Goal Made between 40 and 49 yards |
|`fg_made_50plus`| Field Goal Made from 50 or more yards |
|`fg_missed_0_19`| Field Goal Missed between 0 and 19 yards |
|`fg_missed_20_29`| Field Goal Missed between 20 and 29 yards |
|`fg_missed_30_39`| Field Goal Missed between 30 and 39 yards |
|`fg_missed_40_49`| Field Goal Missed between 40 and 49 yards |
|`fg_missed_50plus`| Field Goal Missed from 50 or more yards |
|`sacks`| Number of Sacks (All Defense) |
|`interceptions`| Number of Interceptions Caught (All Defense) |
|`fumbles_recovered`| Number of Fumbles Recovered (All Defense) |
|`fumbles_forced`| Number of Fumbles Forced (All Defense)  |
|`safeties`| Number of Safeties Caused (All Defense) |
|`touchdowns`| Number of Touchdowns (All Defense) |
|`blocked_kicks`| Number of Blocked Kicks (All Defense) |
|`points_allowed`| Number of Points Allowed to the Opposing Team (All Defense) |
|`yards_allowed`| Number of Yards Allowed to the Opposing Team (All Defense)  |
|`tackle`| Number of Tackles |
|`assisted_tackles`| Number of Assisted Tackles |
|`sack`| Number of Sacks (Individual) |
|`defense_interception`| Number of Interceptions (Individual)|
|`forced_fumble`| Number of Fumbles Forced (Individual) |
|`fumbles_recovery`| Number of Fumbles Recoberd (Individual) |
|`touchdown_interception_return`| Number of Interceptions Returned for a Touchdown (Individual)|
|`touchdown_fumble_return`| Number of Fumbles Returned for a Touchdown (Individual)|
|`touchdown_blocked_kick`| Number of Blocked Kicks Recovered and Returned for a Touchdown (Individual)|
|`blocked_kick_punt_fg_pat`| Number of Blocked Kicks (Individual) |
|`safety`| Number of Safeties (Individual) |
|`pass_defended`| Number of Passes Defended |
|`interception_return_yards`| Number of Interception Return Yards|
|`fumble_return_yards`| Number of Fumble Return Yards|
|`qb_hit`| Number of QB Hits |
|`sack_yards`| Number of Yards Lost by Opposing Team from Sacks |
|`def_2_point_return`| Number of 2 Point Attempts Returned for a Touchdown (Individual) |
|`team_def_2_point_return`| Number of 2 Point Attempts Returned for a Touchdown (All Defense) | 


## Topological Data Analysis - Average Weekly Points

I chose to use the average fantasy points of each player to create my cover of
the point cloud space. Since fantasy points are essentially a linear combination
of their performance statistics, I felt it was a good low-dimensional 
representation of each player that would provide me with a nice cover.

In order to decide how many sets to put in my cover, I inspected dendrograms 
created from single, ward, and complete linkage hierarchical clustering. I 
decided that ward clustering would give me the results I wanted. 

As well, I limited each position to the top 100 performers. Since I am dealing 
with fantasy football data, anybody outside of this range is unlikely to be on
a team even in a deep 12-man league.

### Process
For each position, I looked at ward linkage dendrograms to decide how many sets
to include in my cover.

Then, I normalized the full data set and pulled back the cover from the 
low-dimensional projection to the high-dimensional data. I constructed two 
types of simplicial complexes utilizing the cover and data. First, I computed
the centroid of each set in my cover and used these points as vertices in my 
observation complexes. Then, I used each data point in my data set as the 
vertices for my landmark complexes. So players are landmarks and the centroids 
of my cover were observations.

For the observation complex, a k-simplex was added when (k+1) distinct 
observation points were visible to a common landmark. For the landmark complex,
a k-simplex was added when (k+1) distinct landmark points were visible to a 
common observation.

Visibility was defined by being within a certain distance threshold of another
point.

Since the goal of the project is to overcome issues with high-dimensional data,
I computed 2-complexes. As well, creating and analyzing higher dimensional 
complexes becomes computationaly expensive and the curse of dimensionality 
resurfaces.

The relevant code is located in the [tda.py] script.

Then I constructed filtrations consisting of successive simplicial 2-complexes
as the distance threshold was increased. Then I computed persistent homologies
of these filtrations and inspected the barcode diagrams.

The barcode diagrams show when a connected component is "born" until the time it 
"dies." So longer bars indicate that the component lived longer through the 
filtration and has might be more meaningful.

After inspecting the barcode diagrams I also looked at the the simplicial 
complexes at the key times when components were born and what the vertices were.
The observation complexes could be used to collapse the initial clusters into 
larger clusters, and the landmark complexes

### Weekly Data

After deciding the number of sets I wanted in my covers of each position, I 
created complexes for each position and inserted them into a MongoDB which would
be accessible via my [web app]. The code for this is in the [tda.py] script.

The plots for the weekly barcode diagrams can be found [here].

As well, I have a script that is currently scheduled to scrape the weekly data,
clean it, and insert it into the relevant databases for use on the web app.
Details can be found in the [get_new_data.py] script

## Visualization

I used plotly to visualize the simplicial complexes. I exported the plotly 
figures as json files and inserted them into a MongoDB database using PyMongo. 
The relevant code is in [tda.py]. 

## [Web App](http://benedictaquino.com/mapping-the-clutch-gene)

I used Flask and Dash to create my web app. Dash was used to create an 
interactive dashboard that allows one to explore the various simplicial 
complexes and filtrations.

I embedded the Dash app in one of the pages in my Flask web app.

I used a Bootstrap template by Creative Tim, [Material Kit] for my layout.


#### [Quarterbacks](notebooks/qb-landmark-observer.ipynb)
###### Figure 1.
![alt text][qb_avg_dendrogram]

After visual inspection of the dendrogram shown in Figure 1 I decided to use 5 
sets for my cover.

###### Figure 2.
![alt text][qb_avg_cover]
###### Here the data points are colored by which set in the cover they are in

###### Figure 3.
![alt text][qb_avg_barcode_landmark]
![alt text][qb_avg_barcode_observer]
###### Barcode diagrams for the two filtrations

#### [Runningbacks](notebooks/rb-landmark-observer.ipynb)
###### Figure 1.
![alt text][rb_avg_dendrogram]

After visual inspection of the dendrogram shown in Figure 1 I decided to use 12 
sets for my cover.

###### Figure 2.
![alt text][rb_avg_cover]

###### Figure 3.
![alt text][rb_avg_barcode_landmark]
![alt text][rb_avg_barcode_observer]

#### [Wide Receivers](notebooks/wr-landmark-observer.ipynb)
###### Figure 4.
![alt text][wr_avg_dendrogram]

After visual inspection of the dendrogram shown in Figure 1 I decided to use 12 
sets for my cover.

###### Figure 5.
![alt text][wr_avg_cover]

###### Figure 6.
![alt text][wr_avg_barcode_landmark]
![alt text][wr_avg_barcode_observer]

#### [Tight Ends](notebooks/te-landmark-observer.ipynb)
###### Figure 7.
![alt text][te_avg_dendrogram]

After visual inspection of the dendrogram shown in Figure 1 I decided to use 11 
sets for my cover.

###### Figure 8.
![alt text][te_avg_cover]

###### Figure 9.
![alt text][te_avg_barcode_landmark]
![alt text][te_avg_barcode_observer]

#### [Kickers](notebooks/k-landmark-observer.ipynb)
###### Figure 10.
![alt text][k_avg_dendrogram]

After visual inspection of the dendrogram shown in Figure 1 I decided to use 8 
sets for my cover.

###### Figure 11.
![alt text][k_avg_cover]

###### Figure 12.
![alt text][k_avg_barcode_landmark]
![alt text][k_avg_barcode_observer]

#### [Team Defenses and Special Teams](notebooks/def-landmark-observer.ipynb)
###### Figure 13.
![alt text][def_avg_dendrogram]

After visual inspection of the dendrogram shown in Figure 1 I decided to use 7 
sets for my cover.

###### Figure 14.
![alt text][def_avg_cover]

###### Figure 15.
![alt text][def_avg_barcode_landmark]
![alt text][def_avg_barcode_observer]

#### [Linebackers](notebooks/lb-landmark-observer.ipynb)
###### Figure 16.
![alt text][lb_avg_dendrogram]

After visual inspection of the dendrogram shown in Figure 1 I decided to use 11 
sets for my cover.

###### Figure 17.
![alt text][lb_avg_cover]

###### Figure 18.
![alt text][lb_avg_barcode_landmark]
![alt text][lb_avg_barcode_observer]

#### [Defensive Backs](notebooks/db-landmark-observer.ipynb)
###### Figure 19.
![alt text][db_avg_dendrogram]

After visual inspection of the dendrogram shown in Figure 1 I decided to use 7 
sets for my cover.

###### Figure 20.
![alt text][db_avg_cover]

###### Figure 21.
![alt text][db_avg_barcode_landmark]
![alt text][db_avg_barcode_observer]

#### [Defensive Lineman](notebooks/dl-landmark-observer.ipynb)
###### Figure 22.
![alt text][dl_avg_dendrogram]

After visual inspection of the dendrogram shown in Figure 1 I decided to use 10 
sets for my cover.

###### Figure 23.
![alt text][dl_avg_cover]

###### Figure 24.
![alt text][dl_avg_barcode_landmark]![alt text][dl_avg_barcode_observer]

## Insights

Notice the barcode diagrams are exactly the same by position. This is expected,
since the two complexes are equivalent representations of the data.

I believe the barcode diagrams and persistent homologies can be a good tool to 
use to refine the initial clusters.

[RotoGuru]: http://rotoguru.net/
[NFL Fantasy API]: http://api.fantasy.nfl.com/
[data_pipeline.py]: src/data_pipeline.py
[Here]: data/nfl.sql
[qb_avg_dendrogram]: plots/2017/avg/qb_avg_dendrogram.png "QB Average Dendrogram"
[qb_avg_cover]: plots/2017/avg/qb_avg_scatter.png "QB Average Scatterplot"
[qb_avg_barcode_landmark]: plots/2017/avg/qb_avg_barcode_landmark.png "QB Average: Barcode Diagram for beta_0 of the Landmark Complex"
[qb_avg_barcode_observer]: plots/2017/avg/qb_avg_barcode_observer.png "QB Average: Barcode Diagram for beta_0 of the Observer Complex"
[rb_avg_dendrogram]: plots/2017/avg/rb_avg_dendrogram.png "RB Average Dendrogram"
[rb_avg_cover]: plots/2017/avg/rb_avg_scatter.png "RB Average Scatterplot"
[rb_avg_barcode_landmark]: plots/2017/avg/rb_avg_barcode_landmark.png "RB Average: Barcode Diagram for beta_0 of the Landmark Complex"
[rb_avg_barcode_observer]: plots/2017/avg/rb_avg_barcode_observer.png "RB Average: Barcode Diagram for beta_0 of the Observer Complex"
[wr_avg_dendrogram]: plots/2017/avg/wr_avg_dendrogram.png "WR Average Dendrogram"
[wr_avg_cover]: plots/2017/avg/wr_avg_scatter.png "WR Average Scatterplot"
[wr_avg_barcode_landmark]: plots/2017/avg/wr_avg_barcode_landmark.png "WR Average: Barcode Diagram for beta_0 of the Landmark Complex"
[wr_avg_barcode_observer]: plots/2017/avg/wr_avg_barcode_observer.png "WR Average: Barcode Diagram for beta_0 of the Observer Complex"
[te_avg_dendrogram]: plots/2017/avg/te_avg_dendrogram.png "TE Average Dendrogram"
[te_avg_cover]: plots/2017/avg/te_avg_scatter.png "TE Average Scatterplot"
[te_avg_barcode_landmark]: plots/2017/avg/te_avg_barcode_landmark.png "TE Average: Barcode Diagram for beta_0 of the Landmark Complex"
[te_avg_barcode_observer]: plots/2017/avg/te_avg_barcode_observer.png "TE Average: Barcode Diagram for beta_0 of the Observer Complex"
[k_avg_dendrogram]: plots/2017/avg/k_avg_dendrogram.png "K Average Dendrogram"
[k_avg_cover]: plots/2017/avg/k_avg_scatter.png "K Average Scatterplot"
[k_avg_barcode_landmark]: plots/2017/avg/k_avg_barcode_landmark.png "K Average: Barcode Diagram for beta_0 of the Landmark Complex"
[k_avg_barcode_observer]: plots/2017/avg/k_avg_barcode_observer.png "K Average: Barcode Diagram for beta_0 of the Observer Complex"
[def_avg_dendrogram]: plots/2017/avg/def_avg_dendrogram.png "DEF Average Dendrogram"
[def_avg_cover]: plots/2017/avg/def_avg_scatter.png "DEF Average Scatterplot"
[def_avg_barcode_landmark]: plots/2017/avg/def_avg_barcode_landmark.png "DEF Average: Barcode Diagram for beta_0 of the Landmark Complex"
[def_avg_barcode_observer]: plots/2017/avg/def_avg_barcode_observer.png "DEF Average: Barcode Diagram for beta_0 of the Observer Complex"
[lb_avg_dendrogram]: plots/2017/avg/lb_avg_dendrogram.png "LB Average Dendrogram"
[lb_avg_cover]: plots/2017/avg/lb_avg_scatter.png "LB Average Scatterplot"
[lb_avg_barcode_landmark]: plots/2017/avg/lb_avg_barcode_landmark.png "LB Average: Barcode Diagram for beta_0 of the Landmark Complex"
[lb_avg_barcode_observer]: plots/2017/avg/lb_avg_barcode_observer.png "LB Average: Barcode Diagram for beta_0 of the Observer Complex"
[db_avg_dendrogram]: plots/2017/avg/db_avg_dendrogram.png "DB Average Dendrogram"
[db_avg_cover]: plots/2017/avg/db_avg_scatter.png "DB Average Scatterplot"
[db_avg_barcode_landmark]: plots/2017/avg/db_avg_barcode_landmark.png "DB Average: Barcode Diagram for beta_0 of the Landmark Complex"
[db_avg_barcode_observer]: plots/2017/avg/db_avg_barcode_observer.png "DB Average: Barcode Diagram for beta_0 of the Observer Complex"
[dl_avg_dendrogram]: plots/2017/avg/dl_avg_dendrogram.png "DL Average Dendrogram"
[dl_avg_cover]: plots/2017/avg/dl_avg_scatter.png "DL Average Scatterplot"
[dl_avg_barcode_landmark]: plots/2017/avg/dl_avg_barcode_landmark.png "DL Average: Barcode Diagram for beta_0 of the Landmark Complex"
[dl_avg_barcode_observer]: plots/2017/avg/dl_avg_barcode_observer.png "DL Average: Barcode Diagram for beta_0 of the Observer Complex"
[web app]:(http://benedictaquino.com/mapping-the-clutch-gene)
[tda.py]: (src/tda.py)
[here]: (plots)
[get_new_data.py]: src/get_new_dat.py
[Material Kit]: (https://demos.creative-tim.com/material-kit/index.html)