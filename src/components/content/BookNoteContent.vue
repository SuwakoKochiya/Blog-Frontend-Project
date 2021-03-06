<template>
  <div class="book-note-content layout-content" v-if="Object.keys(bookNote).length > 0">
    <i-row v-if="!needAuth">
      <i-col :xs="24" :sm="24" :md="24" :lg="17">
        <div class="layout-left">
          <article-page-header :article="bookNote"></article-page-header>
          <article-page-content>
            <div class="article-details" id="article-main-page" slot="content" ref="article">
              <div class="detail" v-if="bookNote !== undefined" v-for="detail in bookNote.details">
                <article class="typo container article-main-content" v-html="detail.formatted_content">
                </article>
                <div class="detail-footer">Append At / {{ detail.add_time | socialDate
                  }} &nbsp;&nbsp;&nbsp; Update At / {{ detail.update_time | socialDate }}
                </div>
              </div>
            </div>
          </article-page-content>
          <article-page-footer :article="bookNote"></article-page-footer>
        </div>
      </i-col>
      <i-col :xs="0" :sm="0" :md="0" :lg="7">
        <div class="layout-right">
          <book-info :book="bookNote.book"></book-info>
          <recommend style="margin-top: 15px;"></recommend>
          <i-affix :offset-top="60">
            <side-toc style="margin-top: 15px;" ref="sideToc"></side-toc>
          </i-affix>
        </div>
      </i-col>
    </i-row>
  </div>
</template>

