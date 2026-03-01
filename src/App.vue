<template>
  <div class="saas-container">
    
    <div v-if="currentStep === 'onboarding'" class="step-onboarding">
      <div class="welcome-card">
        <h1 class="title">✨ 创作者数字资产工作台</h1>
        <p class="subtitle">请建立您的专属商业档案，系统将为您匹配专属算法模型</p>
        
        <el-form :model="creatorInfo" label-width="80px" class="profile-form">
          <el-form-item label="博主昵称">
            <el-input v-model="creatorInfo.name" placeholder="请输入您的IP名称" size="large" />
          </el-form-item>
          <el-form-item label="内容赛道">
            <el-select v-model="creatorInfo.niche" placeholder="请选择主攻赛道，影响后续诊断模型" size="large" style="width: 100%">
              <el-option label="科技数码 (硬核/高客单)" value="科技数码" />
              <el-option label="美妆穿搭 (颜值/高复购)" value="美妆穿搭" />
              <el-option label="知识干货 (专业/高留存)" value="知识干货" />
              <el-option label="泛娱乐/剧情 (流量池)" value="泛娱乐" />
            </el-select>
          </el-form-item>
          <el-button type="primary" size="large" class="enter-btn" @click="enterWorkspace" :disabled="!creatorInfo.name || !creatorInfo.niche">
            加载专属商业模型 🚀
          </el-button>
        </el-form>
      </div>
    </div>

    <div v-else-if="currentStep === 'home'" class="step-home">
      <div class="home-header">
        <h2>👋 欢迎，{{ creatorInfo.name }}</h2>
        <p>专属模型：<strong style="color: #409EFF">{{ creatorInfo.niche }}</strong> | 请选择今天的商业诊断方向</p>
        <el-button plain size="small" @click="currentStep = 'onboarding'">重置档案</el-button>
      </div>

      <el-row :gutter="30" class="feature-cards">
        <el-col :xs="24" :md="8">
          <div class="feature-card" @click="openDataDialog('single')">
            <div class="icon-wrap" style="background: #ecf5ff; color: #409EFF;"><el-icon><DataLine /></el-icon></div>
            <h3>内容诊断与受众留存</h3>
            <p>基于完播曲线与互动深度进行复盘</p>
          </div>
        </el-col>
        <el-col :xs="24" :md="8">
          <div class="feature-card" @click="openDataDialog('account')">
            <div class="icon-wrap" style="background: #fdf6ec; color: #E6A23C;"><el-icon><PieChart /></el-icon></div>
            <h3>高净值客群商业大盘</h3>
            <p>挖掘受众购买力与账号基本盘稳定性</p>
          </div>
        </el-col>
        <el-col :xs="24" :md="8">
          <div class="feature-card" @click="openDataDialog('commercial')">
            <div class="icon-wrap" style="background: #fef0f0; color: #F56C6C;"><el-icon><Money /></el-icon></div>
            <h3>品牌方商单 ROI 测算</h3>
            <p>生成极具说服力的专业报价与转化预测</p>
          </div>
        </el-col>
      </el-row>
    </div>

    <div v-else-if="currentStep === 'report'" class="dashboard-area" id="report-capture-area">
      <div class="report-header">
        <el-button class="back-btn" plain @click="currentStep = 'home'"><el-icon><Back /></el-icon> 返回大厅</el-button>
        <h2>📊 {{ creatorInfo.name }} · 专属商业洞察简报</h2>
        <p class="mode-badge">
          <span v-if="currentMode === 'single'">单篇爆款内容力复盘</span>
          <span v-else-if="currentMode === 'account'">账号客群高净值大盘</span>
          <span v-else>品牌方商单转化测算单</span>
        </p>
        <el-button type="success" plain round class="export-btn" @click="exportToImage" :loading="isExporting" data-html2canvas-ignore>
          <el-icon style="margin-right: 5px;"><Download /></el-icon> {{ isExporting ? '生成中...' : '导出商业长图' }}
        </el-button>
      </div>

      <div class="report-grid">
        <el-alert 
          v-if="aiInsightText"
          :title="`💡 ${creatorInfo.niche}赛道专属诊断结论：`" 
          :description="aiInsightText" 
          type="success" 
          show-icon 
          :closable="false" 
          class="custom-alert"
        />

        <template v-if="currentMode === 'single'">
          <el-row :gutter="20">
            <el-col :span="24" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header><h3>📉 流量留存与黄金法则曲线</h3></template>
                <div id="chart-retention" class="chart-box"></div>
              </el-card>
            </el-col>
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header><h3>🏆 核心传播力对标矩阵</h3></template>
                <div id="chart-radar" class="chart-box"></div>
              </el-card>
            </el-col>
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header><h3>🌸 社交货币与种草转化结构</h3></template>
                <div id="chart-rose" class="chart-box"></div>
              </el-card>
            </el-col>
          </el-row>
        </template>

        <template v-if="currentMode === 'account'">
          <el-row :gutter="20">
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header><h3>👥 客群消费力多维图谱</h3></template>
                <div id="chart-audience" class="chart-box"></div>
              </el-card>
            </el-col>
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header><h3>📈 账号底盘与爆发稳定性 (10期)</h3></template>
                <div id="chart-stability" class="chart-box"></div>
              </el-card>
            </el-col>
          </el-row>
        </template>

        <template v-if="currentMode === 'commercial'">
          <el-row :gutter="20">
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header><h3>🎯 甲方预期 ROI 推演树</h3></template>
                <div id="chart-roi" class="chart-box" style="height: 400px;"></div>
              </el-card>
            </el-col>
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header><h3>💰 科学报价与溢价拆解 (RMB)</h3></template>
                <div id="chart-price" class="chart-box" style="height: 400px;"></div>
              </el-card>
            </el-col>
          </el-row>
        </template>
        
        <div class="report-footer">
          <p>报告生成时间：{{ currentTime }} | 算法模型：{{ creatorInfo.niche }} 商业核算 V3.0</p>
        </div>
      </div>
    </div>

    <el-dialog v-model="dataDialogVisible" :title="dialogTitle" width="750px" top="5vh">
      
      <div class="preset-area">
        <span style="font-size: 13px; color: #909399; margin-right: 10px;">不知如何填写？可一键代入同赛道参考模板：</span>
        <el-button v-for="(preset, index) in currentPresets" :key="index" size="small" type="primary" plain @click="applyPreset(preset.data)">
          {{ preset.label }}
        </el-button>
        <el-button size="small" type="danger" plain @click="clearForm">清空</el-button>
      </div>

      <el-form :model="formData" label-width="130px" label-position="left">
        <template v-if="currentMode === 'single'">
          <el-divider content-position="left">基础流量与留存指标</el-divider>
          <el-row :gutter="20">
            <el-col :span="8"><el-form-item label="总播放量"><el-input-number v-model="formData.single.views" :controls="false" style="width: 100%" placeholder="必填" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="5秒完播率(%)"><el-input-number v-model="formData.single.fiveSecRate" :controls="false" style="width: 100%" placeholder="必填" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="整体完播率(%)"><el-input-number v-model="formData.single.finishRate" :controls="false" style="width: 100%" placeholder="必填" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="点赞数" label-width="70px"><el-input-number v-model="formData.single.likes" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="评论数" label-width="70px"><el-input-number v-model="formData.single.comments" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="转发数" label-width="70px"><el-input-number v-model="formData.single.shares" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="收藏数" label-width="70px"><el-input-number v-model="formData.single.saves" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
        </template>
        <template v-if="currentMode === 'account'">
          <el-divider content-position="left">受众购买力画像 (百分比 %)</el-divider>
          <el-row :gutter="20">
            <el-col :span="8"><el-form-item label="女性占比(%)"><el-input-number v-model="formData.account.femaleRatio" :controls="false" style="width: 100%" max="100" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="18-30岁占比(%)"><el-input-number v-model="formData.account.youngRatio" :controls="false" style="width: 100%" max="100" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="一二线城市(%)"><el-input-number v-model="formData.account.tier1Ratio" :controls="false" style="width: 100%" max="100" /></el-form-item></el-col>
            <el-col :span="12"><el-form-item label="近期最低播放"><el-input-number v-model="formData.account.minViews" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="12"><el-form-item label="近期最高播放"><el-input-number v-model="formData.account.maxViews" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
        </template>
        <template v-if="currentMode === 'commercial'">
          <el-row :gutter="20">
            <el-col :span="12"><el-form-item label="CPM底价(元)"><el-input-number v-model="formData.commercial.baseCpm" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="12"><el-form-item label="承诺播放量"><el-input-number v-model="formData.commercial.targetViews" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="12"><el-form-item label="预估点击率(%)"><el-input-number v-model="formData.commercial.ctr" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="12"><el-form-item label="产品客单价(元)"><el-input-number v-model="formData.commercial.aov" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
        </template>
      </el-form>
      <template #footer>
        <el-button @click="dataDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="generateReport">深度推演图表</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, nextTick, onMounted, computed, watch } from 'vue'
