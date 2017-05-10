---
layout: single
title: The Simpsons by Data
---

# Testing Simpsons Chart


<style>
    circle.dimple-series-1 {
      fill: red;
    }

    h2 {
      text-align: center;
    }
  </style>




<div id='d3div'></div>


## Testing 123



<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/dimple/2.3.0/dimple.latest.min.js"></script>
<script type="text/javascript">
    function draw(data) {

      "use strict";
      var margin = 30,
          width = 1500 - margin,
          height =550 - margin;

      d3.select("#d3div")
        .append("h2")
        .text("The Simpsons by Data")

      var svg = d3.select("#d3div")
        .append("svg")
          .attr("width", width + margin)
          .attr("height", height + margin)
        .append('g')
            .attr('class','chart');




      var myChart = new dimple.chart(svg, data);
      myChart.setBounds(60, 60, 1000, 350);   
      var x = myChart.addTimeAxis("x", "Air Date","%Y-%m-%d","%Y-%m-%d");
      x.tickFormat ="%Y"
      var y = myChart.addCategoryAxis("y", "IMDB Rating");
      y.overrideMin = 4.5;
      y.overrideMax = 9.2;
      var z = myChart.addMeasureAxis("z", "US Viewers in Millions");
      z.overrideMin = 0;
      z.overrideMax = 100;
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
      
      myChart.draw();
      x.tickFormat ="%Y-%m-%d"




    };
</script>


<script type="text/javascript">
/*
  Use D3 (not dimple.js) to load the TSV file
  and pass the contents of it to the draw function
  */
d3.tsv("https://github.com/jjchoi08/DSProj/blob/gh-pages/d3_episodes.tsv", draw);
</script>
