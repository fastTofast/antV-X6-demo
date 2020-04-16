<template>
  <div class="editor-com">
    <div class="editor-left">
      <li class="list" v-for="(item, index) in list" :key="index">
        <div
          class="list-item"
          draggable
          @dragstart="dragstart"
          :nodeType="item.nodeType"
          :class="item.classStr"
        >
          {{ item.name }}
        </div>
      </li>
    </div>
    <div class="editor-section" @drop="drop">
      <div ref="x6Editor" class="x6-editor-container"></div>
    </div>
    <slot name="rightPane">
      <modal :show.sync="showAttrConfig" title="属性配置" @keyup.stop.native="">
        <div slot="content">
          <el-form
            :model="nodeFrmData"
            label-width="80px"
            label-position="left"
          >
            <el-form-item label="名称" prop="name">
              <el-input v-model="nodeFrmData.name"></el-input>
            </el-form-item>
            <el-form-item label="属性1" prop="属性1">
              <el-input v-model="nodeFrmData.属性1"></el-input>
            </el-form-item>
            <el-form-item label="属性2" prop="属性2">
              <el-input v-model="nodeFrmData.属性2"></el-input>
            </el-form-item>
          </el-form>
        </div>
        <span slot="footer">
          <el-button @click="close">取消</el-button>
          <el-button type="primary" @click="confirm">确定</el-button>
        </span>
      </modal>
    </slot>
    <!-- <div ref="minimap" class="mini-map"></div> -->
  </div>
</template>

