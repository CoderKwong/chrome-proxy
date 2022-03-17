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
        <el-input id="port" v-model="form.port"></el-input>
      </el-form-item>
      <el-form-item label="username">
        <el-input id="username" v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="password">
        <el-input id="password" v-model="form.password"></el-input>
      </el-form-item>
      <el-form-item label="sn">
        <el-input id="sn" v-model="form.sn"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button id="save" type="primary" @click="onSubmit">保存</el-button>
        <el-button id="change" type="primary" @click="changePorxy"
          >更换代理ip</el-button
        >
        <el-button id="clear" type="primary" @click="clear">清除代理</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import axios from "axios";
import { Message } from "element-ui";

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
        sn: "",
      },
    };
  },
  mounted() {
    // 读取数据，第一个参数是指定要读取的key以及设置默认值
    chrome.storage.local.get(
      ["scheme", "host", "port", "username", "password"],
      (items) => {
        console.log(items);
        this.form.scheme = items.scheme || "http";
        this.form.host = items.host || "35.171.2.66";
        this.form.port = items.port || "2333";
        this.form.username = items.username || "";
        this.form.password = items.password || "";
        this.form.sn = items.sn || "";
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
      Message.success("清除代理成功");
    },
    onSubmit() {
      console.log("submit!");
      this.$refs["form"].validate(async (valid) => {
        if (valid) {
          console.log(this.form);
          this.chromePrxoy();
        } else {
          Message.error("error submit!!");
          return false;
        }
      });
    },
    async changePorxy() {
      const session = this.form.username.match(/-session-(\w+)/)[1];
      const sn = this.form.sn;
      const account = this.form.username.split("-")[0];
      this.cleanChrome();
      this.$refs["form"].validate(async (valid) => {
        if (valid) {
          console.log(this.form);
          const res = await axios.get(
            `http://tiqu.ipidea.io/changeAccountSession?sn=${sn}&account=${account}&session=${session}`
          );
          if (res.status === 200 && res.data.Code === 0) {
            console.log(res);
            Message.success("更换代理成功!");
            setTimeout(() => {
              Message.closeAll();
            }, 1000);
          } else {
            console.log("error");
          }
        } else {
          Message.error("error submit!!");
          return false;
        }
      });
      this.onSubmit();
    },
    cleanChrome() {
      const data = {
        cookies: true,
      };
      chrome.browsingData.remove(
        {
          originTypes: {
            extension: true,
          },
        },
        data,
        function () {
          //弹出框
          Message.success("清理缓存成功!");
        }
      );
    },
    chromePrxoy() {
      const { scheme, host, port, username, password } = this.form;
      console.log(
        "🚀 ~ file: App.vue ~ line 85 ~ chromePrxoy ~ this.form",
        this.form
      );
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
                port: parseInt(port),
              },
              // 域名地址
              bypassList: ["*.ipidea.io", "2captcha.com", "*.2captcha.com"],
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
        (details, asyncCallback) => {
          console.log(
            "🚀 ~ file: App.vue ~ line 100 ~ chromePrxoy ~ details",
            username,
            password,
            details
          );
          asyncCallback({
            authCredentials: {
              username,
              password,
            },
          });
        },
        { urls: ["<all_urls>"] },
        ["asyncBlocking"]
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
