<template>
  <div id="deviceEdit" v-loading="isLoging">
    <el-dialog
      title="设备编辑"
      width="40%"
      top="2rem"
      :close-on-click-modal="false"
      :visible.sync="showDialog"
      :destroy-on-close="true"
      @close="close()"
    >
      <div id="shared" style="margin-top: 1rem;margin-right: 100px;">
        <el-form ref="form" :rules="rules" :model="form" label-width="200px" >
          <el-form-item label="设备编号" prop="deviceId">
            <el-input v-if="isEdit" v-model="form.deviceId" disabled></el-input>
            <el-input v-if="!isEdit" v-model="form.deviceId" clearable></el-input>
          </el-form-item>

          <el-form-item label="设备名称" prop="name">
            <el-input v-model="form.name" clearable></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="form.password" clearable></el-input>
          </el-form-item>
          <el-form-item label="收流IP" prop="sdpIp">
            <el-input type="sdpIp" v-model="form.sdpIp" clearable></el-input>
          </el-form-item>
          <el-form-item label="流媒体ID" prop="mediaServerId">
            <el-select v-model="form.mediaServerId" style="float: left; width: 100%" >
              <el-option key="auto" label="自动负载最小" value="auto"></el-option>
              <el-option
                v-for="item in mediaServerList"
                :key="item.id"
                :label="item.id"
                :value="item.id">
              </el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="字符集" prop="charset" >
            <el-select v-model="form.charset" style="float: left; width: 100%" >
                <el-option key="GB2312" label="GB2312" value="gb2312"></el-option>
              <el-option key="UTF-8" label="UTF-8" value="utf-8"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="其他选项">
            <el-checkbox label="SSRC校验" v-model="form.ssrcCheck" style="float: left"></el-checkbox>
            <el-checkbox label="作为消息通道" v-model="form.asMessageChannel" style="float: left"></el-checkbox>
            <el-checkbox label="收到ACK后发流" v-model="form.broadcastPushAfterAck" style="float: left"></el-checkbox>
          </el-form-item>
          <el-form-item>
            <div style="float: right;">
              <el-button type="primary" @click="onSubmit" >确认</el-button>
              <el-button @click="close">取消</el-button>
            </div>

          </el-form-item>
        </el-form>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import MediaServer from '../service/MediaServer'
export default {
  name: "deviceEdit",
  props: {},
  computed: {},
  created() {},
  data() {
    return {
      listChangeCallback: null,
      showDialog: false,
      isLoging: false,
      hostNames:[],
      mediaServerList: [], // 滅体节点列表
      mediaServerObj : new MediaServer(),
      form: {},
      isEdit: false,
      rules: {
        deviceId: [{ required: true, message: "请输入设备编号", trigger: "blur" }]
      },
    };
  },
  methods: {
    openDialog: function (row, callback) {
      console.log(row)
      this.showDialog = true;
      this.isEdit = false;
      if (row) {
        this.isEdit = true;
      }
      this.form = {};
      this.listChangeCallback = callback;
      if (row != null) {
        this.form = row;
      }
      this.getMediaServerList();
    },
    getMediaServerList: function (){
      let that = this;
      that.mediaServerObj.getOnlineMediaServerList((data)=>{
        that.mediaServerList = data.data;
      })
    },
    onSubmit: function () {
      console.log("onSubmit");
      console.log(this.form);
      this.$axios({
        method: 'post',
        url:`/api/device/query/device/${this.isEdit?'update':'add'}`,
        data: this.form
      }).then((res) => {
        console.log(res.data)
        if (res.data.code === 0) {
          this.listChangeCallback()
        }else {
          this.$message({
            showClose: true,
            message: res.data.msg,
            type: "error",
          });
        }
      }).catch(function (error) {
        console.log(error);
      });
    },
    close: function () {
      this.showDialog = false;
      this.$refs.form.resetFields();
    },
  },
};
</script>
