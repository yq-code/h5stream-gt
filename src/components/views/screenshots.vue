<template>
  <div>
    <!-- 内容 -->
    <div class="row">
      <div class="col-sm-3">
        <!-- 模糊查询搜查 -->
        <el-input
          placeholder="输入关键字进行过滤"
          v-model="filterText">
        </el-input>
        <!-- 这是原下拉框代码 -->
        <el-tree
          id="el-tree"
          style="height: 700px;overflow: auto;"
          :data="cameraData"
          show-checkbox
          default-expand-all
          node-key="id"
          :filter-node-method="filterNode"
          ref="tree"
          highlight-current
          :props="defaultProps">
                        <span slot-scope="{ node, data }">
                            <i :class="data.iconclass" style="color:rgb(142, 132, 132);"></i>
                            <span style="padding-left: 4px;">{{data.label}}</span>
                        </span>
        </el-tree>
      </div>
      <!-- 表格 -->
      <div class="col-sm-9">
        <!-- 查询按钮 -->
        <div style="margin-bottom: 10px;">
          <!-- 时间表 -->
          <el-date-picker
            v-model="value"
            type="datetimerange"
            :picker-options="pickerOptions"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            :default-time="['0:00:00', '24:00:00']">
          </el-date-picker>
          <el-button @click="getCheckedNodes" icon="el-icon-search">查询</el-button>
          <el-button @click="tableDatak">清空</el-button>
        </div>
        <!-- 有按钮 -->
        <el-table
          :data="tableData1.slice((currentPage-1)*pageSize,currentPage*pageSize)"
          style="width: 100%;">
          <el-table-column
            prop="token"
            label="名称">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.token }}</span>
            </template>
          </el-table-column>
          <el-table-column
            prop="name"
            label="token">
            <template slot-scope="scope">
              <span>{{ scope.row.name }}</span>
            </template>
          </el-table-column>
          <el-table-column
            prop="starf"
            label="截图时间">
            <template slot-scope="scope">
              <i class="el-icon-time"></i>
              <span>{{ scope.row.starf }}</span>
            </template>
          </el-table-column>
          <el-table-column>
            <template slot-scope="scope">
              <el-button
                size="mini"
                type="success"><a :href="scope.row.url" :download="scope.row.urlto">下载</a></el-button>
              <el-button size="mini" @click="Refresh1(scope.$index, scope.row)" data-toggle="modal"
                         data-target="#myModal">预览
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <!-- 分页 -->
        <el-pagination
          style="text-align: center;"
          layout="prev, pager, next"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :total="total">
        </el-pagination>
      </div>
    </div>
    <!-- 预览模态框 -->
    <div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
              &times;
            </button>
            <h4 class="modal-title" id="myModalLabel">
              预览
            </h4>
            <!-- 开始结束时间 -->
            <div class="kai">
              <span>截图时间:{{rowstarf}}</span>
            </div>
          </div>
          <div class="modal-body text-center">
            <img :src="url" class="imgmin"/>
          </div>
        </div><!-- /.modal-content -->
      </div><!-- /.modal -->
    </div>
  </div>
