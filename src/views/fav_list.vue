<template>
  <div id="goods_list">
    <div class="goods-list-main">
      <div class="list-wrapper" v-show="goodsList.length">
        <ul class="list">
          <li class="item" v-for="(item,index) in goodsList" :key="item.goods_id" @click="openDetail(item.goods_id)">
            <div class="item-main">
              <div class="icon" :class="{'active':item.isStorages}" @click.stop="storageGoods(index)">
                <span class="iconfont icon-fav-full"></span>
              </div>
              <div class="img">
                <img class="lazy-goods" v-lazy="item.goods_image" alt="">
              </div>
              <div class="desc">
                <p class="name">{{item.goods_name}}</p>
                <p class="price">￥{{item.goods_price}}</p>
              </div>
            </div>
          </li>
        </ul>
      </div>
      <div class="load" @click="loadMore" v-show="goodsList.length">
        <p>{{loadText}}</p>
      </div>
      <div class="nomal-data" v-show="!goodsList.length&&!hasMore">
        <div class="desc">
          <img src="./../assets/images/fav_nomal.png" alt="">
          <p>暂无收藏的商品哦～</p>
        </div>
      </div>
    </div>
    <!-- footer -->
    <app-footer></app-footer>
    <!-- loading -->
    <loading v-if="loading"></loading>
  </div>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";
import appFooter from "../components/footer";
import Loading from "../components/loading";

export default {
  components: { appFooter, Loading },
  data() {
    return {
      loading: true,
      ids: 0,
      size: 10,
      page: 1,
      goodsList: [],
      hasMore: false,
      loadText: "加载更多"
    };
  },
  created() {
    this.init();
  },
  methods: {
    init() {
      this.ids = localStorage.getItem("goodsID");
      if (this.ids) {
        this.getGoodsList();
      } else {
        this.loading = false;
      }
    },
    getGoodsList() {
      this.$http
        .get(this.$URL + "wap/goods/cookie-list", {
          params: {
            ids: this.ids,
            size: this.size,
            page: this.page
          }
        })
        .then(res => {
          if (res.data.code == 0) {
            this.goodsList = this.goodsList.concat(res.data.data);

            if (this.loading) {
              setTimeout(() => {
                this.loading = false;
              }, 300);
            }
            var s = localStorage.getItem("goodsID");
            if (s) {
              var t = s.split(",");
              for (var i = 0; i < this.goodsList.length; i++) {
                for (var j = 0; j < t.length; j++) {
                  if (this.goodsList[i].goods_id == t[j]) {
                    this.$set(this.goodsList[i], "isStorages", true);
                  }
                }
              }
            }

            this.hasMore =
              res.data.pagination.current_page >=
              res.data.pagination.total_pages
                ? false
                : true;
            if (!this.hasMore) {
              this.loadText = "没有更多";
              return;
            }else{
              this.loadText = "加载更多";
            }
            this.page++;
          }
        });
    },
    loadMore() {
      if (this.hasMore) {
        this.getGoodsList();
      }
    },
    storageGoods(index) {
      var s = localStorage.getItem("goodsID");
      var t = [];
      var id = this.goodsList[index].goods_id.toString();
      if (s) {
        t = s.split(",");
        t = t.filter(function(item) {
          return item != id;
        });
      }
      t.unshift(id);

      if (this.goodsList[index].isStorages) {
        this.$set(this.goodsList[index], "isStorages", false);
        t.splice(t.indexOf(id), 1);
        this.goodsList.splice(index, 1);
      } else {
        this.$set(this.goodsList[index], "isStorages", true);
      }

      localStorage.setItem("goodsID", t);
    },
    openDetail(id) {
      this.$router.push({
        path: `/goodsDetails/${id}`
      });
    }
  },
  watch: {
    $route(to, from) {
      this.page = 1;
      this.goodsList = [];
      this.init();
    }
  }
};
</script>

<style scoped lang="scss">
@keyframes fade {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
.lazy-goods[lazy="loaded"] {
  animation: fade 0.5s;
}
#goods_list {
  min-height: calc(100% - 4.4rem - 19.2rem);
  background: #f0f0f0;
  .goods-list-main {
    min-height: 43.1rem;
  }
  .list-wrapper {
    padding: 0.5rem;
    .list {
      display: flex;
      flex-wrap: wrap;
      .item {
        width: 50%;
        .item-main {
          position: relative;
          margin: 0.5rem;
          padding: 1rem;
          background: #fff;
          box-shadow: 0 1px 2px 0 rgba(192, 192, 192, 0.3);
          .icon {
            position: absolute;
            top: 1.8rem;
            left: 1.8rem;
            z-index: 2;
            width: 1.6rem;
            height: 1.6rem;
            line-height: 1.6rem;
            text-align: center;
            background: #c0c0c0;
            border-radius: 0.2rem;
            .iconfont {
              font-size: 1rem;
              color: #444444;
            }
            &.active {
              background: #c8a165;
              .iconfont {
                color: #ffffff;
              }
            }
          }
          .img {
            position: relative;
            padding-top: 100%;
            width: 100%;
            background: url("./../assets/images/goods_default.png") no-repeat;
            background-size: 100% 100%;
            & > img {
              position: absolute;
              top: 0;
              left: 0;
              width: 100%;
              height: 100%;
            }
          }
          .desc {
            padding: 1rem 0;
            .name {
              padding-bottom: 1rem;
              white-space: nowrap;
              text-overflow: ellipsis;
              overflow: hidden;
              font-size: 1.3rem;
              color: #666666;
            }
            .price {
              white-space: nowrap;
              text-overflow: ellipsis;
              overflow: hidden;
              text-align: center;
              font-weight: 600;
              font-size: 1.6rem;
              color: #444444;
            }
          }
        }
      }
    }
  }
  .load {
    width: 100%;
    padding-bottom: 1rem;
    height: 4.6rem;
    p {
      line-height: 4.6rem;
      text-align: center;
      font-size: 1.4rem;
      color: #666666;
      background: #ffffff;
    }
  }
  .m-search-tag {
    padding: 1rem 1rem 0;
    height: 100%;
  }
  .nomal-data {
    height: 40.1rem;
    padding: 1.5rem;
    display: flex;
    align-items: center;
    justify-content: center;
    .desc {
      & > img {
        display: inline-block;
        width: 12rem;
        height: 12rem;
        margin-bottom: 1.6rem;
      }
      & > p {
        font-size: 1.4rem;
        color: #666;
      }
    }
  }
}
</style>
