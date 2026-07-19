<template>
  <div class="app-container">
    <!-- 顶部生成成功浮动 Toast -->
    <transition name="fade">
      <div v-if="showSuccessToast" class="top-success-toast">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
        <span>聊天记录诊断与情感评估成功！</span>
      </div>
    </transition>

    <!-- 右上角常驻分享按钮 -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="share-icon">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享</span>
    </button>

    <!-- 顶部 App Header (已安全移除未登录提示栏) -->
    <header>
      <h1>{{ appTitle }}</h1>
      <p>智能 AI 体验引擎 · 开启科学聊天记录透视与关系诊断</p>
    </header>

    <!-- 活跃动态 -->
    <UserTicker />

    <!-- 核心卡片 / 结果与历史记录展示 -->
    <main class="glass-card input-group">
      <div class="card-tabs">
        <button class="tab-btn" :class="{ active: !showHistory }" @click="showHistory = false">
          聊天诊断
        </button>
        <button class="tab-btn" :class="{ active: showHistory }" @click="showHistory = true">
          历史记录 ({{ historyList.length }})
        </button>
      </div>

      <!-- 本地历史记录视图 -->
      <div v-if="showHistory" class="history-view">
        <div class="history-header">
          <span>本地记录分析历史</span>
          <button v-if="historyList.length > 0" class="clear-all-btn" @click="clearAllHistory">清空全部</button>
        </div>

        <div v-if="historyList.length === 0" class="empty-state">
          <p>暂无历史诊断记录</p>
        </div>

        <div v-else class="history-grid">
          <div v-for="item in historyList" :key="item.id" class="history-card">
            <div class="h-card-header">
              <span class="h-card-style">{{ item.styleLabel }}</span>
              <span class="h-card-time">{{ item.timestamp }}</span>
            </div>
            
            <div class="h-card-body">
              <div class="h-card-nomad-title">
                <span class="h-city-tag">关系: {{ item.userRoles }}</span>
                <span class="h-score-badge">情感亲密度: {{ getAverageScore(item) }}</span>
              </div>

              <p class="h-card-excerpt"><strong>方案摘要：</strong>{{ cleanExcerpt(item.output) }}</p>
            </div>

            <div class="h-card-actions">
              <button class="h-action-btn load-btn" @click="selectHistoryItem(item)">
                加载详情
              </button>
              <button class="h-action-btn delete-btn" @click="deleteHistoryRecord(item.id)">
                删除
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-else>
        <!-- 主输入区域 (无多余滑动条，保持极简流畅) -->
        <div v-if="!result" class="divination-setup">
          <div class="selector-group">
            <label class="selector-label">对话双方的角色与关系 (如：相亲对象，男生A与女生B)</label>
            <input 
              type="text" 
              v-model="userRoles" 
              class="city-text-input" 
              placeholder="例如：相亲认识一个月，男生A（主动）与女生B（偏冷淡）..."
            />
          </div>

          <div class="selector-group">
            <label class="selector-label">真实的聊天对话历史记录 (换行输入)</label>
            <textarea 
              v-model="userInput" 
              placeholder="请输入对话记录。例如：&#10;A: 下班了吗？&#10;B: 刚下。&#10;A: 今天辛苦了，晚上吃什么？&#10;B: 还没想好。"
              rows="5"
            ></textarea>
          </div>

          <div class="selector-group">
            <label class="selector-label">选择聊天分析流派</label>
            <select v-model="activeStyle" class="style-select">
              <option 
                v-for="style in styleOptions" 
                :key="style.value" 
                :value="style.value"
              >
                {{ style.label }}
              </option>
            </select>
          </div>

          <button 
            class="action-btn" 
            :disabled="!userRoles.trim() || !userInput.trim() || loading" 
            @click="handleGenerate"
          >
            {{ loading ? '聊天记录关系透视中...' : '提交记录并生成关系诊断' }}
          </button>

          <!-- 异常提示 -->
          <div v-if="errorMsg" class="error-banner">
            {{ errorMsg }}
          </div>
        </div>

        <!-- 结果展现 -->
        <div v-else class="divination-result">
          <!-- 情意指数探测雷达打卡印章交互板块 -->
          <div class="stamp-section">
            <div class="stamp-canvas">
              <svg 
                class="stamp-svg" 
                :class="{ stamping: isStamping }" 
                @click="stampCheckin" 
                viewBox="0 0 160 160"
              >
                <!-- 雷达印章 -->
                <circle cx="80" cy="80" r="70" fill="rgba(168, 85, 247, 0.08)" stroke="#a855f7" stroke-width="4" stroke-dasharray="6,3" />
                <circle cx="80" cy="80" r="62" fill="none" stroke="#a855f7" stroke-width="2" />
                <!-- 雷达扫描十字线与同心圆 -->
                <circle cx="80" cy="80" r="40" fill="none" stroke="rgba(168, 85, 247, 0.3)" stroke-width="1" />
                <line x1="80" y1="18" x2="80" y2="142" stroke="rgba(168, 85, 247, 0.3)" stroke-width="1" />
                <line x1="18" y1="80" x2="142" y2="80" stroke="rgba(168, 85, 247, 0.3)" stroke-width="1" />
                
                <text x="80" y="72" font-size="20" font-weight="900" fill="#a855f7" text-anchor="middle">RADAR</text>
                <text x="80" y="102" font-size="14" font-weight="bold" fill="#a855f7" text-anchor="middle">情感探测</text>
              </svg>
              <!-- 漂浮在看动效 (严格无 Emoji) -->
              <transition-group name="float-up">
                <span 
                  v-for="item in floatingItems" 
                  :key="item.id" 
                  class="floating-merit"
                  :style="{ transform: `translate(${item.x}px, ${item.y}px)` }"
                >
                  {{ item.text }}
                </span>
              </transition-group>
            </div>
            <div class="merit-counter-display">
              情感雷达累计扫描：<strong style="color: #a855f7;">{{ totalCheckin }}</strong> 次
              <p class="wood-fish-tip">每日点击上方 Radar 雷达探测仪，破译潜台词对接电波信号</p>
            </div>
          </div>

          <!-- 单轨关系指标多维评估看板 (AI共识判定) -->
          <div v-if="aiScores" class="comparison-dashboard">
            <h3 class="dashboard-title">双向沟通质量多维评估 (AI共识判定)</h3>
            <div class="comparison-grid">
              <div v-for="metric in metricsList" :key="metric.key" class="comparison-row">
                <div class="metric-info">
                  <span class="metric-label">{{ metric.label }}</span>
                  <span class="metric-scores-text">
                    指数值: <strong style="color: var(--accent-color)">{{ aiScores[metric.key] }} / 5</strong>
                  </span>
                </div>
                <div class="comparison-bars">
                  <div class="bar-container">
                    <div class="bar-bg">
                      <div class="bar-fill ai-fill" :style="{ width: aiScores[metric.key] * 20 + '%' }"></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="result-header">
            <div class="result-title-group">
              <span class="result-title">记录分析报告</span>
              <span class="success-badge">生成成功</span>
            </div>
            <div class="button-actions">
              <button class="icon-btn" @click="copyText">
                {{ copied ? '已复制全文' : '复制结果' }}
              </button>
              <button class="icon-btn highlight" @click="showShareCard = true">
                生成分享卡片
              </button>
              <button class="icon-btn restart-btn" @click="resetReview">
                重新生成
              </button>
            </div>
          </div>

          <!-- 加载中骨架屏 -->
          <div v-if="loading" class="skeleton">
            <div class="skeleton-line" style="width: 80%"></div>
            <div class="skeleton-line" style="width: 95%"></div>
            <div class="skeleton-line" style="width: 60%"></div>
            <div class="skeleton-line" style="width: 75%"></div>
          </div>

          <!-- 渲染回复 -->
          <div class="ai-response-wrapper">
            <div class="output-content scroll-box" style="text-align: left;">{{ displayResultText }}</div>
          </div>
        </div>
      </div>
    </main>

    <!-- 演示案例区组件 (模块三：30 条聊天记录与关系情感分析精选案例展示) -->
    <DemoShowcase @use-sample="handleUseSample" />

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的隐私。您在本应用中输入的对话角色背景、敏感聊天记录细节等提示词仅用于实时大模型透视与关系诊断，我们不会在服务器端进行永久存储或记录。</p>
          <p>为了记录您的诊断历史、免费额度和打卡数据，本应用会在您的浏览器本地（localStorage）保存相关状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 智能聊天记录与关系分析辅助评估微应用。生成的建议、潜台词翻译、情感报告及避雷指南仅供个人参考，人际沟通需要真诚与包容，切勿盲目将其作为感情评判的绝对依据。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 (自适应高度 + weixin.png & dingtalk.png 展示) -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信联系" class="contact-qr-img" />
              <span class="contact-qr-label">微信联系</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉交流" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉交流</span>
            </div>
          </div>
          <p class="contact-email">反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 (模块四：裂变机制) -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />

    <!-- 分享卡片弹窗 (模块二扩展) -->
    <ShareCardModal
      :visible="showShareCard"
      :content="displayResultText"
      :wechat-id="wechatId"
      @close="showShareCard = false"
    />

    <!-- 微信 H5 分享引导浮层 -->
    <div v-if="showShareGuide" class="share-guide-overlay" @click="handleShareClose">
      <div class="share-guide-arrow">↗</div>
      <div class="share-guide-content">
        <p>点击右上角菜单 <strong>•••</strong></p>
        <p>选择 <strong>「分享到朋友圈」</strong></p>
        <p class="share-guide-sub">分享这款好用的 AI 智能聊天记录与关系分析微应用</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import DemoShowcase from './components/DemoShowcase.vue';
