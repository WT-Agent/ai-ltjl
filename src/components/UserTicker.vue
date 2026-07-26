<template>
  <div class="glass-card ticker-card">
    <div class="ticker-wrapper">
      <div v-for="(item, index) in tickerItems" :key="index" class="ticker-item">
        <span>{{ item.text }}</span>
        <span style="color: var(--success-color); font-weight: bold;">[生成完成]</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tickerItems = ref([
  {
    "text": "用户 136****4411 分析了相亲对象连续 3 天单字回复的真实心理"
  },
  {
    "text": "用户 188****9020 解读了领导深夜微信发“收到”后的隐性关注"
  },
  {
    "text": "用户 132****6612 完成了情侣争吵聊天记录的潜台词拆解与破局"
  },
  {
    "text": "用户 151****8830 分析了重要客户推迟签约对话背后的真实顾虑"
  },
  {
    "text": "用户 176****1129 获得了破解暧昧对象忽冷忽热聊天法门"
  }
]);
</script>
