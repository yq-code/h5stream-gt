<template>
  <div class="row">
    <!-- Device tree -->
    <div class="col-sm-3">
      <!-- 模糊查询搜查 -->
      <el-input
        placeholder="输入关键字进行过滤"
        v-model="filterText">
      </el-input>
      <el-tree
        id="el-tree"
        style="height: 700px;overflow: auto;"
        :data="cameraData"
        accordion
        default-expand-all
        node-key="id"
        :filter-node-method="filterNode"
        ref="tree"
        highlight-current
        @node-click="handleNodeClick"
        :props="defaultProps">
                        <span slot-scope="{ node, data }">
                            <i :class="data.iconclass" style="color:rgb(142, 132, 132);"></i>
                            <span style="padding-left: 4px;">{{data.label}}</span>
                        </span>
      </el-tree>
    </div>
    <!-- Video 1 4 9 16 -->
    <div class="col-sm-9" id="videoPanel">
      <div name='flex' class="videoColor" v-for="r in rows" :key="r">
        <div calss="videoflexitem" style="flex:1; border:1px solid black;" name="flex" v-for="c in cols"
             @contextmenu.prevent="stopVideo($event)" @click="videoClick(r,c,$event)" :key="c">
          <v-liveplayer v-bind:id="'h'+r+c" :h5id="'h'+r+c" :h5videoid="'hvideo'+r+c"></v-liveplayer>
        </div>
      </div>
      <div class="btn-group blocks">
        <button type="button" class="btn btn-default layout1x1 waves-effect" data-row="1|1"
                @click="changePanel($event)">
        </button>
        <button type="button" class="btn btn-default layout2x2 waves-effect" data-row="2|2"
                @click="changePanel($event)">
        </button>
        <button type="button" class="hidden-xs btn btn-default layout3x3 waves-effect" data-row="3|3"
                @click="changePanel($event)">
        </button>
        <button type="button" class="hidden-xs btn btn-default layout4x4 waves-effect" data-row="4|4"
                @click="changePanel($event)">
        </button>
        <button type="button" class="btn btn-default layoutfull waves-effect"
                @click="panelFullScreen($event)"></button>
      </div>
    </div>
  </div>
</template>

