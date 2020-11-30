---
layout: single
title: The Simpsons by The Data
---

The iconic animated show "The Simpson" is in its 28th season with almost 600 episodes. The first season started in 1989 and it almost aired for three decades. I've been a fan of The Simpsons growing up and when I came across a data set for Simpsons in Kaggle, I had to dive in and do something about it. The data can be found <a href="https://www.kaggle.com/wcukierski/the-simpsons-by-the-data">here</a>.

I created a D3.js / Dimple.js chart for all episodes in 27 seasons of the show. I wanted to be able to identify which episodes are the best / worst ones in a glance. The chart shows all episodes as data points with x-axis being air date in year and y-axis for IMDB rating. The size of the circle represents US viewership number. I gathered additional writer/director information from Wikipedia and included in the toolip. Enjoy!

<div id='d3div'></div>

<style>
    circle.dimple-series-1 {
      fill: red;
    }

    .dimple-custom-tooltip-box {
       fill: #ffd700 !important;
    }

    text.dimple-tooltip { fill: navy !important;

      font-family: "Comic Sans MS", cursive !important;
      font-size: 14px !important;
      font-weight: 50 !important;
       }
    }
</style>

<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/dimple/2.3.0/dimple.latest.min.js"></script>
  
<script type="text/javascript">
  function draw(data) {

    "use strict";
    var margin = 5,
        width = 1100 - margin,
        height =900 - margin;

    var svg = d3.select("#d3div")
      .append("svg")
      .attr("width", width + margin)
      .attr("height", height + margin)
      .append('g')
      .attr('class','chart');

    svg.append("text")
     .attr("x", width / 2 + 75)
     .attr("y", 20)
     .style("text-anchor", "middle")
     .style("font-family", "Comic Sans MS")
     .style("font-weight", "bold")
     .style("fill","#005580")
     .style("font-size", "25px")
     .text("Three Decades of The Simpsons");


    var myChart = new dimple.chart(svg, data);
    myChart.setBounds(80, 25, 1000, 700);   


    var myColor = myChart.defaultColors = [
        new dimple.color("#ffcc00","#cca300",0.9), //another yellow
        new dimple.color("#ff6347","#e62200",0.9), // Tomato
        new dimple.color("#99cc33","#6b8e23",0.9), // Olive Drab       
        new dimple.color("#a0ff80","#99cc00", 0.9), //R:223, G:255, B:128
        new dimple.color("#0077b3","#005580", 0.9), //RoyalBlue
        
        new dimple.color("#cdab7e","#bf935a", 0.9), // Tan     
        new dimple.color("#ffd700","#ccad00",0.9), //Simpson Yellow
        new dimple.color("#80bfff","#4da6ff",0.9),
        new dimple.color("#ff8000","#cc6600",0.9), //rgb(255, 128, 0)
        new dimple.color("#708090","#596673",0.9)

    ];


    var x = myChart.addTimeAxis("x", "Air Date","%Y-%m-%d","%Y-%m-%d");
    x.tickFormat = "%Y";
    x.timeInterval = 1;
    x.overrideMin = new Date("1988-01-01");
    x.overrideMax = new Date("2017-01-01");
    x.fontSize = 13;

    var y = myChart.addMeasureAxis("y", "IMDB Rating");
    y.ticks = 12;
    y.tickFormat ="0.1f"
    y.overrideMin = 4.5;
    y.overrideMax = 9.5;
    y.fontSize = 13;

    var z = myChart.addMeasureAxis("z", "US Viewers in Millions");
    //z.overrideMin = 100;
    z.overrideMax = 180;
    var mySeries = myChart.addSeries(["Air Date","Episode","Title","US Viewers in Millions","IMDB Rating","Director","Writer","Season"], dimple.plot.bubble);

    mySeries.getTooltipText = function (e) {
        return [
            "Season: " + e.aggField[7],
            "Episode: " + e.aggField[1],
            "Title: " + e.aggField[2],
            "Air Date: " + e.aggField[0],
            "Director: " + e.aggField[5],
            "Writer: " + e.aggField[6],
            "IMDB Rating: " + e.aggField[4],
            "US Viewers in Millions: " + e.aggField[3]
        ];
    };


      var img = svg.selectAll("image").data([0]);
    img.enter()
    .append("svg:image")
    .attr("xlink:href","https://jjchoi08.github.io/DSProj/image/simpsons_icon_1.png")
    .attr("x", "100")
    .attr("y", "520")
    .attr("width", "200")
    .attr("height", "200");
    myChart.draw();
    x.tickFormat ="%Y-%m-%d"

  };
</script>

<script type="text/javascript">

  d3.tsv("https://jjchoi08.github.io/_data/d3_episodes.tsv", draw);
  
</script>

# What is happening?

The IMDB rating declined over the 27 seasons. We could also see that the circle sizes being decreased gradually. As most of the Simpsons fans would agree, the first 8 seasons have great ratings. So if you are planning to watch The Simpsons again or for the first time, you know which ones to watch!