<script>
import Modal from './Modal'
import { Graph, UndoManager } from '@antv/x6'
export default {
  data () {
    return {
      showAttrConfig: false,
      nodeFrmData: {},
      initScale: 1,
      viewTranslate: { graphX: 0, graphY: 0 },
      list: [
        {
          name: '节点2',
          icon: '节点2',
          classStr: 'outer',
          nodeType: '2'
        },
        {
          name: '节点1',
          icon: '节点1',
          nodeType: '1'
        },
        {
          name: '节点3',
          icon: '节点3',
          classStr: 'circular',
          nodeType: '3'
        }
      ]
    }
  },
  components: {
    Modal
  },
  beforeDestroy () {},
  mounted () {
    const container = this.$refs.x6Editor
    container.addEventListener('dragover', function (event) {
      event.preventDefault()
    })
    const graph = new Graph(container, {
      keyboard: {
        enabled: true, // 是否开启快捷键系统
        /*
         * 为 true 时绑定键盘事件到 document 上，否则绑定在 container 上。
         * 当绑定键盘事件在 container 上时，需要设置 container 的 tabIndex = -1 才能捕获键盘事件
         */
        global: true,
        escape: true // 点击 esc 时是否终止连线、移动等交互
      },
      nodeStyle: {
        resizable: false,
        strokeWidth: 1
      },
      folding: {
        enabled: false // 禁止折叠
      },
      connection: {
        enabled: true // 开启连线交互
      },
      edgeStyle: {
        edge: 'orth',
        endArrow: 'classic',
        curved: true,
        fontSize: 16,
        movable: false
      },
      guide: {
        enabled: true,
        dashed: true,
        stroke: '#555555'
      },
      getAnchors (cell) {
        if (cell != null && cell.isNode()) {
          // 返回 6 个相对定位的锚点
          return [
            [0.25, 0],
            [0.5, 0],
            [0.75, 0],

            // [0, 0.5],
            // [1, 0.5],

            [0.25, 1],
            [0.5, 1],
            [0.75, 1]
          ]
        }
        return null
      }
      // getHtml: cell => {
      //   const wrap = document.createElement('div')
      //   wrap.style.width = '100%'
      //   wrap.style.height = '100%'
      //   wrap.innerHTML = cell.style.html
      //   wrap.addEventListener('click', () => {
      //     console.log('getHtml', cell)
      //     window.graph = graph
      //     this.nodeFrmData = Object.assign(cell.data, {
      //       isNode: cell._isNode,
      //       isEdge: cell._isEdge
      //     })
      //     this.showAttrConfig = true
      //   })
      //   wrap.innerHTML = cell.style.html
      //   return wrap
      // }
    })
    const undoManager = UndoManager.create(graph)
    this.undoManager = undoManager
    window.undoManager = undoManager
    graph.bindKey(
      ['del', 'backspace'],
      e => {
        graph.deleteCells()
        this.showAttrConfig = false
      },
      'keyup'
    )
    graph.bindKey(
      ['ctrl+z'],
      e => {
        undoManager.undo()
      },
      'keyup'
    )
    graph.bindKey(['ctrl+plus'], e => this.scaleView(e, 'up'))
    graph.bindKey(['ctrl+-'], e => this.scaleView(e, 'down'))
    graph.on('selection:changed', data => {
      console.log('graph up', data)
      // if (data.selected.length === 0) {
      //   this.showAttrConfig = false
      // } else {
      //   this.showAttrConfig = true
      //   let cell = data.selected[0]
      //   this.nodeFrmData = Object.assign(cell.data || {}, {
      //     isNode: cell._isNode,
      //     isEdge: cell._isEdge
      //   })
      // }
    })
    graph.on('mouseEvent', data => {
      let e = data.e
      switch (data.eventName) {
        case 'mouseDown':
          console.log('e.state', e.state)
          this.mouseDownMoveFlag = !e.state
          // 记录点击位置
          this.mouseDownPos = e
          break
        case 'mouseUp':
          this.mouseDownMoveFlag = false
          // 一次移动结束，记录最终view偏移量
          this.viewTranslate = Object.assign({}, this.viewOffset)
          break
        case 'mouseMove':
          if (this.mouseDownMoveFlag) {
            let vt = this.viewTranslate
            // 设置偏移位置
            let tx = e.graphX - this.mouseDownPos.graphX + vt.graphX
            let ty = e.graphY - this.mouseDownPos.graphY + vt.graphY
            // 保存view偏移量
            this.viewOffset = { graphX: tx, graphY: ty }
            graph.getView().setTranslate(tx, ty)
          }
          break

        default:
          break
      }
    })
    graph.on('click', ev => {
      let cell = ev.cell
      if (cell) {
        this.showAttrConfig = true
        this.nodeFrmData = Object.assign(cell.data || {}, {
          isNode: cell._isNode,
          isEdge: cell._isEdge
        })
      } else {
        this.showAttrConfig = false
      }
    })
    // 创建边
    // const edge = graph.addEdge({
    //   data: { a: 123 },
    //   id: 'edge1',
    //   source: hello, // 源节点
    //   target: world // 目标节点
    // })
    this.graph = graph
    window.graph = graph
    window.UndoManager = UndoManager
  },
  methods: {
    dragstart (e) {
      let target = e.target
      let data = {
        nodeType: target.getAttribute('nodeType'),
        name: target.innerText
      }
      e.dataTransfer.setData('data', JSON.stringify(data))
    },
    drop (e) {
      this.createNode(e)
    },
    createNode (e) {
      let data = JSON.parse(e.dataTransfer.getData('data'))
      const container = this.$refs.x6Editor
      const pos = container.getBoundingClientRect()
      let nodeOp = {}
      let defaultOp = {
        width: 120,
        height: 60,
        data
      }
      switch (data.nodeType) {
        case '1':
          nodeOp = Object.assign(defaultOp, {
            strokeOpacity: 1,
            shape: 'html',
            html: `<div class="default">
                      <span attr="name">${data.name}</div>
                  </div>`
          })
          break
        case '2':
          nodeOp = Object.assign(defaultOp, {
            label: false,
            shape: 'html',
            html: `<div class="outer">
                      <span class="inner" attr="name">${data.name}</span>
                  </div>`
          })
          break
        case '3':
          nodeOp = Object.assign(defaultOp, {
            width: 80,
            height: 80,
            label: false, // 不渲染 label
            // shape: 'ellipse',
            perimeter: 'ellipse',
            strokeOpacity: 0.01,
            // 提供 html 字符串或 HTMLElement 实例
            // label: data.name
            shape: 'html',
            html: `<div class="circular">
                     <span attr="name">${data.name}</span>
                  </div>`
          })
          break
        default:
          break
      }
      let x = e.x - pos.left
      let y = e.y - pos.top
      if (x < 0 || y < 0) {
        return
      }
      let vt = graph.getView().translate
      let scale = graph
        .getView()
        .getScale()
        .toFixed(1)
      //scale为视图缩放倍数
      nodeOp.x = x / scale - nodeOp.width / 2 - vt.x
      nodeOp.y = y / scale - nodeOp.height / 2 - vt.y
      console.log(nodeOp)
      const world = this.graph.addNode(nodeOp)
    },
    scaleView (e, flag) {
      e.preventDefault()
      if (flag === 'up') {
        this.initScale += 0.1
      } else {
        this.initScale -= 0.1
      }
      graph.getView().setScale(this.initScale)
    },
    close () {
      this.showAttrConfig = false
    },
    confirm () {
      let graph = this.graph
      let selectedCell = graph.getSelectedCell()
      selectedCell.setData(this.nodeFrmData)
      let style = selectedCell.style
      if (selectedCell.style.shape === 'html') {
        let html = selectedCell.style.html
        let temDom = document.createElement('div')
        temDom.innerHTML = style.html
        temDom.querySelector('[attr="name"]').innerText = this.nodeFrmData.name
        style.html = temDom.innerHTML
        selectedCell.setStyle(style)
      } else {
        selectedCell.style.label = this.nodeFrmData.name
      }
      graph.refresh(selectedCell)
    }
  }
}
</script>

<style lang="scss">
.outer {
  width: 100%;
  height: 100%;
  background-color: #4093ff;
  display: flex;
  justify-content: center;
  align-items: center;
  .inner {
    font-size: 16px;
  }
}
.circular {
  width: 76px;
  height: 76px;
  border-radius: 76px;
  border: 1px solid #555555;
  display: flex;
  justify-content: center;
  align-items: center;
}
.default {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.editor-com {
  display: flex;
  .editor-left {
    flex: 0 0 120px;
    .list {
      list-style: none;
      .list-item {
        width: 100px;
        height: 30px;
        line-height: 30px;
        // border-radius: 8px;
        cursor: pointer;
        background-color: #fcfcfc;
        border: 1px solid #999999;
        margin: 10px auto;
        &.outer {
          @extend .outer;
          width: 100px;
        }
        &.circular {
          @extend .circular;
        }
        &.default {
          @extend .default;
        }
      }
    }
  }
  .editor-section {
    flex: 1;
    height: 400px;
    position: relative;
    .x6-editor-container {
      height: 100%;
      div {
        user-select: none;
      }
    }
  }
  // .mini-map {
  //   position: absolute;
  //   right: 10px;
  //   top: 10px;
  //   padding: 10px;
  //   height: 300px;
  //   width: 400px;
  // }
}
</style>
