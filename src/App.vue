<template>
<div class="p-6 max-w-4xl mx-auto bg-gray-50 min-h-screen">
  <!-- 標題 -->
  <h1 class="text-3xl font-bold text-center mb-6 text-blue-600">台北YouBike 即時資訊查詢</h1>

  <!-- 搜尋區域 -->
  <div class="flex gap-2 mb-8 justify-center item-center">
    <input
      v-model="keyword"
      placeholder="輸入站點名稱或地址"
      @keyup.enter="searchStations"
      class="input input-bordered w-64"
    />
    <button @click="searchStations" class="btn btn-success">
      搜尋
    </button>
  </div>

  <!-- 查詢結果列表 -->
  <div class="mb-8">
    <h2 class="text-xl font-semibold mb-4 text-gray-700">查詢結果 ({{ filteredStations.length }})筆資料</h2>
    <ul class="overflow-y-auto max-h-60 divide-y divide-gray-200">
      <li
        v-for="station in filteredStations"
        :key="station.sno"
        @click="selectStation(station)"
        class="cursor-pointer hover:bg-gray-100 px-4 py-2 flex justify-between items-center"
      >
        <div>
          <div class="font-medium text-gray-800">{{ station.sna }}</div>
          <div class="text-sm text-gray-500">{{ station.orgn }}</div>
        </div>
        <div class="text-sm text-gray-400">{{ station.ar }}</div>
      </li>
    </ul>
  </div>

  <!-- 選中的站點詳細資訊 -->
  <dialog id="my_modal_1" class="modal">
  <div v-if="selectedStation" class="modal-box w-11/12 max-w-5xl">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-xl font-semibold text-gray-700">站點詳細資訊</h2>
      <button @click="deselectStation" class="btn btn-error btn-sm">關閉</button>
    </div>
    <div class="space-y-2">
      <p><span class="font-semibold text-gray-600">站名：</span>{{ selectedStation.sna }}</p>
      <p><span class="font-semibold text-gray-600">地址：</span>{{ selectedStation.ar }}</p>
      <p><span class="font-semibold text-gray-600">可租借數：</span>{{ selectedStation.available_rent_bikes }}</p>
      <p><span class="font-semibold text-gray-600">經緯度：</span>{{ selectedStation.latitude }} , {{ selectedStation.longitude }}</p>
    </div>
    <div class="mt-4 flex items-center space-x-4">
      <button
        @click="toggleFavorite(selectedStation)"
        :class="isFavorite(selectedStation) ? 'btn btn-warning' : 'btn btn-primary'"
      >
        {{ isFavorite(selectedStation) ? '取消收藏' : '加入收藏' }}
      </button>
    </div>
  </div>
</dialog>

  <!-- 收藏站點 -->
  <div class="mb-8">
    <h2 class="text-xl font-semibold mb-4 text-gray-700">常用站點</h2>
    <ul class="divide-y divide-gray-200 bg-white border border-gray-300 rounded-lg shadow-inner max-h-60 overflow-y-auto">
      <li
        v-for="fav in favorites"
        :key="fav.sno"
        class="flex justify-between items-center px-4 py-2 hover:bg-gray-100"
      >
        <div class="font-medium text-gray-800">{{ fav.sna }}</div>
        <button
          @click="removeFavorite(fav)"
          class="btn btn-sm btn-error"
          title="刪除"
        >
          刪除
        </button>
      </li>
      <li v-if="favorites.length === 0" class="px-4 py-2 text-gray-500">無收藏的站點</li>
    </ul>
  </div>
</div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue';

    const stations = ref([]);
    const filteredStations = ref([]);
    const keyword = ref('');
    const selectedStation = ref(null);
    const favorites = ref([]);

    // 載入收藏資料
    const loadFavorites = () => {
      const favs = localStorage.getItem('favorites');
      if (favs) {
        try {
          favorites.value = JSON.parse(favs);
        } catch (e) {
          favorites.value = [];
        }
      }
    };

    // 儲存收藏資料
    const saveFavorites = () => {
      localStorage.setItem('favorites', JSON.stringify(favorites.value));
    };

    // 檢查是否為收藏
    const isFavorite = (station) => {
      return favorites.value.some((fav) => fav.sno === station.sno);
    };

    // 加入或取消收藏
    const toggleFavorite = (station) => {
      if (isFavorite(station)) {
        removeFavorite(station);
      } else {
        addFavorite(station);
      }
    };

    const addFavorite = (station) => {
      favorites.value.push(station);
      saveFavorites();
      my_modal_1.close()
    };

    const removeFavorite = (station) => {
      favorites.value = favorites.value.filter(
        (fav) => fav.sno !== station.sno
      );
      saveFavorites();
    };

    // 搜尋站點
    const searchStations = async () => {

      await fetchData();
      const kw = keyword.value.trim().toLowerCase();

      if (!stations.value.length) return;
      
      if (kw === '') {
        filteredStations.value = stations.value; // 輸入空白則顯示全部
        return;
      }
      
      filteredStations.value = stations.value.filter(
        (station) =>
          station.sna.toLowerCase().includes(kw) ||
          station.ar.toLowerCase().includes(kw)
      );
    };

    // 選擇一個站點顯示詳細資訊
    const selectStation = (station) => {
      my_modal_1.showModal()
      selectedStation.value = station;
    };

    const deselectStation = () => {
      my_modal_1.close()
      selectedStation.value = null;
    };

    // 從 API 取得資料
    const fetchData = async () => {
      const url =
        'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json';

      try {
        const response = await fetch(url);
        const data = await response.json();
        stations.value = data;
        filteredStations.value = data; // 預設顯示全部
      } catch (error) {
        console.error('資料載入失敗:', error);
      }
    };

    onMounted(() => {
      loadFavorites();
      
    });
</script>

<style scoped>
body {
  font-family: Arial, sans-serif;
  padding: 20px;
}

h1 {
  text-align: center;
}

.search-section {
  margin-bottom: 20px;
  text-align: center;
}

input {
  padding: 8px;
  width: 300px;
  margin-right: 10px;
}

button {
  padding: 8px 12px;
  cursor: pointer;
}

.station-list ul,
.favorites ul {
  list-style: none;
  padding: 0;
}

.station-list li,
.favorites li {
  padding: 8px;
  border-bottom: 1px solid #ccc;
  cursor: pointer;
}

.station-details {
  border: 1px solid #ccc;
  padding: 15px;
  margin-top: 20px;
}

.favorites {
  margin-top: 20px;
}
.header-row {
  display: flex;
  align-items: center;
  justify-content: space-between; /* 讓標題在左，按鈕在右 */
}

.close-btn {
  /* 可以加一些樣式，例如尺寸、間距等 */
  padding: 4px 12px;
  font-size: 14px;
  cursor: pointer;
}

</style>
