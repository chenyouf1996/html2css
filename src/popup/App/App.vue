<template>
  <div class="main_app">
    <div class="title-wrap">
      <div class="title">
        <span class="tip">scss类名结构生成工具</span>
        <el-button type="primary" size="medium" @click="generateHandler">生成</el-button>
      </div>
      <el-input v-model="htmlString" placeholder="输入html字符串" type="textarea" :autosize="{minRows: 6, maxRows: 9}" resize="none" />
    </div>
    <div v-if="cssStruct" class="title-wrap">
      <div class="title" style="margin-top: 10px">
        <span class="tip">scss类名结构</span>
        <el-button type="primary" size="medium" @click="copyHandler">复制</el-button>
      </div>
      <el-input id="scss-content" v-model="cssStruct" type="textarea" :autosize="{minRows: 6, maxRows: 9}" resize="none" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      htmlString: '',
      cssStruct: ''
    }
  },
  methods: {
    generateHandler() {
      this.cssStruct = this.html2CssStruct(this.htmlString)
    },
    async copyHandler() {
      try {
        await navigator.clipboard.writeText(this.cssStruct)
        this.$message.success('复制成功')
      } catch (error) {
        console.log(error)
        this.$message.error('复制失败')
      }
    },
    generateScssStruct(dom) {
      const re = {}
      const childNodes = dom.children
      Array.from(childNodes).forEach(childNode => {
        const classNameList = Array.from(childNode.classList).map(item => `.${item}`)
        let className = classNameList.join('&')
        if (className) {
          re[className] = {}
        } else {
          className = childNode.tagName.toLowerCase()
          re[className] = {}
        }
        if (childNode.children.length) {
          const scssObject = this.generateScssStruct(childNode)
          Object.assign(re[className], scssObject)
        }
      })
      return re
    },
    formatScssStruct(scssObject) {
      const spaces = 4
      let scssString = JSON.stringify(scssObject, null, spaces)
      scssString = scssString.substring(1, scssString.length - 1)
      scssString = scssString.replace('\n', '')
      scssString = scssString.replaceAll('"', '')
      scssString = scssString.replaceAll(':', '')
      scssString = scssString.replaceAll(',', '')
      scssString = scssString.replaceAll('&', ',')
      scssString = scssString.split('\n').map(item => item.substring(spaces, item.length)).join('\n')
      return scssString
    },
    html2CssStruct(domString) {
      const dom = document.createElement('div')
      dom.innerHTML = domString
      const scssObject = this.generateScssStruct(dom)
      const scssString = this.formatScssStruct(scssObject)
      return scssString
    }
  }
}
</script>

<style lang="scss">
  html,
  body {
    margin: 0;
    padding: 0;
  }

  .main_app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    width: 500px;
    min-height: 500px;
    padding: 10px;

    .title-wrap {
      margin-bottom: 10px;

      .title {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 10px;

        .tip {
          font-size: 18px;
          font-weight: bold;
          color: #000000;
        }
      }
    }
  }
</style>
