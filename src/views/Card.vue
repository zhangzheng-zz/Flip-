<template>
  <div class="card">
    <ul class="card__list" id="card">
      <li class="card__item" v-for="item of tableData" :key="item.id" name="card" :id="item.id">
          <p class="card__item-title">
          Title - {{ item.id }}
          <span class="card__item-title--del" @click="handleDelItem(item.id)">Delete</span>
          </p>
          <p class="card__item-text">{{ item.text }}</p>
      </li>
    </ul>
    <input type="text" v-model="posIdx" />
    <button @click="handleAddItem">添加</button>
  </div>
</template>

<script>
export default {
  data () {
    return {
      tableData: [{ id: 1, text: 'This is Card! This is FLIP!' }, { id: 0, text: 'This is Card! This is FLIP!' }],
      // 插入位置
      posIdx: 0,
      // 卡片序号
      num: 1
    }
  },
  methods: {
    /**
       *添加卡片
       * */
    handleAddItem () {
      const data = [...this.tableData] || []
      const index = this.posIdx
      this.num++
      // 添加图片的回调函数
      this.scheduleAnimation(() => {
        this.tableData = data.slice(0, index).concat(
          [{
            id: this.num,
            text: 'This is Card! This is FLIP!'
          }],
          data.slice(index)
        )
      })
    },
    /**
       *减少卡片
       * */
    handleDelItem (id) {
      console.log(id)
      // 减少图片的回调函数
      this.scheduleAnimation(() => {
        let idx = 0
        this.tableData.forEach((item, index) => {
          if (item.id === id) {
            idx = index
          }
        })
        this.tableData.splice(idx, 1)
      })
    },
    /**
       * 动画方法
       * */
    scheduleAnimation (update) {
      let prevCardList = Array.from(document.getElementsByName('card'))
      prevCardList = prevCardList.map(item => ({
        item,
        id: item.getAttribute('id')
      }))

      // 更新前的位置
      const prevSrcRectMap = this.createSrcRectMap(prevCardList)
      // console.log('prevSrcRectMap', prevSrcRectMap)

      // 更新位置
      update()

      /**
       * 倒置 next执行在DOM变化之后 渲染之前
       * */
      this.$nextTick(() => {
        let curCardList = Array.from(document.getElementsByName('card'))
        curCardList = curCardList.map(item => ({
          item,
          id: item.getAttribute('id')
        }))
        // 更新后的位置
        const curSrcRectMap = this.createSrcRectMap(curCardList)
        // console.log('curSrcRectMap', curSrcRectMap)

        // 旧的图片位置做一次遍历
        Object.keys(prevSrcRectMap).forEach((src) => {
          // 改变后的位置
          const currentRect = curSrcRectMap[src]
          // 改变前的位置
          const prevRect = prevSrcRectMap[src]

          const invert = {
            left: prevRect.left - currentRect.left,
            top: prevRect.top - currentRect.top
          }
          const keyframes = [
            // 初始位置是倒置后的位置
            {
              transform: `translate(${invert.left}px, ${invert.top}px)`
            },
            // 图片更新后本来应该在的位置
            { transform: 'translate(0, 0)' }
          ]

          const options = {
            duration: 300,
            easing: 'cubic-bezier(0,0,0.32,1)'
          }
          // 添加动画
          currentRect.item.item.animate(keyframes, options)
        })
      })
    },
    // 获取位置方法
    createSrcRectMap (list) {
      return list.reduce((prev, item) => {
        const rect = item.item.getBoundingClientRect()
        const { left, top } = rect
        prev[item.id] = { left, top, item }
        return prev
      }, {})
    }
  }
}
</script>

<style lang='less' scoped>
.card {
  width: 100%;
  height: 90vh;
  background: rgb(236, 236, 236);

  &__list {
    display: flex;
    justify-content: flex-start;
    flex-wrap: wrap;
    padding: 70px 200px;
  }

  &__item {
    position: relative;
    display: inline-block;
    margin-left: 30px;
    margin-top: 30px;
    width: 260px;
    height: 200px;
    overflow: hidden;
    border-radius: 10px;
    border: 1px solid #e8e8e8;
    background-color: #fff;

    &-title {
      height: 50px;
      width: 100%;
      line-height: 50px;
      font-size: 18px;
      text-align: left;
      padding: 0 30px;
      border-bottom: 1px solid gray;

      &--del {
        color: blue;
        margin-left: 40px;
      }
    }
  }
}
</style>
