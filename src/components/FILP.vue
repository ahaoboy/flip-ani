<template>
  <div id="app" v-cloak>
    <div class="action">
      <button @click="add">
        <span v-show="adding">下载中……</span>
        <span v-show="!adding">追加图片</span>
      </button>
      <button @click="shuffle">乱序图片</button>
      <button @click="reset">重置</button>
    </div>
    <div v-if="loading" class="loading">正在加载图片……</div>
    <div v-else class="wrap">
      <div class="img-wrap" v-for="(img,i) in imgs" :key="img.key">
        <img :ref="el => { imgDoms[i] = el }" class="img" :src="img.src"/>
      </div>
    </div>
  </div>
</template>

<script>
  import {ref, nextTick} from 'vue'
  import {getImgList, loadImage, createSrcRectMap} from "../util";
  import {random, shuffle as _shuffle} from 'lodash-es'

  export default {
    name: 'FILP',
    props: {
      msg: String
    },
    async setup() {
      const initSize = 8

      const adding = ref(false)
      const loading = ref(false)

      let initList = await Promise.all(getImgList(initSize).map(loadImage))
      console.log(initList)
      const imgs = ref(initList.slice())
      console.log('imgs.value', imgs.value)
      const shuffle = () => {
        console.log('shuffle')
        scheduleAnimation(() => {
          imgs.value = _shuffle(imgs.value)
        })
      }
      const reset = () => {
        console.log('reset')
        imgs.value = initList
      }
      const imgDoms = ref([])
      const add = async () => {
        console.log('add')
        let list = await Promise.all(getImgList(random(1, initSize)).map(loadImage))
        scheduleAnimation(() => {
          imgs.value.unshift(...list)
        })
      }
      console.log('nextTick', nextTick)
      const scheduleAnimation = (update) => {
        console.log('imgDoms', imgDoms.value)
        const imgDomList = imgDoms.value.slice()
        // 获取旧图片的位置
        const prevImgs = [...imgDomList]
        console.log(prevImgs.length)
        const prevSrcRectMap = createSrcRectMap(prevImgs)
        // 更新数据
        update()
        // console.log('len', [...document.getElementsByClassName('img')].length)
        // DOM更新后
        nextTick(() => {
          // console.log('len', [...document.getElementsByClassName('img')].length)
          // console.log(prevImgs.length)
          const currentSrcRectMap = createSrcRectMap(prevImgs)
          Object.keys(prevSrcRectMap).forEach((src) => {
            const currentRect = currentSrcRectMap[src]
            // if (!currentRect) {
            //   let k1 = Object.keys(currentSrcRectMap)
            //   let k2 = Object.keys(prevSrcRectMap)
            //   // console.log(k1.length, k2.length)
            //   let s1 = new Set(k1)
            //   let s2 = new Set(k2)
            //   // console.log(s1, s2)
            //   // console.log('====', currentRect, src, prevSrcRectMap[src], currentSrcRectMap[src])
            // }
            const prevRect = prevSrcRectMap[src]

            const invert = {
              left: prevRect.left - currentRect.left,
              top: prevRect.top - currentRect.top,
            }

            const keyframes = [
              {
                transform: `translate(${invert.left}px, ${invert.top}px)`,
                opacity: 0.1,
                offset: 0,
                // width: "0",
                // height: "0",
              },
              {
                transform: "translate(0, 0)",
                offset: 1,
                opacity: 1
                // width: `${invert.width}px`,
                // height: `${invert.height}px`,
              },
            ]
            const options = {
              duration: 300,
              easing: "cubic-bezier(0,0,0.32,1)",
            }
            // debugger

            // https://developer.mozilla.org/en-US/docs/Web/API/Element/animate
            const animation = currentRect.img.animate(keyframes, options)


            // const keyframes2 = [
            //   {
            //     // transform: `translate(${invert.left}px, ${invert.top}px)`,
            //     // width: "0",
            //     // height: "0",
            //     left: `${prevRect.left}px`,
            //     top: `${prevRect.top}px`
            //   },
            //   {
            //     // transform: "translate(0, 0)",
            //     // width: `${invert.width}px`,
            //     // height: `${invert.height}px`,
            //     left: `${currentRect.left}px`,
            //     top: `${currentRect.top}px`
            //   },
            // ]
            // const options2 = {
            //   duration: 300,
            //   // easing: "cubic-bezier(0,0,0.32,1)",
            // }
            //
            // const animation2 = currentRect.img.animate(keyframes2, options2)
          })
        })
      }

      return {
        loading,
        adding,
        reset,
        shuffle,
        add,
        imgs,
        imgDoms
      }
    },
    data() {
      return {
        count: 0
      }
    }
  }
</script>

<style>
  * {
    margin: 0;
    box-sizing: border-box;
    padding: 0;
  }

  body {
    max-width: 500px;
    margin: auto;
  }

  .loading {
    margin: 12px;
    text-align: center;
  }

  .action {
    margin: 12px;
    text-align: center;
  }

  .action button {
    margin-right: 8px;
  }

  .link {
    display: block;
    padding: 8px;
    text-align: center;
    color: red;
  }

  .wrap {
    display: flex;
    flex-wrap: wrap;
    column-count: 4;
  }

  .img-wrap {
    width: 25%;
    padding: 8px;
    /*position: relative;*/
  }

  .img {
    /*width: 100%;*/
    height: 100px;
    border-radius: 8px;
    object-fit: cover;
    /*position: absolute;*/
  }

  .move {
    transition: transform 1s;
  }

  [v-cloak] {
    display: none;
  }

</style>