import { ElMessage } from 'element-plus'
import { DataAnalysis, DataLine, PieChart, Money, EditPen, User, Download, Back } from '@element-plus/icons-vue'
import * as echarts from 'echarts'
import html2canvas from 'html2canvas'

// 状态管理
const currentStep = ref('onboarding')
const currentMode = ref('')
const dataDialogVisible = ref(false)
const dialogTitle = ref('')
const isExporting = ref(false)
const currentTime = ref('')
const aiInsightText = ref('')

const creatorInfo = reactive({ name: '', niche: '' })

// 表单数据初始化结构
const formData = reactive({
  single: { views: null, fiveSecRate: null, finishRate: null, likes: null, comments: null, shares: null, saves: null },
  account: { femaleRatio: null, youngRatio: null, tier1Ratio: null, minViews: null, maxViews: null },
  commercial: { baseCpm: null, targetViews: null, ctr: null, aov: null }
})

// 垂直赛道行业标准预设配置字典
const nicheConfig = {
  '科技数码': {
    single: [ { label: '💻 硬核万粉爆款', data: { views: 350000, fiveSecRate: 58, finishRate: 35, likes: 12000, comments: 2500, shares: 6000, saves: 15000 } } ],
    account: [ { label: '📱 数码极客大盘', data: { femaleRatio: 15, youngRatio: 88, tier1Ratio: 75, minViews: 80000, maxViews: 500000 } } ],
    commercial: [ { label: '🎧 高客单外设商单', data: { baseCpm: 50, targetViews: 200000, ctr: 1.8, aov: 899 } } ]
  },
  '美妆穿搭': {
    single: [ { label: '💄 种草带货爆款', data: { views: 500000, fiveSecRate: 65, finishRate: 25, likes: 25000, comments: 4500, shares: 3000, saves: 28000 } } ],
    account: [ { label: '👗 颜值经济大盘', data: { femaleRatio: 85, youngRatio: 92, tier1Ratio: 68, minViews: 120000, maxViews: 800000 } } ],
    commercial: [ { label: '✨ 高复购护肤商单', data: { baseCpm: 45, targetViews: 300000, ctr: 2.5, aov: 299 } } ]
  },
  '知识干货': {
    single: [ { label: '📚 沉浸式教学', data: { views: 150000, fiveSecRate: 45, finishRate: 40, likes: 5000, comments: 800, shares: 12000, saves: 20000 } } ],
    account: [ { label: '🎓 高知群体大盘', data: { femaleRatio: 55, youngRatio: 75, tier1Ratio: 80, minViews: 50000, maxViews: 150000 } } ],
    commercial: [ { label: '📖 课程售卖转化', data: { baseCpm: 60, targetViews: 100000, ctr: 3.0, aov: 199 } } ]
  },
  '泛娱乐': {
    single: [ { label: '😂 搞笑百万爆款', data: { views: 2000000, fiveSecRate: 75, finishRate: 50, likes: 150000, comments: 20000, shares: 80000, saves: 5000 } } ],
    account: [ { label: '🎡 泛下沉流量盘', data: { femaleRatio: 50, youngRatio: 65, tier1Ratio: 40, minViews: 500000, maxViews: 3000000 } } ],
    commercial: [ { label: '🍔 零食/APP推广', data: { baseCpm: 20, targetViews: 1500000, ctr: 1.0, aov: 39 } } ]
  }
}