import ShareCardModal from './components/ShareCardModal.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

const appTitle = ref(appConfig.title || '网腾无限AI - 聊天记录与关系分析');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const userRoles = ref('男生A（舔狗）与女生B（高冷女神）');
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);
const showSuccessToast = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showShareCard = ref(false);

const isStamping = ref(false);
const totalCheckin = ref(parseInt(localStorage.getItem('ltjl_total_radar_stamp') || '0', 10));
const floatingItems = ref<{ id: number; text: string; x: number; y: number }[]>([]);
let floatingId = 0;

// 关系分析评估指标
const metricsList = [
  { key: 'closeness', label: '双向亲密程度 (Closeness)' },
  { key: 'balance', label: '话语权平衡度 (Balance)' },
  { key: 'subtext', label: '潜台词暴露度 (Subtext)' },
  { key: 'risk', label: '关系雷区风险 (Risk)' },
  { key: 'progress', label: '推进约会潜力 (Progress)' }
];

const aiScores = ref<{ closeness: number; balance: number; subtext: number; risk: number; progress: number; } | null>(null);

const styleOptions = [
  { label: '情感亲密度评估 (分析词汇温差/判定双方实际情感黏度深度)', value: '情感亲密度评估' },
  { label: '潜台词深度剖析 (翻译短句嗯/好的/去洗澡等高冷回复背后意图)', value: '潜台词深度剖析' },
  { label: '关系话术诊断书 (诊断低情商道德绑架/卑微索取等致命聊天雷区)', value: '关系话术诊断书' },
  { label: '破局行动指南针 (定制话题重置切入点/无痛推进线下聚会试探)', value: '破局行动指南针' },
  { label: '舔狗红线预警机 (亮起过度投资预警红灯/指引找回人设自我框架)', value: '舔狗红线预警机' }
];

