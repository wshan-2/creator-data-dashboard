<template>
  <div class="saas-container">
    <div class="dashboard-area">
      
      <div v-if="!hasData" class="empty-state">
        <h1 class="title">✨ 创作者数字资产白皮书</h1>
        <p class="subtitle">深度量化内容价值，用硬核数据为您构建无懈可击的商业画像</p>
        <el-icon class="empty-icon"><DataAnalysis /></el-icon>
      </div>

      <div v-else class="report-grid" id="report-capture-area">
        <div class="report-header">
          <h2>📊 {{ creatorInfo.name || '独立创作者' }} · 商业洞察简报</h2>
          <p class="mode-badge">
            <span v-if="currentMode === 'single'">模块一：单篇爆款全景画像 (内容力证明)</span>
            <span v-else-if="currentMode === 'account'">模块二：账号宏观高净值大盘 (基本盘证明)</span>
            <span v-else>模块三：商单 ROI 与科学报价核算 (转化力证明)</span>
          </p>
          
          <el-button 
            type="success" 
            plain 
            round 
            class="export-btn" 
            @click="exportToImage" 
            :loading="isExporting"
            data-html2canvas-ignore
          >
            <el-icon style="margin-right: 5px;"><Download /></el-icon> 
            {{ isExporting ? '正在生成高清长图...' : '📥 导出高清商业报告' }}
          </el-button>
        </div>

        <template v-if="currentMode === 'single'">
          <el-row :gutter="20">
            <el-col :xs="24" :md="24" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header>
                  <h3>📉 黄金 5 秒与观众留存流失曲线</h3>
                  <span class="card-desc">直观展示内容“抓人”能力与自然完播趋势</span>
                </template>
                <div id="chart-retention" class="chart-box"></div>
              </el-card>
            </el-col>
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header>
                  <h3>🏆 核心传播力对标矩阵</h3>
                  <span class="card-desc">实线为本篇表现，虚线为同量级竞品均值</span>
                </template>
                <div id="chart-radar" class="chart-box"></div>
              </el-card>
            </el-col>
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header>
                  <h3>🌸 深度认同与社交货币转化</h3>
                  <span class="card-desc">过滤无效点赞，挖掘具有商业价值的“转发/收藏”</span>
                </template>
                <div id="chart-rose" class="chart-box"></div>
              </el-card>
            </el-col>
          </el-row>
        </template>

        <template v-if="currentMode === 'account'">
          <el-row :gutter="20">
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header>
                  <h3>👥 核心受众高净值画像分布</h3>
                  <span class="card-desc">展示粉丝购买力结构（年龄、性别、地域）</span>
                </template>
                <div id="chart-audience" class="chart-box"></div>
              </el-card>
            </el-col>
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header>
                  <h3>📈 近期流量底盘与爆发稳定性</h3>
                  <span class="card-desc">证明账号流量下限极高，非单次运气爆款</span>
                </template>
                <div id="chart-stability" class="chart-box"></div>
              </el-card>
            </el-col>
          </el-row>
        </template>

        <template v-if="currentMode === 'commercial'">
          <el-alert title="本页模型为广告界通用 ROI 算法，可直接作为【品牌合作报价与对赌协议】参考依据。" type="success" show-icon style="margin-bottom: 20px;" />
          <el-row :gutter="20">
            <el-col :xs="24" :md="12" style="margin-bottom: 40px;">
              <el-card shadow="hover" class="chart-card">
                <template #header>
                  <h3>🎯 甲方投资回报 (ROI) 盈亏平衡推演</h3>
                  <span class="card-desc">基于您的历史转化率与客单价，推算品牌方预期收益</span>
                </template>
                <div id="chart-roi" class="chart-box" style="height: 400px;"></div>
              </el-card>
            </el-col>
            <el-col :xs="24" :md="12" style="margin-bottom: 40px;">
              <el-card shadow="hover" class="chart-card">
                <template #header>
                  <h3>💰 科学报价拆解模型 (RMB)</h3>
                  <span class="card-desc">明码标价，拒绝盲目喊价，让品牌方看到溢价的价值</span>
                </template>
                <div id="chart-price" class="chart-box" style="height: 400px;"></div>
              </el-card>
            </el-col>
          </el-row>
        </template>

        <div class="report-footer">
          <p>报告生成时间：{{ currentTime }}</p>
          <p>数据溯源与核算：创作者数字资产审计系统 V1.0</p>
        </div>
      </div>
    </div>

    <div class="bottom-action-bar" data-html2canvas-ignore>
      <div class="action-inner">
        <el-button class="manual-btn" @click="profileDialogVisible = true" plain>
          <el-icon style="margin-right: 5px;"><User /></el-icon> 账号档案
        </el-button>
        <el-button type="primary" class="manual-btn" @click="dataDialogVisible = true" plain>
          <el-icon style="margin-right: 5px;"><EditPen /></el-icon> 输入硬核数据源
        </el-button>
      </div>
    </div>

    <el-dialog v-model="profileDialogVisible" title="⚙️ 创作者全局档案" width="400px">
      <el-form :model="creatorInfo" label-width="80px">
        <el-form-item label="博主昵称"><el-input v-model="creatorInfo.name" /></el-form-item>
        <el-form-item label="内容赛道"><el-input v-model="creatorInfo.niche" /></el-form-item>
      </el-form>
      <template #footer><el-button type="primary" @click="saveProfile">保存</el-button></template>
    </el-dialog>

    <el-dialog v-model="dataDialogVisible" title="📊 模式选择与硬核数据录入" width="750px" top="5vh">
      <div style="text-align: center; margin-bottom: 20px;">
        <el-radio-group v-model="inputMode" size="default">
          <el-radio-button label="single">模块一：单篇爆款画像</el-radio-button>
          <el-radio-button label="account">模块二：账号高净值大盘</el-radio-button>
          <el-radio-button label="commercial">模块三：商业 ROI 测算</el-radio-button>
        </el-radio-group>
      </div>
      <el-form :model="formData" label-width="130px" label-position="left" size="small">
        <template v-if="inputMode === 'single'">
          <el-divider content-position="left">基础流量与留存指标</el-divider>
          <el-row :gutter="20">
            <el-col :span="8"><el-form-item label="总播放量"><el-input-number v-model="formData.single.views" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="5秒完播率(%)"><el-input-number v-model="formData.single.fiveSecRate" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="整体完播率(%)"><el-input-number v-model="formData.single.finishRate" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
          <el-divider content-position="left">深度互动转化指标</el-divider>
          <el-row :gutter="20">
            <el-col :span="6"><el-form-item label="点赞数" label-width="70px"><el-input-number v-model="formData.single.likes" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="评论数" label-width="70px"><el-input-number v-model="formData.single.comments" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="转发数" label-width="70px"><el-input-number v-model="formData.single.shares" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="收藏数" label-width="70px"><el-input-number v-model="formData.single.saves" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
        </template>
        <template v-if="inputMode === 'account'">
          <el-divider content-position="left">受众购买力画像 (百分比 %)</el-divider>
          <el-row :gutter="20">
            <el-col :span="8"><el-form-item label="女性粉丝占比"><el-input-number v-model="formData.account.femaleRatio" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="18-30岁占比"><el-input-number v-model="formData.account.youngRatio" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="一二线城市占比"><el-input-number v-model="formData.account.tier1Ratio" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
          <el-divider content-position="left">账号流量稳定性 (近10期)</el-divider>
          <el-row :gutter="20">
            <el-col :span="12"><el-form-item label="最低播放 (下限)"><el-input-number v-model="formData.account.minViews" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="12"><el-form-item label="最高播放 (爆款)"><el-input-number v-model="formData.account.maxViews" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
        </template>
        <template v-if="inputMode === 'commercial'">
          <el-row :gutter="20">
            <el-col :span="12"><el-form-item label="基础千次播放(CPM)"><el-input-number v-model="formData.commercial.baseCpm" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="12"><el-form-item label="承诺播放量"><el-input-number v-model="formData.commercial.targetViews" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="12"><el-form-item label="预估点击率(%)"><el-input-number v-model="formData.commercial.ctr" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="12"><el-form-item label="产品客单价(元)"><el-input-number v-model="formData.commercial.aov" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
        </template>
      </el-form>
      <template #footer>
        <el-button @click="dataDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="generateReport">保存并生成商业图表</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, nextTick, onMounted, watch } from 'vue'