// 动态计算适用的预设模板
const currentPresets = computed(() => {
  if (!creatorInfo.niche || !nicheConfig[creatorInfo.niche]) return []
  return nicheConfig[creatorInfo.niche][currentMode.value] || []
})

// 应用模板数据至当前表单状态
const applyPreset = (presetData) => {
  Object.keys(presetData).forEach(key => {
    formData[currentMode.value][key] = presetData[key]
  })
  ElMessage.success('已自动代入行业参考数据')
}

// 清除表单状态
const clearForm = () => {
  Object.keys(formData[currentMode.value]).forEach(key => { formData[currentMode.value][key] = null })
}

// 本地存储状态同步与初始化挂载
onMounted(() => {
  const savedProfile = localStorage.getItem('vlog_creator')
  if (savedProfile) {
    Object.assign(creatorInfo, JSON.parse(savedProfile))
    if (creatorInfo.name && creatorInfo.niche) currentStep.value = 'home'
  }
  const savedData = localStorage.getItem('vlog_data')
  if (savedData) Object.assign(formData, JSON.parse(savedData))
})

// 监听状态变化以持久化存储
watch([creatorInfo, formData], () => {
  localStorage.setItem('vlog_creator', JSON.stringify(creatorInfo))
  localStorage.setItem('vlog_data', JSON.stringify(formData))
}, { deep: true })

