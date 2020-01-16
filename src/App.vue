<template>
  <div id="app">
    <div>{{selectionIndex}}</div>
    <div>{{ value && value.slice(selectionIndex - 2, selectionIndex) }}</div>
    <div class="preview">{{ value }}</div>
    <hm-textarea-autocomplete
      class="form-control"
      placeholder="Type something here..."
      v-model="value"
      :auto-list="autoList"
      tag-class="form-control-tag"
      @input="doSomething"
      @updateSelection="selectionIndex = $event"
    />
  </div>
</template>

<script>
import HmTextareaAutocomplete from './components/textarea-autocomplete'

export default {
  name: 'app',
  components: {
    HmTextareaAutocomplete
  },
  data() {
    return {
      autoList: ["Khanh OC", "Yến Đặng", "Trần Toản"],
      value: null,
      selectionIndex: null
    };
  },
  methods: {
    doSomething() {
      // eslint-disable-next-line no-console
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin-top: 60px;
  padding: 5rem;
  .preview {
    background-color: #f96666;
    border-radius: 0.5rem;
    color: #fff;
    display: flex;
    font-size: 0.875rem;
    line-height: 1.38;
    margin: 1.5rem 0;
    overflow-wrap: break-word;
    padding: 0.375rem 0.75rem;
    width: 100%;
    white-space: pre-wrap;
  }
  .form-control {
    border: 1px solid #e4e4e4;
    border-radius: 0.5rem;
    font-size: 0.875rem;
    &-tag {
      background-color: #f96666;
      border-radius: 0.25rem;
      color: #ffffff;
      cursor: text;
    }
  }
}
</style>
