<script setup lang="ts">
import { data } from '~/static/data.js'
import domtoimage from 'dom-to-image'


function parseCSV(csvData) {
  const rows = csvData.split('\n')
  const keys = rows[0].split(',').filter((item: string) => item !== '') // 提取第一行的键
  const jsonData:any[] = []

  for (let i = 1; i < rows.length; i++) {
    const values = rows[i].split(',').filter((item: string) => item !== '')
    const entry = {}

    for (let j = 0; j < keys.length; j++)
      entry[keys[j]] = values[j]

    jsonData.push(entry)
  }

  return jsonData
}

function getWidthAndHeights(data) {
  const widthsAndHeights = new Set()

  for (let i = 0; i < data.length; i++)
    if (data[i]['外框（mm)'] && data[i]['外框（mm)'] !== '外框（mm)')
      widthsAndHeights.add(data[i]['外框（mm)'])

  return  widthsAndHeights
}

function getWAndH(str:string){
  if (!str) return [0, 0]
  return str.split('*')
}

// 调用parseCSV函数来解析CSV数据
const parsedData = parseCSV(data)

const renderData = ref(parsedData)

const widthsAndHeights = getWidthAndHeights(parsedData)

widthsAndHeights.add('--请选择--')

console.log(widthsAndHeights)

const selectedWidthAndHeight = ref('--请选择--')

watch(selectedWidthAndHeight, (val) => {
  console.log(val)

  if (val === '--请选择--') return

  if (!val) return

  renderData.value = parsedData.filter((item) => {
    return item['外框（mm)'] === val
  })
})

function print2Image(){
  var node = document.getElementById('preview');

  domtoimage.toPng(node)
      .then(function (dataUrl) {
          var img = new Image();
          img.src = dataUrl;
          document.body.appendChild(img);

          // 创建一个下载链接
          var downloadLink = document.createElement('a');

          // 设置下载链接的属性
          downloadLink.href = dataUrl;
          downloadLink.download = '我的图像.png'; // 设置下载文件名

          // 设置下载链接的文本
          downloadLink.innerText = '下载图像';

          // 将下载链接添加到文档的 body 元素中
          document.body.appendChild(downloadLink);

          downloadLink.click();

          // 移除下载链接
          document.body.removeChild(downloadLink);
      })
      .catch(function (error) {
          console.error('oops, something went wrong!', error);
      });
}

const canvasWidth = ref(1200)
const canvasHeight = ref(800)
</script>

<template>
  <div>
    <div class="flex gap-4 items-center my-4">
    <!-- 设置画布大小 -->
    <div flex gap-2>
      <div>画布: {{ canvasWidth }} * {{ canvasHeight }} </div>
      <div>
        <label for="canvasWidth">画布宽度: </label>
        <input type="number" id="canvasWidth" v-model="canvasWidth" class="w-16"/>
      </div>
      <div>
        <label for="canvasHeight">画布高度: </label>
        <input type="number" id="canvasHeight" v-model="canvasHeight" class="w-16" />
      </div>
    </div>


    <!-- 下拉筛选 -->
    <div>
      <label for="widthsAndHeights">选择外框尺寸</label>
      <select v-model="selectedWidthAndHeight">
        <option v-for="(item,index) in widthsAndHeights" :key="index" :value="item">{{ item }}</option>
      </select>
    </div>

    <!-- 按钮 筛选 -->
    <button class="btn">筛选</button>

    <button class="btn" @click="print2Image()">导出</button>
  </div>


    <!-- 预览区 START -->
    <!-- <textarea  :value="JSON.stringify(parsedData)" /> -->
    <!-- 预览区 END -->
    <!-- 画布区 -->
    <div :style="{
      width: canvasWidth + 'px',
      height: canvasHeight + 'px'
    }" id="preview" class="p-4">
      <div class="container flex flex-wrap gap-4">
        <div v-for="(item,index) in renderData" :key="index" >
          <div class="border inline-block border-1 border-gray-500 p-4">
            <div class="content inline-flex items-center justify-around gap-1 flex-col border-1 border-red-400" :style="{
              width: getWAndH(item['外框（mm)'])[0] + 'px',
              height: getWAndH(item['外框（mm)'])[1] + 'px'
            }">
              <div>{{ item['KKS码'] }}</div>
              <div>{{ item['设备标识名称'] }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>
