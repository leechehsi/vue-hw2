<script setup>
import { ref, onMounted } from 'vue' // 載入 ref 建立響應式變數
import axios from 'axios' // 使用 axios HTTP client 載入 YouBike API 資料

// 定義 data 用來接收 JSON 資料
const data = ref([]) // 響應式變數，是一個「Ref 物件」，data = {value: []  // 真正的資料藏在 value 裡}

// 定義輸入欄位值
const inputData = ref('') // 響應式變數，是一個「Ref 物件」，初始值 ''，綁定搜尋輸入

// 符合搜尋資料
const matchDatas = ref([])

// 我的收藏
const collectItems = ref([])

// onMounted 時讀取 localStorage，與 YouBike API 資料
onMounted(async () => {
  // 載入localStorage中的我的收藏
  const saved = localStorage.getItem('collect-items') // 獲取 localStorage中 collect-item
  if (saved) {
    // 有收藏資料
    // Vue 會自動把 JSON 轉成響應式 的 Proxy
    collectItems.value = JSON.parse(saved) // 解析與更新我的收藏
    //console.log('collectItems:', collectItems.value) // 響應式的 Proxy 陣列
    // [測試]，它的確是array
    //console.log(Array.isArray(collectItems.value)) // true
  }

  // 獲取 YouBike API 資料
  retrieveAPIData()
})

// 搜尋符合API資料
function findData() {
  console.log(`開始搜尋 API 資料, 輸入資料: ${inputData.value}`)

  // 取得輸入資料
  const trimInputData = inputData.value.trim() // 輸入資料去除空白

  // 搜尋符合清單
  const tempMatchDatas = []
  data.value.forEach((element) => {
    // 名稱 sna 或 地址 arr 符合
    if (element.sna.includes(trimInputData) || element.ar.includes(trimInputData)) {
      //console.log(element['sna'])
      tempMatchDatas.push(element) // sna(場站中文名稱)
    }
  })
  console.log(`完成搜尋, 共有 ${tempMatchDatas.length} 筆符合`)
  matchDatas.value = tempMatchDatas // 更新符合清單
}

async function retrieveAPIData() {
  console.log('開始獲取 API 資料...')
  // 獲取 YouBike API 資料
  const resp = await axios.get(
    'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json',
  )
  data.value = resp.data // 更新響應式變數 data 的值
  //console.log(resp.data) // array
  console.log(`獲取 API 資料完成，資料數:${data.value.length}`)
}

// 新增收藏
function addItem(element) {
  console.log(`新增收藏, 站名:${element.sna}`)

  // 判斷是否重覆
  const found = collectItems.value.find((item) => item.sno == element.sno) // find, 回應第一個比對成功item，比對失敗回傳undefined
  console.log(found)

  if (found === undefined) {
    // 尚未加入收藏
    // 加入收藏
    collectItems.value.unshift(element)

    // 儲存到localStorage
    localStorage.setItem('collect-items', JSON.stringify(collectItems.value)) // 存入整個清單

    console.log(`「${element.sna}」 新增收藏完成 !`)
  } else {
    console.log(`「${element.sna}」 已在收藏清單內 !`)
    // 已加入收藏
    alert(`「${element.sna}」 已在收藏清單內 !`)
  }
}

// 取消收藏
function clearItem(element) {
  console.log(`取消收藏, 站名:${element.sna}`)

  // 獲取index
  const index = collectItems.value.findIndex((item) => {
    return item.sno == element.sno
  }) // 找詢成功回傳 index，無匹配回傳 -1
  console.log(`index:${index}`)

  if (index >= 0) {
    // 刪除再確認
    const confirmMsg = `確定要刪除「${element.sna}」嗎？`
    if (confirm(confirmMsg)) {
      // 確認刪除

      // 隱藏 收藏 detail，否則重新加入時會直接顯示 detail
      toggleCollectDetail(element.sno)

      // 移除收藏
      collectItems.value.splice(index, 1) // 移除收藏，從位置 index 開始，刪除 1 個元素。
      // 儲存到localStorage
      localStorage.setItem('collect-items', JSON.stringify(collectItems.value)) // 存入更新後清單

      console.log(`取消收藏完成!`)
    }
  }
}

