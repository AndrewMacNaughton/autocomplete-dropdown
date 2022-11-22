<template>
  <div class="filters">
    <label for="filter" class="basic-dropdown">Auto Complete ComboBox</label>

    <auto-complete-drop-down
      @input="handleSelectChanged"
      :selectOptions="items"
      :value="selectedValue"
      :nestedList="nestedList"
    />
  </div>
</template>

<script>
import AutoCompleteDropDown from "./AutoCompleteDropDown.vue";

export default {
  components: { AutoCompleteDropDown },
  name: "Filters",
  data() {
    return {
      selectedValue: {},
      items: [
        {
          value: "Vancouver",
          id: 0,
          innerItems: [
            {
              value: "Gastown",
              id: 0,
            },
            {
              value: "Hastings",
              id: 1,
            },
            {
              value: "Granville Street",
              id: 2,
            },
            {
              value: "Kitsilano",
              id: 3,
            },
          ],
        },
        {
          value: "Squamish",
          id: 1,
          innerItems: [
            {
              value: "Downtown",
              id: 4,
            },
            {
              value: "Dentville",
              id: 5,
            },
            {
              value: "Highlands",
              id: 6,
            },
          ],
        },
        {
          value: "Whistler",
          id: 2,
          innerItems: [
            {
              value: "Creekside",
              id: 7,
            },
            {
              value: "Village",
              id: 8,
            },
            {
              value: "Village North",
              id: 9,
            },
          ],
        },
      ],
      singleItems: [
        {
          id: 0,
          value: "Whistler",
        },
        {
          id: 1,
          value: "Squamish",
        },
        {
          id: 2,
          value: "Vancouver",
        },
      ],
      nestedList: true,
    };
  },
  methods: {
    handleSelectChanged(event) {
      if (!("id" in event)) {
        this.selectedValue = {};
        return;
      }
      if (this.nestedList) {
        const { id, innerId } = event;
        const parentIndex = this.items.findIndex((item) => item.id === id);

        const parent = this.items[parentIndex].value;

        const child = this.items[parentIndex].innerItems.find(
          (item) => item.id === innerId
        );

        this.selectedValue = {
          text: `${parent} - ${child.value}`,
          id,
          innerId,
        };
      } else {
        const { id } = event;
        this.selectedValue = { id, text: this.items[id].value };
      }
    },
  },
};
</script>

<style scoped>
.filters {
  width: 300px;
  margin: 0 auto;
}
.filters ::v-deep .auto-complete {
  height: 36px;
  font-size: 16px;
}
::v-deep .suggestion-list {
  top: 42px;
}
.filter {
  text-align: left;
}
.result {
  margin-top: 30px;
  text-align: left;
}
label {
  display: block;
}
</style>