const enterWorkspace = () => { currentStep.value = 'home'; ElMessage.success(`专属 ${creatorInfo.niche} 算法已加载！`) }

const openDataDialog = (mode) => {
  currentMode.value = mode
  dialogTitle.value = mode === 'single' ? '录入单篇数据' : mode === 'account' ? '录入大盘画像' : '录入商业变现参数'
  dataDialogVisible.value = true
}

// 根据垂直类目与当前数据生成结构化报告文本
const generateAIInsights = () => {
  const d = formData[currentMode.value]
  const niche = creatorInfo.niche

  if (currentMode.value === 'single') {
    if (d.fiveSecRate > 60 && d.finishRate < 20) {
      aiInsightText.value = `【跳出率异常分析】该作品前5秒具备高留存特征，但整体完播率处于低位。结合 ${niche} 赛道特性，建议审查视频中后段内容密度，减少非必要冗余以提升整体播放深度。`
    } else if (d.fiveSecRate < 30 && d.finishRate > 40) {
      aiInsightText.value = `【长尾留存特征】视频初期跳出率较高，但核心受众留存情况良好。建议针对前3秒引入核心钩子，优化信息前置逻辑，有望扩大整体流量池开口。`
    } else if (d.saves > d.likes) {
      aiInsightText.value = `【商业转化潜力】在 ${niche} 维度下，该作品表现出“收藏量高于点赞量”的强实用属性特征，具备优质的种草转化能力，建议作为商业合作的核心案例展示。`
    } else {
      aiInsightText.value = `【各项指标稳定】互动指标分布均衡，体现了在 ${niche} 领域内成熟的内容控盘能力。`
    }
  } else if (currentMode.value === 'account') {
    const isMaleHeavy = d.femaleRatio < 50
    const genderTarget = isMaleHeavy ? '男性' : '女性'
    const power = d.tier1Ratio > 60 ? '较高' : '大众'
    aiInsightText.value = `【用户画像特征分析】当前客群呈现明显的 ${genderTarget} 主导结构，一二线城市占比达 ${d.tier1Ratio}%。基于 ${niche} 赛道属性及客群购买力评估（评估等级：${power}），建议拓展 ${isMaleHeavy ? '3C数码/汽车/游戏' : '美妆/母婴/轻奢'} 相关的高净值商业接洽。`
  } else if (currentMode.value === 'commercial') {
    const roiVal = (d.targetViews * (d.ctr/100) * 0.02 * d.aov) / ((d.targetViews/1000)*d.baseCpm)
    if (roiVal > 5) {
      aiInsightText.value = `【高 ROI 转化预估】系统推演显示，该商业计划预期 ROI 高达 1:${roiVal.toFixed(1)}。结合 ${niche} 赛道流量分发机制，转化率处于行业头部区间，建议在合作谈判中引入阶梯式 CPS 抽成协议。`
    } else {
      aiInsightText.value = `【长效曝光价值评估】本次商业方案侧重于品牌曝光度建设。针对客单价为 ${d.aov} 元的 SKU，当前报价策略稳健，符合 ${niche} 赛道的标准市场定价逻辑。`
    }
  }
}