</template>
<script>
  import '../../assets/adapter.js'
  import {H5sPlayerWS, H5sPlayerHls, H5sPlayerRTC} from '../../assets/h5splayer.js'
  import {H5siOS, H5sPlayerCreate} from '../../assets/h5splayerhelper.js'

  export default {

    name: "screenshots",
    data() {
      return {
        timelink: 0,//滑块
        max: 0,//滑块最大值
        value: [new Date(new Date().getTime() - 3600 * 1000 * 1), new Date()],
        //分页
        currentPage: 1, // 当前页码
        total: 0, // 总条数
        pageSize: 10,//一页数量
        search: '',
        filterText: '',
        data: [],
        cameraData: [],
        defaultProps: {
          children: 'children',
          label: 'label',
          token: "token",
          iconclass: "iconclass"
        },
        tableData1: [],
        pickerOptions: {
          shortcuts: [{
            text: '最近一小时',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 1);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一天',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }]
        },
        rowstarf: "",//跟进进度条开始时间
        url: "",//图片地址
      }
    },
    mounted() {
      $(".el-tree").height(window.innerHeight - 110);
      this.loadLocalSrc();
    },
    methods: {
      //播放
      Refresh1(index, row) {
        console.log(index, row);
        this.rowstarf = row.starf;
        this.url = row.url;
      },
      //按钮搜索
      getCheckedNodes() {
        this.tableDatak();
        console.log("node值", this.$refs.tree.getCheckedNodes());
        var nodes = this.$refs.tree.getCheckedNodes();
        if (nodes.length == 0) {
          this.$Notice.error({
            title: "请选择摄像机"
          });
          return false;
        }
        var timevalue = this.value;
        console.log(timevalue, "nodes", nodes);
        var timevalues = timevalue[0].toISOString();
        var timevaluee = timevalue[1].toISOString();
        let _this = this;
        var root = process.env.API_ROOT;
        var wsroot = process.env.WS_HOST_ROOT;
        if (root == undefined) {
          root = window.location.protocol + '//' + window.location.host + window.location.pathname;
        }
        if (wsroot == undefined) {
          wsroot = window.location.host;
        }
        for (let i = 0; i < nodes.length; i++) {
          if (nodes[i].rtspurl) {
            var idname = nodes[i].token;
            var idname1 = nodes[i].label;
            var url = root + "/api/v1/Search?type=snapshot&token=" + idname + "&start=" + timevalues + "&end=" + timevaluee + "&session=" + this.$store.state.token;
            $.ajax({
              type: "GET",
              url: url,
              async: false,
              dataType: "json",
              success: function (data) {
                if (data.record && data.record.length != 0) {
                  for (var i = 0; i < data.record.length; i++) {
                    var item = data.record[i];
                    var urlto = item["strPath"].split("/");
                    var timeitem = {
                      name: idname,
                      token: idname1,
                      starf: new Date(item["strStartTime"]).Format("yyyy-MM-dd hh:mm"),
                      percentage: 0,
                      url: item["strPath"],
                      urlto: urlto[urlto.length - 1],
                      strFileName: "",
                    };
                    _this.tableData1.push(timeitem);
                  }
                  _this.total = _this.tableData1.length;
                  console.log("length", _this.total)
                }
              },
              error: function (error) {
                console.log('Snapshot failed!', error);
                _this.$Notice.info({
                  title: "Query failed !"
                })
              }
            });
          }
        }
      },
      //清空表格
      tableDatak() {
        this.tableData1 = [];
        this.total = 0;
      },
      //分页
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.currentPage = 1;
        this.pageSize = val;
      },
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        this.currentPage = val;
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
      // 机舱
      loadOneDevice(toplevels) {
        let _this = this;
        var root = process.env.API_ROOT;
        var wsroot = process.env.WS_HOST_ROOT;
        if (root == undefined) {
          root = window.location.protocol + '//' + window.location.host + window.location.pathname;
        }
        if (wsroot == undefined) {
          wsroot = window.location.host;
        }
        var url = root + "/api/v1/GetDeviceSrc?token=" + toplevels.strToken + "&session=" + this.$store.state.token;

        this.$http.get(url).then(result => {
          if (result.status == 200) {
            var data = result.data;
            var topGroup = {children: []};
            topGroup.label = toplevels.strName;
            topGroup.iconclass = "mdi mdi-view-sequential fa-fw";
            for (var i = 0; i < data.src.length; i++) {
              var item = data.src[i];
              var topitem = {
                id: i,
                token: item['strToken'],
                label: item['strName'],
                iconclass: "mdi mdi-camcorder fa-fw"
              };
              topGroup.children.push(topitem);
              if (!item['bOnline'])
                topitem['iconclass'] = 'mdi mdi-camcorder-off fa-fw';

              if (item['nType'] == 'H5_CLOUD')
                topitem['iconclass'] = 'mdi mdi-cloud-upload fa-fw';
            }
            this.data.push(topGroup);
          }
        })
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

            var data = result.data;
            for (var i = 0; i < data.dev.length; i++) {
              var item = data.dev[i];
              var toplevel = [];
              toplevel["strToken"] = item.strToken;
              toplevel["strName"] = item.strName;
              this.loadOneDevice(toplevel);
            }

          }
        })
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

              var newItem = {
                token: item['strToken'],
                label: item['strName'],
                iconclass: 'mdi mdi-camcorder fa-fw',
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
      //模糊查询
      filterNode(value, data) {
        if (!value) return true;
        return data.label.indexOf(value) !== -1;
      },
    },
    //模糊查询
    watch: {
      filterText(val) {
        this.$refs.tree.filter(val);
      }
    },

  }
</script>
<style scoped>
  a {
    color: #FFFFFF;
  }

  .imgmin {
    width: 100%;
  }

  .kai {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
  }
</style>
