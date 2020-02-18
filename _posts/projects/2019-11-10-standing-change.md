---
title: "Visualizing the Change in Standings of NBA teams over the Decade"
date: 2019-11-10
permalink: /projects/2019/11/10/standing-change
--- 

I wanted to make a cool visualization of the standings change over the last 5 seasons.

Hence, I used Basketball-Reference to get historical standings data over the 2015-16, 2016-17, 2017-18, and 2018-19
seasons.

Then, I used d3 to create charts.

These charts have a couple added effects:
* On hover, it labels the team by name
* On click, it decreases opacity of all other teams
* We also have implemented d3's brush functionality

Overall, I'm pretty satisfied with it! I've published it on my [blo.cks](https://bl.ocks.org/vishaalagartha) profile:

- [West version](https://bl.ocks.org/vishaalagartha/raw/3802243163558f4d230a59c64671b07c/)
- [East version](https://bl.ocks.org/vishaalagartha/raw/cd0220d31a97a2c73ddf289404a50c8c/)
