<template>
  <div>
    <div class="i-layout-page-header header-title">
      <div class="fl_header">
        <span>
          <Button icon="ios-arrow-back" size="small" type="text" @click="$router.go(-1)">返回</Button>
        </span>
        <Divider type="vertical" />
        <span class="ivu-page-header-title">{{ $route.meta.title }}</span>
      </div>
    </div>
    <Card :bordered="false" dis-hover class="ivu-mt">
      <Form
        ref="formValidate"
        :model="formValidate"
        :label-width="labelWidth"
        :label-position="labelPosition"
        class="tabform"
        @submit.native.prevent
      >
        <Row :gutter="24" type="flex">
          <Col span="24">
            <FormItem label="选择商品：">
              <div class="box">
                <div class="box-item" v-for="(item, index) in goodsList" :key="index">
                  <img :src="item.image" alt="" />
                  <Icon type="ios-close-circle" size="20" @click="bindDelete(index, item)" />
                </div>
                <div class="upload-box" @click="selectGoods">
                  <Icon type="ios-camera-outline" size="36" />
                </div>
              </div>
            </FormItem>
          </Col>
        </Row>
      </Form>
      <div class="active-btn" v-if="goodsList.length > 0">
        <Button type="success" @click="liveGoods">生成直播商品</Button>
      </div>
      <div class="table-box" v-if="isShowBox">
        <Table
          :columns="columns1"
          :data="tabList"
          ref="table"
          class="mt25"
          :loading="loading"
          no-userFrom-text="暂无数据"
          no-filtered-userFrom-text="暂无筛选结果"
        >
          <template slot-scope="{ row, index }" slot="img">
            <div class="product_box">
              <img :src="row.image" alt="" />
              <span>{{ row.store_name }}</span>
            </div>
          </template>
          <template slot-scope="{ row, index }" slot="action">
            <a @click="del(row, index)">删除</a>
          </template>
        </Table>

        <div class="sub_btn">
          <Button type="primary" style="width: 8%" @click="bindSub" :disabled="disabled" :loading="loadings"
            >提交</Button
          >
        </div>
      </div>
    </Card>
    <Modal v-model="modals" title="商品列表" class="paymentFooter" scrollable width="900" :footer-hide="true">
      <goods-list
        ref="goodslist"
        :selectIds="selectIds"
        @getProductId="getProductId"
        v-if="modals"
        :ischeckbox="true"
      ></goods-list>
    </Modal>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import goodsList from '@/components/goodsList';
import { liveGoodsCreat, liveGoodsAdd } from '@/api/live';
export default {
  name: 'add_goods',
  components: {
    goodsList,
  },
  computed: {
    ...mapState('media', ['isMobile']),
    labelWidth() {
      return this.isMobile ? undefined : 100;
    },
    labelPosition() {
      return this.isMobile ? 'top' : 'right';
    },
  },
  data() {
    return {
      isShowBox: false,
      loading: false,
      modals: false,
      goodsList: [],
      tempGoods: {},
      formValidate: {},
      columns1: [
        { key: 'id', title: '商品ID' },
        { slot: 'img', title: '商品信息' },
        { key: 'price', title: '直播售价' },
        // {
        //   key: 'price',
        //   title: '直播售价',
        //   render: (h, params) => {
        //     return h('Input', {
        //       props: {
        //         type: 'number',
        //         value: params.row.price,
        //       },
        //       on: {
        //         input: (val) => {
        //           this.tabList[params.index].price = val;
        //         },
        //       },
        //     });
        //   },
        // },
        // {
        //   key: 'cost_price',
        //   title: '直播原价',
        //   render: (h, params) => {
        //     return h('Input', {
        //       props: {
        //         type: 'number',
        //         value: params.row.cost_price,
        //       },
        //       on: {
        //         input: (val) => {
        //           this.tabList[params.index].cost_price = val;
        //         },
        //       },
        //     });
        //   },
        // },
        { key: 'stock', title: '库存' },
        { slot: 'action', fixed: 'right', title: '操作' },
      ],
      tabList: [],
      disabled: false,
      loadings: false,
    };
  },
  methods: {
    selectGoods() {
      this.modals = true;
      this.selectIds = this.goodsList.map((i) => {
        return i.product_id;
      });
    },
    // 生成直播商品
    liveGoods() {
      let array = [];
      this.goodsList.map((el) => {
        array.push(el.product_id);
      });
      liveGoodsCreat({
        product_id: array,
      })
        .then((res) => {
          this.tabList = res.data;
          this.isShowBox = true;
        })
        .catch((error) => {
          this.$Message.error(error.msg);
        });
    },
    getProductId(data) {
      this.goodsList = data;
      this.$nextTick((res) => {
        setTimeout(() => {
          this.modals = false;
        }, 300);
      });
    },
    bindDelete(index, item) {
      this.goodsList.splice(index, 1);
      let i = this.tabList.findIndex((e) => e.id == item.product_id);
      this.tabList.splice(i, 1);
      if (!this.goodsList.length) {
        this.isShowBox = false;
      }
    },
    del(row, index) {
      this.tabList.splice(index, 1);
      let i = this.goodsList.findIndex((e) => e.product_id == row.id);
      this.goodsList.splice(i, 1);
      if (!this.tabList.length) {
        this.isShowBox = false;
      }
    },
    // 提交
    bindSub() {
      this.disabled = true;
      this.loadings = true;
      liveGoodsAdd({
        goods_info: this.tabList,
      })
        .then((res) => {
          this.$Message.success('添加成功');
          this.disabled = false;
          setTimeout(() => {
            this.$router.push({ path: this.$routeProStr + '/marketing/live/live_goods' });
          }, 500);
        })
        .catch((error) => {
          this.disabled = false;
          this.$Message.error(error.msg);
        });
    },
  },
};
</script>

<style lang="stylus" scoped>
.upload-box {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 60px;
  height: 60px;
  background: #ccc;
}

.box {
  display: flex;
  flex-wrap: wrap;

  .box-item {
    position: relative;
    margin-right: 20px;

    .ivu-icon {
      position: absolute;
      right: -10px;
      top: -8px;
      color: #999;
      cursor: pointer;
    }
  }

  .upload-box, .box-item {
    width: 60px;
    height: 60px;
    margin-bottom: 10px;

    img {
      width: 100%;
      height: 100%;
    }
  }
}

.active-btn {
  padding-left: 96px;
}

.table-box {
  margin: 0 107px;
}

.sub_btn {
  margin-top: 10px;
}

.product_box {
  display: flex;

  img {
    width: 36px;
    height: 36px;
    margin-right: 10px;
  }
}
</style>
