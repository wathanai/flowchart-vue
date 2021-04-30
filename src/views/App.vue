<template>
  <div class="container">
    <h1 class="title">Flowchart Vue</h1>
    <h5 class="subtitle">
      Flowchart & Flowchart designer component for Vue.js.
    </h5>
    <div id="toolbar">
      <button
        @click="
          $refs.chart.add({
            id: +new Date(),
            x: 10,
            y: 10,
            name: 'New',
            type: 'operation',
          })
        "
      >
        Add(Double-click canvas)
      </button>
      <button @click="$refs.chart.remove()">Delete(Del)</button>
      <button @click="$refs.chart.editCurrent()">
        Edit(Double-click node)
      </button>
      <button @click="$refs.chart.save()">Save</button>
    </div>
    <flowchart
      :nodes="nodes"
      :connections="connections"
      @editnode="handleEditNode"
      :width="'100%'"
      :height="500"
      :readonly="false"
      @dblclick="handleDblClick"
      @editconnection="handleEditConnection"
      @save="handleChartSave"
      ref="chart"
      :render="render"
    >
    </flowchart>
    <node-dialog
      :visible.sync="nodeDialogVisible"
      :node.sync="nodeForm.target"
    ></node-dialog>
    <connection-dialog
      :visible.sync="connectionDialogVisible"
      :connection.sync="connectionForm.target"
      :operation="connectionForm.operation"
    >
    </connection-dialog>
  </div>
</template>
<script>
/* eslint-disable no-unused-vars */

import ConnectionDialog from "../components/ConnectionDialog";
import NodeDialog from "../components/NodeDialog";
import Flowchart from "../components/flowchart/Flowchart";
import * as d3 from "d3";
import { roundTo20 } from "../utils/math";

