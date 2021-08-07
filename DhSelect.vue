<template>
  <!-- 自定义封装的条件选择框 -->
  <div>
    <el-popover trigger="click" :width="optionWidth" popper-class="my-popper">
      <div class="select-option">
        <!-- 搜索区 -->
        <div v-if="filterable" class="option-search">
          <div class="search-icon">
            <i class="el-icon-search" />
          </div>
          <div class="search-input">
            <input v-model="search" placeholder="搜索" @input="handleSearch">
          </div>
          <div class="search-clear">
            <i class="el-icon-error" @click="handleSearchClear" />
          </div>
        </div>
        <!-- 信息栏 -->
        <div v-if="isClear" class="option-top">
          <div class="top-label">已选{{ getLength }}项</div>
          <div class="top-line" />
          <div class="top-clear" @click="handleClear">清除已选</div>
        </div>
        <!-- 选择项 -->
        <div v-loading="loading" class="option">
          <el-scrollbar ref="scrollbar">
            <div
              v-for="(item,index) in option"
              :key="index"
              class="option-item"
              :style="`width:${optionWidth}px`"
              @click="check(item)"
            >
              <div class="item-left">
                <div v-if="icon" class="item-icon">
                  <img :src="item.icon">
                </div>
                <div class="item-title" :title="item.label">{{ item.label }}</div>
              </div>
              <div class="item-check">
                <i v-show="isCheck(item.value)" class="el-icon-check" />
              </div>
            </div>
          </el-scrollbar>
        </div>
      </div>
      <!-- 外面文本框 -->
      <div slot="reference" class="select-label" :class="labelBg ? 'select-label-bg' : ''" :style="`height:${labelHeight};`">
        <div class="left">
          <span class="title">{{ title }}</span>
          <span class="label" :style="`width: ${labelWidth};min-width:20px `">{{ getLabel }}</span>
        </div>
        <div class="icon"><i class="el-icon-caret-bottom" /></div>
      </div>
    </el-popover>
  </div>
</template>

