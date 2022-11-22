<template>
  <div class="dropdown component form-input input-item">
    <label v-html="label"></label>
    <div class="input-wrapper">
      <input
        @focus="handleFocus"
        type="text"
        v-model="searchText"
        @input="handleInputInSearchField"
        @keyup.up="handleKeyStroke('up')"
        @keyup.down="handleKeyStroke('down')"
        @keyup.enter="handleKeyStroke('enter')"
        @keyup.esc="isSuggestionListVisible = false"
        class="auto-complete"
      />
      <span v-if="showClearIcon" @click="handleClearClicked" class="clear"
        >X</span
      >
    </div>

    <ul v-if="isSuggestionListVisible" class="suggestion-list">
      <template v-if="matches.length">
        <li
          v-for="(item, index) in matches"
          :key="item.id"
          @click="handleClickParent(item.id)"
          :class="{
            list: !nestedList,
            selected: !nestedList && selectedIndex === index,
          }"
        >
          <div
            class="outer-item"
            :class="{
              'selected-disabled':
                !valueSet && nestedList && item.overall === selectedIndex,
            }"
            @mouseover.self="selectedIndex = item.overall"
          >
            <i></i> {{ item.value }}
          </div>
          <ul v-if="nestedList" class="inner-list">
            <li
              v-for="innerItem in item.innerItems"
              :key="innerItem.id"
              class="list"
              @click="handleClickInner(item.id, innerItem.id)"
              @mouseover.self="selectedIndex = innerItem.overall"
              :class="{
                selected: innerItem.overall === selectedIndex,
              }"
            >
              <i></i>{{ innerItem.value }}
            </li>
          </ul>
        </li>
      </template>
      <li v-else>{{ noResultsMessage }}</li>
    </ul>
    <div
      @click="handleClickAway"
      v-if="isSuggestionListVisible"
      class="click-away-wrapper"
      data-testid="editable-clickaway"
    ></div>
  </div>
</template>