<script>
  import * as types from '@/store/types'
  import '../../assets/material/js/custom.min.js'
  import '../../assets/adapter.js'
  import '../../assets/h5splayer.js'

  import {H5siOS, H5sPlayerCreate} from '../../assets/h5splayerhelper.js'
  import qs from 'qs'
  import Vue from 'vue'
  import 'patternfly-bootstrap-treeview/dist/bootstrap-treeview.min.css'
  import 'patternfly-bootstrap-treeview/dist/bootstrap-treeview.min.js'
  import Liveplayer from './liveplayer'

  export default {
    name: "liveview",
    components: {
      'v-liveplayer': Liveplayer
    },
    data() {
      return {
        //过滤文字
        filterText: "",
        rows: 2,
        cols: 2,
        selectCol: 1,
        selectRow: 1,
        proto: this.$store.state.rtc,
        contentHeight: '',
        contentWidth: '',
        data: [],
        cameraData: [],
        defaultProps: {
          children: 'children',
          label: 'label',
          token: "token",
          iconclass: "iconclass"
        },
      };
    },
    computed: {
      count() {
        return this.$store.state.rtc;
      }
    },
    mounted() {
      $(".el-tree").height(window.innerHeight - 110);
      this.updateUI();
      this.loadLocalSrc();
    },
    methods: {
      //树形节点点击
      handleNodeClick(data, checked, indeterminate) {
        let vid = 'h' + this.selectRow + this.selectCol;
        if (data.rtspurl && data.token) {
          this.$root.bus.$emit('liveplay', data.token, data.streamprofile, vid);
        }
      },
      updateUI() {
        $(".content").innerHeight(window.outerHeight - 50);
        //$('div[name="flex"]').height(($(".content").height() - 50) / this.rows);
        if ($(document.body).width() < 768) {
          this.contentHeight = $(document.body).height() * 0.4;
        } else {
          this.contentHeight = $(document.body).height() * 0.8;
        }
        $('div[name="flex"]').height(this.contentHeight / this.rows);
        //this.contentHeight = $(document.body).height()*0.8;
        let _this = this;
        if (H5siOS() === true) {
          $('.h5video').prop("controls", true);
        }
        $(".right-side-toggle").on("click", function () {
          $(".right-sidebar").slideDown(50).toggleClass("shw-rside");
          $(".fxhdr").on("click", function () {
            body.toggleClass("fix-header");
            /* Fix Header JS */
          });
          $(".fxsdr").on("click", function () {
            body.toggleClass("fix-sidebar");
            /* Fix Sidebar JS */
          });

          /* ===== Service Panel JS ===== */
          var fxhdr = $('.fxhdr');
          console.log(fxhdr);
          if (body.hasClass("fix-header")) {
            fxhdr.attr('checked', true);
          } else {
            fxhdr.attr('checked', false);
          }
        });
        $(".content").removeAttr("style");
      },
      updateUIEnterFullScreen() {
        $('div[name="flex"]').height(screen.height / this.rows);
      },
      updateUIExitFullScreen() {
        if (!document.fullscreenElement && !document.webkitIsFullScreen && !document.mozFullScreen && !document.msFullscreenElement) {
          $('div[name="flex"]').height(this.contentHeight / this.rows);
        }
      },
      loadCamera() {
        let _this = this;
        var root = process.env.API_ROOT;
        if (root == undefined) {
          root = window.location.protocol + '//' + window.location.host + window.location.pathname;
        }
        let url = _this.$root.camera_ip;
        this.$http.get(url).then(result => {
          if (result.status == 200 && result.data.code == 0) {
            var data = result.data;
            for (var i = 0; i < data.data.length; i++) {
              var itemJq = data.data[i];
              var newItemJq = {
                token: itemJq['jqdm'],
                label: itemJq['jqhz'],
                children: [],
                iconclass: 'mdi mdi-home fa-fw'
              };
              if (itemJq.jsEntity && itemJq.jsEntity.length != 0) {
                for (var j = 0; j < itemJq.jsEntity.length; j++) {
                  var itemJs = itemJq.jsEntity[j];
                  var newItemJs = {
                    token: itemJs['jsdm'],
                    label: itemJs['jshz'],
                    children: [],
                    iconclass: 'mdi mdi-hotel fa-fw'
                  };
                  if (itemJs.cameraEntity && itemJs.cameraEntity.length != 0) {
                    for (var k = 0; k < itemJs.cameraEntity.length; k++) {
                      var itemCamera = itemJs.cameraEntity[k];
                      var newItemCamera = {
                        token: itemCamera['cameraid'],
                        label: itemCamera['cameraname'],
                        rtspurl: itemCamera['rtspurl'],
                        equipmentip: itemCamera['equipmentip'],
                        equipmentloginname: itemCamera['equipmentloginname'],
                        equipmentloginpwd: itemCamera['equipmentloginpwd'],
                        iconclass: 'mdi mdi-camcorder fa-fw'
                      };
                      let item = _this.ifInData(itemCamera['cameraid']);
                      if (item != null) {
                        newItemCamera['iconclass'] = item['iconclass'];
                        newItemJs.children.push(newItemCamera);
                      }
                    }
                  }
                  if (newItemJs.children.length != 0)
                    newItemJq.children.push(newItemJs);
                }
              }
              if (newItemJq.children.length != 0)
                this.cameraData.push(newItemJq);
            }
          }
          _this.playVideo(_this.data);
        })
      },
      // 判断token是否存在
      ifInData(id) {
        var _data = this.data;
        for (var i = 0; i < _data.length; i++) {
          if (_data[i]['token'] == id) {
            return _data[i];
          }
        }
        return null;
      },
      // 加载本地资源
      loadLocalSrc() {
        let _this = this;
        var root = process.env.API_ROOT;
        if (root == undefined) {
          root = window.location.protocol + '//' + window.location.host + window.location.pathname;
        }
        var url = root + "/api/v1/GetSrcWithoutDevice?session=" + _this.$store.state.token;
        this.$http.get(url).then(result => {
          if (result.status == 200) {
            var data = result.data;
            for (var i = 0; i < data.src.length; i++) {
              var item = data.src[i];
              if (item['nOriginalType'] == 'H5_CH_GB') {
                continue;
              } else {
                var newItem = {
                  token: item['strToken'],
                  label: item['strName'],
                  iconclass: 'mdi mdi-camcorder fa-fw'
                };
                if (!item['bOnline'])
                  newItem['iconclass'] = 'mdi mdi-camcorder-off fa-fw';
                if (item['nType'] == 'H5_CLOUD')
                  newItem['iconclass'] = 'mdi mdi-cloud-upload fa-fw';
                _this.data.push(newItem);
              }
            }
            this.loadCamera();
          }
        });
      },
      // 视频播放
      playVideo(data) {
        let _this = this;
        let rc = this.rows * this.cols;
        for (var i = 0; i < data.length; i++) {
          if (i < rc) {
            let vid = 'h' + this.selectRow + this.selectCol;
            if (this.selectRow == this.rows) {
              this.selectRow = this.rows;
              this.selectCol++;
            } else if (this.selectRow != this.rows) {
              if (this.selectCol == this.cols) {
                this.selectRow++;
                this.selectCol = 1;
              } else if (this.selectCol != this.cols) {
                this.selectCol++;
              }
            }
            _this.$root.bus.$emit('liveplay', data[i].token, 'main', vid);
          }
          if (i == rc - 1) {
            this.selectRow = 1;
            this.selectCol = 1;
          }
        }
      },
      loadDevice() {
        let _this = this;
        var root = process.env.API_ROOT;
        var wsroot = process.env.WS_HOST_ROOT;
        if (root == undefined) {
          root = window.location.protocol + '//' + window.location.host + window.location.pathname;
        }
        if (wsroot == undefined) {
          wsroot = window.location.host;
        }
        //url
        var url = root + "/api/v1/GetDevice?session=" + this.$store.state.token;

        //重组
        this.$http.get(url).then(result => {
          if (result.status == 200) {
            var srcData = [];
            var data = result.data;
            for (var i = 0; i < data.dev.length; i++) {
              var item = data.dev[i];
              var srclevel = [];
              srclevel["strToken"] = item.strToken;
              srclevel["strName"] = item.strName;
              this.loadSrc(srclevel, srcData);
            }
          }
        })
      },
      loadSrc(srclevel, srcData) {
        let _this = this;
        var root = process.env.API_ROOT;
        var wsroot = process.env.WS_HOST_ROOT;
        if (root == undefined) {
          root = window.location.protocol + '//' + window.location.host + window.location.pathname;
        }
        if (wsroot == undefined) {
          wsroot = window.location.host;
        }
        var url = root + "/api/v1/GetDeviceSrc?token=" + srclevel.strToken + "&session=" + this.$store.state.token;
        this.$http.get(url).then(result => {
          if (result.status == 200) {
            var data = result.data;
            var srcGroup = {children: []};
            srcGroup.label = srclevel.strName;
            srcGroup.iconclass = "mdi mdi-view-sequential fa-fw";
            for (var i = 0; i < data.src.length; i++) {
              var item = data.src[i];
              // 主副流
              var node = [{
                token: item['strToken'],
                streamprofile: "main",
                label: this.$t('message.live.mainstream'),
                iconclass: 'mdi mdi-playlist-play fa-fw'
              }, {
                token: item['strToken'],
                streamprofile: "sub",
                label: this.$t('message.live.substream'),
                iconclass: 'mdi mdi-playlist-play fa-fw'
              }]
              var newItem = {
                token: item['strToken'],
                label: item['strName'],
                iconclass: 'mdi mdi-camcorder fa-fw',
                children: node
              };

              if (!item['bOnline'])
                newItem['iconclass'] = 'mdi mdi-camcorder-off fa-fw';

              if (item['nType'] == 'H5_CLOUD')
                newItem['iconclass'] = 'mdi mdi-cloud-upload fa-fw';

              srcGroup.children.push(newItem);
            }
            this.data.push(srcGroup);
          }
        }).catch(error => {
          console.log('GetSrc failed', error);
        });
      },
      //数字仓机
      NumberDevice() {
        let _this = this;
        var root = process.env.API_ROOT;
        var wsroot = process.env.WS_HOST_ROOT;
        if (root == undefined) {
          root = window.location.protocol + '//' + window.location.host + window.location.pathname;
        }
        if (wsroot == undefined) {
          wsroot = window.location.host;
        }
        //url
        var url = root + "/api/v1/GetGbDevice?session=" + this.$store.state.token;

        //重组
        this.$http.get(url).then(result => {
          if (result.status == 200) {
            var srcData = [];
            var data = result.data;
            for (var i = 0; i < data.dev.length; i++) {
              var item = data.dev[i];
              var srclevel = [];
              srclevel["strToken"] = item.strToken;
              srclevel["strName"] = item.strName;
              this.NumberSrc(srclevel, srcData);
            }
          }
        })
      },
      NumberSrc(srclevel, srcData) {

        let _this = this;
        var root = process.env.API_ROOT;
        var wsroot = process.env.WS_HOST_ROOT;
        if (root == undefined) {
          root = window.location.protocol + '//' + window.location.host + window.location.pathname;
        }
        if (wsroot == undefined) {
          wsroot = window.location.host;
        }

        var url = root + "/api/v1/GetGbDeviceSrc?token=" + srclevel.strToken + "&session=" + this.$store.state.token;

        this.$http.get(url).then(result => {
          if (result.status == 200) {
            var data = result.data;
            var srcGroup = {children: []};
            srcGroup.label = srclevel.strName;
            srcGroup.iconclass = "mdi mdi-view-sequential fa-fw";
            for (var i = 0; i < data.src.length; i++) {
              var item = data.src[i];
              // 主副流
              var node = [{
                token: item['strToken'],
                streamprofile: "main",
                label: this.$t('message.live.mainstream'),
                iconclass: 'mdi mdi-playlist-play fa-fw'
              }, {
                token: item['strToken'],
                streamprofile: "sub",
                label: this.$t('message.live.substream'),
                iconclass: 'mdi mdi-playlist-play fa-fw'
              }]
              var newItem = {
                token: item['strToken'],
                label: item['strName'],
                iconclass: 'mdi mdi-camcorder fa-fw',
                children: node
              };

              if (!item['bOnline'])
                newItem['iconclass'] = 'mdi mdi-camcorder-off fa-fw';

              if (item['nType'] == 'H5_CLOUD')
                newItem['iconclass'] = 'mdi mdi-cloud-upload fa-fw';

              srcGroup.children.push(newItem);
            }
            this.data.push(srcGroup);
          }
        }).catch(error => {
          console.log('GetSrc failed', error);
        });
      },

      changePanel(event) {
        let data = $(event.target).data('row');
        let cols = data.split('|')[1];
        let rows = data.split('|')[0];
        let _this = this;
        Object.assign(this.$data, {
          rows: parseInt(rows),
          cols: parseInt(cols)
        });
        Vue.nextTick(function () {
          $('div[name="flex"]').height(_this.contentHeight / rows);
        });
      },

      changeWS(event) {
        this.proto = "WS";
        var proto = this.proto;
        this.$store.commit(types.RTCSW, proto);
        this.$root.bus.$emit('liveplayproto', "WS");
      },
      changeRTC(event) {
        this.proto = "RTC";
        var proto = this.proto;
        this.$store.commit(types.RTCSW, proto);
        this.$root.bus.$emit('liveplayproto', "RTC");
      },

      panelFullScreen(event) {
        var elem = document.getElementById('videoPanel');
        //var elem = $("#videoPanel");
        console.log('panelFullScreen', event);
        if (
          document.fullscreenEnabled ||
          document.webkitFullscreenEnabled ||
          document.mozFullScreenEnabled ||
          document.msFullscreenEnabled
        ) {
          if (
            document.fullscreenElement ||
            document.webkitFullscreenElement ||
            document.mozFullScreenElement ||
            document.msFullscreenElement
          ) {
            if (document.exitFullscreen) {
              document.exitFullscreen();
            } else if (document.webkitExitFullscreen) {
              document.webkitExitFullscreen();
            } else if (document.mozCancelFullScreen) {
              document.mozCancelFullScreen();
            } else if (document.msExitFullscreen) {
              document.msExitFullscreen();
            }
            console.log("========  updateUIExitFullScreen");
            this.updateUIExitFullScreen();
          } else {
            console.log('panelFullScreen3');

            if (elem.requestFullscreen) {
              elem.requestFullscreen();
            } else if (elem.webkitRequestFullscreen) {
              elem.webkitRequestFullscreen();
            } else if (elem.mozRequestFullScreen) {
              elem.mozRequestFullScreen();
            } else if (elem.msRequestFullscreen) {
              elem.msRequestFullscreen();
            }
            this.updateUIEnterFullScreen();
            if (document.addEventListener) {
              document.addEventListener('webkitfullscreenchange', this.updateUIExitFullScreen, false);
              document.addEventListener('mozfullscreenchange', this.updateUIExitFullScreen, false);
              document.addEventListener('fullscreenchange', this.updateUIExitFullScreen, false);
              document.addEventListener('MSFullscreenChange', this.updateUIExitFullScreen, false);
            }
          }
        } else {
          console.log('Fullscreen is not supported on your browser.');
        }
      },
      videoClick(r, c, $event) {
        this.selectCol = c;
        this.selectRow = r;
        console.log(r, c);
        if ($($event.target).parent().hasClass('videoClickColor')) {
          $($event.target).parent().removeClass('videoClickColor');
        } else {
          $('#videoPanel div[class*="videoClickColor"]').removeClass('videoClickColor');
          $('#videoPanel>div').eq(r - 1).children('div').eq(c - 1).addClass('videoClickColor');
          //$('#videoPanel>div').eq(r - 1).children('div').eq(c - 1).children(".h5videowrapper").children(".h5video").style.opacity = "0.25";
        }
      },
      stopVideo(event) {
        return;
      },
      //模糊查询
      filterNode(value, data, node) {
        // 如果什么都没填就直接返回
        if (!value) return true;
        // 如果传入的value和data中的label相同说明是匹配到了
        if (data.label.indexOf(value) !== -1) {
          return true;
        }
        // 否则要去判断它是不是选中节点的子节点
        return this.checkBelongToChooseNode(value, data, node);
      },
      // 判断传入的节点是不是选中节点的子节点
      checkBelongToChooseNode(value, data, node) {
        const level = node.level;
        // 如果传入的节点本身就是一级节点就不用校验了
        if (level === 1) {
          return false;
        }
        // 先取当前节点的父节点
        let parentData = node.parent;
        // 遍历当前节点的父节点
        let index = 0;
        while (index < level - 1) {
          // 如果匹配到直接返回
          if (parentData.data.label.indexOf(value) !== -1) {
            return true;
          }
          // 否则的话再往上一层做匹配
          parentData = parentData.parent;
          index++;
        }
        // 没匹配到返回false
        return false;
      }
    },
    //模糊查询
    watch: {
      filterText(val) {
        console.log("filter", val);
        this.$refs.tree.filter(val);
      }
    }
  }