<script>
export default {
  name: 'DhSelect',
  props: {
    // 选择框的标题
    title: {
      type: String,
      required: true
    },
    // 可供选择项：{ icon: '', label: '', value: '' }
    option: {
      type: Array,
      required: true
    },
    value: {
      type: [String, Number, Array],
      default: ''
    },
    // 是否开启清除已选
    isClear: {
      type: Boolean,
      default: true
    },
    // 是否开启图标
    icon: {
      type: Boolean,
      default: false
    },
    // 选择区的宽度
    optionWidth: {
      type: String,
      default: '200'
    },
    // 选择器的高度
    labelHeight: {
      type: String,
      default: '32px'
    },
    // 选择器文本的宽度
    labelWidth: {
      type: String,
      default: 'auto'
    },
    // 是否开启多选
    multiple: {
      type: Boolean,
      default: false
    },
    // 是否可筛选
    filterable: {
      type: Boolean,
      default: false
    },
    labelBg: {
      type: Boolean,
      default: false
    },
    // 是否查询数据中
    loading: {
      type: Boolean,
      default: false
    },
    // 是否可加载更多
    more: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      data: [],
      label: [],
      search: '',
      oneData: '',
      oneLabel: '',
      TimeId: -1 // 作为节流的id
    }
  },
  computed: {
    getLabel() {
      if (this.multiple) {
        return this.label.length > 0 ? this.label.join('，') : '全部'
      }
      return this.oneLabel || '全部'
    },

    getLength() {
      if (this.multiple) {
        return this.data.length
      }
      return this.oneLabel ? 1 : 0
    }
  },
  watch: {
    value() {
      this.data = []
      this.label = []
      this.oneData = ''
      this.oneLabel = ''
      if (this.multiple) {
        this.data = JSON.parse(JSON.stringify(this.value))
        this.option.forEach(v => {
          if (this.value.includes(v.value)) {
            this.label.push(v.label)
          }
        })
      } else {
        this.oneData = this.value
        for (let i = 0; i < this.option.length; i++) {
          if (this.option[i].value === this.value) {
            this.oneLabel = this.option[i].label
            break
          }
        }
      }
    }
  },
  created() {
    this.data = []
    this.label = []
    this.oneData = ''
    this.oneLabel = ''
    if (this.multiple) {
      this.data = JSON.parse(JSON.stringify(this.value))
      this.option.forEach(v => {
        if (this.value.includes(v.value)) {
          this.label.push(v.label)
        }
      })
    } else {
      this.oneData = this.value
      for (let i = 0; i < this.option.length; i++) {
        if (this.option[i].value === this.value) {
          this.oneLabel = this.option[i].label
          break
        }
      }
    }
  },
  mounted() {
    if (this.more) {
      this.handleScroll()
    }
  },
  methods: {
    // 是否选中
    isCheck(value) {
      if (this.multiple) {
        return this.data.includes(value)
      }
      return this.oneData === value
    },
    // 选中
    check(item) {
      // 多选的情况
      if (this.multiple) {
        const index = this.data.indexOf(item.value)
        // 代表取消选中
        if (index !== -1) {
          this.data.splice(index, 1)
          this.label.splice(index, 1)
        } else {
        // 代表选择
          this.data.push(item.value)
          this.label.push(item.label)
        }
        this.$emit('input', this.data)
        this.$emit('change', this.data)
      } else {
        // 单选的情况
        if (this.oneData === item.value) {
          this.oneData = ''
          this.oneLabel = ''
        } else {
          this.oneData = item.value
          this.oneLabel = item.label
        }
        this.$emit('input', this.oneData)
        this.$emit('change', this.oneData)
      }
    },
    // 清除选中
    handleClear() {
      if (this.multiple) {
        this.data.splice(0, this.data.length)
        this.label.splice(0, this.label.length)
        this.$emit('input', this.data)
        this.$emit('change', this.data)
      } else {
        this.oneData = ''
        this.oneLabel = ''
        this.$emit('input', this.oneData)
        this.$emit('change', this.oneData)
      }
    },
    // 搜索清除
    handleSearchClear() {
      this.search = ''
      this.$emit('search', this.search)
    },

    // 监听底部滚动
    handleScroll() {
      const scrollbarEl = this.$refs.scrollbar.wrap
      scrollbarEl.onscroll = () => {
        if (scrollbarEl.scrollTop + scrollbarEl.offsetHeight >= scrollbarEl.scrollHeight) {
          this.$emit('more')
        } else {
          this.isFixedTop = false
        }
      }
    },

    // 防抖算法
    handleSearch() {
      clearTimeout(this.TimeId)
      this.TimeId = setTimeout(() => {
        this.$emit('search', this.search)
      }, 500)
    }
  }
}
</script>
<style>
.my-popper {
  padding: 0!important;
}
</style>
<style lang="scss" scoped>
// 外面文本框
.select-label {
  display: flex;
  align-items: center;
  border-radius: 4px;
  padding: 0 12px;
  font-size: 14px;
  cursor: pointer;
  justify-content: space-between;
  border: 1px solid #E6E6E6;
  background-color: #E9E9E9;

  // &:hover {
  //   background-color: #f5f5f5;
  // }

  .left {
    display: flex;
  }
  .title {
    color: #999999;
  }
  .label {
    color: #333333;
    margin: 0 10px;
    overflow: hidden;
    text-overflow:ellipsis;
    white-space: nowrap;
  }
  .icon {
    display: flex;
    width: 14px;
    height: 100%;
    align-items: center;
    color: #C0C4CC;
  }
}
.select-label-bg {
  background-color: #f5f5f5;
}
// 选择区
.select-option {
  font-size: 14px;
  padding-bottom: 6px;

  // 搜索区
  .option-search {
    display: flex;
    height: 40px;
    padding: 6px 12px;
    align-items: center;
    box-shadow: 0 1px 0 0 rgba(0,0,0,.05);

    .search-input {
      margin-left: 10px;

      input {
        width: 100%;
        background:none;
        outline:none;
        border: none;
      }
      input:focus {
        border:none;
      }
      input::-webkit-input-placeholder{
        color: #D9D9D9;
      }
    }
    .search-clear {
      cursor: pointer;
    }
  }

  // 信息区
  .option-top {
    display: flex;
    color: #999999;
    padding: 6px 12px;
    height: 30px;
    font-size: 12px;
    align-items: center;

    .top-line {
      height: 14px;
      border-left: 1px solid #cccccc;
      margin: 0 6px;
    }
    .top-clear {
      color: #00CC88;
      cursor: pointer;
    }
  }
  // 选择区
  .option-item {
    display: flex;
    height: 36px;
    line-height: 36px;
    color: #333333;
    padding: 0 12px;
    cursor: pointer;
    box-sizing: border-box;
    justify-content: space-between;

    &:hover {
      background-color: #f5f7fa;
    }

    .item-left {
      display: flex;
      width: calc(100% - 14px);
    }

    .item-icon {
      height: 100%;
      margin-right: 6px;
      display: flex;
      align-items: center;
      img {
        margin-top: 2px;
        width: 16px;
        height: 16px;
        background-color: #f5f7fa;
        border-radius: 50%;
      }
    }
    .item-title {
      flex: 1;
      overflow: hidden;
      text-overflow:ellipsis;
      white-space: nowrap;
    }
    .item-check {
      width: 14px;
      color: #00CC88;
      font-weight: bold;
    }
  }
}
::v-deep .el-scrollbar {
	.el-scrollbar__wrap {
    max-height: 250px; // 最大高度
    overflow-x: hidden !important;
	}
}
::v-deep .el-scrollbar__bar.is-horizontal {
  display: none;
}
</style>

