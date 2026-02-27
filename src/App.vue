<template>
  <div class="saas-container">
    
    <div v-if="currentStep === 'onboarding'" class="step-onboarding">
      <div class="welcome-card">
        <h1 class="title">✨ 创作者数字资产工作台</h1>
        <p class="subtitle">请先建立您的专属商业档案，开启数据变现之旅</p>
        
        <el-form :model="creatorInfo" label-width="80px" class="profile-form">
          <el-form-item label="博主昵称">
            <el-input v-model="creatorInfo.name" placeholder="请输入账号名称" size="large" />
          </el-form-item>
          <el-form-item label="内容赛道">
            <el-select v-model="creatorInfo.niche" placeholder="请选择您的主攻赛道" size="large" style="width: 100%">
              <el-option label="科技数码 (高客单)" value="科技数码" />
              <el-option label="美妆穿搭 (高复购)" value="美妆穿搭" />
              <el-option label="知识干货 (高留存)" value="知识干货" />
              <el-option label="泛娱乐 (高流量)" value="泛娱乐" />
            </el-select>
          </el-form-item>
          <el-button type="primary" size="large" class="enter-btn" @click="enterWorkspace" :disabled="!creatorInfo.name || !creatorInfo.niche">
            进入专属工作台 🚀
          </el-button>
        </el-form>
      </div>
    </div>

    <div v-else-if="currentStep === 'home'" class="step-home">
      <div class="home-header">
        <h2>👋 欢迎回来，{{ creatorInfo.name }}</h2>
        <p>当前赛道：{{ creatorInfo.niche }} | 请选择您今天要进行的商业分析</p>
        <el-button plain size="small" @click="currentStep = 'onboarding'">修改档案</el-button>
      </div>

      <el-row :gutter="30" class="feature-cards">
        <el-col :xs="24" :md="8">
          <div class="feature-card" @click="openDataDialog('single')">
            <div class="icon-wrap" style="background: #ecf5ff; color: #409EFF;"><el-icon><DataLine /></el-icon></div>
            <h3>单篇爆款诊断</h3>
            <p>分析单期视频留存率与社交传播力</p>
          </div>
        </el-col>
        <el-col :xs="24" :md="8">
          <div class="feature-card" @click="openDataDialog('account')">
            <div class="icon-wrap" style="background: #fdf6ec; color: #E6A23C;"><el-icon><PieChart /></el-icon></div>
            <h3>账号高净值大盘</h3>
            <p>分析粉丝消费画像与近期流量盘</p>
          </div>
        </el-col>
        <el-col :xs="24" :md="8">
          <div class="feature-card" @click="openDataDialog('commercial')">
            <div class="icon-wrap" style="background: #fef0f0; color: #F56C6C;"><el-icon><Money /></el-icon></div>
            <h3>商单 ROI 核算</h3>
            <p>生成给品牌方的专业转化报价单</p>
          </div>
        </el-col>
      </el-row>
    </div>

    <div v-else-if="currentStep === 'report'" class="dashboard-area" id="report-capture-area">
      <div class="report-header">
        <el-button class="back-btn" plain @click="currentStep = 'home'">
          <el-icon><Back /></el-icon> 返回功能大厅
        </el-button>
        
        <h2>📊 {{ creatorInfo.name }} · 商业洞察简报</h2>
        <p class="mode-badge">
          <span v-if="currentMode === 'single'">单篇爆款全景画像</span>
          <span v-else-if="currentMode === 'account'">账号宏观高净值大盘</span>
          <span v-else>商单 ROI 与科学报价核算</span>
        </p>
        
        <el-button type="success" plain round class="export-btn" @click="exportToImage" :loading="isExporting" data-html2canvas-ignore>
          <el-icon style="margin-right: 5px;"><Download /></el-icon> {{ isExporting ? '生成中...' : '导出商业长图' }}
        </el-button>
      </div>

      <div class="report-grid">
        <template v-if="currentMode === 'single'">
          <el-row :gutter="20">
            <el-col :span="24" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header><h3>📉 黄金 5 秒与受众留存漏斗</h3></template>
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
                <template #header><h3>🌸 社交货币转化结构</h3></template>
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
                  <h3>👥 客群消费力多维图谱</h3>
                  <span class="card-desc">自动补充男女比例，直击高净值商业属性</span>
                </template>
                <div class="ai-insight-box">
                  <p><strong>💡 商业匹配建议：</strong>该账号女性占比 {{ formData.account.femaleRatio }}%，男性 {{ 100 - formData.account.femaleRatio }}%，且 {{ formData.account.youngRatio }}% 为 18-30 岁青壮年。完美契合<strong style="color: #E6A23C;">【{{ formData.account.femaleRatio > 50 ? '美妆/零食/轻奢' : '3C数码/汽车/游戏' }}】</strong>品类的商业投放。</p>
                </div>
                <div id="chart-audience" class="chart-box" style="height: 280px;"></div>
              </el-card>
            </el-col>
            <el-col :xs="24" :md="12" style="margin-bottom: 20px;">
              <el-card shadow="hover" class="chart-card">
                <template #header>
                  <h3>📈 近期流量盘与爆发稳定性 (10期)</h3>
                  <span class="card-desc">鼠标悬浮可丝滑查看每期明细，无死角展示底盘下限</span>
                </template>
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
                <template #header><h3>💰 科学报价溢价拆解 (RMB)</h3></template>
                <div id="chart-price" class="chart-box" style="height: 400px;"></div>
              </el-card>
            </el-col>
          </el-row>
        </template>
        
        <div class="report-footer">
          <p>报告生成时间：{{ currentTime }}</p>
          <p>数据溯源：创作者数字资产审计系统 V2.0</p>
        </div>
      </div>
    </div>

    <el-dialog v-model="dataDialogVisible" :title="dialogTitle" width="700px" top="8vh">
      <el-form :model="formData" label-width="130px" label-position="left">
        
        <template v-if="currentMode === 'single'">
          <el-row :gutter="20">
            <el-col :span="8"><el-form-item label="总播放量"><el-input-number v-model="formData.single.views" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="5秒完播率(%)"><el-input-number v-model="formData.single.fiveSecRate" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="8"><el-form-item label="整体完播率(%)"><el-input-number v-model="formData.single.finishRate" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="点赞数" label-width="70px"><el-input-number v-model="formData.single.likes" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="评论数" label-width="70px"><el-input-number v-model="formData.single.comments" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="转发数" label-width="70px"><el-input-number v-model="formData.single.shares" :controls="false" style="width: 100%" /></el-form-item></el-col>
            <el-col :span="6"><el-form-item label="收藏数" label-width="70px"><el-input-number v-model="formData.single.saves" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
        </template>

        <template v-if="currentMode === 'account'">
          <el-alert title="我们将自动推算男性受众比例，为您构建完整的画像" type="info" :closable="false" style="margin-bottom:15px;" />
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
            <el-col :span="12"><el-form-item label="客单价(元)"><el-input-number v-model="formData.commercial.aov" :controls="false" style="width: 100%" /></el-form-item></el-col>
          </el-row>
        </template>
      </el-form>
      <template #footer>
        <el-button @click="dataDialogVisible = false">暂不生成 (已为您保留数据)</el-button>
        <el-button type="primary" @click="generateReport">开始生成图表 🚀</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, nextTick, onMounted, watch } from 'vue'