export default {
  components: {
    ConnectionDialog,
    NodeDialog,
    Flowchart,
  },
  data: function () {
    return {
      nodes: [
        { id: 1, x: 180, y: 20, name: "Start", type: "start" },
        { id: 2, x: 500, y: 180, name: "End", type: "end" },
        {
          id: 3,
          x: 180,
          y: 100,
          name: "To Do",
          type: "operation",
        },
        {
          id: 4,
          x: 180,
          y: 180,
          name: "In Progress",
          type: "operation",
        },
        {
          id: 5,
          x: 180,
          y: 260,
          name: "Ready to Merge",
          type: "operation",
        },
        {
          id: 6,
          x: 180,
          y: 340,
          name: "Prepare for SIT",
          type: "operation",
        },
        {
          id: 7,
          x: 340,
          y: 340,
          name: "In QA",
          type: "operation",
        },
        {
          id: 8,
          x: 500,
          y: 340,
          name: "QA Testing in Progress",
          type: "operation",
        },
        {
          id: 9,
          x: 500,
          y: 260,
          name: "Prepare for UAT",
          type: "operation",
        },
        {
          id: 10,
          x: 340,
          y: 260,
          name: "Test Failed",
          type: "operation",
        }
      ],
      connections: [
        {
          source: { id: 1, position: "bottom" },
          destination: { id: 3, position: "top" },
          id: 1,
          type: "pass",
        },
        {
          source: { id: 3, position: "bottom" },
          destination: { id: 4, position: "top" },
          id: 2,
          type: "pass",
        },
        {
          source: { id: 4, position: "bottom" },
          destination: { id: 5, position: "top" },
          id: 3,
          type: "pass",
        },
        {
          source: { id: 5, position: "bottom" },
          destination: { id: 6, position: "top" },
          id: 4,
          type: "pass",
        },
        {
          source: { id: 6, position: "right" },
          destination: { id: 7, position: "left" },
          id: 5,
          type: "pass",
        },
        {
          source: { id: 7, position: "right" },
          destination: { id: 8, position: "left" },
          id: 6,
          type: "pass",
        },
        {
          source: { id: 8, position: "top" },
          destination: { id: 9, position: "bottom" },
          id: 7,
          type: "pass",
        },
        {
          source: { id: 9, position: "top" },
          destination: { id: 2, position: "bottom" },
          id: 8,
          type: "pass",
        },
        {
          source: { id: 7, position: "top" },
          destination: { id: 10, position: "bottom" },
          id: 9,
          type: "reject",
        },
        {
          source: { id: 10, position: "top" },
          destination: { id: 4, position: "right" },
          id: 10,
          type: "reject",
        }
      ],
      nodeForm: { target: null },
      connectionForm: { target: null, operation: null },
      nodeDialogVisible: false,
      connectionDialogVisible: false,
    };
  },
  async mounted() {},
  methods: {
    handleDblClick(position) {
      this.$refs.chart.add({
        id: +new Date(),
        x: position.x,
        y: position.y,
        name: "New",
        type: "operation"
      });
    },
    async handleChartSave(nodes, connections) {
      // axios.post(url, {nodes, connection}).then(resp => {
      //   this.nodes = resp.nodes;
      //   this.connections = resp.connections;
      //   // Flowchart will refresh after this.nodes and this.connections changed
      // });
    },
    handleEditNode(node) {
      this.nodeForm.target = node;
      this.nodeDialogVisible = true;
    },
    handleEditConnection(connection) {
      this.connectionForm.target = connection;
      this.connectionDialogVisible = true;
    },
    render: function (g, node, isSelected) {
      node.width = node.width || 120;
      node.height = node.height || 60;
      let borderColor = isSelected ? "#666666" : "#bbbbbb";
      // body
      // if (node.id === 3) {
      //   let body = g.append("ellipse").attr("class", "body");
      //   body.attr("cx", node.x + node.width / 2);
      //   body.attr("cy", node.y + node.height / 2);
      //   body.attr("rx", node.width / 2);
      //   body.attr("ry", node.height / 2);
      //   body.style("fill", "#0000ff"); // Blue
      //   body.style("stroke-width", "1px");
      //   body.classed(node.type, true);
      //   body.attr("stroke", borderColor);
      // } else {
        let body = g.append("rect").attr("class", "body");
        body
          .style("width", node.width + "px")
          .style("stroke-width", "1px");
        if (node.type !== "start" && node.type !== "end") {
          body.attr("x", node.x).attr("y", node.y);
          body.style("height", roundTo20(node.height) + "px")
              .style("fill", "#ffff00") // Yellow
        } else {
          body
            .style("fill", "#ff0000") // Red
            .attr("x", node.x)
            .attr("y", node.y)
            .classed(node.type, true)
            .attr("rx", 30);
          body.style("height", roundTo20(node.height) + "px");
        }
        body.attr("stroke", borderColor);
      // }

      // body text
      let text =
        node.type === "start"
          ? "Start"
          : node.type === "end"
          ? "End"
          : node.name
      let bodyTextY;
      if (node.type !== "start" && node.type !== "end") {
        // if (node.id === 3) {
        //   bodyTextY = node.y + 25;
        // } else {
          bodyTextY = node.y + 15 + roundTo20(node.height - 20) / 2;
        // }
      } else {
        bodyTextY = node.y + 5 + roundTo20(node.height) / 2;
      }
      g.append("text")
        .attr("x", node.x + node.width / 2)
        .attr("y", bodyTextY)
        .attr("class", "unselectable")
        .attr("text-anchor", "middle")
        .text(function () {
          return text;
        })
        .each(function wrap() {
          let self = d3.select(this),
            textLength = self.node().getComputedTextLength(),
            text = self.text();
          while (textLength > node.width - 2 * 4 && text.length > 0) {
            text = text.slice(0, -1);
            self.text(text + "...");
            textLength = self.node().getComputedTextLength();
          }
        });
    },
  },
};
</script>
<style scoped>
#toolbar {
  margin-bottom: 10px;
}

.title {
  margin-top: 10px;
  margin-bottom: 0;
}

.subtitle {
  margin-bottom: 10px;
}

#toolbar > button {
  margin-right: 4px;
}

.container {
  width: 800px;
  margin: auto;
}
</style>
