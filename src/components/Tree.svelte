<script lang="ts">
  import * as d3 from "d3";

  type Data = {
    name: string;
    children?: Data[];
  };

  export let data: Data = {
    name: "A",
    children: [
      { name: "A1" },
      { name: "A2", children: [{ name: "B" }, { name: "C" }] },
    ],
  };

  const width = 954;
  const tree = (data: Data) => {
    const root = d3.hierarchy(data);
    const dx = 10;
    const dy = width / (root.height + 1);
    return {
      root: d3.tree<Data>().nodeSize([dx, dy])(root),
      dx,
      dy,
    };
  };

  const chart = (el: HTMLElement) => {
    const { root, dx, dy } = tree(data);

    let x0 = Infinity;
    let x1 = -x0;
    root.each((d) => {
      if (d.x > x1) x1 = d.x;
      if (d.x < x0) x0 = d.x;
    });

    const svg = d3
      .create("svg")
      .attr("viewBox", [0, 0, width, x1 - x0 + dx * 2].join(", "));

    const g = svg
      .append("g")
      .attr("font-family", "sans-serif")
      .attr("font-size", 10)
      .attr("transform", `translate(${dy / 3},${dx - x0})`);

    const link = g
      .append("g")
      .attr("fill", "none")
      .attr("stroke", "#555")
      .attr("stroke-opacity", 0.4)
      .attr("stroke-width", 1.5)
      .selectAll("path")
      .data(root.links())
      .join("path")
      .attr(
        "d",
        d3
          .linkHorizontal<any, any, any>()
          .x((d) => d.y)
          .y((d) => d.x)
      );

    const node = g
      .append("g")
      .attr("stroke-linejoin", "round")
      .attr("stroke-width", 3)
      .selectAll("g")
      .data(root.descendants())
      .join("g")
      .attr("transform", (d) => `translate(${d.y},${d.x})`)
      .classed("node", true);

    node
      .append("circle")
      .attr("fill", (d) => (d.children ? "#555" : "#999"))
      .attr("r", 2.5);

    node
      .append("text")
      .attr("dy", "0.31em")
      .attr("x", (d) => (d.children ? -6 : 6))
      .attr("text-anchor", (d) => (d.children ? "end" : "start"))
      .text((d) => d.data.name)
      .clone(true)
      .lower()
      .attr("stroke", "white");

    el.appendChild(svg.node());
  };
</script>

<div use:chart />

<style>
  :global(.node:hover) {
    fill: cornflowerblue;
    cursor: pointer;
  }
</style>