const activeStyle = ref(styleOptions[0].value);

interface HistoryItem {
  id: string;
  timestamp: string;
  userRoles: string;
  input: string;
  styleLabel: string;
  aiScores: { closeness: number; balance: number; subtext: number; risk: number; progress: number; } | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('ltjl_history_records');
    historyList.value = raw ? JSON.parse(raw) : [];
  } catch (e) {
    historyList.value = [];
  }
};

const saveHistory = () => {
  localStorage.setItem('ltjl_history_records', JSON.stringify(historyList.value));
};

const addHistoryRecord = () => {
  const matched = styleOptions.find(o => o.value === activeStyle.value);
  const styleLabel = matched ? matched.label : '分析流派';

  const newItem: HistoryItem = {
    id: Date.now().toString(),
    timestamp: new Date().toLocaleString(),
    userRoles: userRoles.value,
    input: userInput.value,
    styleLabel,
    aiScores: aiScores.value ? { ...aiScores.value } : null,
    output: result.value
  };
  historyList.value.unshift(newItem);
  saveHistory();
};

const deleteHistoryRecord = (id: string) => {
  historyList.value = historyList.value.filter(item => item.id !== id);
  saveHistory();
};

const clearAllHistory = () => {
  if (confirm('确认清空所有历史聊天记录诊断记录吗？此操作不可恢复。')) {
    historyList.value = [];
    saveHistory();
  }
};

