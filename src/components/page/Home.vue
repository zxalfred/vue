<template>
  <div class="daily">
    <div class="daily-menu">
      <div class="daily-menu-item" 
        :class="{ 'on': type ==='recommend'}"
        @click="handleToRecommend">每日推荐</div>
      <div class="daily-menu-item" 
        :class="{ 'on': type === 'daily'}"
        @click="showThemes = !showThemes; type = 'daily'">主题日报</div>
      <ul v-show="showThemes">
        <li v-for="item in themes" :key="item.id">
          <a class="daily-theme-link" :class="{ on: item.id === themeId && type === 'daily'}"
            @click="handleToTheme(item.id)">{{ item.name }}</a>
        </li>
      </ul>
    </div>
    <div class="daily-list" ref="list">
      <template v-if="type === 'recommend'">
        <div v-for="list in recommendList" :key="list.date">
          <div class="daily-date">{{ formatDay(list.date) }}</div>
          <Item
            v-for="item in list.stories"
            :data="item"
            :key="item.id"
            @click.native="handleClick(item.id)"
          ></Item>
        </div>
      </template>
      <template v-if="type === 'daily'">
        <Item
          v-for="item in list"
          :data="item"
          :key="item.id"
          @click.native="handleClick(item.id)"
        ></Item>
      </template>
    </div>
    <daily-article :id="articleId"></daily-article>
  </div>
</template>
<script>
import $ from '../../../build/axios.conf';
import Item from './item.vue';
import dailyArticle from './daily-article.vue';

export default {
  components: {
    Item,
    dailyArticle,
  },
  data() {
    return {
      themes: [],
      showThemes: false,
      type: 'recommend',
      list: [],
      themeId: 13,
      recommendList: [],
      dailyTime: $.getTodayTime(),
      isLoading: false,
      articleId: 0,
    };
  },
  methods: {
    getThemes() {
      // axios 发起 get 请求
      $.ajax.get('themes').then((res) => {
        this.themes = res.others;
        // 默认选中第一项
      });
    },
    handleToTheme(id) {
      this.type = 'daily';
      this.themeId = id;
      this.list = [];
      $.ajax.get(`theme/${id}`).then((res) => {
        this.list = res.stories.filter(item => item.type !== 1);
      });
    },
    handleToRecommend() {
      this.type = 'recommend';
      this.recommendList = [];
      this.dailyTime = $.getTodayTime();
      this.getRecommendList();
    },
    getRecommendList() {
      this.isLoading = true;
      this.showThemes = false;
      const prevDay = $.prevDay(this.dailyTime + 86400000);
      $.ajax.get(`news/before/${prevDay}`).then((res) => {
        this.recommendList.push(res);
        this.isLoading = false;
      });
    },
    formatDay(date) {
      let month = date.substr(4, 2);
      let day = date.substr(6, 2);
      if (month.substr(0, 1) === '0') month = month.substr(1, 1);
      if (day.substr(0, 1) === '0') day = day.substr(1, 1);
      return `${month} 月 ${day} 日`;
    },
    handleClick(id) {
      this.articleId = id;
    },
  },
  mounted() {
    this.getThemes();
    this.getRecommendList();
    const $list = this.$refs.list;
    const that = this;
    $list.addEventListener('scroll', () => {
      // 在主题日报或正在加载推荐列表时停止操作
      if (this.type === 'daily' || this.isLoading) return;
      if ($list.scrollTop + document.body.clientHeight >= $list.scrollHeight - 10) {
        // 时间相对减少一天
        that.dailyTime -= 86400000;
        that.getRecommendList();
      } 
    });
  },
};
</script>
<style>
.daily-menu ul{
  list-style: none;
}
.daily-menu ul li a{
  display: block;
  color: inherit;
  text-decoration: none;
  padding: 5px 0 5px 15px;
  margin: 5px 0;
  cursor: pointer;
}
.daily-menu ul li a:hover, .daily-menu ul li a.on{
  color: #3399ff;
}
.daily-date{
  text-align: center;
  margin: 10px 0;
}
</style>