<script type="text/ecmascript-6">
  import {
    mapState,
    mapGetters,
    mapMutations,
    mapActions
  } from 'vuex';
  import ArticlePageHeader from '@/components/views/Article/ArticlePageHeader';
  import ArticlePageContent from '@/components/views/Article/ArticlePageContent';
  import ArticlePageFooter from '@/components/views/Article/ArticlePageFooter';
  import About from '@/components/views/About';
  import SideToc from '@/components/views/SideToc';
  import Recommend from '@/components/views/Recommend';
  import BookInfo from '@/components/views/Book/BookInfo';
  // highlight.js引入
  import hljs from '@/common/js/highlight.pack';
  // 样式文件
  import 'highlight.js/styles/atom-one-light.css';
  // TOC
  import tocbot from 'tocbot';
  // 加密
  import {hexMd5} from '@/common/js/md5';

  let HLJS = hljs;

  export default {
    name: 'book-note-content',
    data() {
      return {
        id: undefined,
        browse_auth: undefined
      };
    },
    metaInfo() {
      return {
        title: this.documentMeta.title,
        meta: [
          {name: 'description', content: this.documentMeta.description},
          {name: 'keywords', content: this.documentMeta.keywords}
        ]
      };
    },
    beforeRouteLeave(to, from, next) {
      // 导航离开时清空vuex中图书笔记数据
      this.clearBookNoteInfo();
      next();
    },
    beforeRouteUpdate(to, from, next) {
      next();
      this.refreshData();
    },
    asyncData({store, route}) {
      this.id = route.params.id;
      this.browse_auth = route.query.browse_auth;
      return Promise.all([
        store.dispatch('bookNote/GET_BOOKNOTE_DETAIL_INFO', {
          params: {
            browse_auth: this.browse_auth
          },
          id: this.id
        })
      ]);
    },
    mounted() {
      console.log('mounted');
      this.id = this.$route.params.id;
      this.browse_auth = this.$route.query.browse_auth;
      let that = this;
      if (this.needAuth) {
        this.$Notice.error({
          title: '您输入的阅读密码错误',
          duration: 3,
          closable: true,
          onClose: () => {
            that.checkPassword('该文章为加密文章，<br />您输入的阅读密码错误，请重新验证');
          }
        });
      } else {
        if (Object.keys(this.$store.state.bookNote.bookNote).length === 0) {
          console.log('non ssr');
          // 未SSR的情况
          this.refreshData();
        } else {
          // SSR的情况
          this.refreshContent();
        }
      }
    },
    computed: {
      ...mapState({
        bookNote: state => state.bookNote.bookNote,
        needAuth: state => state.bookNote.needAuth
      }),
      ...mapGetters({
        documentMeta: 'DOCUMENT_META'
      })
    },
    methods: {
      ...mapMutations({
        updateBookNoteAuth: 'bookNote/UPDATE_BOOKNOTE_AUTH',
        clearBookNoteInfo: 'bookNote/CLAER_BOOKNOTE_DETAIL_INFO'
      }),
      ...mapActions({
        getBookNoteDetailInfo: 'bookNote/GET_BOOKNOTE_DETAIL_INFO'
      }),
      refreshData() {
        let that = this;
        this.getBookNoteDetailInfo({
          params: {
            browse_auth: this.browse_auth
          },
          id: this.id
        }).then(response => {
          this.refreshContent();
        }).catch((error) => {
          console.log('book note detail need auth');
          if (error.code === 401) {
            if (that.browse_auth) {
              that.$Notice.error({
                title: '您输入的阅读密码错误',
                duration: 3,
                closable: true,
                onClose: () => {
                  that.checkPassword('该文章为加密文章，<br />您输入的阅读密码错误，请重新验证');
                }
              });
            } else {
              that.checkPassword('该文章为加密文章，请输入阅读密码');
            }
          }
        });
      },
      checkPassword(message) {
        let checkAuth = (browseAuthInput, isAutoRemove) => {
          this.browse_auth = hexMd5(browseAuthInput);
          this.$router.push({
            name: this.$router.name,
            params: {id: this.id},
            query: {browse_auth: this.browse_auth}
          });
          if (isAutoRemove) {
            this.$Modal.remove();
          }
        };
        this.$Modal.confirm({
          autoClosable: true,
          render: (h) => {
            let children = [];
            children.push(h('h2', {
              domProps: {
                innerHTML: '提示'
              },
              'class': {
                'modal-title': true
              }
            }));
            children.push(h('p', {
              domProps: {
                innerHTML: message
              },
              'class': {
                'modal-message': true
              }
            }));
            children.push(h('i-input', {
              props: {
                type: 'password',
                autofocus: true,
                placeholder: '请输入阅读密码'
              },
              'class': {
                'modal-input': true
              },
              on: {
                input: (value) => {
                  this.browse_auth_input = value;
                }
              },
              nativeOn: {
                keyup: (event) => {
                  if (event.keyCode === 13) {
                    checkAuth(this.browse_auth_input, true);
                  }
                }
              }
            }));
            return h('div', {}, children);
          },
          onOk: () => {
            checkAuth(this.browse_auth_input, false);
          }
        });
      },
      // 更新文章结构，高亮、图片、代码行号
      refreshContent() {
        this.$nextTick(() => {
          // 添加图片前缀
          this.resolveImageTagsUrl(this.$refs.article.querySelectorAll('img'));
          this.addCodeLineNumber();
          this.addTocScrollSpy();
        });
      },
      addTocScrollSpy() {
        /* eslint-disable */
        if (!this.$refs.article) return;
        tocbot.init({
          tocSelector: '#side-toc',
          contentSelector: '#article-main-page',
          headingSelector: 'h1, h2, h3, h4, h5',
          linkClass: 'toc-link',
          activeLinkClass: 'is-active-link',
          listClass: 'toc-list',
          isCollapsedClass: 'is-collapsed',
          collapsibleClass: 'is-collapsible',
          listItemClass: 'toc-list-item',
          collapseDepth: 0,
          scrollSmooth: true,
          scrollSmoothDuration: 420,
          headingsOffset: 1,
          throttleTimeout: 50,
          positionFixedClass: 'is-position-fixed',
          fixedSidebarOffset: 'auto',
          includeHtml: false,
          onClick: false
        });
      },
      addCodeLineNumber() {
        // 添加行号
        let blocks = this.$refs.article.querySelectorAll('pre code');
        blocks.forEach((block) => {
          HLJS.highlightBlock(block);
          // 去前后空格并添加行号
          let reg = /<ul(.*?)><li(.*?)>[\s\S]*?<\/li><\/ul>/gm;
          if (reg.test(block.innerHTML)) return;
          block.innerHTML = '<ul><li>' + block.innerHTML.replace(/(^\s*)|(\s*$)/g, '').replace(/\n/g, '\n</li><li>') + '\n</li></ul>';
        });
      }
    },
    components: {
      'article-page-header': ArticlePageHeader,
      'article-page-content': ArticlePageContent,
      'article-page-footer': ArticlePageFooter,
      'about': About,
      'book-info': BookInfo,
      'side-toc': SideToc,
      'recommend': Recommend
    }
  };
</script>

<style lang="stylus" type="text/stylus" rel="stylesheet/stylus">
</style>
