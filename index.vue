<template>
  <div>
    <el-form ref="skuTitleListRef"
             size="small"
             @submit.native.prevent
             label-width="0">
      <el-form-item prop="skuTitleList"
                    v-for="(skuItem,skuIndex) in skuTitleList"
                    :key="skuItem.key">
        <div>
          <el-input v-model="skuItem.skuLabel"
                    class="standard_list"
                    :maxlength="10"
                    :disabled="operateType==='edit' && skuItem.disabled"
                    clearable>
            <span slot="suffix">{{skuItem.skuLabel.length}}/10</span>
          </el-input>
          <el-link type="danger"
                   :underline="false"
                   v-if="handleDeleteBtn(skuIndex+1)"
                   @click.native="removeSkuItem(skuIndex)">删除</el-link>
        </div>
        <div class="mt5">
          <el-button :disabled="!handleAddSkuBtn(skuIndex+1)"
                     class="tags">
            <el-icon class="el-icon-plus" />
          </el-button>
          <el-tag class="tags"
                  v-for="(item,tagIndex) in handleskuTitleList(skuIndex+1)"
                  :key="tagIndex">
            {{item.label}}
            <el-icon class="el-icon-close"
                     @click.native.stop="removeStandard(item,`skuTag_${skuIndex+1}`)" />
          </el-tag>
        </div>
      </el-form-item>
      <el-form-item>
        <el-button @click="addSkuItem"
                   :disabled="skuTitleList.length>=3"> 添加</el-button>
      </el-form-item>
    </el-form>

    <innerTable :skuTag_1="skuTag_1"
                :skuTag_2="skuTag_2"
                :skuTag_3="skuTag_3"
                :skuTitleList="skuTitleList"
                :keyList.sync="keyList"
                :skuPriceGroup.sync="skuPriceGroup" />
  </div>
</template>

<script>
import innerTable from './inner-table.vue'
// interface SkuPriceGroup {
//   stock: string;
//   price: string;
//   id: string;
// }
export default {
  components: {
    innerTable
  },
  computed: {
    operateType() {
      return this.$route.query.operateType
    }
  },
  data() {
    return {
      skuTitleList: ['A', 'B', 'C'],
      skuTag_1: ['a1', 'a2'],
      skuTag_2: ['b1', 'b2'],
      skuTag_3: ['c1', 'c2'],
      keyList: [], // string[]
      skuPriceGroup: [] // SkuPriceGroup[]
    }
  },
  methods: {
    // 编辑状态下不可删除
    handleDeleteBtn(index) {
      const s = this[`skuTag_${index}`]
      return s && !s.find(e => e.disabled)
    },
    // 最多20个
    handleAddSkuBtn(index = 1, max = 10) {
      const rjl = this[`skuTag_${index}`].length
      return rjl < max
    },
    handleskuTitleList(index = 1) {
      return this[`skuTag_${index}`]
    },
    addSkuItem() {
      let { skuTitleList } = this
      if (skuTitleList.length >= 3) {
        return
      }
      skuTitleList.push({
        key: Date.now(),
        skuLabel: '',
        disabled: false
      })
    },
    /**
     * @description 删除规格大类
     * @param index 索引
     * @param needConfirm 是否要confirm
     * @param removeTitle 是否保留原来的规格大类title
     */
    removeSkuItem(index = 0, needConfirm = true, removeTitle = true) {
      const fn = () => {
        let { skuTitleList } = this
        removeTitle && skuTitleList.splice(index, 1)
        let sl = skuTitleList.length
        if (index === 0) {
          this.skuTag_1 = [...this.skuTag_2]
          this.skuTag_2 = [...this.skuTag_3]
        }
        if (index === 1) {
          this.skuTag_2 = [...this.skuTag_3]
        }
        sl === 1 && (this.skuTag_2 = [])
        sl === 0 && (this.skuTag_1 = [])
        this.skuTag_3 = []
      }
      needConfirm &&
        this.$confirm('确定删除吗', '提示')
          .then(fn)
          .catch(() => {})
      !needConfirm && fn()
    }
  },
  removeStandard(item, tagArrName = 'skuTag_1') {
    const index = this[tagArrName].findIndex(e => e.label === item.label)
    index > -1 && this[tagArrName].splice(index, 1)
    if (this.skuTag_3.length === 0) {
      this.removeSkuItem(2, false, tagArrName === 'skuTag_3')
    }
    if (this.skuTag_2.length === 0) {
      this.removeSkuItem(1, false, tagArrName === 'skuTag_2')
    }
    if (this.skuTag_1.length === 0) {
      this.removeSkuItem(0, false, tagArrName === 'skuTag_1')
    }
  }
}
</script>