const selectHistoryItem = (item: HistoryItem) => {
  userRoles.value = item.userRoles;
  userInput.value = item.input;
  aiScores.value = item.aiScores ? { ...item.aiScores } : null;
  result.value = item.output;
  showHistory.value = false;
};

const getAverageScore = (item: HistoryItem) => {
  if (!item.aiScores) return '4.0';
  const s = item.aiScores;
  const avg = (s.closeness + s.balance + s.subtext + s.risk + s.progress) / 5;
  return avg.toFixed(1);
};

// 纯前端利用 Web Audio API 动态合成情感探测雷达声呐碰击声效
const stampCheckin = () => {
  isStamping.value = true;
  setTimeout(() => {
    isStamping.value = false;
  }, 100);

  // 增加雷达扫描计数
  totalCheckin.value++;
  localStorage.setItem('ltjl_total_radar_stamp', totalCheckin.value.toString());

  // 漂浮动效 (严格无 Emoji)
  const id = floatingId++;
  const x = (Math.random() - 0.5) * 60;
  const y = -40 - Math.random() * 20;
  floatingItems.value.push({ id, text: '电波对接成功', x, y });
  setTimeout(() => {
    floatingItems.value = floatingItems.value.filter(item => item.id !== id);
  }, 800);

  try {
    const audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    
    // 模拟雷达的高音调短暂音
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.type = 'sine';
    
    osc.frequency.setValueAtTime(1200, audioCtx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(600, audioCtx.currentTime + 0.15);

    gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.15);

    osc.connect(gain);
    gain.connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + 0.16);
  } catch (err) {
    // 捕获异常
  }
};

const parseAIScores = (text: string) => {
  const match = text.match(/\[LTJL_SCORES\](.*?)\[\/LTJL_SCORES\]/);
  if (match) {
    const scoreStr = match[1].trim();
    const scores: Record<string, number> = {};
    scoreStr.split(',').forEach(item => {
      const [key, val] = item.split(':');
      if (key && val) {
        scores[key.trim()] = Math.min(5, Math.max(1, parseInt(val.trim(), 10) || 3));
      }
    });
    return {
      closeness: scores.closeness || 3,
      balance: scores.balance || 3,
      subtext: scores.subtext || 3,
      risk: scores.risk || 3,
      progress: scores.progress || 3
    };
  }
  return null;
};

const cleanResponseText = (text: string) => {
  return text.replace(/\[LTJL_SCORES\][\s\S]*?\[\/LTJL_SCORES\]/gi, '').trim();
};

const displayResultText = computed(() => {
  return cleanResponseText(result.value);
});

const cleanExcerpt = (text: string) => {
  const cleaned = cleanResponseText(text);
  return cleaned.length > 80 ? cleaned.slice(0, 80) + '...' : cleaned;
};

onMounted(() => {
  loadHistory();
});

const handleShareClose = () => {
  showShareGuide.value = false;
  localStorage.setItem('shared_fission', 'true');
};

const isLimitReached = computed(() => {
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const triggerSuccessToast = () => {
  showSuccessToast.value = true;
  setTimeout(() => {
    showSuccessToast.value = false;
  }, 3000);
};

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const fullPrompt = `【对话双方角色与关系】：${userRoles.value}\n【真实的聊天对话记录】：${userInput.value}\n【选定分析流派】：${activeStyle.value}\n${promptTopic.value}`;

    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: fullPrompt,
        style: activeStyle.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      aiScores.value = parseAIScores(data.result);
      addHistoryRecord();
      triggerSuccessToast();
      
      const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
      localStorage.setItem('free_uses', (currentUses + 1).toString());
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const resetReview = () => {
  result.value = '';
  aiScores.value = null;
};

const handleUseSample = (sampleTopic: string, sampleRoles: string) => {
  // 解析样例填入
  userRoles.value = sampleRoles;
  userInput.value = sampleTopic;
  showHistory.value = false;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(displayResultText.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>