// 报告生成主线逻辑与参数校验
const generateReport = async () => {
  const currentData = formData[currentMode.value]
  const hasEmpty = Object.values(currentData).some(v => v === null)
  if (hasEmpty) { ElMessage.warning('检测到空值项，请完善表单参数或应用参考模板'); return }

  dataDialogVisible.value = false
  currentStep.value = 'report'
  const now = new Date()
  currentTime.value = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`
  
  generateAIInsights()
  
  // 利用 nextTick 确保 DOM 更新完毕后进行 ECharts 实例化
  await nextTick()
  if (currentMode.value === 'single') initSingleCharts()
  else if (currentMode.value === 'account') initAccountCharts()
  else if (currentMode.value === 'commercial') initCommercialCharts()
}

// 基于 html2canvas 的节点快照导出逻辑
const exportToImage = async () => {
  isExporting.value = true
  try {
    const canvas = await html2canvas(document.getElementById('report-capture-area'), { scale: 2, backgroundColor: '#f5f7fa' })
    const link = document.createElement('a')
    link.href = canvas.toDataURL('image/png')
    link.download = `${creatorInfo.name}_商业图谱_${currentMode.value}.png`
    link.click()
  } catch (error) { ElMessage.error('导出任务异常中止') } finally { isExporting.value = false }
}

// 图表渲染模块：单篇数据分析
const initSingleCharts = () => {
  const d = formData.single
  const retentionChart = echarts.init(document.getElementById('chart-retention'))
  retentionChart.setOption({
    tooltip: { trigger: 'axis', axisPointer: { type: 'cross' } }, grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
    xAxis: { type: 'category', boundaryGap: false, data: ['0s', '3s', '5s', '中段', '完播'] }, yAxis: { type: 'value', max: 100 },
    series: [{ type: 'line', smooth: true, areaStyle: { opacity: 0.3 }, itemStyle: { color: '#409EFF' }, data: [100, Math.floor(d.fiveSecRate*1.1), d.fiveSecRate, Math.floor((d.fiveSecRate+d.finishRate)/2), d.finishRate] }]
  })
  
  const radarChart = echarts.init(document.getElementById('chart-radar'))
  radarChart.setOption({
    tooltip: {}, radar: { indicator: [{ name: '曝光', max: 100 }, { name: '5秒留存', max: 100 }, { name: '完播', max: 100 }, { name: '浅互动', max: 100 }, { name: '深种草', max: 100 }] },
    series: [{ type: 'radar', data: [{ value: [90, d.fiveSecRate, d.finishRate, 85, 95], name: '本篇', areaStyle: { opacity: 0.4 } }] }]
  })

  const roseChart = echarts.init(document.getElementById('chart-rose'))
  roseChart.setOption({
    tooltip: { trigger: 'item' }, series: [{ type: 'pie', roseType: 'area', radius: [20, 100], data: [{ value: d.saves, name: '收藏' }, { value: d.shares, name: '转发' }, { value: d.comments, name: '评论' }, { value: d.likes, name: '点赞' }] }]
  })
  window.addEventListener('resize', () => { retentionChart.resize(); radarChart.resize(); roseChart.resize() })
}

// 图表渲染模块：账号基本盘分析
const initAccountCharts = () => {
  const d = formData.account
  const maleRatio = 100 - d.femaleRatio
  const audienceChart = echarts.init(document.getElementById('chart-audience'))
  audienceChart.setOption({
    tooltip: { trigger: 'item' },
    series: [
      { name: '性别分布', type: 'pie', radius: ['40%', '60%'], center: ['25%', '50%'], label: { show: true, position: 'center', formatter: '性别' }, data: [{ value: d.femaleRatio, name: '女性', itemStyle: {color: '#ff9f7f'} }, { value: maleRatio, name: '男性', itemStyle: {color: '#83bff6'} }] },
      { name: '购买力画像', type: 'pie', radius: ['40%', '60%'], center: ['75%', '50%'], label: { show: true, position: 'center', formatter: '客单' }, data: [{ value: d.tier1Ratio, name: '一二线', itemStyle: {color: '#fac858'} }, { value: d.youngRatio, name: '18-30岁', itemStyle: {color: '#91cc75'} }] }
    ]
  })

  const stChart = echarts.init(document.getElementById('chart-stability'))
  const range = d.maxViews - d.minViews
  const mock10Data = [ d.minViews+range*0.2, d.minViews+range*0.5, d.minViews+range*0.1, d.minViews+range*0.8, d.minViews, d.maxViews, d.minViews+range*0.4, d.minViews+range*0.6, d.minViews+range*0.3, d.maxViews*0.9 ].map(Math.floor)

  stChart.setOption({
    tooltip: { trigger: 'axis', axisPointer: { type: 'cross' } }, grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
    xAxis: { type: 'category', boundaryGap: false, data: ['10期前','9期前','8期前','7期前','6期前','5期前','4期前','3期前','2期前','上一期'] }, yAxis: { type: 'value' },
    series: [{ name: '播放量', type: 'line', smooth: true, data: mock10Data, itemStyle: { color: '#67C23A' }, markArea: { itemStyle: { color: 'rgba(103,194,58,0.1)' }, data: [[ { yAxis: d.minViews, name: '保底流量护城河' }, { yAxis: d.maxViews } ]] } }]
  })
  window.addEventListener('resize', () => { audienceChart.resize(); stChart.resize() })
}

// 图表渲染模块：商业转化测算
const initCommercialCharts = () => {
  const d = formData.commercial
  const clk = Math.floor(d.targetViews * (d.ctr / 100)); const ord = Math.floor(clk * 0.02); const gmv = ord * d.aov
  
  const roiChart = echarts.init(document.getElementById('chart-roi'))
  roiChart.setOption({ tooltip: { trigger: 'item' }, series: [{ type: 'funnel', sort: 'descending', label: { show: true, position: 'inside', formatter: '{b}\n{c}' }, data: [{ value: d.targetViews, name: '总曝光' }, { value: clk, name: '点击' }, { value: ord, name: '成交' }, { value: gmv, name: 'GMV预估 (¥)' }] }] })
  
  const bq = Math.floor((d.targetViews / 1000) * d.baseCpm); const pq = Math.floor(gmv * 0.1)
  const priceChart = echarts.init(document.getElementById('chart-price'))
  priceChart.setOption({ tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' } }, xAxis: { type: 'category', data: ['底价', '溢价', '最终报价'] }, yAxis: { type: 'value' }, series: [{ type: 'bar', barWidth: '40%', data: [{ value: bq, itemStyle:{color:'#91cc75'}}, { value: pq, itemStyle:{color:'#fac858'}}, { value: bq+pq, itemStyle:{color:'#ee6666'}}], label: { show: true, position: 'top', formatter: '¥ {c}' } }] })
  
  window.addEventListener('resize', () => { roiChart.resize(); priceChart.resize() })
}
</script>

<style scoped>
.saas-container { min-height: 100vh; background: #f5f7fa; padding: 20px; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }

/* 引导页与首页样式 */
.step-onboarding { display: flex; justify-content: center; align-items: center; height: 90vh; }
.welcome-card { background: #fff; padding: 50px; border-radius: 16px; box-shadow: 0 20px 40px rgba(0,0,0,0.08); text-align: center; max-width: 500px; width: 100%; }
.welcome-card .title { color: #2d3748; font-size: 2rem; margin-bottom: 10px; }
.welcome-card .subtitle { color: #718096; margin-bottom: 30px; }
.enter-btn { width: 100%; margin-top: 20px; font-weight: bold; border-radius: 8px; }

.step-home { max-width: 1000px; margin: 0 auto; padding-top: 50px; }
.home-header { text-align: center; margin-bottom: 50px; }
.home-header h2 { font-size: 2.2rem; color: #2d3748; }
.home-header p { color: #718096; margin-bottom: 20px; font-size: 1.1rem; }

.feature-cards { margin-top: 30px; }
.feature-card { background: #fff; border-radius: 16px; padding: 40px 20px; text-align: center; cursor: pointer; transition: all 0.3s ease; border: 1px solid #edf2f7; height: 100%; display: flex; flex-direction: column; align-items: center; }
.feature-card:hover { transform: translateY(-5px); box-shadow: 0 15px 30px rgba(0,0,0,0.05); border-color: #cbd5e0; }
.icon-wrap { width: 70px; height: 70px; border-radius: 50%; display: flex; justify-content: center; align-items: center; font-size: 32px; margin-bottom: 20px; }
.feature-card h3 { color: #2d3748; margin-bottom: 10px; font-size: 1.2rem; }
.feature-card p { color: #a0aec0; font-size: 0.9rem; line-height: 1.5; }

/* 报告展示区域样式 */
.report-grid { max-width: 1200px; margin: 0 auto; background: #fff; padding: 40px; border-radius: 16px; box-shadow: 0 10px 40px rgba(0,0,0,0.03); }
.report-header { text-align: center; margin-bottom: 30px; position: relative; }
.back-btn { position: absolute; left: 0; top: 0; }
.report-header h2 { color: #2d3748; font-size: 2rem; margin: 15px 0; font-weight: 800; }
.mode-badge span { background-color: #2b6cb0; color: white; padding: 6px 15px; border-radius: 20px; font-size: 0.9rem; font-weight: bold; }
.export-btn { margin-top: 20px; font-weight: bold; }

.custom-alert { margin-bottom: 25px; border-radius: 8px; line-height: 1.6; font-size: 14px; border: 1px solid #e1f3d8;}

.chart-card { border-radius: 12px; border: 1px solid #edf2f7; box-shadow: none; margin-bottom: 20px;}
.chart-box { height: 350px; width: 100%; }

.report-footer { margin-top: 40px; padding-top: 20px; border-top: 1px dashed #e2e8f0; text-align: center; color: #a0aec0; font-size: 0.85rem; }

/* 弹窗及快捷预设区样式 */
.preset-area { background: #f4f4f5; padding: 12px 20px; border-radius: 8px; margin-bottom: 20px; display: flex; align-items: center; flex-wrap: wrap; gap: 10px;}

.bottom-action-bar { position: fixed; bottom: 0; left: 0; width: 100%; background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(20px); border-top: 1px solid rgba(226, 232, 240, 0.8); box-shadow: 0 -10px 30px rgba(0, 0, 0, 0.05); padding: 15px 0; z-index: 1000; display: none; }
:deep(.el-form-item__label) { font-weight: 600; color: #4a5568; }
:deep(.el-divider__text) { font-weight: bold; color: #2b6cb0; background-color: #fff; padding: 0 15px;}

/* 移动端响应式适配：修复卡片排列与间距问题 */
@media (max-width: 768px) {
  .feature-cards {
    display: flex !important;
    flex-direction: column !important;
    gap: 20px !important;
    margin-top: 20px !important;
  }
  
  .feature-cards .el-col {
    width: 100% !important;
    max-width: 100% !important;
    padding: 0 !important; 
  }

  .feature-card {
    height: auto !important;
    position: static !important;
    transform: none !important;
    margin: 0 !important;
  }
}
</style>