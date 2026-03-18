<script setup lang="ts">
import { ref } from 'vue'

// 组件类型定义
interface ChartComponent {
  id: string
  name: string
  type: 'bar' | 'line' | 'pie' | 'stat' | 'text'
  x: number
  y: number
  width: number
  height: number
  data?: any
}

// 大屏组件列表
const components = ref<ChartComponent[]>([
  { id: '1', name: '销售统计', type: 'stat', x: 0, y: 0, width: 3, height: 2, data: { title: '总销售额', value: '¥1,234,567', trend: '+12.5%' } },
  { id: '2', name: '柱状图', type: 'bar', x: 3, y: 0, width: 4, height: 4, data: { title: '月度销售', categories: ['1月', '2月', '3月', '4月'], values: [120, 200, 150, 280] } },
  { id: '3', name: '折线图', type: 'line', x: 7, y: 0, width: 5, height: 4, data: { title: '访问趋势', categories: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'], values: [820, 932, 901, 934, 1290, 1330, 1320] } },
  { id: '4', name: '饼图', type: 'pie', x: 0, y: 2, width: 3, height: 2, data: { title: '用户分布', values: [{ name: '华东', value: 35 }, { name: '华南', value: 25 }, { name: '华北', value: 20 }, { name: '其他', value: 20 }] } },
])

// 组件库
const componentLibrary = [
  { type: 'bar', name: '柱状图', icon: '📊' },
  { type: 'line', name: '折线图', icon: '📈' },
  { type: 'pie', name: '饼图', icon: '🥧' },
  { type: 'stat', name: '统计卡', icon: '💰' },
  { type: 'text', name: '文本', icon: '📝' },
]

// 添加组件
const addComponent = (type: string) => {
  const libItem = componentLibrary.find(c => c.type === type)
  const newComponent: ChartComponent = {
    id: Date.now().toString(),
    name: libItem?.name || '新组件',
    type: type as ChartComponent['type'],
    x: 0,
    y: components.value.length * 2,
    width: 4,
    height: 3,
    data: getDefaultData(type)
  }
  components.value.push(newComponent)
}

const getDefaultData = (type: string): any => {
  switch (type) {
    case 'bar': return { title: '柱状图', categories: ['A', 'B', 'C', 'D'], values: [100, 200, 150, 180] }
    case 'line': return { title: '折线图', categories: ['周一', '周二', '周三', '周四'], values: [100, 150, 120, 180] }
    case 'pie': return { title: '饼图', values: [{ name: 'A', value: 30 }, { name: 'B', value: 20 }] }
    case 'stat': return { title: '统计', value: '0', trend: '0%' }
    case 'text': return { content: '双击编辑文本' }
    default: return {}
  }
}

// 删除组件
const removeComponent = (id: string) => {
  components.value = components.value.filter(c => c.id !== id)
}

// 选中组件
const selectedId = ref<string | null>(null)
const selectComponent = (id: string) => {
  selectedId.value = id
}

// 选中颜色
const selectedColor = ref('#1890ff')
const colors = ['#1890ff', '#52c41a', '#faad14', '#f5222d', '#722ed1', '#13c2c2']

// 计算最大值用于图表比例
const getMaxValue = (values: number[]): number => {
  return Math.max(...values, 1)
}

// 计算饼图总值
const getPieTotal = (values: any[]): number => {
  return values?.reduce((a: number, b: any) => a + (b.value || 0), 0) || 1
}
</script>

<template>
  <div class="editor-container">
    <!-- 左侧组件库 -->
    <div class="component-library">
      <h3>组件库</h3>
      <div class="library-items">
        <div 
          v-for="item in componentLibrary" 
          :key="item.type"
          class="library-item"
          @click="addComponent(item.type)"
        >
          <span class="icon">{{ item.icon }}</span>
          <span>{{ item.name }}</span>
        </div>
      </div>
      
      <h3>配色方案</h3>
      <div class="color-picker">
        <div 
          v-for="color in colors" 
          :key="color"
          class="color-item"
          :style="{ backgroundColor: color }"
          :class="{ active: selectedColor === color }"
          @click="selectedColor = color"
        ></div>
      </div>
    </div>

    <!-- 中间画布 -->
    <div class="canvas-area">
      <div class="canvas">
        <div 
          v-for="comp in components" 
          :key="comp.id"
          class="chart-component"
          :class="{ selected: selectedId === comp.id }"
          :style="{
            gridColumn: `span ${comp.width}`,
            gridRow: `span ${comp.height}`,
            borderColor: selectedColor
          }"
          @click="selectComponent(comp.id)"
        >
          <!-- 统计卡 -->
          <div v-if="comp.type === 'stat'" class="stat-card" :style="{ background: `linear-gradient(135deg, ${selectedColor}22, ${selectedColor}44)` }">
            <div class="stat-title">{{ comp.data?.title }}</div>
            <div class="stat-value">{{ comp.data?.value }}</div>
            <div class="stat-trend" :class="{ positive: comp.data?.trend?.includes('+') }">
              {{ comp.data?.trend }}
            </div>
          </div>

          <!-- 柱状图 -->
          <div v-else-if="comp.type === 'bar'" class="bar-chart">
            <div class="chart-title">{{ comp.data?.title }}</div>
            <div class="bar-container">
              <div 
                v-for="(val, idx) in comp.data?.values" 
                :key="idx"
                class="bar-item"
                :style="{ 
                  height: `${(val / getMaxValue(comp.data?.values || [])) * 100}%`,
                  backgroundColor: selectedColor
                }"
              >
                <span class="bar-value">{{ val }}</span>
              </div>
            </div>
            <div class="chart-labels">
              <span v-for="(cat, idx) in comp.data?.categories" :key="idx">{{ cat }}</span>
            </div>
          </div>

          <!-- 折线图 -->
          <div v-else-if="comp.type === 'line'" class="line-chart">
            <div class="chart-title">{{ comp.data?.title }}</div>
            <svg class="line-svg" viewBox="0 0 300 150">
              <polyline
                :points="comp.data?.values?.map((v: number, i: number, arr: number[]) => 
                  `${(i / ((arr.length || 1) - 1)) * 280 + 10},${150 - (v / getMaxValue(comp.data?.values || [])) * 120 - 10}`
                ).join(' ')"
                fill="none"
                :stroke="selectedColor"
                stroke-width="3"
              />
              <circle 
                v-for="(v, i) in comp.data?.values" 
                :key="i"
                :cx="(i / ((comp.data.values?.length || 1) - 1)) * 280 + 10"
                :cy="150 - (v / getMaxValue(comp.data?.values || [])) * 120 - 10"
                r="4"
                :fill="selectedColor"
              />
            </svg>
            <div class="chart-labels">
              <span v-for="(cat, idx) in comp.data?.categories" :key="idx">{{ cat }}</span>
            </div>
          </div>

          <!-- 饼图 -->
          <div v-else-if="comp.type === 'pie'" class="pie-chart">
            <div class="chart-title">{{ comp.data?.title }}</div>
            <svg viewBox="0 0 200 200">
              <circle 
                v-for="(slice, idx) in comp.data?.values" 
                :key="idx"
                cx="100" cy="100" r="80"
                fill="transparent"
                :stroke="colors[idx % colors.length]"
                stroke-width="40"
                :stroke-dasharray="`${(slice.value / getPieTotal(comp.data?.values || [])) * 502} 502`"
                :stroke-dashoffset="`-${comp.data?.values?.slice(0, idx).reduce((a: number, b: any) => a + (b.value / getPieTotal(comp.data?.values || [])) * 502, 0)}`"
                transform="rotate(-90 100 100)"
              />
            </svg>
            <div class="pie-legend">
              <div v-for="(slice, idx) in comp.data?.values" :key="idx" class="legend-item">
                <span class="legend-color" :style="{ backgroundColor: colors[idx % colors.length] }"></span>
                {{ slice.name }}: {{ slice.value }}
              </div>
            </div>
          </div>

          <!-- 删除按钮 -->
          <button class="delete-btn" @click.stop="removeComponent(comp.id)">×</button>
        </div>
      </div>
    </div>

    <!-- 右侧属性面板 -->
    <div class="property-panel" v-if="selectedId">
      <h3>属性设置</h3>
      <div v-for="comp in components.filter(c => c.id === selectedId)" :key="comp.id">
        <div class="prop-item">
          <label>组件名称</label>
          <input v-model="comp.name" />
        </div>
        <div class="prop-item">
          <label>宽度 (格)</label>
          <input type="number" v-model="comp.width" min="1" max="12" />
        </div>
        <div class="prop-item">
          <label>高度 (格)</label>
          <input type="number" v-model="comp.height" min="1" max="8" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.editor-container {
  display: flex;
  height: 100vh;
  background: #1a1a2e;
  color: #fff;
}

