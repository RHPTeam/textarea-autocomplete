<template>
  <textarea
    :style="computedStyles"
    spellcheck="false"
    v-model="val"
    @focus="resize"
    ref="editor"
  ></textarea>
</template>

<script>
  export default {
    name: "TextareaAutosize",
    props: {
      value: {
        type: [String, Number],
        default: ""
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
      /*
       * Force !important for style properties
       */
      important: {
        type: [Boolean, Array],
        default: false
      }
    },
    data() {
      return {
        height: "auto",
        isShowTagDialog: false,
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
      });
    },
    methods: {
      getSelection(e) {
        if (e.target.type === "textarea") {
          if (e.target.selectionStart) {
            this.selectionIndex = e.target.selectionEnd;
            if (this.val.slice(this.selectionIndex - 2, this.selectionIndex) === "{{") {
              this.isShowTagDialog = true;
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
      }
    }
  };
</script>

<style lang="scss">
  textarea {
    background-color: transparent;
    border: 0;
    font-family: inherit;
    outline: 0;
    &::placeholder {
      color: #999999;
    }
  }
</style>
