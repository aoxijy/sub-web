<template>
  <div>
    <el-row style="margin-top: 10px">
      <el-col>
        <el-card>
          <div slot="header">
            Subscription Converter
            <svg-icon icon-class="github" style="margin-left: 20px" @click="goToProject" />

            <div style="display: inline-block; position:absolute; right: 20px">{{ backendVersion }}</div>
          </div>
          <el-container>
            <el-form :model="form" label-width="140px" label-position="left" style="width: 100%">
              <el-form-item label="模式设置:">
                <el-radio v-model="advanced" label="1">基础模式</el-radio>
                <el-radio v-model="advanced" label="2">进阶模式</el-radio>
              </el-form-item>
              <el-form-item label="订阅链接:">
                <el-input v-model="form.sourceSubUrl" type="textarea" rows="3"
                  placeholder="支持订阅或ss/ssr/vmess链接，多个链接每行一个或用 | 分隔" @blur="saveSubUrl" />
              </el-form-item>
              <el-form-item label="客户端:">
                <el-select v-model="form.clientType" style="width: 100%">
                  <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>

              <div v-if="advanced === '2'">
                <!-- 后端地址输入框已删除 -->
                <el-form-item label="远程配置:">
                  <el-select v-model="form.remoteConfig" allow-create filterable placeholder="请选择" style="width: 100%">
                    <el-option-group v-for="group in options.remoteConfig" :key="group.label" :label="group.label">
                      <el-option v-for="item in group.options" :key="item.value" :label="item.label"
                        :value="item.value"></el-option>
                    </el-option-group>
                    <el-button slot="append" @click="gotoRemoteConfig" icon="el-icon-link">配置示例</el-button>
                  </el-select>
                </el-form-item>
                <!-- 其他选项省略也保留 -->

                <!-- 可选：你的其他 el-form-item 都保留不变 -->
              </div>

              <el-form-item label="定制订阅:">
                <el-input class="copy-content" disabled v-model="customSubUrl">
                  <el-button slot="append" v-clipboard:copy="customSubUrl" v-clipboard:success="onCopy"
                    icon="el-icon-document-copy">复制</el-button>
                </el-input>
              </el-form-item>
            </el-form>
          </el-container>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  data() {
    return {
      backendVersion: "",
      advanced: "2",

      options: {
        clientTypes: {
          Clash: "clash",
          Surge: "surge&ver=4",
          QuantumultX: "quanx",
          V2Ray: "v2ray",
          ssr: "ssr"
        },
        remoteConfig: [] // 可填入你的 config 列表
      },
      form: {
        sourceSubUrl: "",
        clientType: "clash",
        remoteConfig: "",
        // 删除 customBackend 控制项，直接写死后端地址
      },
      customSubUrl: ""
    };
  },
  mounted() {
    this.getBackendVersion();
  },
  methods: {
    goToProject() {
      window.open("https://github.com/CareyWang/sub-web");
    },
    gotoRemoteConfig() {
      window.open("https://github.com/tindy2013/subconverter/blob/master/README-cn.md");
    },
    getBackendVersion() {
      fetch("https://ns.gqru.com/version")
        .then(res => res.text())
        .then(text => {
          this.backendVersion = text.replace(/backend\n$/gm, "").replace("subconverter", "");
        });
    },
    makeUrl() {
      if (this.form.sourceSubUrl === "" || this.form.clientType === "") {
        this.$message.error("订阅链接与客户端为必填项");
        return;
      }

      const backend = "https://ns.gqru.com/sub?";

      let sourceSub = this.form.sourceSubUrl.replace(/(\n|\r|\n\r)/g, "|");

      this.customSubUrl =
        backend +
        "target=" +
        this.form.clientType +
        "&url=" +
        encodeURIComponent(sourceSub);

      if (this.form.remoteConfig) {
        this.customSubUrl +=
          "&config=" + encodeURIComponent(this.form.remoteConfig);
      }

      this.$copyText(this.customSubUrl);
      this.$message.success("定制订阅已复制到剪贴板");
    },
    saveSubUrl() {
      if (this.form.sourceSubUrl !== '') {
        localStorage.setItem('sourceSubUrl', this.form.sourceSubUrl);
      }
    },
    onCopy() {
      this.$message.success("已复制！");
    }
  }
};
</script>