.component-library {
  width: 200px;
  background: #16213e;
  padding: 20px;
  border-right: 1px solid #0f3460;
}

.component-library h3 {
  margin-bottom: 15px;
  font-size: 14px;
  color: #e94560;
}

.library-items {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-bottom: 30px;
}

.library-item {
  background: #0f3460;
  padding: 15px 10px;
  border-radius: 8px;
  cursor: pointer;
  text-align: center;
  transition: all 0.3s;
}

.library-item:hover {
  background: #e94560;
  transform: translateY(-2px);
}

.library-item .icon {
  display: block;
  font-size: 24px;
  margin-bottom: 5px;
}

.color-picker {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.color-item {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  cursor: pointer;
  border: 2px solid transparent;
}

.color-item.active {
  border-color: #fff;
  transform: scale(1.1);
}

.canvas-area {
  flex: 1;
  padding: 20px;
  overflow: auto;
}

.canvas {
  display: grid;
  grid-template-columns: repeat(12, 100px);
  grid-auto-rows: 100px;
  gap: 15px;
  min-height: 600px;
  background: 
    linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px);
  background-size: 100px 100px;
  padding: 20px;
  border-radius: 12px;
}

.chart-component {
  background: #0f3460;
  border-radius: 12px;
  padding: 15px;
  cursor: pointer;
  position: relative;
  border: 2px solid transparent;
  transition: all 0.3s;
  min-width: 100px;
  min-height: 100px;
}

