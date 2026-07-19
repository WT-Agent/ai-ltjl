<template>
  <section class="glass-card showcase-container">
    <div class="showcase-header">
      <div class="showcase-title-group">
        <h2 class="showcase-title">智能聊天记录剖析与关系诊断案例库 (30 精选样例)</h2>
        <p class="showcase-subtitle">体验不同人际关系的 AI 对话透视与情意探测，点击“一键同款生成”即可即刻核算</p>
      </div>
      <div class="showcase-badge">情感雷达 · 免费体验</div>
    </div>

    <!-- 搜索与分类筛选 -->
    <div class="showcase-filter-bar">
      <div class="category-tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          class="category-tab"
          :class="{ active: currentCategory === cat }"
          @click="currentCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="搜索聊天对话、角色关系、潜台词解读或情感诊断关键字..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 样例列表格 Grid -->
    <div class="sample-grid">
      <div 
        v-for="sample in paginatedSamples" 
        :key="sample.id" 
        class="sample-card"
      >
        <div class="sample-card-header">
          <span class="topic-category-tag">{{ sample.category }}</span>
          <span class="style-name-tag">{{ sample.style }}</span>
        </div>
        <div class="sample-original">
          <span class="sample-label">角色关系与对话：</span>“角色：{{ sample.roles }}。记录：{{ sample.topic }}”
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">诊断导向：</span>{{ sample.core }}
        </div>
        <div class="sample-card-footer">
          <button class="use-sample-btn" @click="$emit('use-sample', sample.topic, sample.roles)">
            一键同款生成
          </button>
        </div>
      </div>
    </div>

    <!-- 空状态提示 -->
    <div v-if="filteredSamples.length === 0" class="empty-showcase">
      未找到匹配的聊天记录案例，请尝试切换分类或重置搜索关键词。
    </div>

    <!-- 分页组件 -->
    <div v-if="filteredSamples.length > pageSize" class="pagination-bar">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        上一页
      </button>
      <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页 (共 {{ filteredSamples.length }} 条)</span>
      <button 
        class="page-btn" 
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        下一页
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

defineEmits<{
  (e: 'use-sample', text: string, roles: string): void;
}>();

const categories = ['全部', '暧昧破冰', '热恋拉扯', '冷淡敷衍', '相亲试探', '关系警报'];
const currentCategory = ref('全部');
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 6;

interface ChatRecordSample {
  id: number;
  category: string;
  roles: string;
  topic: string;
  style: string;
  core: string;
}