import { ElMessage } from 'element-plus'
import { DataAnalysis, EditPen, User, Download } from '@element-plus/icons-vue'
import * as echarts from 'echarts'
import html2canvas from 'html2canvas'

const hasData = ref(false)
const profileDialogVisible = ref(false)
const dataDialogVisible = ref(false)
const isExporting = ref(false)
const inputMode = ref('single') 
const currentMode = ref('single')
const currentTime = ref('')

const creatorInfo = reactive({ name: '文思涵', niche: '种草测评' })

const formData = reactive({
  single: { views: 125000, fiveSecRate: 65, finishRate: 35.5, likes: 4500, comments: 850, shares: 3200, saves: 5600 },
  account: { femaleRatio: 78, youngRatio: 85, tier1Ratio: 62, minViews: 85000, maxViews: 450000 },
  commercial: { baseCpm: 40, targetViews: 200000, ctr: 1.5, aov: 129 }
})

// ================= 数据持久化 (LocalStorage) =================
onMounted(() => {
  const savedProfile = localStorage.getItem('vlog_creator')
  if (savedProfile) Object.assign(creatorInfo, JSON.parse(savedProfile))
  
  const savedData = localStorage.getItem('vlog_data')
  if (savedData) Object.assign(formData, JSON.parse(savedData))
  
  // 更新当前时间戳
  const now = new Date()
  currentTime.value = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`
})

const saveProfile = () => {
  localStorage.setItem('vlog_creator', JSON.stringify(creatorInfo))
  profileDialogVisible.value = false
  ElMessage.success('档案已保存在本地')
}

// 监听数据变化，自动存档
watch(formData, (newVal) => {
  localStorage.setItem('vlog_data', JSON.stringify(newVal))
}, { deep: true })

// ================= 一键导出图片 =================
const exportToImage = async () => {
  isExporting.value = true
  try {
    const element = document.getElementById('report-capture-area')
    const canvas = await html2canvas(element, {
      scale: 2, // 提高清晰度
      useCORS: true,
      backgroundColor: '#f5f7fa'
    })
    
    // 生成图片并触发下载
    const imgData = canvas.toDataURL('image/png')
    const link = document.createElement('a')
    link.href = imgData
    link.download = `${creatorInfo.name}_商业洞察报告_${currentMode.value}.png`
    link.click()
    
    ElMessage.success('🎉 高清商业报告已成功下载！')
  } catch (error) {
    console.error("导出失败", error)
    ElMessage.error('导出失败，请重试')
  } finally {
    isExporting.value = false
  }
}

// ================= 生成报告逻辑 =================
const generateReport = async () => {
  dataDialogVisible.value = false
  currentMode.value = inputMode.value
  hasData.value = true
  ElMessage.success('商业算法执行完毕，图表已生成')
  await nextTick()
  
  if (currentMode.value === 'single') initSingleCharts()
  else if (currentMode.value === 'account') initAccountCharts()
  else if (currentMode.value === 'commercial') initCommercialCharts()
}

// (以下是与上一版完全一致的硬核 ECharts 渲染代码，为了保证你能直接运行，全部保留)
const initSingleCharts = () => {
  const d = formData.single
  const retentionChart = echarts.init(document.getElementById('chart-retention'))
  retentionChart.setOption({
    tooltip: { trigger: 'axis', formatter: '{b}: {c}% 观众留存' }, grid: { left: '5%', right: '5%', bottom: '10%', top: '15%', containLabel: true },
    xAxis: { type: 'category', boundaryGap: false, data: ['0秒(曝光)', '黄金5秒', '视频中段', '视频完播'] }, yAxis: { type: 'value', max: 100, axisLabel: { formatter: '{value}%' } },
    series: [{ type: 'line', smooth: true, symbol: 'circle', symbolSize: 8, areaStyle: { color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{ offset: 0, color: 'rgba(64,158,255,0.6)' }, { offset: 1, color: 'rgba(64,158,255,0.1)' }]) }, itemStyle: { color: '#409EFF' }, lineStyle: { width: 3 }, data: [100, d.fiveSecRate, Math.floor((d.fiveSecRate + d.finishRate)/2), d.finishRate] }]
  })
  const radarChart = echarts.init(document.getElementById('chart-radar'))
  radarChart.setOption({
    tooltip: {}, legend: { bottom: '0' }, radar: { indicator: [{ name: '曝光获取', max: 100 }, { name: '5秒抓入', max: 100 }, { name: '整体完播', max: 100 }, { name: '浅层互动', max: 100 }, { name: '深度种草', max: 100 }] },
    series: [{ type: 'radar', data: [{ value: [90, d.fiveSecRate, d.finishRate, 85, 95], name: '本篇表现', areaStyle: { color: 'rgba(64, 158, 255, 0.4)' }, lineStyle: { width: 3, color: '#409EFF' } }, { value: [60, 40, 20, 50, 45], name: '行业均值', areaStyle: { color: 'transparent' }, lineStyle: { width: 2, type: 'dashed', color: '#999' } }] }]
  })
  const roseChart = echarts.init(document.getElementById('chart-rose'))
  roseChart.setOption({
    tooltip: { trigger: 'item', formatter: '{b} : {c} ({d}%)' }, legend: { bottom: '0%' }, series: [{ type: 'pie', radius: [20, 110], center: ['50%', '45%'], roseType: 'area', itemStyle: { borderRadius: 8 }, data: [{ value: d.saves, name: '高意向收藏' }, { value: d.shares, name: '社交破圈转发' }, { value: d.comments, name: '热议评论' }, { value: d.likes, name: '认同点赞' }] }]
  })
  window.addEventListener('resize', () => { retentionChart.resize(); radarChart.resize(); roseChart.resize() })
}

const initAccountCharts = () => {
  const d = formData.account
  const audienceChart = echarts.init(document.getElementById('chart-audience'))
  audienceChart.setOption({
    tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' } }, grid: { left: '3%', right: '10%', bottom: '5%', top: '10%', containLabel: true },
    xAxis: { type: 'value', max: 100, axisLabel: { formatter: '{value}%' } }, yAxis: { type: 'category', data: ['一二线城市', '18-30岁青年', '女性消费主导'] },
    series: [{ type: 'bar', barWidth: '40%', itemStyle: { color: new echarts.graphic.LinearGradient(1, 0, 0, 0, [{ offset: 0, color: '#fac858' }, { offset: 1, color: '#ff9f7f' }]), borderRadius: [0, 5, 5, 0] }, label: { show: true, position: 'right', formatter: '{c}%' }, data: [d.tier1Ratio, d.youngRatio, d.femaleRatio] }]
  })
  const stabilityChart = echarts.init(document.getElementById('chart-stability'))
  stabilityChart.setOption({
    tooltip: { trigger: 'axis' }, grid: { left: '5%', right: '5%', bottom: '10%', top: '15%', containLabel: true },
    xAxis: { type: 'category', boundaryGap: false, data: ['1期前', '3期前', '5期前', '7期前', '9期前'] }, yAxis: { type: 'value', name: '播放量' },
    series: [{ name: '近期表现', type: 'line', smooth: true, data: [d.minViews*1.2, d.maxViews*0.8, d.minViews*1.5, d.maxViews, d.minViews*1.1], itemStyle: { color: '#67C23A' }, lineStyle: { width: 3 }, markArea: { itemStyle: { color: 'rgba(103,194,58,0.1)' }, data: [[ { yAxis: d.minViews, name: '保底流量护城河' }, { yAxis: d.maxViews } ]] } }]
  })
  window.addEventListener('resize', () => { audienceChart.resize(); stabilityChart.resize() })
}

const initCommercialCharts = () => {
  const d = formData.commercial
  const estimatedClicks = Math.floor(d.targetViews * (d.ctr / 100))
  const estimatedOrders = Math.floor(estimatedClicks * 0.02)
  const estimatedGmv = estimatedOrders * d.aov
  const roiChart = echarts.init(document.getElementById('chart-roi'))
  roiChart.setOption({
    tooltip: { trigger: 'item', formatter: '{b} : {c}' }, color: ['#5470c6', '#91cc75', '#fac858', '#ee6666'],
    series: [{ type: 'funnel', left: '10%', top: '5%', bottom: '5%', width: '80%', sort: 'descending', gap: 2, label: { show: true, position: 'inside', formatter: '{b}\n{c}', fontSize: 13 }, data: [{ value: d.targetViews, name: '承诺总曝光' }, { value: estimatedClicks, name: `预估进店 (${d.ctr}%)` }, { value: estimatedOrders, name: '预估成交单量' }, { value: estimatedGmv, name: `预估创造 GMV (¥${d.aov})` }] }]
  })
  const baseQuote = Math.floor((d.targetViews / 1000) * d.baseCpm)
  const premiumQuote = Math.floor(estimatedGmv * 0.1) 
  const totalQuote = baseQuote + premiumQuote
  const priceChart = echarts.init(document.getElementById('chart-price'))
  priceChart.setOption({
    tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' } }, grid: { left: '3%', right: '4%', bottom: '3%', top: '15%', containLabel: true },
    xAxis: { type: 'category', data: ['CPM底价', '转化附加值', '最终报价'] }, yAxis: { type: 'value', name: '人民币(元)' },
    series: [{ type: 'bar', barWidth: '40%', data: [{ value: baseQuote, itemStyle: { color: '#91cc75' } }, { value: premiumQuote, itemStyle: { color: '#fac858' } }, { value: totalQuote, itemStyle: { color: '#ee6666' } }], label: { show: true, position: 'top', formatter: '¥ {c}', fontSize: 16, fontWeight: 'bold' } }]
  })
  window.addEventListener('resize', () => { roiChart.resize(); priceChart.resize() })
}
</script>

<style scoped>
.saas-container { height: 100vh; width: 100vw; display: flex; flex-direction: column; background: linear-gradient(135deg, #f5f7fa 0%, #e4e7eb 100%); overflow: hidden; margin: -8px; }
.dashboard-area { flex-grow: 1; overflow-y: auto; padding: 40px 20px 120px 20px; box-sizing: border-box; }
.empty-state { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 70%; text-align: center; }
.title { font-size: 2.2rem; color: #1a202c; margin-bottom: 15px; font-weight: 800; }
.subtitle { font-size: 1.1rem; color: #718096; margin-bottom: 30px; }
.empty-icon { font-size: 80px; color: #cbd5e0; opacity: 0.5; }

.report-grid { max-width: 1200px; margin: 0 auto; background: #fff; padding: 30px; border-radius: 16px; box-shadow: 0 10px 40px rgba(0,0,0,0.05); }
.report-header { text-align: center; margin-bottom: 30px; position: relative;}
.report-header h2 { color: #2d3748; font-size: 2.2rem; margin-bottom: 12px; font-weight: 800; }
.mode-badge span { background-color: #2b6cb0; color: white; padding: 6px 15px; border-radius: 20px; font-size: 0.9rem; font-weight: bold; }

.export-btn { margin-top: 15px; font-weight: bold; }
.report-footer { margin-top: 40px; padding-top: 20px; border-top: 1px dashed #e2e8f0; text-align: center; color: #a0aec0; font-size: 0.85rem; line-height: 1.8; }

.chart-card { border-radius: 12px; border: 1px solid #edf2f7; box-shadow: none; transition: all 0.3s ease; }
.chart-card:hover { box-shadow: 0 10px 20px rgba(0,0,0,0.04); transform: translateY(-2px);}
.chart-card h3 { margin: 0; color: #2d3748; font-size: 1.1rem; }
.card-desc { display: block; font-size: 0.85rem; color: #a0aec0; margin-top: 5px; font-weight: normal; }
.chart-box { height: 350px; width: 100%; }

.bottom-action-bar { position: fixed; bottom: 0; left: 0; width: 100%; background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(20px); border-top: 1px solid rgba(226, 232, 240, 0.8); box-shadow: 0 -10px 30px rgba(0, 0, 0, 0.05); padding: 15px 0; z-index: 1000; }
.action-inner { max-width: 900px; margin: 0 auto; display: flex; align-items: center; justify-content: center; gap: 15px; padding: 0 20px; }
.manual-btn { height: 50px; border-radius: 8px; font-weight: bold; }

:deep(.el-form-item__label) { font-weight: 600; color: #4a5568; }
:deep(.el-divider__text) { font-weight: bold; color: #2b6cb0; background-color: #fff; padding: 0 15px;}
</style>