.chart-component:hover {
  box-shadow: 0 8px 25px rgba(0,0,0,0.3);
}

.chart-component.selected {
  border-width: 2px;
}

.delete-btn {
  position: absolute;
  top: 5px;
  right: 5px;
  width: 24px;
  height: 24px;
  border: none;
  background: #f5222d;
  color: #fff;
  border-radius: 50%;
  cursor: pointer;
  opacity: 0;
  transition: opacity 0.3s;
}

.chart-component:hover .delete-btn {
  opacity: 1;
}

.stat-card {
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 8px;
}

.stat-title {
  font-size: 14px;
  opacity: 0.8;
}

.stat-value {
  font-size: 32px;
  font-weight: bold;
  margin: 10px 0;
}

.stat-trend {
  font-size: 14px;
  color: #f5222d;
}

.stat-trend.positive {
  color: #52c41a;
}

.chart-title {
  font-size: 16px;
  margin-bottom: 15px;
  text-align: center;
}

.bar-chart, .line-chart {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.bar-container {
  flex: 1;
  display: flex;
  align-items: flex-end;
  justify-content: space-around;
  padding: 0 10px;
}

.bar-item {
  width: 30px;
  min-height: 10px;
  border-radius: 4px 4px 0 0;
  position: relative;
}

.bar-value {
  position: absolute;
  top: -20px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 12px;
}

.chart-labels {
  display: flex;
  justify-content: space-around;
  padding: 5px 10px;
  font-size: 12px;
  opacity: 0.7;
}

.line-svg {
  flex: 1;
  width: 100%;
}

.pie-chart {
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pie-chart svg {
  width: 120px;
  height: 120px;
}

.pie-legend {
  font-size: 12px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 5px;
  margin: 3px 0;
}

.legend-color {
  width: 12px;
  height: 12px;
  border-radius: 2px;
}

.property-panel {
  width: 250px;
  background: #16213e;
  padding: 20px;
  border-left: 1px solid #0f3460;
}

.property-panel h3 {
  margin-bottom: 20px;
  color: #e94560;
}

.prop-item {
  margin-bottom: 15px;
}

.prop-item label {
  display: block;
  margin-bottom: 5px;
  font-size: 12px;
  opacity: 0.7;
}

.prop-item input {
  width: 100%;
  padding: 8px;
  background: #0f3460;
  border: 1px solid #1a1a2e;
  color: #fff;
  border-radius: 4px;
}

.prop-item input:focus {
  outline: none;
  border-color: #e94560;
}
</style>