// 精选 30 条聊天记录诊断案例
const raw30Samples: ChatRecordSample[] = [
  {
    id: 1,
    category: '暧昧破冰',
    roles: '男生A（暗恋者）与女生B',
    topic: 'A: 睡了吗？\nB: 没呢，在看电影。\nA: 看什么电影呢？推荐一下呗。\nB: 一部挺老的故事片，你估计不喜欢看。',
    style: '情感亲密度评估',
    core: '破译“睡了吗”开局的社交温差，分析女方低意向社交距离。'
  },
  {
    id: 2,
    category: '冷淡敷衍',
    roles: '女生A与相亲对象男生B',
    topic: 'A: 今天周末去哪里玩了吗？\nB: 没，在家加班。\nA: 辛苦啦，那要多注意休息呀。\nB: 嗯。',
    style: '潜台词深度剖析',
    core: '剖析单音节字“嗯”背后的社交敷衍状态与沟通阻抗意图。'
  },
  {
    id: 3,
    category: '热恋拉扯',
    roles: '异地恋情侣（女生A与男生B）',
    topic: 'A: 你是不是不爱我了，一下午都不给我发微信。\nB: 我在开会，手机静音了，一结束就给你回了。\nA: 哼，以前开会你都会偷偷回我的，变了就是变了。',
    style: '关系话术诊断书',
    core: '诊断异地依赖情绪对关系造成的负荷，提供得体沟通建议。'
  },
  {
    id: 4,
    category: '关系警报',
    roles: '男生A（极度卑微）与女生B',
    topic: 'A: 早安！今天也要元气满满哦！\nA: 中午记得按时吃饭，我给你点了奶茶送到你公司了。\nB: 谢谢，以后别点了，我们同事都笑话我了。\nA: 没事没事，只要你开心就行！',
    style: '舔狗红线预警机',
    core: '拉响单方面自我感动投资的红色警戒，分析框架丧失深度。'
  },
  {
    id: 5,
    category: '约会邀约',
    roles: '相亲初聊（男生A与女生B）',
    topic: 'A: 听介绍人说你喜欢吃辣？\nB: 对，超喜欢川菜。\nA: 那太好了，我知道有一家很地道的九宫格火锅，周末一起去尝尝？\nB: 这周末我可能不太方便，下次吧。',
    style: '破局行动指南针',
    core: '翻译“下次吧”的真实拒绝成色，制定降温观察或软性重置方案。'
  },
  {
    id: 6,
    category: '暧昧破冰',
    roles: '女生A（主动）与暗恋学长B',
    topic: 'A: 学长，今天在图书馆看到你了，你旁边的书好深奥。\nB: 哈哈，是吗，在准备一门考试。\nA: 那祝学长备考顺利！有不懂的我能请教你吗？\nB: 可以啊。',
    style: '情感亲密度评估',
    core: '分析学长回复礼貌但温吞的态度，制定低频接触与针对性提问策略。'
  },
  {
    id: 7,
    category: '热恋拉扯',
    roles: '新婚夫妻（女生A与男生B）',
    topic: 'A: 老公，你今天回来能顺路买个蛋糕吗？想吃草莓的。\nB: 今天会很晚，要不改天？\nA: 好吧，那你路上注意安全。\nB: 算了，我快点做完，开完会去给你买。',
    style: '情感亲密度评估',
    core: '评估对话中双方双向奔赴的亲密值，判定高额情感连接。'
  },
  {
    id: 8,
    category: '冷淡敷衍',
    roles: '男生A与冷淡期女友B',
    topic: 'A: 这周末我们去郊游吧，我攻略都做好了。\nB: 周末我想在家睡觉，最近太累了。\nA: 那我去你家做饭给你吃？\nB: 别折腾了，我想一个人静静。',
    style: '潜台词深度剖析',
    core: '破译“想一个人静静”所暗示的冷暴力倾向，提供合理的抽离断联指南。'
  },
  {
    id: 9,
    category: '相亲试探',
    roles: '女生A与相亲对象B',
    topic: 'A: 你平时周末都有什么爱好呀？\nB: 爬山、看书，偶尔打打游戏。\nA: 爬山挺好的！你一般去哪座山？\nB: 附近随便爬爬。',
    style: '关系话术诊断书',
    core: '诊断男方低频话题承接度，指出“问答式”沟通泥潭，规划话题破局点。'
  },
  {
    id: 10,
    category: '关系警报',
    roles: '男生A（舔狗）与女生B',
    topic: 'A: 宝，你在干嘛呀？我今天看到一个很好笑的视频，链接发你。\nA: 怎么不回我，是遇到什么烦心事了吗？\nA: 别不开心哦，我一直都在的。\nB: （6小时后）刚在忙。',
    style: '舔狗红线预警机',
    core: '警示多频连环追问造成的社交窒息感，分析权力不对等的恋爱困境。'
  },
  {
    id: 11,
    category: '暧昧破冰',
    roles: '男生A与暧昧中女生B',
    topic: 'A: 刚刚在街上看到一个人，长得特别像你。\nB: 真的吗？我有那么大众脸吗？\nA: 哈哈不是，因为她笑起来也特别治愈。\nB: 贫嘴，今天嘴抹蜜啦？',
    style: '情感亲密度评估',
    core: '评估男女调情性互动的亲密度，判定情绪张力已达拉扯阶段。'
  },
  {
    id: 12,
    category: '热恋拉扯',
    roles: '情侣（男生A与女生B）',
    topic: 'A: 宝贝，今天晚上去吃烤肉吗？\nB: 随便，都可以。\nA: 那吃那家新开的韩式烤肉？\nB: 太多人了，不想排队。\nA: 那日式铁板烧？\nB: 太油腻了，不想吃。',
    style: '关系话术诊断书',
    core: '破译“随便”背后的情绪疲劳或隐性挑剔，提供一锤定音决策策略。'
  },
  {
    id: 13,
    category: '冷淡敷衍',
    roles: '女生A（暗恋者）与冷淡男生B',
    topic: 'A: 听说你喜欢喝这家的咖啡，我给你带了一杯放在你工位了。\nB: 谢谢，但我不怎么喝甜的，以后不用麻烦了。\nA: 没关系没关系，下次我帮你点美式！\nB: 真的不用了。',
    style: '舔狗红线预警机',
    core: '诊断送礼越界后的社交警戒机制，提示女方适可而止以保留体面。'
  },
  {
    id: 14,
    category: '相亲试探',
    roles: '男生A与相亲对象B',
    topic: 'A: 看你朋友圈经常加班，你们行业挺卷的吧？\nB: 差不多吧，大家都一样。\nA: 听说卷的行业收入高，你应该赚挺多的？\nB: 混口饭吃而已。',
    style: '关系话术诊断书',
    core: '诊断相亲初期“查户口/探资产”的越界低情商提问，提供合规话术调整。'
  },
  {
    id: 15,
    category: '关系警报',
    roles: '女生A（备胎）与男生B',
    topic: 'A: 今晚你有空吗，我抢到了话剧票。\nB: 今晚和朋友喝酒聚聚，下次吧。\nA: 那明天呢，明天我们去吃海鲜？\nB: 明天要加班，改天吧，乖。',
    style: '潜台词深度剖析',
    core: '剖析高频“改天吧加抚慰词（乖）”背后的高超吊备胎技巧，拉响备胎警报。'
  },
  {
    id: 16,
    category: '暧昧破冰',
    roles: '男生A与暗恋女生B',
    topic: 'A: 听说你家那只猫咪特别黏人？\nB: 对啊，它现在正躺在我腿上呼噜呼噜呢。\nA: 真羡慕它，能有这么好的专属猫抓板。\nB: 哈哈，你也可以来当它的副猫抓板。',
    style: '情感亲密度评估',
    core: '解读宠物话题带来的越界邀请暗示，评估极高暧昧指数。'
  },
  {
    id: 17,
    category: '热恋拉扯',
    roles: '吵架后情侣（女生A与男生B）',
    topic: 'A: 我到家了。\nB: 哦，知道了。\nA: 你这是什么态度？还在为下午的事生气吗？\nB: 没，我累了，先睡了。',
    style: '矛盾化解与降温',
    core: '破译冷战期“累了先睡”背后的消极逃避心理，制定降温和解话术。'
  },
  {
    id: 18,
    category: '冷淡敷衍',
    roles: '相亲初期（女生A与冷感对象B）',
    topic: 'A: 今天去那家新开的日料店了，味道真的很棒！\nB: 挺好。\nA: 哈哈，下次我们也可以一起去尝尝呀。\nB: 好的。',
    style: '潜台词深度剖析',
    core: '分析“挺好/好的”等标准客套回复的敷衍成色，提供止损或策略性退守。'
  },
  {
    id: 19,
    category: '相亲试探',
    roles: '男生A与相亲对象B',
    topic: 'A: 感觉你平时挺温柔的，介绍人也一直夸你。\nB: 哈哈，那都是客套话，我其实脾气挺急的。\nA: 急点好啊，办事效率高，直来直去挺可爱的。',
    style: '破局行动指南针',
    core: '分析顺承情绪与合理解释技巧，评估相亲初期情绪破冰成色。'
  },
  {
    id: 20,
    category: '关系警报',
    roles: '男生A与冷淡女友B',
    topic: 'A: 我们下个月去大理旅游吧？我假都请好了。\nB: 我下个月可能没时间，要备考。\nA: 你备考都备了半年了，抽三天陪我就这么难吗？\nB: 你觉得难就是难吧，别说了。',
    style: '关系话术诊断书',
    core: '诊断道德绑架提问引发的逆反心理，剖析“难就是难吧”代表的恋爱破裂红线。'
  },
  {
    id: 21,
    category: '暧昧破冰',
    roles: '女生A与暗恋对象B',
    topic: 'A: 听说你打篮球受伤了？严重吗？\nB: 还好，就是轻微扭伤，休息几天就行。\nA: 哎呀怎么这么不小心，好心疼啊。要我去看你吗？\nB: 没事，不用麻烦了。',
    style: '潜台词深度剖析',
    core: '分析过快释放强关心（心疼、看望）导致的对方心理边界防御，提示退避。'
  },
  {
    id: 22,
    category: '热恋拉扯',
    roles: '情侣（男生A与女生B）',
    topic: 'A: 宝贝我今天和部门同事聚餐。\nB: 有女同事吗？\nA: 有两个新来的实习生，怎么啦？\nB: 没事，多喝点，注意安全。',
    style: '潜台词深度剖析',
    core: '破译“没事，多喝点”背后的隐形吃醋与安全感缺乏，提供标准自律报备话术。'
  },
  {
    id: 23,
    category: '冷淡敷衍',
    roles: '女生A（卑微）与相亲对象B',
    topic: 'A: 在干嘛呢？\nB: 忙。\nA: 哦哦，那我不打扰你了，忙完记得找我哈。\nB: 嗯。',
    style: '舔狗红线预警机',
    core: '针对单字“忙、嗯”的极度敷衍，提供强制切断对话、重树自身框架的行动蓝图。'
  },
  {
    id: 24,
    category: '相亲试探',
    roles: '女生A与男生B',
    topic: 'A: 你以前谈过几次恋爱呀？\nB: 就两次，都很短暂。\nA: 怎么分手的呢？\nB: 性格不合吧，过去的事了。',
    style: '关系话术诊断书',
    core: '评估相亲试探前任话题时的边界把控，警示过度追问雷区。'
  },
  {
    id: 25,
    category: '关系警报',
    roles: '男生A与闹脾气女友B',
    topic: 'A: 宝贝我错了，别生气了嘛。\nB: 你错哪了？\nA: 我不该开会没及时回你消息。\nB: 呵呵，你没错，是我无理取闹了。',
    style: '矛盾化解与降温',
    core: '针对经典死循环问题“错哪了”，拆解女方隐藏的情绪认同诉求，给出破局模板。'
  },
  {
    id: 26,
    category: '暧昧破冰',
    roles: '男生A（幽默）与女生B',
    topic: 'A: 你是不是偷偷学过魔法？\nB: 啊？为什么这么说？\nA: 不然你怎么一出现，就把我身边别的人都变模糊了。\nB: 哈哈，你这也太土味了。',
    style: '情感亲密度评估',
    core: '评估土味情话下的关系破冰反馈，判定对方情绪回温。'
  },
  {
    id: 27,
    category: '热恋拉扯',
    roles: '异地情侣（女生A与男生B）',
    topic: 'A: 我今天生病了，一个人去挂水，感觉好可怜。\nB: 多喝热水，吃点退烧药，早点睡。\nA: 我不要吃药，我就要你陪我说话。\nB: 乖，别闹，我方案还没写完呢。',
    style: '关系话术诊断书',
    core: '诊断典型的“理工男敷衍”与“异地女缺乏关心”冲突，优化双方表达方式。'
  },
  {
    id: 28,
    category: '冷淡敷衍',
    roles: '男生A（卑微）与高冷女神B',
    topic: 'A: 你今天穿这条裙子真好看，像仙女一样！\nB: 谢谢。\nA: 晚上有空一起吃个饭吗？我订了你最爱吃的泰餐。\nB: 没空。',
    style: '舔狗红线预警机',
    core: '判定女方极度低社交窗口，拉响舔狗终极警报，命令即刻停止邀约。'
  },
  {
    id: 29,
    category: '相亲试探',
    roles: '男生A与相亲对象B',
    topic: 'A: 你平时周末加班多吗？\nB: 还行，偶尔加。\nA: 那下周六有空一起去逛逛那个新艺术展吗？\nB: 下周的事下周再说吧。',
    style: '潜台词深度剖析',
    core: '解译“下周再说”的推脱心理，提供得体后撤、降低暴露感的话术策略。'
  },
  {
    id: 30,
    category: '关系警报',
    roles: '女生A与回避型男友B',
    topic: 'A: 我们能聊聊以后的打算吗？你打算什么时候买房？\nB: 以后再说，现在压力大。\nA: 每次谈到未来你都回避，你根本就没打算娶我吧？\nB: 你要这么想我也没办法。',
    style: '关系话术诊断书',
    core: '诊断关系走到尽头的回避型特征，深度拆解“我也没办法”的绝杀潜台词。'
  }
];

const samples = ref<ChatRecordSample[]>(raw30Samples);

const filteredSamples = computed(() => {
  return samples.value.filter(s => {
    const matchCat = currentCategory.value === '全部' || s.category === currentCategory.value;
    const matchQuery = !searchQuery.value.trim() || 
      s.topic.includes(searchQuery.value) || 
      s.roles.includes(searchQuery.value) ||
      s.style.includes(searchQuery.value) || 
      s.core.includes(searchQuery.value);
    return matchCat && matchQuery;
  });
});

const totalPages = computed(() => Math.ceil(filteredSamples.value.length / pageSize) || 1);

const paginatedSamples = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredSamples.value.slice(start, start + pageSize);
});

watch([currentCategory, searchQuery], () => {
  currentPage.value = 1;
});
</script>