</script>


<style scoped>

  .content-header .breadcrumb {
    font-size: 1.5rem;
    position: static;
    float: left;
  }

  .content {
    min-height: 50px;
  }

  div[name='flex'] {
    display: flex;
    border-bottom: 0px !important;

  }

  div[name='flex'] + [name='flex'] {
    border-left: 0px !important;
  }

  #videoPanel > div:nth-last-child(2) {
    border-bottom: 1px solid rgb(22, 22, 22) !important;
  }

  #videoPanel:-webkit-full-screen {
    background-color: rgb(73, 74, 75) !important;
    display: block;
    width: 100%;
    height: 100%;
    margin-left: auto;
    margin-right: auto;
    margin-top: auto;
    margin-bottom: auto;
    top: 0;
    left: 0;
    padding: 0px;
    box-shadow: 0px 0px 50px #000;
  }

  #videoPanel:-moz-full-screen {
    background-color: rgb(73, 74, 75) !important;
    display: block;
    width: 100%;
    height: 100%;
    margin-left: auto;
    margin-right: auto;
    margin-top: auto;
    margin-bottom: auto;
    top: 0;
    left: 0;
    padding: 0px;
    box-shadow: 0px 0px 50px #000;
  }

  div[name="flex"]:hover {
    /*background-color: #3c8dbc;*/
    cursor: pointer;
  }

  .videoClickColor {
    background-color: #616263 !important;
    opacity: 0.80;
  }

  .videoColor {
    background-color: rgb(73, 74, 75) !important;
  }

  .pre-scrollable {
    max-height: 480px;
    overflow-y: scroll;
  }

  .layout1x1 {
    background: url('../../assets/img/layout/1x1.png') #f2f2f2;
    background-repeat: no-repeat;
    background-size: 32px 32px;
    color: #000;
    height: 32px;
    width: 32px;
  }

  .layout1x1:hover {
    background: url('../../assets/img/layout/1x1.png') #7a7878;
    background-size: 32px 32px;
    color: rgb(187, 184, 184);
    height: 32px;
    width: 32px;
  }

  .layout2x2 {
    background: url('../../assets/img/layout/2x2.png') #f2f2f2;
    background-repeat: no-repeat;
    background-size: 32px 32px;
    color: #000;
    height: 32px;
    width: 32px;
  }

  .layout2x2:hover {
    background: url('../../assets/img/layout/2x2.png') #7a7878;
    background-size: 32px 32px;
    color: rgb(187, 184, 184);
    height: 32px;
    width: 32px;
  }

  .layout3x3 {
    background: url('../../assets/img/layout/3x3.png') #f2f2f2;
    background-repeat: no-repeat;
    background-size: 32px 32px;
    color: #000;
    height: 32px;
    width: 32px;
  }

  .layout3x3:hover {
    background: url('../../assets/img/layout/3x3.png') #7a7878;
    background-size: 32px 32px;
    color: rgb(187, 184, 184);
    height: 32px;
    width: 32px;
  }

  .layout4x4 {
    background: url('../../assets/img/layout/4x4.png') #f2f2f2;
    background-repeat: no-repeat;
    background-size: 32px 32px;
    color: #000;
    height: 32px;
    width: 32px;
  }

  .layout4x4:hover {
    background: url('../../assets/img/layout/4x4.png') #7a7878;
    background-size: 32px 32px;
    color: rgb(187, 184, 184);
    height: 32px;
    width: 32px;
  }

  .layoutfull {
    background: url('../../assets/img/layout/fullscreen.png') #f2f2f2;
    background-repeat: no-repeat;
    background-size: 32px 32px;
    color: #000;
    height: 32px;
    width: 32px;
  }

  .layoutfull:hover {
    background: url('../../assets/img/layout/fullscreen.png') #7a7878;
    background-size: 32px 32px;
    color: rgb(187, 184, 184);
    height: 32px;
    width: 32px;
  }
</style>
