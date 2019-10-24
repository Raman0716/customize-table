<template>
  <table class="tablesd"
         cellspacing="0"
         v-show="skuTag_1.length>0">
    <thead>
      <tr>
        <template v-for="(skuItem,index) in skuTitleList">
          <th :key="skuItem.key"
              v-if="skuItem.skuLabel && handleskuTitleList(index+1).length>0">
            {{skuItem.skuLabel}}
          </th>
        </template>
        <th>零售价格(元)</th>
        <th v-if="hasStock">库存</th>
      </tr>
    </thead>
    <tbody>
      <!-- body 里面 动态生成tr  -->
      <tr v-for="(len,index) in len_1*len_2*len_3"
          :key="index">
        <template v-if="(len-1)%(len_2*len_3)===0">
          <td v-if="skuTag_1[ parseInt((len-1) / (len_2*len_3)) ]"
              :rowspan="len_2*len_3"
              data-level='1'>
            {{ skuTag_1[ parseInt((len-1) / (len_2*len_3)) ].label }}
          </td>
        </template>
        <!-- 第二格 手动控制一下 -->
        <template v-if="((len-1) % len_3) === 0 && skuTag_2.length>0">
          <td v-if="skuTag_2[parseInt((len-1)/len_3) % len_2]"
              :rowspan="len_3"
              data-level='2'>
            {{ skuTag_2[parseInt((len-1)/len_3) % len_2].label }}
          </td>
        </template>

        <template v-if="skuTag_3.length>0">
          <td v-if="skuTag_3[(len-1) % len_3]"
              rowspan="1"
              data-level='3'>
            {{ skuTag_3[(len-1) % len_3].label }}
          </td>
        </template>
        <td>
          <template v-if="_skuPriceGroup[handlePriceKey(len)]">
            <span v-if="priceDisabled || disabled"> {{_skuPriceGroup[handlePriceKey(len)].price}} </span>
            <el-input v-else
                      v-model="_skuPriceGroup[handlePriceKey(len)].price"
                      size="mini"
                      type="number"
                      placeholder="请输入金额" />
          </template>
        </td>
        <td v-if="hasStock">
          <template v-if="_skuPriceGroup[handlePriceKey(len)]">
            <span v-if="disabled"> {{_skuPriceGroup[handlePriceKey(len)].stock}} </span>
            <el-input v-if="!disabled"
                      v-model="_skuPriceGroup[handlePriceKey(len)].stock"
                      size="mini"
                      type="number"
                      placeholder="请输入库存数量" />
          </template>
        </td>
      </tr>
    </tbody>
  </table>

</template>

<script>
export default {
  props: {
    DEVIDE_CHAR: {
      // 全局格式 key: `a${DEVIDE_CHAR}b`
      type: String,
      default: '-=-'
    },
    // 是否有库存选项
    hasStock: {
      type: Boolean,
      default: false
    },
    keyList: {
      type: Array,
      default: () => []
    },
    // { key: { price } }形式，key: `a${DEVIDE_CHAR}b`
    skuPriceGroup: {
      type: Object,
      default: () => {
        return {}
      }
    },
    // table 的自定义title: string[]
    skuTitleList: {
      type: Array,
      default: () => []
    },
    // 三个正常的 {label, disabled}[]，按照顺序排列 skuTag_1 ，skuTag_2 ，skuTag_3
    skuTag_1: {
      type: Array,
      default: () => []
    },
    skuTag_2: {
      type: Array,
      default: () => []
    },
    skuTag_3: {
      type: Array,
      default: () => []
    },
    // 是否禁用编辑
    disabled: {
      type: Boolean,
      default: false
    },
    // 是否禁用编辑价格
    priceDisabled: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    _keyList: {
      get() {
        return this.keyList
      },
      set(val) {
        this.$emit('update:keyList', val)
      }
    },
    _skuPriceGroup: {
      get() {
        return this.skuPriceGroup
      },
      set(val) {
        this.$emit('update:skuPriceGroup', val)
      }
    },
    len_1() {
      return this.skuTag_1.length || 1
    },
    len_2() {
      return this.skuTag_2.length || 1
    },
    len_3() {
      return this.skuTag_3.length || 1
    },
    tmpRes() {
      return [...this.skuTag_1, ...this.skuTag_2, ...this.skuTag_3]
    }
  },
  data() {
    return {
      parseInt: window.parseInt
    }
  },
  watch: {
    tmpRes: {
      deep: true,
      immediate: true,
      // 重新设置价格表
      handler: function(val) {
        const { DEVIDE_CHAR } = this
        let len = this.len_1 * this.len_2 * this.len_3
        let sl_2 = this.skuTag_2.length
        let sl_3 = this.skuTag_3.length
        this._keyList.splice(0)
        const handle = key => {
          this._keyList.push(key)
          if (!this._skuPriceGroup[key]) {
            let res = { price: '' }
            this.hasStock && (res.stock = '')
            this.$set(this._skuPriceGroup, key, res)
          }
        }
        this.skuTag_1.forEach(e1 => {
          !sl_2 && handle(e1.label)
          sl_2 &&
            this.skuTag_2.forEach(e2 => {
              !sl_3 && handle(`${e1.label}${DEVIDE_CHAR}${e2.label}`)
              sl_3 &&
                this.skuTag_3.forEach(e3 => {
                  handle(
                    `${e1.label}${DEVIDE_CHAR}${e2.label}${DEVIDE_CHAR}${e3.label}`
                  )
                })
            })
        })
      }
    }
  },
  methods: {
    handleskuTitleList(index) {
      return this[`skuTag_${index}`]
    },
    /**
     * @description 设置key，格式与上面 setPriceGroup 一致
     * @param len 当前循环到的位置
     */
    handlePriceKey(len) {
      const { len_1, len_2, len_3, DEVIDE_CHAR } = this
      let a = this.skuTag_1[parseInt(String((len - 1) / (len_2 * len_3)))]
      let b = this.skuTag_2[parseInt(String((len - 1) / len_3)) % len_2]
      let c = this.skuTag_3[parseInt(String((len - 1) % len_3))]
      return `${a ? a.label : ''}${b ? DEVIDE_CHAR + b.label : ''}${
        c ? DEVIDE_CHAR + c.label : ''
      }`
    }
  }
}
</script>
<style lang="scss" scoped>
.tablesd {
  $bc: 1px solid #dedede;
  table-layout: fixed;
  width: 100%;
  border-right: $bc;
  border-bottom: $bc;
  td,
  th {
    width: 120px;
    padding: 5px;
    text-align: center;
    border-left: $bc;
    border-top: $bc;
  }
}
</style>