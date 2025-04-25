<template>
  <div class="p-4 w-auto">
    <div v-if="root">
      <PersonNode :person="root" :depth="0" @load-children="loadChildren" />
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
import PersonNode from './Person.vue';

export default {
  components: { PersonNode },
  data() {
    return {
      root: null,
      map: {},
    };
  },
  mounted() {
    // Load the CSV data
    d3.csv('/Giga Corp (40k) - Sheet1.csv').then(data => {
      // Clean the data and add to map
      const map = {};
      data.forEach(p => {
        p.salary = parseFloat(p.Salary) || 0;
        p.id = p['Employee Id'];
        p.managerId = p.Manager;
        p.Name = p.Name;
        map[p.id] = p;
      });

      this.map = map;

      // Use d3.stratify to create the root node
      const stratify = d3
        .stratify()
        .id(d => d.id)
        .parentId(d => d.managerId);

      const root = stratify(data);
      this.root = root;


      this.addCalculationsToNode(root);
    });
  },
  methods: {
    addCalculationsToNode(node) {
      // Assign bgColor early based on the node's level and department
      if (parseInt(node.data.level) === 2) {
        // Apply bgColor for root children based on their department
        if (node.data.Department === 'Business Intelligence') {
          node.bgColor = 'bg-green-50';
        } else if (node.data.Department === 'Network Operations') {
          node.bgColor = 'bg-blue-50';
        } else if (node.data.Department === 'Finance and Accounting') {
          node.bgColor = 'bg-orange-50';
        } else if (node.data.Department === 'Software Development') {
          node.bgColor = 'bg-purple-50';
        } else {
          node.bgColor = 'bg-white';
        }
      } else if (parseInt(node.data.level) > 2) {
        node.bgColor = node.parent?.bgColor || 'bg-white';
      }

      let totalDescendants = 0;
      let nonLeafDescendants = 0;
      let icCost = 0;
      let managementCost = 0;

      node.children?.forEach(child => {
        const childStats = this.addCalculationsToNode(child);
        totalDescendants += 1 + childStats.totalDescendants;
        nonLeafDescendants += (child.children?.length > 0 ? 1 : 0) + childStats.nonLeafDescendants;
        icCost += childStats.icCost;
        managementCost += childStats.managementCost;
      });

      if (node.children?.length > 0) {
        managementCost += node.data.salary;
      } else {
        icCost += node.data.salary;
      }

      node.totalDescendants = totalDescendants;
      node.nonLeafDescendants = nonLeafDescendants;
      node.icCost = icCost;
      node.managementCost = managementCost;
      node.totalCost = icCost + managementCost;
      node.managementRatio = managementCost ? icCost / managementCost : 0;

      return node;
    },
    loadChildren(node) {
      // Dynamically load children for a given node
      if (!node.children) {
        node.children = Object.values(this.map)
          .filter(p => p.managerId === node.id)
          .map(child => {
            const childNode = d3.hierarchy(child);
            childNode.parent = node; // Explicitly set the parent property
            this.enrich(childNode);
            return childNode;
          });
      }
    },
  },
};
</script>

<style>
body {
  font-family: sans-serif;
}
</style>