import { ElMessage } from 'element-plus'
import { DataAnalysis, DataLine, PieChart, Money, EditPen, User, Download, Back } from '@element-plus/icons-vue'
import * as echarts from 'echarts'
import html2canvas from 'html2canvas'

// 核心状态机：onboarding -> home -> report
const currentStep = ref('onboarding')
const currentMode = ref('')
const dataDialogVisible = ref(false)
const dialogTitle = ref('')
const isExporting = ref(false)
const currentTime = ref('')

// 清空了默认名字，强制用户填写
const creatorInfo = reactive({ name: '', niche: '' })

// 数据持久化载体 (初始为空或基础值，供用户修改)
const formData = reactive({
  single: { views: 100000, fiveSecRate: 60, finishRate: 30, likes: 2000, comments: 500, shares: 1000, saves: 1500 },
  account: { femaleRatio: 70, youngRatio: 80, tier1Ratio: 60, minViews: 50000, maxViews: 200000 },
  commercial: { baseCpm: 30, targetViews: 100000, ctr: 1.5, aov: 99 }
})

// ================= 持久化逻辑 =================
onMounted(() => {
  const savedProfile = localStorage.getItem('vlog_creator')
  if (savedProfile) {
    Object.assign(creatorInfo, JSON.parse(savedProfile))
    // 如果已有名字，直接跳过迎新大厅，进入功能主页！极大提升体验
    if (creatorInfo.name) currentStep.value = 'home'
  }
  const savedData = localStorage.getItem('vlog_data')
  if (savedData) Object.assign(formData, JSON.parse(savedData))
})

