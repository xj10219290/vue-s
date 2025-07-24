<template>
  <body>
    <h1>我是你的小助手，有什么问题尽管问我吧！</h1>
    <div>
      <el-input
        type="textarea"
        :rows="3"
        placeholder="请输入内容..."
        v-model="query"
      >
      </el-input>
      <el-button
        style="margin-top: 8px"
        type="primary"
        @click="clickHandler()"
        icon="el-icon-s-promotion"
        >发送</el-button
      >
      <div v-html="compiledMarkdown"></div>
    </div>
    <div v-loading="loading" class="custom-loading-container"></div>

    <div class="container">
      <div>
        <label>文件：</label>
        <input
          class="input"
          type="file"
          accept=".jpg, .jpeg, .png, .gif"
          @change="updateBase64Data"
        />
        <button @click="update" :disabled="computedData">提交</button>
      </div>
      <div class="output">
        <div class="preview">
          <img v-if="imgBase64Data" :src="imgBase64Data" alt="preview" />
        </div>
        <div>{{ content }}</div>
      </div>
    </div>
  </body>
</template>

<script>
import marked from "marked";
import hljs from "highlight.js";

marked.setOptions({
  highlight: (code, lang) => {
    if (lang && hljs.getLanguage(lang)) {
      return hljs.highlight(lang, code).value;
    }
    return hljs.highlightAuto(code).value;
  },
});
export default {
  name: "App",
  data() {
    return {
      markdownText: ``,
      query: "",
      loading: false,
      imgBase64Data: '',
      content: '',
      //isValid: 
    };
  },
  methods: {
    updateBase64Data: async function (e) {
      this.imgBase64Data = "";
      const file = e.target.files?.[0];
      if (!file) {
        return;
      }
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = () => {
        this.imgBase64Data = reader.result;
      };
    },
    update: async function () {
      if (!this.imgBase64Data) {
        return;
      }
      const endpoint = "https://api.moonshot.cn/v1/chat/completions";
      const headers = {
        "Content-Type": "application/json",
        Authorization: `Bearer sk-QqDjkxXPQesTCCEWdyJtTV6spnrHAZt5tZiBbP6J951D6Vuj`,
      };
      this.content = "思考中...";
      const response = await fetch(endpoint, {
        method: "POST",
        headers: headers,
        body: JSON.stringify({
          model: "moonshot-v1-8k-vision-preview",
          messages: [
            {
              role: "user",
              content: [
                {
                  type: "image_url",
                  image_url: { url: this.imgBase64Data },
                },
                { type: "text", text: "请描述图片的内容。" },
              ],
            },
          ],
          stream: false,
        }),
      });
      const data = await response.json();
      this.content = data.choices[0].message.content;
    },
    clickHandler: function () {
      if (this.query) {
        this.loading = true;
        this.markdownText = "";
        this.getResultFromDeepseek();
      }
    },
    getResultFromDeepseek: async function () {
      //async function a() {
      const endpoint = "https://api.deepseek.com/chat/completions";
      const headers = {
        "Content-Type": "application/json",
        Authorization: `Bearer sk-f25e0992c2f14bb2bbe2eeacfcd6bf19`,
      }; //sk-QqDjkxXPQesTCCEWdyJtTV6spnrHAZt5tZiBbP6J951D6Vuj
      const payload = {
        model: "deepseek-chat",
        messages: [
          { role: "system", content: "You are a helpful assistant." },
          { role: "user", content: `${this.query}` },
        ],
        stream: false,
      };
      const response = await fetch(endpoint, {
        method: "POST",
        headers: headers,
        body: JSON.stringify(payload),
      });
      const data = await response.json();
      // document.getElementById("reply").style.display = "none";
      this.loading = false;
      this.markdownText = data.choices[0].message.content;
      // }
    },
  },
  computed: {
    compiledMarkdown() {
      return marked(this.markdownText);
    },
    computedData() { 
      if(this.imgBase64Data !== ''){
        return  false
      } else {
        return  true
      }
    }
  },
  created: function () {},
};
</script>

<style>
@import "~highlight.js/styles/github.css";
textarea {
  display: block;
}

.custom-loading-container {
  height: 200px;
  /* border: 1px solid #eee; */
  position: relative; /* 确保 Loading 动画定位正确 */
}

.container {
  display: flex;
  flex-direction: column;
  align-items: start;
  justify-content: start;
  height: 100vh;
  font-size: 0.85rem;
}
.input {
  width: 200px;
}
.output {
  margin-top: 10px;
  min-height: 300px;
  width: 100%;
  text-align: left;
}
.preview img {
  max-width: 100%;
}
button {
  padding: 0 10px;
  margin-left: 6px;
}
/* #app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
} */
</style>