<script>
export default {
  name: "AutoCompleteDropDown",
  props: {
    value: {
      type: Object,
    },
    selectOptions: {
      required: true,
      type: Array,
    },
    nestedList: {
      default: false,
    },
    label: {
      type: String,
    },
    noResultsMessage: {
      type: String,
      default: () => "No Results found",
    },
    showClearIcon: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      searchText: "",
      isSuggestionListVisible: false,
      valueSet: false,
      selectedIndex: -1,
    };
  },
  watch: {
    value: {
      handler(newval) {
        // If the id property is not present then don't do anything. This means the input has been cleared
        if ("id" in this.value) {
          this.searchText = newval.text;
          this.valueSet = true;
        } else {
          this.valueSet = false;
          this.searchText = "";
        }
      },
    },
  },
  computed: {
    // Figure out the number of total visisble matches including nested
    totalMatches() {
      return this.matches.reduce((acc, item) => {
        // If its nested add the number of nested items
        if (this.nestedList) {
          acc += item.innerItems.length;
        }
        // Otherwise just return it.
        return acc;
        // Start the counter at the number of outer items -1
      }, this.matches.length - 1);
    },
    // This controls all the items in the list
    matches() {
      if (this.nestedList) {
        // If the value has already been set, make sure the correct values are returned
        if (this.value && this.valueSet) {
          return this.handleShowingCorrectValuesWhenNestedListItemSelected;
        }
        // Filter the data here based on what is in the text box
        let outerCounter = 0;
        return this.handleMatchesInNestedList.map((item) => {
          // This map function handles adding the overall index to the nested items to allow the arrows and enter
          // keys to be used for selection
          // Set this here as the outeritem needs a lower index than the nested items
          const overall = outerCounter;
          // Return the inner items with the current overall index
          const newItems = item.innerItems.map((inner) => {
            // Add 1 BEFORE adding to the list
            outerCounter++;
            return {
              ...inner,
              overall: outerCounter,
            };
          });
          // Increment the counter here
          outerCounter++;
          // Assign the inner items here
          item.innerItems = newItems;
          // Return with the item with overall index as well
          return {
            ...item,
            overall,
          };
        });
      } else {
        return this.handleMatchesInSingleList;
      }
    },
    // This is used to show the selected value once an item has been selected in nested mode
    handleShowingCorrectValuesWhenNestedListItemSelected() {
      // Find the parent item,
      const parentItem = this.copiedSearchOptions().find(
        (item) => item.id === this.value.id
      );
      // Find the single inner item
      const innerItem = [
        parentItem.innerItems.find((item) => item.id === this.value.innerId),
      ];
      // Assign the inner to the parent
      parentItem.innerItems = innerItem;
      // Needs to be an array to work
      return [parentItem];
    },
    handleMatchesInNestedList() {
      return this.copiedSearchOptions().filter((item) => {
        // If the search term exists in the parent object return the entire object
        if (item.value.toUpperCase().match(this.searchText.toUpperCase())) {
          return true;
        } else {
          // If it doesnt exist in the parent object, Check all the child objects to see if it exists
          // Filter all the items that don't contain the search string
          const filteredFruits = item.innerItems.filter((fruit) =>
            fruit.value.toUpperCase().match(this.searchText.toUpperCase())
          );
          // If there is a matching item return true. This ensures it shows up in the list
          if (filteredFruits.length) {
            // Only show the filtered items that match the search term
            item.innerItems = filteredFruits;
            return true;
          } else {
            // If no items are found return false so there are no empty datasets
            return false;
          }
        }
      });
    },
    // If its not a nested list just return the parent
    handleMatchesInSingleList() {
      return this.copiedSearchOptions().filter((item) => {
        // If the search term exists in the parent object return the entire object
        if (item.value.toUpperCase().match(this.searchText.toUpperCase())) {
          return true;
        }
      });
    },
  },
  methods: {
    handleClickInner(id, innerId) {
      this.isSuggestionListVisible = false;
      this.$emit("input", { id, innerId });
    },
    handleClickParent(id) {
      if (!this.nestedList) {
        this.$emit("input", { id });
        this.isSuggestionListVisible = false;
      }
    },
    handleInputInSearchField() {
      this.isSuggestionListVisible = this.searchText.length > 0;
      this.valueSet = false;
    },
    handleFocus() {
      this.isSuggestionListVisible = true;
    },
    // Used to allow the user to navigate search results with keyboard
    handleKeyStroke(e) {
      switch (e) {
        case "up":
          // Don't go above the the first match
          if (this.selectedIndex > -1) this.selectedIndex--;
          break;
        case "down":
          // Don't go below the bottom match
          if (this.selectedIndex < this.totalMatches) this.selectedIndex++;
          break;
        // Allow select with the enter key
        case "enter": {
          // Very different behaviour for nested lists so seperate there
          if (this.nestedList) {
            this.handleEnterPressedNestedList();
          } else {
            const match = this.matches[this.selectedIndex];
            this.$emit("input", { id: match.id });
          }
          // Hide the list right away
          this.isSuggestionListVisible = false;
          break;
        }
      }
    },
    handleClickAway() {
      this.isSuggestionListVisible = false;
    },
    // Use this a method as computed caches the results
    copiedSearchOptions() {
      return JSON.parse(JSON.stringify(this.selectOptions));
    },
    handleEnterPressedNestedList() {
      const checkIfParent = this.matches.find(
        (item) => item.overall === this.selectedIndex
      );
      // Return right away without doing anything as the user can't select the parent
      if (checkIfParent) return;

      // Find the overall index so that we can figure out the ids of what was selected
      this.matches.forEach((item) => {
        const innerItem = item.innerItems.find(
          (inner) => inner.overall === this.selectedIndex
        );
        if (innerItem) {
          this.$emit("input", { id: item.id, innerId: innerItem.id });
        }
      });
    },
    handleClearClicked() {
      this.searchText = "";
      this.$emit("input", {});
      this.valueSet = false;
      this.selectedIndex = -1;
    },
  },
};
</script>

<style scoped>
.dropdown {
  position: relative;
}
.input-wrapper {
  position: relative;
  width:100%;
}
.input-wrapper .clear {
  position: absolute;
  height: 30px;
  right: 13px;
  top: 13px;
  font-size: 16px;
  color: #aaa;
  cursor: pointer;
}
.suggestion-list {
  background-color: rgba(255, 255, 255, 0.95);
  border: 1px solid #eee;
  list-style: none;
  display: block;
  margin: 0;
  width: 100%;
  overflow: hidden;
  position: absolute;
  top: 65px;
  left: 0;
  z-index: 2;
  text-align: left;
  border-radius: 4px;
  padding: 8px;
  margin-top: 1px;
  color: #555;
  cursor: default;
}
.list {
  cursor: pointer;
  padding: 8px;
}

.selected {
  background-color: #ddd;
}
.selected-disabled {
  background-color: #fafafa;
  cursor: not-allowed;
}

.inner-list {
  padding-left: 20px;
  list-style: none;
}
input {
  width: 100%;
}
input:focus-visible {
  outline: none;
}
.click-away-helper {
  position: relative;
  z-index: 2 !important;
  max-height: 36px;
}

.click-away-wrapper {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0px;
  left: 0px;
  z-index: 1;
}
.outer-item {
  padding: 8px;
}
</style>
