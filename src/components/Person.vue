<template>
  <div class="relative">
    <!-- Parent Node -->
    <div
      class="rounded-3xl shadow-sm border border-slate-200 p-4 mb-6 mx-auto w-[250px] hover:shadow-lg transition-all duration-300"
      :class="person.bgColor"
    >
      <div class="text-center">
        <div class="mx-auto w-10 h-10 flex items-center justify-center rounded-full bg-blue-500 text-white font-bold mb-2">
          {{ initials }}
        </div>
        <div class="font-bold">{{ person.data.Name }}</div>
        <div class="text-sm text-gray-500">{{ person.data['Job Title'] }}</div>
        <div class="text-xs text-gray-400">{{ person.data.Department }}</div>
        <div class="rounded-md border border-slate-300 py-0.5 px-3 text-center text-xs transition-all shadow-sm text-slate-600 inline-block bg-slate-300 dark:bg-white/10">
          {{ person.data.Location }}
        </div>
      </div>
      <div 
        class="text-sm cursor-pointer" 
        @click="showDetails = !showDetails" 
        :title="showDetails ? 'Hide management calculations' : 'Show management calculations'"
      >
        <hr v-if="!showDetails" class="h-0.5 border-t-0 bg-slate-100 dark:bg-white/10 my-2" />
        <div v-if="!showDetails"
          class="rounded-md border border-slate-300 py-0.5 px-3 text-center text-xs transition-all shadow-sm text-slate-600 inline-block"
        >
          Total Cost: {{ formatMoney(person.totalCost) }}
        </div>
        <div v-if="!showDetails"
          class="rounded-md border border-slate-300 my-1 py-0.5 px-3 text-center text-xs transition-all shadow-sm text-slate-600 inline-block"
        >
          Management Cost Ratio: {{ person.managementRatio.toFixed(2) }}
        </div>
        <div v-if="!showDetails" class="text-xs text-slate-400">
          Show calculations ⌄
        </div>
        <div v-if="showDetails" class="mt-2">
          <hr class="h-0.5 border-t-0 bg-slate-100 dark:bg-white/10 my-2" />
          <div class="rounded-md border border-slate-300 mb-0 py-0.5 px-3 text-center text-xs transition-all shadow-sm text-slate-600 inline-block">
            Management Cost: {{ formatMoney(person.managementCost) }}
          </div>
          <div class="my-0">+</div>
          <div class="rounded-md border border-slate-300 my-0 py-0.5 px-3 text-center text-xs transition-all shadow-sm text-slate-600 inline-block">
            IC Cost: {{ formatMoney(person.icCost) }}
          </div>
          <div class="my-0">=</div>
          <div class="rounded-md border border-slate-300 py-0.5 px-3 text-center text-xs transition-all shadow-sm text-slate-600 inline-block">
            Total Cost: {{ formatMoney(person.totalCost) }}
          </div>
          <div class="rounded-md border border-slate-300 my-1 py-0.5 px-3 text-center text-xs transition-all shadow-sm text-slate-600 inline-block">
            Management Cost Ratio: {{ person.managementRatio.toFixed(2) }}
          </div>
          <div class="text-xs text-slate-400">
            Hide calculations ^
          </div>
        </div>
      </div>
      <div class="mt-2 text-xs text-center">
        <button
          v-if="person.children?.length"
          class="mt-2 px-2 py-1 !bg-white shadow-sm rounded hover:bg-gray-800 text-xs"
          @click="toggleExpanded"
        >
          {{ expanded ? 'Hide' : 'Show' }} {{ person.children?.length || 0 }}
          / {{ person.totalDescendants }} {{ expanded ? '^' : '⌄' }}
        </button>
        <div class="text-xs text-slate-400">
          Management Level {{ person.data.level || '?' }}
        </div>
      </div>
    </div>

    <!-- Children Container -->
    <div
      v-if="expanded"
      class="flex flex-col items-center"
    >
      <div class="flex justify-evenly gap-4 w-full">
        <PersonNode
          v-for="child in person.children"
          :key="child.id"
          :person="child"
        />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PersonNode',
  props: ['person'],
  data() {
    return {
      expanded: false,
      showDetails: false // State to toggle accordion details
    };
  },
  computed: {
    initials() {
      return this.person.data.Name?.split(' ').map(n => n[0]).join('').slice(0, 2).toUpperCase();
    }
  },
  methods: {
    toggleExpanded() {
      this.expanded = !this.expanded;
      if (this.expanded && !this.person.children) {
        this.$emit('load-children', this.person);
      }
    },
    formatMoney(val) {
      if (!val) return '$0';
      if (val >= 1000000000) {
        return `$${(val / 1000000000).toFixed(1)}B`;
      } else if (val >= 1000000) {
        return `$${(val / 1000000).toFixed(1)}M`;
      }
      return `$${(val / 1000).toFixed(1)}K`;
    },
  }
};
</script>

<style scoped>
</style>