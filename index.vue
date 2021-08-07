<template>
  <dh-select
    v-model="searchParam.shop_id"
    title="店铺"
    class="filter"
    more
    filterable
    :loading="filterLoading"
    :option="shop_option"
    @change="handleSearchByPage"
    @search="handleFilterShopSearch"
    @more="handleMoreOption"
  />
</template>

<script>
import DhSelect from '@/components/DhFilter/DhSelect'
import { getShopList } from '@/api/POS/shop/shop'
export default {
  name: 'GoodsList',
  components: { DhSelect },
  data() {
    return {
      filterLoading: false,
      // 筛选信息
      searchObj: {
        search: '',
        page: 1,
        size: 20
      },
      shop_option: [],
      optionTotal: 0,
    }
  },
  created() {
    this.getShopList(this.searchObj)
  },
  methods: {


    /**
     * 网络请求函数
     */
    // 获取店铺列表
    async getShopList(param) {
      try {
        this.filterLoading = true
        const { data: res } = await getShopList(param)
        if (res.code !== 0) {
          return this.$message.warning(res.message)
        }
        this.shop_option.push(... res.data.records.map(v => ({
          value: v.id,
          label: v.shop_name
        })))
        this.optionTotal = res.data.total
      } finally {
        this.filterLoading = false
      }
    },

    /**
     * 筛选事件
     */
    // 店铺选择器
    handleFilterShopSearch(search) {
      this.shop_option = []
      this.searchObj.search = search
      this.searchObj.page = 1
      this.getShopList(this.searchObj)
    },
    // 店铺加载更多
    handleMoreOption() {
      console.log(this.optionTotal, this.shop_option.length)
      if (this.optionTotal > this.shop_option.length) {
        this.searchObj.page += 1
        this.getShopList(this.searchObj)
      }
    },


  }
}
</script>
