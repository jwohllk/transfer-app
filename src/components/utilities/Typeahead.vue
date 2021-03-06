<template>
  <div :class="typeaheadState">
      <div class="typeahead__toggle" ref="toggle" @mousedown.prevent="toggle">
          <input type="text" class="typeahead__search" ref="search"
              v-model="search"
              @focus="onFocus"
              @blur="onBlur"
              @keydown.esc="onEscape"
              @keydown.down="onDownKey"
              @keydown.up="onUpKey"
              @keydown.enter="onEnterKey"
              >
          <span class="typeahead__text" ref="text">{{displayText}}</span>
      </div>
      <ul class="typeahead__list" ref="list" v-if="open">
          <li class="typeahead__item" v-for="(option, index) in filteredOptions" :key="index">
              <a class="typeahead__link" @mousedown.prevent="select(option)"
                  :class="[selectIndex === index ? 'typeahead__active':'']"
                  >
                  {{option[displayProperty]}}
              </a>
          </li>
      </ul>
  </div>
</template>
<script type="text/javascript">

export default {
  props: {
    options: {
      type: Array,
      default() {
        return [];
      }
    },
    value: {
      type: [String, Number],
      default: null
    },
    searchProperty: {
      type: [String],
      default: "text"
    },
    displayProperty: {
      type: [String],
      default: "text"
    }
  },
  data() {
    return {
      open: false,
      selectIndex: 0,
      displayText: "",
      search: ""
    };
  },
  computed: {
    typeaheadState() {
      return this.open ? "typeahead typeahead__open" : "typeahead";
    },
    filteredOptions() {
      const exp = new RegExp(this.search, "i");
      return this.options.filter(option => {
        return exp.test(option[this.searchProperty]);
      });
    }
  },
  methods: {
    onDownKey() {
      if (this.filteredOptions.length - 1 > this.selectIndex) {
        this.selectIndex++;

        // scroll when overflow
        if (this.selectIndex > 2) {
          this.$refs.list.scrollTop += 20 + this.selectIndex;
        }
      }
    },
    onUpKey() {
      if (this.selectIndex > 0) {
        this.selectIndex--;

        // scroll when overflow
        if (this.selectIndex > 0) {
          this.$refs.list.scrollTop -= 20 + this.selectIndex;
        }
      }
    },
    onEnterKey() {
      const option = this.filteredOptions[this.selectIndex];

      if (option) {
        this.select(option);
      }
    },
    select(option) {
      (this.displayText = option[this.displayProperty]),
        this.$emit("input", option.id);
      this.$refs.search.blur();
    },
    toggle(e) {
      if (
        e.target === this.$refs.toggle ||
        e.target === this.$refs.search ||
        e.target === this.$refs[this.displayProperty]
      ) {
        if (this.open) {
          if (
            e.target !== this.$refs.search &&
            e.target !== this.$refs[this.displayProperty]
          ) {
            this.$refs.search.blur();
          }
        } else {
          this.$refs.search.focus();
        }
      }
    },
    onFocus() {
      this.open = true;
    },
    onBlur() {
      this.search = "";
      this.selectIndex = 0;
      this.$refs.list.scrollTop = 0;
      this.open = false;
    },
    onEscape() {
      this.$refs.search.blur();
    }
  }
};

</script>
<style lang="scss">

.typeahead {
  border-radius: 3px;
  border: 1px solid #ccc;
  position: relative;
  z-index: 1;
  width: 100%;
  font-size: 14px;
}
.typeahead__open {
  border: 1px solid rgb(204, 56, 56);
}
.typeahead__open .typeahead__text {
  color: #999;
  opacity: 0.2;
}
.typeahead__toggle {
  position: relative;
  z-index: 1;
  width: 100%;
}
.typeahead__search {
  position: absolute;
  top: 0;
  left: 0;
  line-height: 1em;
  font-size: 1em;
  padding: 10px;
  width: 100%;
  display: block;
  cursor: text;
  background: transparent;
  border: none;
  outline: none;
  z-index: 2;
}
.typeahead__text {
  min-height: 36px;
  font-size: 1em;
  line-height: 1em;
  padding: 10px;
  display: inline-block;
  position: relative;
  z-index: 3;
}
.typeahead__list {
  margin: 0;
  padding: 0;
  max-height: 200px;
  overflow-y: scroll;
}
.typeahead__item {
  display: block;
  border-top: 1px solid #f4f4f4;
}
.typeahead__link {
  display: block;
  padding: 10px;
  line-height: 1em;
  font-size: 1em;
  cursor: pointer;
}
.typeahead__active {
  background: rgb(226, 76, 76);
  color: #fff;
}

</style>