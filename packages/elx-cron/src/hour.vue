<template>
  <div class="elx-cron-hour">
    <el-radio
      v-model="valType"
      label="1">
      <span>每小时 允许的通配符[, - * /]</span>
    </el-radio>
    <el-radio
      v-model="valType"
      label="2">
      <span>周期从</span>
      <el-input-number
        v-model="valModel['2'].min"
        :min="1"
        :max="60">
      </el-input-number>
      <span>-</span>
      <el-input-number
        v-model="valModel['2'].max"
        :min="1"
        :max="60">
      </el-input-number>
      <span>小时</span>
    </el-radio>
    <el-radio
      v-model="valType"
      label="3">
      <span>从</span>
      <el-input-number
        v-model="valModel['3'].start"
        :min="0"
        :max="24">
      </el-input-number>
      <span>小时开始，每</span>
      <el-input-number
        v-model="valModel['3'].frequency"
        :min="1"
        :max="100">
      </el-input-number>
      <span>小时执行一次</span>
    </el-radio>
    <el-radio
      v-model="valType"
      label="4">
      <span>指定</span>
      <div
        class="hour-checkbox-group">
        <span>AM: </span>
        <elx-checkbox-group
          v-model="valModel['4']"
          class="hour-checkbox">
          <elx-checkbox
            :key="option.value"
            v-for="(option, index) in options"
            v-if="index<12"
            :label="option.value">
            {{option.label}}
          </elx-checkbox>
        </elx-checkbox-group>
        <span>PM: </span>
        <elx-checkbox-group
          v-model="valModel['4']"
          class="hour-checkbox">
          <elx-checkbox
            :key="option.value"
            v-for="(option, index) in options"
            v-if="index>11"
            :label="option.value">
            {{option.label}}
          </elx-checkbox>
        </elx-checkbox-group>
      </div>
    </el-radio>
  </div>
</template>

<script>
  export default {
    name: 'Hour',
    componentName: 'Hour',
    props: {
      value: {
        type: String,
        default: ''
      }
    },
    data() {
      const options = [];
      for (let i = 0; i < 24; i++) {
        if (i < 10) {
          options.push({value: '' + i, label: '0' + i});
        } else {
          options.push({value: '' + i, label: '' + i});
        }
      }
      return {
        currentValue: '',
        valType: '1',
        valModel: {
          '1': '*',
          '2': {min: 0, max: 2},
          '3': {start: 0, frequency: 1},
          '4': []
        },
        regexp: {
          '1': '^\\*$',
          '2': '^(\\d+)-(\\d+)$',
          '3': '^(\\d+)/(\\d+)$',
          '4': '((^(\\d+)(,\\d+)*$)|(^\\?$))'
        },
        options: options
      };
    },
    methods: {
      validate(val) {
        const self = this;
        let valType;
        const regexp = [];
        Object.keys(this.regexp).forEach(function(key) {
          const item = self.regexp[key];
          const testRegExp = new RegExp(item, 'g');
          const judge = testRegExp.test(val);
          if (judge) {
            valType = key;
          }
          if (judge) {
            regexp.push(item);
          }
          return judge;
        });
        if (regexp.length > 0) {
          return valType;
        } else {
          self.$emit('error', '(小时)格式错误');
          return null;
        }
      },
      updateValue() {
        const val = this.valType;
        let currentVal;
        if (val === '1') {
          currentVal = this.valModel[val];
        } else if (val === '2') {
          currentVal = this.valModel[val].min + '-' + this.valModel[val].max;
        } else if (val === '3') {
          currentVal = this.valModel[val].start + '/' + this.valModel[val].frequency;
        } else if (val === '4') {
          currentVal = this.valModel[val].length === 0 ? '?' : this.valModel[val].join(',');
        }
        this.currentValue = currentVal;
        this.$emit('input', currentVal);
      },
      initData() {
        let valType;
        if (!this.value || this.value === '') {
          this.valType = '1';
          this.$emit('input', this.valModel['1']);
          return false;
        }
        if (this.currentValue === this.value) {
          return false;
        }
        valType = this.validate(this.value);
        const regexp = this.regexp[valType];
        if (valType) {
          this.valType = valType;
          let x = 0;
          let y = 0;
          this.value.replace(new RegExp(regexp, 'g'), function(a, b, c, d, e) {
            x = b;
            y = c;
          });
          if (valType === '2') {
            this.valModel[valType].min = x;
            this.valModel[valType].max = y;
          }
          if (valType === '3') {
            this.valModel[valType].start = x;
            this.valModel[valType].frequency = y;
          }
          if (valType === '4') {
            if (this.value === '?') {
              this.valModel[valType] = [];
            } else {
              this.valModel[valType] = this.value.split(',');
            }
          }
        }
      }
    },
    watch: {
      valType(val, oldVal) {
        this.updateValue();
      },
      valModel: {
        deep: true,
        handler(val, oldVal) {
          this.updateValue();
        }
      },
      value() {
        this.initData();
      }
    },
    created() {
      this.initData();
    },
    mounted() {
    }
  };
</script>