watch([creatorInfo, formData], () => {
  localStorage.setItem('vlog_creator', JSON.stringify(creatorInfo))
  localStorage.setItem('vlog_data', JSON.stringify(formData))
}, { deep: true })

// ================= 导航流转逻辑 =================
const enterWorkspace = () => { currentStep.value = 'home'; ElMessage.success(`欢迎回来，${creatorInfo.name}！`) }

const openDataDialog = (mode) => {
  currentMode.value = mode
  dialogTitle.value = mode === 'single' ? '📊 录入单篇数据' : mode === 'account' ? '👥 录入大盘画像' : '💰 录入商业变现参数'
  dataDialogVisible.value = true
}

const generateReport = async () => {
  dataDialogVisible.value = false
  currentStep.value = 'report' // 切换到图表页
  const now = new Date()
  currentTime.value = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`
  
  await nextTick()
  if (currentMode.value === 'single') initSingleCharts()
  else if (currentMode.value === 'account') initAccountCharts()
  else if (currentMode.value === 'commercial') initCommercialCharts()
}

// 导出长图
const exportToImage = async () => {
  isExporting.value = true
  try {
    const canvas = await html2canvas(document.getElementById('report-capture-area'), { scale: 2, backgroundColor: '#f5f7fa' })
    const link = document.createElement('a')
    link.href = canvas.toDataURL('image/png')
    link.download = `${creatorInfo.name}_商业报告.png`
    link.click()
  } catch (error) { ElMessage.error('导出失败') } finally { isExporting.value = false }
}

// ================= ECharts 渲染逻辑 (针对你的要求进行了大升级) =================
const initSingleCharts = () => {
  const d = formData.single
  const retentionChart = echarts.init(document.getElementById('chart-retention'))
  retentionChart.setOption({
    tooltip: { trigger: 'axis', axisPointer: { type: 'cross', label: { backgroundColor: '#6a7985' } } },
    grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
    xAxis: { type: 'category', boundaryGap: false, data: ['0s(曝光)', '3s', '5s(决断)', '中段', '完播'] },
    yAxis: { type: 'value', max: 100 },
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

const initAccountCharts = () => {
  const d = formData.account
  
  // 1. 升级版客群图谱 (包含男/女环形图 + 年龄地域柱状图)
  const maleRatio = 100 - d.femaleRatio
  const audienceChart = echarts.init(document.getElementById('chart-audience'))
  audienceChart.setOption({
    tooltip: { trigger: 'item' },
    series: [
      {
        name: '性别分布', type: 'pie', radius: ['40%', '60%'], center: ['25%', '50%'],
        itemStyle: { borderRadius: 5, borderColor: '#fff', borderWidth: 2 },
        label: { show: true, position: 'center', formatter: '性别', fontSize: 14, fontWeight: 'bold' },
        data: [{ value: d.femaleRatio, name: '女性', itemStyle: {color: '#ff9f7f'} }, { value: maleRatio, name: '男性', itemStyle: {color: '#83bff6'} }]
      },
      {
        name: '高客单画像', type: 'pie', radius: ['40%', '60%'], center: ['75%', '50%'],
        itemStyle: { borderRadius: 5, borderColor: '#fff', borderWidth: 2 },
        label: { show: true, position: 'center', formatter: '客单\n属性', fontSize: 14, fontWeight: 'bold' },
        data: [{ value: d.tier1Ratio, name: '一二线城市', itemStyle: {color: '#fac858'} }, { value: d.youngRatio, name: '18-30岁青年', itemStyle: {color: '#91cc75'} }]
      }
    ]
  })

  // 2. 极其灵敏的 10 期稳定性图表 (自动生成中间的波动数据，打造真实感)
  const stChart = echarts.init(document.getElementById('chart-stability'))
  const range = d.maxViews - d.minViews
  // 模拟近 10 期的真实波动
  const mock10Data = [
    d.minViews + range*0.2, d.minViews + range*0.5, d.minViews + range*0.1, 
    d.minViews + range*0.8, d.minViews, d.maxViews, d.minViews + range*0.4, 
    d.minViews + range*0.6, d.minViews + range*0.3, d.minViews + range*0.7
  ].map(Math.floor)

  stChart.setOption({
    // 核心：加了 cross 准星，极度灵敏且顺滑
    tooltip: { trigger: 'axis', axisPointer: { type: 'cross', label: { backgroundColor: '#6a7985' } } },
    grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
    xAxis: { type: 'category', boundaryGap: false, data: ['10期前','9期前','8期前','7期前','6期前','5期前','4期前','3期前','2期前','上一期'] },
    yAxis: { type: 'value' },
    series: [{
      name: '单期播放量', type: 'line', smooth: true, symbolSize: 8,
      data: mock10Data, itemStyle: { color: '#67C23A' }, lineStyle: { width: 3 },
      markArea: { itemStyle: { color: 'rgba(103,194,58,0.1)' }, data: [[ { yAxis: d.minViews, name: '保底流量护城河 (即使数据差也有下限)' }, { yAxis: d.maxViews } ]] }
    }]
  })
  window.addEventListener('resize', () => { audienceChart.resize(); stChart.resize() })
}

const initCommercialCharts = () => {
  const d = formData.commercial
  const clk = Math.floor(d.targetViews * (d.ctr / 100))
  const ord = Math.floor(clk * 0.02)
  const gmv = ord * d.aov
  
  const roiChart = echarts.init(document.getElementById('chart-roi'))
  roiChart.setOption({ tooltip: { trigger: 'item' }, series: [{ type: 'funnel', sort: 'descending', label: { show: true, position: 'inside', formatter: '{b}\n{c}' }, data: [{ value: d.targetViews, name: '总曝光' }, { value: clk, name: '点击' }, { value: ord, name: '成交' }, { value: gmv, name: '创造 GMV (¥)' }] }] })
  
  const bq = Math.floor((d.targetViews / 1000) * d.baseCpm); const pq = Math.floor(gmv * 0.1)
  const priceChart = echarts.init(document.getElementById('chart-price'))
  priceChart.setOption({ tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' } }, xAxis: { type: 'category', data: ['底价', '溢价', '最终报价'] }, yAxis: { type: 'value' }, series: [{ type: 'bar', barWidth: '40%', data: [{ value: bq, itemStyle:{color:'#91cc75'}}, { value: pq, itemStyle:{color:'#fac858'}}, { value: bq+pq, itemStyle:{color:'#ee6666'}}], label: { show: true, position: 'top', formatter: '¥ {c}' } }] })
  
  window.addEventListener('resize', () => { roiChart.resize(); priceChart.resize() })
}
</script>

<style scoped>
.saas-container { min-height: 100vh; background: #f5f7fa; padding: 20px; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }

/* 迎新与主页样式 */
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

/* 报告页样式 */
.report-grid { max-width: 1200px; margin: 0 auto; background: #fff; padding: 40px; border-radius: 16px; box-shadow: 0 10px 40px rgba(0,0,0,0.03); }
.report-header { text-align: center; margin-bottom: 30px; position: relative; }
.back-btn { position: absolute; left: 0; top: 0; }
.report-header h2 { color: #2d3748; font-size: 2rem; margin: 15px 0; font-weight: 800; }
.mode-badge span { background-color: #2b6cb0; color: white; padding: 6px 15px; border-radius: 20px; font-size: 0.9rem; font-weight: bold; }
.export-btn { margin-top: 20px; font-weight: bold; }

.chart-card { border-radius: 12px; border: 1px solid #edf2f7; box-shadow: none; margin-bottom: 20px;}
.chart-box { height: 350px; width: 100%; }
.ai-insight-box { background: #fdf6ec; border-left: 4px solid #E6A23C; padding: 12px 15px; margin-bottom: 15px; border-radius: 4px; font-size: 0.95rem; color: #606266; line-height: 1.6; }

.report-footer { margin-top: 40px; padding-top: 20px; border-top: 1px dashed #e2e8f0; text-align: center; color: #a0aec0; font-size: 0.85rem; }
</style>