<script>
  import { onMount, createEventDispatcher } from "svelte";
  import * as d3 from "d3";
  export let data;

  const dispatch = createEventDispatcher();
  let chart;
  const init = () => {
    //Width and height
    let chart_width = window.innerWidth * 1.8;
    let chart_height = window.innerHeight * 1.8;
    let colors = d3.scaleOrdinal(d3.schemeCategory10);

    // Force Layout
    let force = d3
      .forceSimulation(data.nodes)
      .force("charge", d3.forceManyBody().strength(-700))
      .force("link", d3.forceLink(data.links))
      .force(
        "center",
        d3
          .forceCenter()
          .x(chart_width / 2)
          .y(chart_height / 2)
      );

    // SVG
    let svg = d3
      .select(chart)
      .append("svg")
      .attr("width", chart_width)
      .attr("height", chart_height)
      .call(
        d3.zoom().on("zoom", function() {
          svg.attr("transform", d3.event.transform);
        })
      )
      .append("g");

    // Lines AKA Links
    let lines = svg
      .selectAll("line")
      .data(data.links)
      .enter()
      .append("line")
      .style("stroke", "#24292e")
      .style("stroke-width", 2);

    // Nodes
    let nodes = svg
      .selectAll("circle")
      .data(data.nodes)
      .enter()
      .append("circle")
      .attr("r", (d, i) => {
        return d.textHeight * 2 + d.name.length;
      })
      .style("stroke", function(d, i) {
        return "black";
      })
      .style("stroke-width", 2)
      .style("fill", function(d, i) {
        return "white";
      });

    // Texts
    let texts = svg
      .selectAll("text")
      .data(data.nodes)
      .enter()
      .append("text")
      .text(d => d.name)
      .attr("text-anchor", "middle")
      .attr("dy", ".3em")
      .style("fill", function(d, i) {
        return d.color;
      })
      .style("font-size", d => {
        return d.textHeight + "px";
      })
      .style("font-family", "sans-serif");

    // Tooltip
    nodes.append("title").text(function(d) {
      return d.name;
    });
    let counter = 0;
    force.on("tick", function() {
      counter += 1;
      if (counter >= 80) {
        dispatch("unloading");
      }
      lines
        .attr("x1", function(d, i) {
          return d.source.x;
        })
        .attr("y1", function(d, i) {
          return d.source.y;
        })
        .attr("x2", function(d, i) {
          return d.target.x;
        })
        .attr("y2", function(d, i) {
          return d.target.y;
        });
      let radius = 200;
      nodes
        .attr("cx", function(d, i) {
          if (i <= 4) {
            return (d.x = Math.max(
              radius,
              Math.min(chart_width - radius, d.x)
            ));
          }
          return d.x;
        })
        .attr("cy", function(d, i) {
          if (i <= 4) {
            return (d.y = Math.max(
              radius,
              Math.min(chart_height - radius, d.y)
            ));
          }
          return d.y;
        });

      texts
        .attr("x", function(d) {
          return d.x;
        })
        .attr("y", function(d) {
          return d.y;
        });
    });
  };

  onMount(init);
</script>

<div id="chart" bind:this={chart} />
