<template>
  <div
    class="elx-table-form-cell"
    :class="isError ? 'error' : ''">
    <el-tooltip
      class="item"
      effect="light"
      placement="top"
      :content="String(cellText)">
      <span
        class="text"
        v-text="cellText">
      </span>
    </el-tooltip>
    <span
      class="uex-icon-warning"
      v-if="isError">
    </span>
    <div
      class="el-form-item__error"
      v-text="errorMsg">
    </div>
  </div>
</template>
<script>
import AsyncValidator from 'async-validator';

export default {
  name: 'Cell',
  componentName: 'Cell',

  props: {
    cellText: {},
    field: String,
    cellData: Object,
    rules: Object,
    fieldShow: Boolean,
    fieldIndex: {},
    activeRowIndex: {},
    activeColumnKey: {},
    editType: {},
    type: {}
  },
  data() {
    return {
      isError: false,
      errorMsg: ''
    };
  },
  methods: {
    validData(data, rules) {
      const self = this;
      let judge = true;
      const validator = new AsyncValidator(rules);
      validator.validate(data, { firstFields: true }, function(errors, invalidFields) {
        if (errors === null) {
          self.isError = false;
          self.errorMsg = '';
        } else {
          judge = false;
          self.isError = true;
          self.errorMsg = errors[0].message;
        }
      });
      return judge;
    },
    validate() {
      const fieldIndex = this.fieldIndex;
      const activeRowIndex = this.activeRowIndex;
      const activeColumnKey = this.activeColumnKey;
      const editType = this.editType;
      const type = this.type;
      const isStr = (type === 'string');
      let visible;
      if (editType === 'single') {
        visible = isStr || (!isStr && type !== 'index' && (fieldIndex !== activeRowIndex || this.field !== activeColumnKey));
      } else {
        visible = isStr || false;
      }

      if (this.fieldShow && visible) {
        if (this.rules) {
          if (this.field in this.rules) {
            const data = {};
            data[this.field] = this.cellData[this.field];
            const rules = {};
            rules[this.field] = this.rules[this.field];
            return this.validData(data, rules);
          }
        }
      }
      return true;
    }
  },
  watch: {
    cellText(val, oldVal) {
      this.validate();
    }
  },
  created() {
  }
};
</script>
