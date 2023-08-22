<template>
  <div class="bpmn-js">
    <div class="bpmn-container">
      <!-- 创建一个canvas画布 bpmn-js是通过canvas实现绘图的，并设置ref让vue获取到element -->
      <div ref="canvas" class="canvas"></div>
      <!-- 工具栏显示的地方 -->
      <div id="js-properties-panel" class="panel"></div>
    </div>
    <!-- 把操作按钮写在这里面 -->
    <div class="action">
      <el-upload action class="upload-demo" :before-upload="openBpmn">
        <el-button icon="el-icon-folder-opened"></el-button>
      </el-upload>
      <el-button class="new" icon="el-icon-circle-plus" @click="newDiagram"></el-button>
      <el-button icon="el-icon-download" @click="downloadBpmn"></el-button>
      <el-button icon="el-icon-picture" @click="downloadSvg"></el-button>
      <a hidden ref="downloadLink"></a>
    </div>
  </div>
</template>

<script>
import BpmnModeler from 'bpmn-js/lib/Modeler'
// 工具栏相关
import propertiesPanelModule from 'bpmn-js-properties-panel'
import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda'
import camundaModdleDescriptor from 'camunda-bpmn-moddle/resources/camunda'

// 汉化
import customTranslate from './locale/customTranslate'

// 模拟数据
import mockTemplate from './mockTemplate'

export default {
  data () {
    return {
      bpmnModeler: null,
      bpmnTemplate: mockTemplate.bpmnTemplate
    }
  },
  mounted () {
    this.init()

    this.createNewDiagram(this.bpmnTemplate)
  },
  methods: {
    // 初始化
    init () {
      // 获取画布element
      const canvas = this.$refs.canvas
      //  将汉化包装成一个模块
      const customTranslateModule = {
        translate: ['value', customTranslate]
      }

      // 创建Bpmn对象
      this.bpmnModeler = new BpmnModeler({
        // 设置bpmn的绘图容器为上门获取的画布 element
        container: canvas,

        // 加入工具栏支持
        propertiesPanel: {
          parent: '#js-properties-panel'
        },
        additionalModules: [
          // 工具栏模块
          propertiesProviderModule,
          propertiesPanelModule,
          // 汉化模块
          customTranslateModule
        ],
        moddleExtensions: {
          camunda: camundaModdleDescriptor
        }
      })
    },
    // 绘制
    createNewDiagram (bpmnXmlStr) {
      // 将字符串转换成图显示出来
      this.bpmnModeler.importXML(bpmnXmlStr, err => {
        if (err) {
          this.$message.error('打开模型出错,请确认该模型符合Bpmn2.0规范')
          return
        }
        this.bpmnModeler.get('canvas').zoom('fit-viewport')
      })
    },
    // 导入一个bpmn文件
    openBpmn (file) {
      const reader = new FileReader()
      // 读取File对象中的文本信息，编码格式为UTF-8
      reader.readAsText(file, 'utf-8')
      reader.onload = () => {
        // 读取完毕后将文本信息导入到Bpmn建模器
        this.createNewDiagram(reader.result)
      }
      return false
    },
    // 绘制bpmn
    newDiagram () {
      this.createNewDiagram(this.bpmnTemplate)
    },
    // 下载bpmn文件
    downloadBpmn () {
      this.bpmnModeler.saveXML({ format: true }, (err, xml) => {
        if (!err) {
          // 获取文件名
          const name = `${this.getFilename(xml)}.bpmn`
          // 将文件名以及数据交给下载方法
          this.download({ name: name, data: xml })
        }
      })
    },
    // 下载svg
    downloadSvg () {
      this.bpmnModeler.saveXML({ format: true }, (err, xml) => {
        if (!err) {
          // 获取文件名
          const name = `${this.getFilename(xml)}.svg`

          // 从建模器画布中提取svg图形标签
          let context = ''
          const djsGroupAll = this.$refs.canvas.querySelectorAll('.djs-group')
          for (const item of djsGroupAll) {
            context += item.innerHTML
          }
          // 获取svg的基本数据，长宽高
          const viewport = this.$refs.canvas
            .querySelector('.viewport')
            .getBBox()

          // 将标签和数据拼接成一个完整正常的svg图形
          const svg = `
            <svg
              xmlns="http://www.w3.org/2000/svg"
              xmlns:xlink="http://www.w3.org/1999/xlink"
              width="${viewport.width}"
              height="${viewport.height}"
              viewBox="${viewport.x} ${viewport.y} ${viewport.width} ${viewport.height}"
              version="1.1"
              >
              ${context}
            </svg>
          `
          // 将文件名以及数据交给下载方法
          this.download({ name: name, data: svg })
        }
      })
    },
    // 获取文件名称
    getFilename (xml) {
      const start = xml.indexOf('process')
      let filename = xml.substr(start, xml.indexOf('>'))
      filename = filename.substr(filename.indexOf('id') + 4)
      filename = filename.substr(0, filename.indexOf('"'))
      return filename
    },
    // 执行下载
    download ({ name = 'diagram.bpmn', data }) {
      // 这里就获取到了之前设置的隐藏链接
      const downloadLink = this.$refs.downloadLink
      // 把数据转换为URI，下载要用到的
      const encodedData = encodeURIComponent(data)

      if (data) {
        // 将数据给到链接
        downloadLink.href =
          'data:application/bpmn20-xml;charset=UTF-8,' + encodedData
        // 设置文件名
        downloadLink.download = name
        // 触发点击事件开始下载
        downloadLink.click()
      }
    }
  }
}
</script>

<style lang="scss">
  /*左边工具栏以及编辑节点的样式*/
  @import 'bpmn-js/dist/assets/diagram-js.css';
  @import 'bpmn-js/dist/assets/bpmn-font/css/bpmn.css';
  @import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-codes.css';
  @import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-embedded.css';
  /*右边工具栏样式*/
  @import 'bpmn-js-properties-panel/dist/assets/bpmn-js-properties-panel.css';

  .bpmn-js{
    position: relative;
    background-color: #ffffff;
    width: 100%;
    height: 100%;
    .bpmn-container {
      width: 100%;
      height: 100vh;
      display: flex;
    }
    .canvas{
      width: calc(100% - 300px);
      height: 100vh;
    }
    .panel{
      width: 300px;
      height: inherit;
      overflow-y: auto;
    }
    .bjs-powered-by{
      display: none;
    }
    .action {
      position: fixed;
      bottom: 10px;
      left: 10px;
      display: flex;
    }
    .upload-demo {
      margin-right: 10px;
    }
  }
</style>
