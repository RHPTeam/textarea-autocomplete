<template>
  <div class="hmta__wrap">
    <div class="hmta__wrap-body">
      <textarea
        class="hmta__texrarea"
        :style="computedStyles"
        :placeholder="placeholder"
        spellcheck="false"
        v-model="val"
        @focus="resize"
        ref="hmTextarea"
      ></textarea>
      <div class="hmta__wrap-mask" v-html="detectedValue">{{ detectedValue }}</div>
      <div
        class="hmta__auto-box"
        v-if="isShowAutoBox"
        :style="{left: autoBoxPosition.left + 'px', top: autoBoxPosition.top + 'px' }"
      >
        <div
          class="hmta__auto-item"
          v-for="(item, index) in autoList"
          :key="index"
          @click.stop="selectAutoItem(item)"
        >
          <slot name="auto-item">
            <span>{{ item }}</span>
          </slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "TextareaAutosize",
  props: {
    value: {
      type: [String, Number],
      default: ""
    },
    autoList: {
      type: Array
    },
    autosize: {
      type: Boolean,
      default: true
    },
    minHeight: {
      type: [Number],
      default: null
    },
    maxHeight: {
      type: [Number],
      default: null
    },
    disableNewLine: {
      type: Boolean,
      default: false
    },
    name: {
      type: String,
      default: null
    },
    placeholder: {
      type: String,
      default: "Type something here..."
    },
    /*
     * Force !important for style properties
     */
    important: {
      type: [Boolean, Array],
      default: false
    },
    tagClass: {
      type: String
    }
  },
  data() {
    return {
      autoBoxPosition: {
        left: 0,
        top: 0
      },
      height: "auto",
      isShowAutoBox: false,
      // works when content height becomes more then value of the maxHeight property
      maxHeightScroll: false,
      selectionIndex: null,
      // data property for v-model binding with real textarea tag
      val: null
    };
  },
  computed: {
    computedStyles() {
      if (!this.autosize) return {};
      return {
        resize: !this.isResizeImportant ? "none" : "none !important",
        height: this.height,
        overflow: this.maxHeightScroll
          ? "auto"
          : !this.isOverflowImportant
            ? "hidden"
            : "hidden !important"
      };
    },
    detectedValue() {
      // eslint-disable-next-line no-useless-escape
      const regex = /\{\{([^{\}{2}]+(?=}{2}))\}\}/g;
      if (this.val) {
        return this.val.replace(regex, item => {
          return `<span style="pointer-events: all" class="${this.tagClass}">${item}</span>`
        });
      }
      return this.val;
    },
    isResizeImportant() {
      const imp = this.important;
      return imp === true || (Array.isArray(imp) && imp.includes("resize"));
    },
    isOverflowImportant() {
      const imp = this.important;
      return imp === true || (Array.isArray(imp) && imp.includes("overflow"));
    },
    isHeightImportant() {
      const imp = this.important;
      return imp === true || (Array.isArray(imp) && imp.includes("height"));
    }
  },
  watch: {
    value(val) {
      this.val = val;
    },
    val(val) {
      this.$nextTick(this.resize);
      this.$emit("input", val);
    },
    minHeight() {
      this.$nextTick(this.resize);
    },
    maxHeight() {
      this.$nextTick(this.resize);
    },
    autosize(val) {
      if (val) this.resize();
    }
  },
  created() {
    this.val = this.value;
  },
  mounted() {
    this.resize();
    this.$nextTick(function() {
      document.addEventListener("selectionchange", this.getSelection, false);
      document.addEventListener("mouseup", this.getSelection, false);
      document.addEventListener("mousedown", this.getSelection, false);
      document.addEventListener("keyup", this.getSelection, false);
      // Prevent new line handler
      if (this.disableNewLine) {
        const elements = this.$el;
        elements.addEventListener("keydown", e => {
          if (e.code === "Enter" && !e.shiftKey) {
            e.preventDefault();
            this.$emit("onEnterHandler", this.val);
          }
        });
      }
      // Change tag style when click
      document.addEventListener("click", (e) => {
        if(e.target.className === this.tagClass) {
          e.target.style.pointerEvents = "none";
          e.target.style.color = "inherit";
          e.target.style.backgroundColor = "transparent";
        }
      });
    });
  },
  methods: {
    calculatePosition() {
      const str = this.val.slice(0, this.selectionIndex),
        lastNewLineIndex = str.lastIndexOf("\n");
      // Check exist new line character
      if (lastNewLineIndex !== -1) {
        const charactersCount = str.slice(lastNewLineIndex + 1).length,
          lineRegex = /\r\n|\r|\n/g,
          linesCount = str.match(lineRegex).length;
        this.autoBoxPosition.left = charactersCount * 5;
        this.autoBoxPosition.top = (linesCount + 1) * 30;
      } else {
        this.autoBoxPosition.left = str.length * 5;
        this.autoBoxPosition.top = 30;
      }
    },
    getSelection(e) {
      if (e.target.type === "textarea") {
        if (e.target.selectionStart) {
          console.log(e);
          this.selectionIndex = e.target.selectionEnd;
          if (this.val.slice(this.selectionIndex - 2, this.selectionIndex) === "{{") {
            this.calculatePosition()
            this.isShowAutoBox = true;
          } else {
            this.isShowAutoBox = false;
          }
          this.$emit("updateSelection", this.selectionIndex);
        }
      }
    },
    resize() {
      const important = this.isHeightImportant ? "important" : "";
      this.height = `auto${important ? " !important" : ""}`;
      this.$nextTick(() => {
        let contentHeight = this.$el.scrollHeight + 1;

        if (this.minHeight) {
          contentHeight =
            contentHeight < this.minHeight ? this.minHeight : contentHeight;
        }

        if (this.maxHeight) {
          if (contentHeight > this.maxHeight) {
            contentHeight = this.maxHeight;
            this.maxHeightScroll = true;
          } else {
            this.maxHeightScroll = false;
          }
        }

        const heightVal = contentHeight + "px";
        this.height = `${heightVal}${important ? " !important" : ""}`;
      });

      return this;
    },
    selectAutoItem(item) {
      const text = `{{${item}}}`;
      const res = [
        this.val.slice(0, this.selectionIndex - 2),
        text,
        this.val.slice(this.selectionIndex)
      ].join("");

      this.val = res;
      this.selectionIndex += text.length - 2;
      this.isShowAutoBox = false;
      this.$refs.hmTextarea.focus();
    },
  }
};
</script>

<style lang="scss">
.hmta__wrap {
  &-body {
    position: relative;
  }
  &-mask {
    background-color: transparent;
    border: 1px solid rgba(0, 0, 0, 0);
    border-radius: 0.5rem;
    font-size: 0.875rem;
    font-weight: normal;
    height: auto !important;
    left: 0;
    line-height: 1.38;
    overflow-wrap: break-word;
    padding: 0.375rem 0.75rem;
    pointer-events: none;
    position: absolute;
    top: 0;
    user-select: none;
    width: 100%;
    white-space: pre-wrap;
  }
}
.hmta__texrarea {
  background-color: transparent;
  border: 1px solid rgba(0, 0, 0, 0);
  border-radius: 0.5rem;
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: normal;
  line-height: 1.38;
  outline: 0;
  padding: 0.375rem 0.75rem;
  width: 100%;
  &::placeholder {
    color: #999999;
  }
}
.hmta__auto-box {
  background-color: #ffffff;
  border-radius: 0.5rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  padding: 0.25rem 0;
  position: absolute;
}
.hmta__auto-item {
  cursor: pointer;
  padding: 0.25rem 0.75rem;
  transition: all 0.4s ease;
  &:hover {
    background-color: #f2f2f2;
  }
}
</style>
