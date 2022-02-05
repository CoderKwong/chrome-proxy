<template>
  <div class="main_app">
    <h1>Proxy Setting</h1>
    <el-form ref="form" :model="form" label-width="80px">
      <el-form-item label="scheme">
        <el-input id="scheme" v-model="form.scheme"></el-input>
      </el-form-item>
      <el-form-item label="host">
        <el-input id="host" v-model="form.host"></el-input>
      </el-form-item>
      <el-form-item label="port">
        <el-input-number id="port" v-model="form.port"></el-input-number>
      </el-form-item>
      <el-form-item label="username">
        <el-input id="username" v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="password">
        <el-input id="password" v-model="form.password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button id="save" type="primary" @click="onSubmit">保存</el-button>
        <el-button id="clear" type="primary" @click="clear">清除代理</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      form: {
        scheme: "http",
        host: "",
        port: 80,
        username: "",
        password: "",
      },
    };
  },
  mounted() {
    // 读取数据，第一个参数是指定要读取的key以及设置默认值
    chrome.storage.local.get(
      ["scheme", "host", "port", "username", "password"],
      (items) => {
        console.log(items);
        this.form = items;
      }
    );

    chrome.storage.onChanged.addListener(function (changes, namespace) {
      for (var key in changes) {
        var storageChange = changes[key];
        console.log(
          'Storage key "%s" in namespace "%s" changed. ' +
            'Old value was "%s", new value is "%s".',
          key,
          namespace,
          storageChange.oldValue,
          storageChange.newValue
        );
      }
    });
  },
  methods: {
    clear() {
      chrome.proxy.settings.clear({}, function () {});
      console.log("清除代理成功");
    },
    onSubmit() {
      console.log("submit!");
      this.$refs["form"].validate((valid) => {
        if (valid) {
          console.log(this.form);
          this.chromePrxoy();
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    chromePrxoy() {
      const { scheme, host, port, username, password } = this.form;
      chrome.proxy.onProxyError.addListener((errr) => {
        console.log("🚀 ~ file: App.vue ~ line 82 ~ chromePrxoy ~ errr", errr);
      });
      // 保存数据
      chrome.storage.local.set(
        { scheme, host, port, username, password },
        function () {
          console.log("保存成功！");
        }
      );
      chrome.proxy.settings.set(
        {
          value: {
            mode: "fixed_servers",
            rules: {
              singleProxy: {
                scheme,
                host,
                port,
              },
              // 域名地址
              bypassList: ["*.mimvp.com"],
            },
          },
          scope: "regular",
        },
        function (details) {
          console.log(
            "🚀 ~ file: App.vue ~ line 96 ~ chromePrxoy ~ details",
            details
          );
        }
      );

      chrome.webRequest.onAuthRequired.addListener(
        function (details) {
          console.log(
            "🚀 ~ file: App.vue ~ line 100 ~ chromePrxoy ~ details",
            details
          );
          return {
            authCredentials: {
              username,
              password,
            },
          };
        },
        { urls: ["<all_urls>"] },
        ["blocking"]
      );
    },
  },
};
</script>

<style>
.main_app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
