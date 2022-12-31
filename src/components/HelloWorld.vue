<template>
  <div class="container">
    <div class="tip">
      <div class="tip-title">使用步骤</div>
      <p>①.在输入框中输入参数</p>
      <p> ---- 请用英文逗号分隔坐标</p>
      <p>②.点击保存参数</p>
      <p>③.在右侧迷宫图中选择可走的【地图块】</p>
      <p>④.点击【寻找路径】</p>
      <p>⑤.查看结果</p>
      <p>⑥.点击【清除迷宫】清除当前迷宫图</p>
    </div>
    <div class="body">
      <div class="body-left">
        <div class="left-item">
          <div class="item-text">迷宫行数</div>
          <input class="item-input" type="number" min=1 v-model="rowNum">
        </div>
        <div class="left-item">
          <div class="item-text">迷宫列数</div>
          <input class="item-input" type="number" min=1 v-model="colNum">
        </div>
        <div class="left-item">
          <div class="item-text">起 点 [示例：1,1]</div>
          <input class="item-input" type="text" v-model="startInput">
        </div>
        <div class="left-item">
          <div class="item-text">终 点 [示例：1,1]</div>
          <input class="item-input" type="text" v-model="endInput">
        </div>
        <div class="save-button" style="position:absolute;bottom:70px;left:10%" @click="saveParams">保 存 参 数</div>
        <div class="save-button" style="position:absolute;bottom:10px;left:10%" @click="getPath">寻 找 路 径</div>
      </div>
      <div class="body-right">
        <div class="clean-button" @click="saveParams">清 除 迷 宫</div>
        <div class="right-row" v-for="(rowArray, rowindex) in mazeArray" :key="rowindex">
          <div class="right-item" v-for="(item, colindex) in rowArray" :key="colindex"
            @click="setArray(rowindex, colindex)" :class="setColorClass(rowindex, colindex)">{{ item }}</div>
        </div>
      </div>
      <div class="clear"></div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'HelloWorld',
  data: function () {
    return {
      rowNum: 1,
      colNum: 1,
      startInput: "0,0",
      endInput: "4,4",
      startPoint: {
        rowIndex: 0,
        colIndex: 0,
        lastOperation: "",
        deep: 0,
        fn: 0,
      },
      endPoint: {
        rowIndex: 0,
        colIndex: 0
      },
      mazeArray: [
      ]
    }
  },
  methods: {
    /**
     * 设置迷宫块的类名
     * @param {String} rowIndex 行索引
     * @param {String} colIndex 列索引
     */
    setColorClass: function (rowIndex, colIndex) {
      let flag = ""
      switch (this.mazeArray[rowIndex][colIndex]) {
        case 0:
          flag = `selected`
          break;
        case 1:
          break;
        case 2:
          flag = `steped`
          break;
        case 3:
          flag = `start-end`
          break;
      }
      return flag
    },
    /**
     * 迷宫块点击事件
     * @param {String} rowIndex 
     * @param {Srting} colIndex 
     */
    setArray: function (rowIndex, colIndex) {
      // this.mazeArray[rowIndex][colIndex] = 1
      this.$set(this.mazeArray[rowIndex], colIndex, 0)
    },
    /**
     * 设置计算路径前的所有参数 
     */
    saveParams: function () {
      const tempStart = this.startInput.split(",")
      // 先pop列后pop行
      this.startPoint.colIndex = Number(tempStart.pop())
      this.startPoint.rowIndex = Number(tempStart.pop())
      const tempEnd = this.endInput.split(",")
      // 先pop列后pop行
      this.endPoint.colIndex = Number(tempEnd.pop())
      this.endPoint.rowIndex = Number(tempEnd.pop())
      this.initParams()
      this.$set(this.mazeArray[this.startPoint.rowIndex], this.startPoint.colIndex, 0)
      this.$set(this.mazeArray[this.endPoint.rowIndex], this.endPoint.colIndex, 0)
    },
    /**
     * 计算路径并设置迷宫块颜色
     */
    getPath: function () {
      // 先判断起点和终点旁边是否有一步可达的通路，没有则一定不可达
      const isEndConnect = this.selectOperation("", this.endPoint.rowIndex, this.endPoint.colIndex)
      const isStartConnect = this.selectOperation("", this.startPoint.rowIndex, this.startPoint.colIndex)
      if (isEndConnect.length > 0 && isStartConnect.length > 0) {
        // 计算路径
        const closedList = this.computedRoute()
        // 回溯路径
        // 设置起点/终点样式
        this.$set(this.mazeArray[this.startPoint.rowIndex], this.startPoint.colIndex, 3)
        this.$set(this.mazeArray[this.endPoint.rowIndex], this.endPoint.colIndex, 3)
        // TODO：溯源需要修改
        let Father = closedList.pop().father
        while (Father.deep >= 1) {
          this.$set(this.mazeArray[Father.rowIndex], Father.colIndex, 2)
          Father = Father.father
        }
      } else {
        alert("起点-终点之间不可达")
      }

    },
    /**
     * 重写sort排序
     * @param {String} props 排序属性名
     */
    sortBy: function (props) {
      return function (a, b) {
        return a[props] - b[props]
      }
    },
    /**
     * 初始化迷宫二维数组
     */
    initParams: function () {
      //创建二维数组
      const arr = new Array(Number(this.rowNum))
      for (var i = 0; i < arr.length; i++) {
        arr[i] = new Array(Number(this.colNum)).fill(1)
      }
      this.mazeArray = arr
    },
    /**
     * 获取当前迷宫块可选择的操作算子
     * @param {String} lastOperation 上一次的操作算子
     * @param {String} rowIndex 行索引
     * @param {String} colIndex 列索引
     */
    selectOperation: function (lastOperation, rowIndex, colIndex) {
      const selectd = []
      // up操作
      if (lastOperation !== "down" && rowIndex !== 0) {
        if (this.mazeArray[rowIndex - 1][colIndex] === 0) {
          selectd.push("up")
        }
      }
      // down操作
      if (lastOperation !== "up" && rowIndex !== this.rowNum - 1) {
        if (this.mazeArray[rowIndex + 1][colIndex] === 0) {
          selectd.push("down")
        }
      }
      // left操作
      if (lastOperation !== "right" && colIndex !== 0) {
        if (this.mazeArray[rowIndex][colIndex - 1] === 0) {
          selectd.push("left")
        }
      }
      // right操作
      if (lastOperation !== "left" && colIndex !== this.colNum - 1) {
        if (this.mazeArray[rowIndex][colIndex + 1] === 0) {
          selectd.push("right")
        }
      }
      return selectd
    },
    /**
     * 获取执行操作算子后的迷宫块坐标
     * @param {String} operation 操作算子
     * @param {String} currentPoint 执行操作的迷宫块
     */
    getOperatedPoint(operation, currentPoint) {
      switch (operation) {
        case "up":
          currentPoint.rowIndex -= 1
          break;
        case "down":
          currentPoint.rowIndex += 1
          break;
        case "left":
          currentPoint.colIndex -= 1
          break;
        case "right":
          currentPoint.colIndex += 1
          break;
      }
      return currentPoint
    },
    /**
     * 计算曼哈顿距离
     * @param {String} rowIndex 
     * @param {String} colIndex 
     */
    computed_MHT_Distance(rowIndex, colIndex) {
      const distance = Math.abs(rowIndex - this.endPoint.rowIndex) + Math.abs(colIndex - this.endPoint.colIndex)
      return distance
    },
    /**
     * A*思想计算路线函数
     */
    computedRoute: function () {
      const that = this
      let openList = []
      openList.push(that.startPoint)
      const closedList = []
      let count = 1
      while (true) {
        if (openList[0].rowIndex === this.endPoint.rowIndex && openList[0].colIndex === this.endPoint.colIndex) {
          // 更新closed表
          closedList.push(openList[0])
          closedList.shift()
          // 更新open表
          openList.shift()
          // 退出循环
          // console.log("最终closed表", closedList)
          return closedList
        } else {
          // 操作算子集
          const currentOperation = this.selectOperation(openList[0].lastOperation, openList[0].rowIndex, openList[0].colIndex)
          // 遍历操作算子集，构建每一个Point对象
          currentOperation.forEach(operation => {
            const currentPoint = Object.assign({}, openList[0]) // 深拷贝
            const fatherPoint = Object.assign({}, openList[0]) // 深拷贝
            const nextPoint = that.getOperatedPoint(operation, currentPoint)
            const openlistObj = {
              rowIndex: nextPoint.rowIndex,
              colIndex: nextPoint.colIndex,
              lastOperation: operation,
              deep: currentPoint.deep + 1,
              fn: that.computed_MHT_Distance(nextPoint.rowIndex, nextPoint.colIndex),
              father: fatherPoint
            }
            // console.log("open表·元素", openlistObj)
            openList.push(openlistObj)
          })
          // 更新closed表
          closedList.push(openList[0])
          // 更新open表
          openList.shift()
          openList.sort(that.sortBy("fn"))
          count = count + 1
        }
      }
    }
  },
}
</script>

