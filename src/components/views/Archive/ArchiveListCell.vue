<template>
  <div class="archive-list-cell">
    <div class="info" @click="gotoPostDetail(post)">
      <i-tag class="time-tag">{{ post.add_time | formatDate}}</i-tag>
      <div class="base-info">
        <a class="title">
          <i-tool-tip placement="right" :content="routerInfos(post).message" v-if="post.browse_password_encrypt">
            <i-icon type="android-lock" color="#FA5555" v-if="post.browse_password_encrypt" style="margin-right:5px;"></i-icon>
          </i-tool-tip>{{ post.title }}
        </a>
        <a class="see-desc" @click.stop="seeDesc" :class="{show: showDesc}">{{ showDesc ? '隐藏简介' : '查看简介' }} <i-icon type="android-arrow-dropup-circle"></i-icon></a>
        <p class="desc" :class="{show: showDesc}">{{ post.desc }}</p>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import {checkPostAuth} from '@/common/js/utils';

  const POST_TYPE_ARTICLE = 'article';
  const POST_TYPE_ALBUM = 'album';
  const POST_TYPE_MOVIE = 'movie';

  export default {
    name: 'archive-list-cell',
    props: {
      post: {
        Type: Object
      }
    },
    data() {
      return {
        showDesc: false
      };
    },
    methods: {
      seeDesc() {
        this.showDesc = !this.showDesc;
      },
      gotoPostDetail(post) {
        let routerInfos = this.routerInfos(post);
        checkPostAuth.call(this, post, '提示', routerInfos.message, () => {
          this.$router.push({name: post.post_type, params: {id: post.id}});
        }, (encryptedBrowseAuth) => {
          this.$router.push({
            name: post.post_type,
            params: {id: post.id},
            query: {browse_auth: encryptedBrowseAuth}
          });
        }, () => {
          this.$Notice.error({
            title: '密码错误'
          });
        });
      },
      routerInfos(post) {
        let router = {};
        switch (post.post_type) {
          case 'article':
            router.message = '该文章已图集，您需要输入阅读密码';
            break;
          case 'album':
            router.message = '该图集已加密，您需要输入阅读密码';
            break;
          case 'movie':
            router.message = '该文章已图集，您需要输入阅读密码';
            break;
          default:
            router.message = '该文章已图集，您需要输入阅读密码';
        }
        return router;
      }
    },
    computed: {
      typeTag() {
        switch (this.post.postType) {
          case POST_TYPE_ARTICLE:
            return '';
          case POST_TYPE_ALBUM:
            return '';
          case POST_TYPE_MOVIE:
            return '';
        }
      }
    },
    filters: {
      formatDate: function (date) {
        let formatedDate = new Date(date);
        return formatedDate.getFullYear() + '-' + (formatedDate.getMonth() + 1) + '-' + formatedDate.getDate();
      }
    }
  };
</script>

<style lang="stylus" type="text/stylus" rel="stylesheet/stylus">
  @import "../../../common/stylus/theme.styl"

  .archive-list-cell
    position relative
    padding 5px 0
    &::after
      content ''
      position absolute
      min-height 25px
      top 0
      bottom 0
      left 17px
      width 4px
      background-color: $color-main-primary
    .info
      display flex
      font-size 17px
      line-height 25px
      margin-left 45px
      cursor pointer
      @media only screen and (max-width: 720px)
        font-size 13px
      .time-tag
        flex-shrink 0
      .base-info
        a.title
          margin-left 5px
          font-weight 300
          color $color-typegraphy-title
          cursor pointer
          &:hover
            color $color-typegraphy-title-hover
        a.see-desc
          color $color-gradually-gray-21
          font-size 13px
          margin-left 5px
          font-weight 100
          cursor pointer
          > i
            transition: All 0.4s ease-in-out
            transform rotateX(180deg)
          &:hover,&.show
            color $color-gradually-gray-11
          &.show
            > i
              transform rotateX(0deg)
        .desc
          font-size 14px
          font-weight 300
          margin-left 5px
          color $color-gradually-gray-51
          overflow hidden
          max-height 0
          transition All 0.4s ease-in-out
          &.show
            max-height 250px
</style>