const shownFindDetails = ref(new Set()) // 記錄搜尋展開項目

// 控制搜尋展開/關閉
function toggleFindDetail(sno) {
  console.log('點選顯示搜尋detail')
  if (shownFindDetails.value.has(sno)) {
    // 有sno，表示已顯示 detail
    shownFindDetails.value.delete(sno)
  } else {
    // 沒有sno，需已顯示 detail
    shownFindDetails.value.add(sno)
  }
}

// 是否顯示查詢detail
function isFindDetailShown(sno) {
  // 輸入內容 sno
  return shownFindDetails.value.has(sno) // 回傳true or false控制顯示
}

const shownCollectDetails = ref(new Set()) // 記錄收藏展開項目

// 控制收藏展開/關閉
function toggleCollectDetail(sno) {
  console.log(`點選顯示收藏detail, sno:${sno}`)
  if (shownCollectDetails.value.has(sno)) {
    // 有sno，表示已顯示 detail
    shownCollectDetails.value.delete(sno)
  } else {
    // 沒有sno，需已顯示 detail
    shownCollectDetails.value.add(sno)
  }
}

// 是否顯示收藏detail
function isCollectDetailShown(sno) {
  // 輸入內容 sno
  return shownCollectDetails.value.has(sno) // 回傳true or false控制顯示
}

function isCollected(item) {
  return collectItems.value.some((i) => i.sno === item.sno)
}
</script>

<template>
  <h1>YouBike站點收藏小工具</h1>
  <input
    v-model="inputData"
    @keyup.enter="findData"
    type="text"
    placeholder="輸入站點名稱、或地址"
  />
  <button @click="findData">搜尋</button>
  <button @click="retrieveAPIData">更新即時資料</button>

  <hr />
  <!--符合搜尋-->

  <!--這是vue語法，故直接取length，不用再加value-->
  <p>共有 {{ matchDatas.length }} 筆符合搜尋</p>
  <ul>
    <li v-for="(matchData, index) in matchDatas" :key="index">
      <!--Click點選顯示detail-->
      <a href="#" @click.prevent="toggleFindDetail(matchData.sno)">{{ matchData.sna }}</a>
      <!--搜尋 detail-->
      <div class="findDetail" v-show="isFindDetailShown(matchData.sno)">
        <p>地址：{{ matchData.ar }}</p>
        <p>可租借數量：{{ matchData.available_rent_bikes }}</p>
        <p>可歸還數量：{{ matchData.available_return_bikes }}</p>
        <!--點選加入收藏，用isColletecd控制內容顯示，以及button disabled/enabled-->
        <button @click="addItem(matchData)" :disabled="isCollected(matchData)">
          {{ isCollected(matchData) ? '已收藏' : '加入收藏' }}
        </button>
      </div>
    </li>
  </ul>

  <hr />

  <!--我的收藏-->
  <p>目前有 {{ collectItems.length }} 筆收藏站點</p>
  <ul>
    <li v-for="(collectItem, index) in collectItems" :key="index">
      <a href="#" @click.prevent="toggleCollectDetail(collectItem.sno)">{{ collectItem.sna }}</a>
      <!--收藏 detail-->
      <div class="collectDetail" v-show="isCollectDetailShown(collectItem.sno)">
        <p>地址：{{ collectItem.ar }}</p>
        <p>可租借數量：{{ collectItem.available_rent_bikes }}</p>
        <p>可歸還數量：{{ collectItem.available_return_bikes }}</p>
        <button @click="clearItem(collectItem)">取消收藏</button>
      </div>
    </li>
  </ul>
</template>

<style scoped></style>