<style scoped lang="scss">
.container {
  position: relative;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;

  .tip {
    position: fixed;
    top: 50px;
    left: 20px;
    width: 300px;
    height: 200px;
    padding: 5px;
    background-color: whitesmoke;
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;

    .tip-title {
      margin-bottom: 5px;
      font-size: 18px;
      font-weight: 550;
      font-family: Arial, Helvetica, sans-serif;
      color: #03A9F4;
    }

    p {
      height: 25px;
      line-height: 25px;
      font-family: Arial, Helvetica, sans-serif;
    }
  }

  .body {
    margin-top: 50px;
    display: flex;
    width: 60%;
    min-height: 500px;
    box-shadow: rgba(0, 0, 0, 0.3) 0px 19px 38px, rgba(0, 0, 0, 0.22) 0px 15px 12px;

    .body-left {
      position: relative;
      display: flex;
      flex-direction: column;
      background-color: #03A9F4;
      width: 300px;
      min-height: 500px;

      .left-item {
        margin-top: 5px;
        margin-left: 30px;
        margin-bottom: 5px;

        .item-text {
          font-family: Arial, Helvetica, sans-serif;
        }

        .item-input {
          margin-top: 2px;
          font-size: 16px;
          height: 25px;
        }
      }

      .save-button {
        width: 80%;
        height: 40px;
        font-size: 16px;
        line-height: 40px;
        font-weight: 550;
        font-family: Arial, Helvetica, sans-serif;
        border-radius: 10px;
        text-align: center;
        background-color: white;
        color: #757575;
        box-shadow: rgba(60, 64, 67, 0.3) 0px 1px 2px 0px, rgba(60, 64, 67, 0.15) 0px 2px 6px 2px;
      }

      .save-button:hover {
        cursor: pointer;

      }
    }

    .body-right {
      flex-grow: 1;
      height: 100%;
      background-color: white;
      display: flex;
      flex-direction: column;
      align-items: center;

      .clean-button {
        margin-top: 5px;
        margin-bottom: 10px;
        width: 200px;
        height: 50px;
        line-height: 50px;
        border-radius: 10px;
        text-align: center;
        font-size: 16px;
        font-weight: 550;
        font-family: Arial, Helvetica, sans-serif;
        background-color: #757575;
        color: white;
      }

      .clean-button:hover {
        cursor: pointer;
      }

      .right-row {
        display: flex;
        margin-bottom: 2px;

        .right-item {
          width: 70px;
          height: 70px;
          background-color: whitesmoke;
          margin-right: 2px;
        }

        .right-item:hover {
          cursor: pointer;
        }

        .selected {
          background-color: lightgray;
        }

        .steped {
          background-color: #B3E5FC;
        }

        .start-end {
          background-color: #03A9F4;
        }
      }

    }

    .clear {
      clear: both;
    }
  }

}
</style>
