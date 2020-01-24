---
title: "Clustering NBA Teams Using K Means"
date: 2019-06-24
permalink: /projects/2019/06/24/nba-teams-clustering
--- 

We try and define basketball in terms of eras, like 'the early 90's' or 'modern space and pace'. But does the timeline of basketball actually divide itself in such a manner? The goal of this project was to group NBA teams using their stats and see if there is a clear demarcation between these abstractions we've created.

For the programmatically inclined [here is a gist](https://gist.github.com/vishaalagartha/5048cea823f758787c12b295d9c2b63d) of all my work including the web scraper, raw data, and IPython notebook.

The tl;dr of the post is at the bottom in a table along with a graphic.

&#x200B;

**Data Aggregation**

I aggregated all my data from Basketball Reference by web scraping and parsing using Python's BeautifulSoup library. The data includes the following data:

`['Team', 'G', 'MP', 'FG', 'FGA', 'FG%', '3P', '3PA', '3P%', '2P', '2PA', '2P%', 'FT', 'FTA', 'FT%', 'ORB', 'DRB', 'TRB', 'AST', 'STL', 'BLK', 'TOV', 'PF', 'PTS', 'Team.1', 'Opponent G', 'Opponent MP', 'Opponent FG', 'Opponent FGA', 'Opponent FG%', 'Opponent 3P', 'Opponent 3PA', 'Opponent 3P%', 'Opponent 2P', 'Opponent 2PA', 'Opponent 2P%', 'Opponent FT', 'Opponent FTA', 'Opponent FT%', 'Opponent ORB', 'Opponent DRB', 'Opponent TRB', 'Opponent AST', 'Opponent STL', 'Opponent BLK', 'Opponent TOV', 'Opponent PF', 'Opponent PTS']`

Data was aggregated for all teams since the 1980-81 season. This is as far back as I could connect to basketball, so I decided to set the cutoff point there.

Note how I don't include year or team record because I don't want the algorithm to be able to split based on time automatically or by how well the team performed. The goal was to purely group based on playing style.

The raw data can be found [here](https://gist.github.com/vishaalagartha/5048cea823f758787c12b295d9c2b63d#file-team_stats-csv).

**K-Means Clustering**

K-Means is a simple machine learning clustering algorithm that groups n items into k groups by computing a 'centroid' and putting each item in the group that minimizes the distance to the centroid. One crucial element in the algorithm is to determine what k should be (i.e. how many clusters to group the n items into). Of course, you don't want k=1 since that would just put all the teams in 1 group and you also don't want k=2418 since that would give each team its own group.

&#x200B;

I used the [elbow method](https://en.wikipedia.org/wiki/Determining_the_number_of_clusters_in_a_data_set#The_elbow_method), which incrementally increases the number of clusters and calculates the inertia or the within-cluster sum-of-squares of all the data points. Using all values of k from 1 to 20, I created this plot.

![alt_text](https://i.imgur.com/pYYh5Fu.png)


By both eyeballing it and calculating the derivative, I determined the best value of k to be 6. For reference, here are the values of inertia for values of k 5 through 7:

k = 5,  3299334.0773817445

k = 6, 3061189.575190504

k=7, 2932684.8809371283

You can see that the dropoff is less significant and would lead to diminishing returns beyond this point.

**Results**

Using k=6, I determined the following to be the 'centroids'. Based on the statistics and teams in each centroid, I gave each cluster a name and added some known teams to them. I've put the tables in 2 separate sections for clarity.

![alt_text](https://i.imgur.com/ZYSqC97.png)

&#x200B;

![alt_text](https://i.imgur.com/fgxZ3Wq.png)

If you can't see it clearly, you can use a link to it [here](https://gist.github.com/vishaalagartha/5048cea823f758787c12b295d9c2b63d#file-team_clusters-csv).

Takeaways:

* As expected, the small ball lineup of the modern day stands out. You can clearly see this in the points scored and 3PA fields.
* I found it interesting that there was an 'Old School Fast Pace' cluster 0. I had no idea that they ran the ball so much back in the day.
* The points and 3PA vs. 2PA of the Dinosaurs (cluster 3) was shocking to look at. Pretty sure Daryl Morey would have a heart attack just looking at them.
* I was interested to see the success of cluster 5 where teams like the Bulls and Pistons just grinded you on the defensive end and killed you.  It's crazy to think what the game must've been like back then

&#x200B;

Overall, I think we clearly defined 4 groups of time: The Dinosaurs, The Early 90's?, Early 2000's/Middle Era, and Modern Small Ball. I'm not really sure how the Old School Fast Pace teams fit into it all.

&#x200B;

**Visualization.**

Finally, for the visually inclined, I created a T-SNE plot to show the clusters. The T-SNE transformation reduces n-dimensions into 2-dimensions so we can interpret our results to a certain degree.

![alt_text](https://i.imgur.com/Vp9y4LE.png)

Clearly, we see that the small ball cluster 1 towards the center left. Interestingly, we see cluster 0 or Old School Fast Pace and cluster 5 more to the right, but still relatively close. We also see the early 2000's era next to the small ball cluster.

Finally, we see clusters 3 and 4 moving further away from the center as expected. These clusters deviate the most from modern basketball and, hence, are clearly delineated from the central groups.

&#x200B;

I hope you enjoyed this post and let me know if you have any thoughts!
