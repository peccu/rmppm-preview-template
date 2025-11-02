<template>
  <div class="min-h-screen bg-gray-100 p-8">
    <div class="max-w-7xl mx-auto">
      <h1 class="text-3xl font-bold text-gray-800 mb-6">reMarkable Template Preview</h1>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <!-- Editor Panel -->
        <div class="bg-white rounded-lg shadow-lg p-6">
          <h2 class="text-xl font-semibold text-gray-700 mb-4">Template JSON</h2>

          <!-- Device Selection -->
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2"> Device Type </label>
            <select
              v-model="deviceType"
              class="w-full p-2 border-2 border-gray-300 rounded-lg focus:border-blue-500 focus:outline-none"
            >
              <option value="rmpp">reMarkable Paper Pro (rmpp)</option>
              <option value="rmppm">reMarkable Paper Pro Move (rmppm)</option>
            </select>
          </div>

          <!-- Constants UI -->
          <div v-if="templateData" class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">Constants</label>
            <div
              v-for="(constObj, idx) in constantsEdit"
              :key="idx"
              class="flex items-center gap-2 mb-2"
            >
              <input v-model="constObj.key" class="w-32 p-1 border rounded" placeholder="Key" />
              <input v-model="constObj.value" class="w-40 p-1 border rounded" placeholder="Value" />
              <button @click="removeConstant(idx)" class="text-red-500 hover:underline">
                削除
              </button>
            </div>
            <button
              @click="addConstant"
              class="px-2 py-1 bg-blue-100 rounded text-blue-700 text-xs hover:bg-blue-200"
            >
              + 定数追加
            </button>
          </div>

          <!-- Items UI -->
          <div v-if="templateData" class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">Items</label>
            <div
              v-for="(item, idx) in itemsEdit"
              :key="idx"
              class="border p-2 rounded mb-2 bg-gray-50"
            >
              <div class="flex gap-2 items-center">
                <input v-model="item.type" class="w-24 p-1 border rounded" placeholder="Type" />
                <input v-model="item.id" class="w-24 p-1 border rounded" placeholder="ID" />
                <button @click="removeItem(idx)" class="text-red-500 hover:underline text-xs">
                  削除
                </button>
              </div>
              <div class="mt-2 pl-2 text-xs text-gray-500">
                <template v-if="item.type === 'text'">
                  <input
                    v-model="item.text"
                    class="w-full p-1 border rounded mb-1"
                    placeholder="Text"
                  />
                  <input
                    v-model="item.fontSize"
                    class="w-20 p-1 border rounded"
                    placeholder="Font Size"
                    type="number"
                  />
                  <div>
                    x: <input v-model="item.position.x" class="w-16 border rounded p-1" /> y:
                    <input v-model="item.position.y" class="w-16 border rounded p-1" />
                  </div>
                </template>
                <template v-if="item.type === 'group'">
                  <div>
                    <span class="font-bold">BoundingBox:</span>
                    x: <input v-model="item.boundingBox.x" class="w-16 border rounded p-1" /> y:
                    <input v-model="item.boundingBox.y" class="w-16 border rounded p-1" /> w:
                    <input v-model="item.boundingBox.width" class="w-16 border rounded p-1" /> h:
                    <input v-model="item.boundingBox.height" class="w-16 border rounded p-1" />
                  </div>
                  <div>
                    <span class="font-bold">Repeat:</span>
                    rows: <input v-model="item.repeat.rows" class="w-16 border rounded p-1" /> cols:
                    <input v-model="item.repeat.columns" class="w-16 border rounded p-1" />
                  </div>
                  <div>
                    <span class="font-bold">Children:</span>
                    <div
                      v-for="(child, cidx) in item.children"
                      :key="cidx"
                      class="ml-2 flex gap-2 items-center"
                    >
                      <input
                        v-model="child.type"
                        class="w-16 border rounded p-1"
                        placeholder="Type"
                      />
                      <button
                        @click="removeChild(idx, cidx)"
                        class="text-red-400 hover:underline text-xs"
                      >
                        子削除
                      </button>
                    </div>
                    <button
                      @click="addChild(idx)"
                      class="text-xs px-2 py-1 rounded bg-gray-200 mt-1"
                    >
                      + 子追加
                    </button>
                  </div>
                </template>
                <template v-if="item.type === 'path'">
                  <div>
                    <span class="font-bold">Data:</span>
                    <input
                      v-model="item.dataString"
                      class="w-full p-1 border rounded"
                      placeholder='例: ["M", 0, 0, "L", 10, 10]'
                    />
                  </div>
                </template>
              </div>
            </div>
            <button
              @click="addItem"
              class="px-2 py-1 bg-blue-100 rounded text-blue-700 text-xs hover:bg-blue-200"
            >
              + アイテム追加
            </button>
          </div>

          <textarea
            v-model="templateJson"
            class="w-full h-96 font-mono text-sm p-4 border-2 border-gray-300 rounded-lg focus:border-blue-500 focus:outline-none"
            placeholder="Paste your .template JSON here..."
          ></textarea>

          <div v-if="error" class="mt-4 p-4 bg-red-50 border-2 border-red-200 rounded-lg">
            <p class="text-red-700 font-semibold">Error:</p>
            <p class="text-red-600 text-sm mt-1">{{ error }}</p>
          </div>

          <div
            v-if="templateData && !error"
            class="mt-4 p-4 bg-blue-50 border-2 border-blue-200 rounded-lg"
          >
            <p class="text-blue-700 font-semibold">{{ templateData.name }}</p>
            <p class="text-blue-600 text-sm">Author: {{ templateData.author }}</p>
            <p class="text-blue-600 text-sm">Orientation: {{ templateData.orientation }}</p>
            <p class="text-blue-600 text-sm">Device: {{ deviceType.toUpperCase() }}</p>
          </div>
        </div>

        <!-- Preview Panel -->
        <div class="bg-white rounded-lg shadow-lg p-6">
          <h2 class="text-xl font-semibold text-gray-700 mb-4">Preview</h2>

          <div class="flex justify-center items-start overflow-auto">
            <div
              class="border-2 border-gray-300 bg-white relative flex-shrink-0"
              :style="canvasStyle"
            >
              <svg
                :width="canvasWidth"
                :height="canvasHeight"
                :viewBox="`0 0 ${canvasWidth} ${canvasHeight}`"
                class="absolute inset-0"
              >
                <g v-for="(item, index) in renderedItems" :key="index">
                  <template v-if="item.type === 'path'">
                    <path :d="item.d" stroke="#000000" stroke-width="1.5" fill="none" />
                  </template>
                  <template v-else-if="item.type === 'text'">
                    <text
                      :x="item.x"
                      :y="item.y"
                      :font-size="item.fontSize"
                      fill="#000000"
                      font-family="sans-serif"
                    >
                      {{ item.text }}
                    </text>
                  </template>
                </g>
              </svg>
            </div>
          </div>

          <div class="mt-4 text-sm text-gray-600">
            <p>Canvas: {{ canvasWidth }} × {{ canvasHeight }}px</p>
            <p>Scale: {{ scale.toFixed(2) }}x</p>
            <p>Device: {{ deviceDimensions?.name ?? '' }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, nextTick } from 'vue'

// --- INTERFACES ---
interface TemplateData {
  name: string
  author: string
  templateVersion: string
  formatVersion: number
  categories: string[]
  orientation: 'portrait' | 'landscape'
  constants?: Array<Record<string, number | string>>
  items: TemplateItem[]
}

interface TemplateItem {
  id?: string
  type: string
  boundingBox?: BoundingBox
  repeat?: RepeatConfig
  children?: TemplateItem[]
  data?: Array<string | number>
  text?: string
  fontSize?: number
  position?: {
    x: number | string
    y: number | string
  }
  // For UI, keep a string of data for 'path'
  dataString?: string
}

interface BoundingBox {
  x: number | string
  y: number | string
  width: number | string
  height: number | string
}

interface RepeatConfig {
  rows: number | string
  columns?: number | string
}

interface RenderedItem {
  type: string
  d?: string
  text?: string
  x?: number
  y?: number
  fontSize?: number
}

interface DeviceDimensions {
  name: string
  portraitWidth: number
  portraitHeight: number
  landscapeWidth: number
  landscapeHeight: number
}

const DEVICES: Record<string, DeviceDimensions> = {
  rmpp: {
    name: 'reMarkable Paper Pro',
    portraitWidth: 1620,
    portraitHeight: 2160,
    landscapeWidth: 2160,
    landscapeHeight: 1620,
  },
  rmppm: {
    name: 'reMarkable Paper Pro Move',
    portraitWidth: 954,
    portraitHeight: 1696,
    landscapeWidth: 1696,
    landscapeHeight: 954,
  },
}

// ---- STATE ----
const deviceType = ref<'rmpp' | 'rmppm'>('rmpp')

// テンプレート JSON & 編集用
const templateJson = ref(`{
              "name": "Grid 5mm",
              "author": "Custom",
              "templateVersion": "1.0.0",
              "formatVersion": 1,
              "categories": ["Grids"],
              "orientation": "portrait",
              "constants": [
                { "gridSize": 45 },
                { "magicOffset": -22 },
                { "xpos": "templateWidth / 2 - templateHeight / 2 + magicOffset" }
              ],
              "items": [
                {
                  "id": "hlines",
                  "type": "group",
                  "boundingBox": { "x": 0, "y": 0, "width": "templateWidth", "height": "gridSize" },
                  "repeat": { "rows": "infinite", "columns": "infinite" },
                  "children": [
                    {
                      "id": "hline",
                      "type": "path",
                      "data": [ "M", 0, 0, "L", "parentWidth", 0 ]
                    }
                  ]
                },
                {
                  "id": "vlines",
                  "type": "group",
                  "boundingBox": { "x": "xpos", "y": 0, "width": "gridSize", "height": "templateHeight" },
                  "repeat": { "rows": "infinite", "columns": "infinite" },
                  "children": [
                    {
                      "id": "vline",
                      "type": "path",
                      "data": [ "M", 0, 0, "L", 0, "parentHeight" ]
                    }
                  ]
                }
              ]
            }`)

const templateData = ref<TemplateData | null>(null)
const error = ref<string>('')

// UI用の編集データ
const constantsEdit = ref<{ key: string; value: string }[]>([])
const itemsEdit = ref<any[]>([]) // TemplateItem[] but with extra UI fields (dataString)

// ---- DEVICE DIMENSIONS ----
const deviceDimensions = computed(() => DEVICES[deviceType.value] ?? undefined)

const canvasWidth = computed(() => {
  if (!deviceDimensions.value) return 0
  if (!templateData.value) return deviceDimensions.value.portraitWidth
  return templateData.value.orientation === 'landscape'
    ? deviceDimensions.value.landscapeWidth
    : deviceDimensions.value.portraitWidth
})

const canvasHeight = computed(() => {
  if (!deviceDimensions.value) return 0
  if (!templateData.value) return deviceDimensions.value.portraitHeight
  return templateData.value.orientation === 'landscape'
    ? deviceDimensions.value.landscapeHeight
    : deviceDimensions.value.portraitHeight
})

const scale = computed(() => {
  const maxWidth = 500
  const maxHeight = 700
  return Math.min(maxWidth / canvasWidth.value, maxHeight / canvasHeight.value)
})

const canvasStyle = computed(() => ({
  width: `${canvasWidth.value * scale.value}px`,
  height: `${canvasHeight.value * scale.value}px`,
}))

const renderedItems = computed<RenderedItem[]>(() => {
  if (!templateData.value) return []
  if (!deviceDimensions.value) return []

  try {
    const constants: Record<string, number> = {
      templateWidth: canvasWidth.value,
      templateHeight: canvasHeight.value,
    }

    // Parse constants
    if (templateData.value.constants) {
      for (const constObj of templateData.value.constants) {
        for (const [key, value] of Object.entries(constObj)) {
          if (typeof value === 'number') {
            constants[key] = value
          } else if (typeof value === 'string') {
            constants[key] = evaluateExpression(value, constants)
          }
        }
      }
    }

    const items: RenderedItem[] = []

    // Render items
    for (const item of templateData.value.items) {
      renderItem(item, items, constants, constants)
    }

    return items
  } catch (e) {
    console.error('Rendering error:', e)
    return []
  }
})

// ---- UI -> JSON変換 & 編集 ----
function updateTemplateFromUI() {
  if (!templateData.value) return
  const td = { ...templateData.value }

  // Constants: {key, value}[] -> Array<{[key: value}>
  td.constants = constantsEdit.value
    .filter((c) => c.key)
    .map((c) => {
      const valNum = Number(c.value)
      return {
        [c.key]: isNaN(valNum) || c.value.trim() === '' ? c.value : valNum,
      }
    })

  // Items: itemsEdit -> TemplateItem[]
  td.items = itemsEdit.value.map((item: any) => {
    const newItem: any = { ...item }
    // For path, parse dataString to array
    if (item.type === 'path' && item.dataString) {
      try {
        newItem.data = JSON.parse(item.dataString)
      } catch {
        newItem.data = []
      }
    }
    // For group, children cleanup
    if (item.type === 'group' && Array.isArray(item.children)) {
      newItem.children = item.children.map((child: any) => {
        const ch = { ...child }
        if (ch.type === 'path' && ch.dataString) {
          try {
            ch.data = JSON.parse(ch.dataString)
          } catch {
            ch.data = []
          }
        }
        return ch
      })
    }
    return newItem
  })

  // JSON反映
  templateJson.value = JSON.stringify(td, null, 2)
}

// --- UI操作関数 ---
function addConstant() {
  constantsEdit.value.push({ key: '', value: '' })
}
function removeConstant(idx: number) {
  constantsEdit.value.splice(idx, 1)
  updateTemplateFromUI()
}

function addItem() {
  itemsEdit.value.push({
    type: 'text',
    id: '',
    text: '',
    fontSize: 16,
    position: { x: 0, y: 0 },
    boundingBox: { x: 0, y: 0, width: 100, height: 100 },
    repeat: { rows: 1, columns: 1 },
    children: [],
    dataString: '',
  })
}
function removeItem(idx: number) {
  itemsEdit.value.splice(idx, 1)
  updateTemplateFromUI()
}

function addChild(parentIdx: number) {
  itemsEdit.value[parentIdx].children.push({
    type: 'path',
    dataString: '',
  })
}
function removeChild(parentIdx: number, childIdx: number) {
  itemsEdit.value[parentIdx].children.splice(childIdx, 1)
  updateTemplateFromUI()
}

// --- Watchers for UI編集値の反映 ---
watch(constantsEdit, updateTemplateFromUI, { deep: true })
watch(itemsEdit, updateTemplateFromUI, { deep: true })

// テキストエリア直編集時のパース
watch(
  templateJson,
  (newVal) => {
    try {
      if (!newVal.trim()) {
        templateData.value = null
        error.value = ''
        return
      }
      const parsed = JSON.parse(newVal) as TemplateData
      templateData.value = parsed
      error.value = ''

      // UI編集用データを再構築
      constantsEdit.value = []
      if (parsed.constants) {
        for (const constObj of parsed.constants) {
          const key = Object.keys(constObj)[0]
          const value = constObj[key]
          constantsEdit.value.push({ key, value: String(value) })
        }
      }
      itemsEdit.value = []
      if (parsed.items) {
        parsed.items.forEach((item: any) => {
          const newItem = { ...item }
          if (item.type === 'path' && Array.isArray(item.data)) {
            newItem.dataString = JSON.stringify(item.data)
          }
          if (item.type === 'group' && Array.isArray(item.children)) {
            newItem.children = item.children.map((child: any) => {
              const ch = { ...child }
              if (ch.type === 'path' && Array.isArray(ch.data)) {
                ch.dataString = JSON.stringify(ch.data)
              }
              return ch
            })
          }
          itemsEdit.value.push(newItem)
        })
      }
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Invalid JSON'
      templateData.value = null
    }
  },
  { immediate: true },
)

// --- SVG描画ロジック ---
function evaluateExpression(expr: string | number, vars: Record<string, number>): number {
  if (typeof expr === 'number') return expr

  let evaluated = expr

  // Replace variables with their values
  for (const [key, value] of Object.entries(vars)) {
    // Use word boundary to match whole variables only
    const regex = new RegExp(`\\b${key}\\b`, 'g')
    evaluated = evaluated.replace(regex, String(value))
  }

  try {
    // eslint-disable-next-line no-eval
    const result = eval(evaluated)
    return typeof result === 'number' && !isNaN(result) ? result : 0
  } catch (e) {
    console.error(`Failed to evaluate: ${expr} -> ${evaluated}`, e)
    return 0
  }
}

function renderItem(
  item: TemplateItem,
  items: RenderedItem[],
  constants: Record<string, number>,
  parentVars: Record<string, number>,
  offsetX: number = 0,
  offsetY: number = 0,
): void {
  if (item.type === 'text' && item.text && item.position) {
    const x = evaluateExpression(item.position.x, constants) + offsetX
    const y = evaluateExpression(item.position.y, constants) + offsetY
    items.push({
      type: 'text',
      text: item.text,
      x,
      y,
      fontSize: item.fontSize || 16,
    })
  } else if (item.type === 'group' && item.boundingBox && item.repeat && item.children) {
    const x = evaluateExpression(item.boundingBox.x, constants)
    const y = evaluateExpression(item.boundingBox.y, constants)
    const width = evaluateExpression(item.boundingBox.width, constants)
    const height = evaluateExpression(item.boundingBox.height, constants)

    let rows: number
    if (item.repeat.rows === 'infinite') {
      rows = Math.ceil(canvasHeight.value / height) + 1
    } else if (item.repeat.rows === 'down') {
      rows = Math.floor((canvasHeight.value - y) / height)
    } else {
      rows = Number(item.repeat.rows)
    }

    const cols =
      item.repeat.columns === 'infinite'
        ? Math.ceil(canvasWidth.value / width) + 1
        : item.repeat.columns
          ? Number(item.repeat.columns)
          : 1

    for (let row = 0; row < rows; row++) {
      for (let col = 0; col < cols; col++) {
        const childOffsetX = x + col * width
        const childOffsetY = y + row * height

        const childVars = {
          ...constants,
          parentWidth: width,
          parentHeight: height,
        }

        for (const child of item.children) {
          renderItem(child, items, constants, childVars, childOffsetX, childOffsetY)
        }
      }
    }
  } else if (item.type === 'path' && item.data) {
    const d = buildPathData(item.data, parentVars, offsetX, offsetY)
    items.push({ type: 'path', d })
  }
}

function buildPathData(
  data: Array<string | number>,
  vars: Record<string, number>,
  offsetX: number = 0,
  offsetY: number = 0,
): string {
  const parts: string[] = []
  let i = 0

  while (i < data.length) {
    const item = data[i]

    if (typeof item === 'string') {
      const command = item.toUpperCase()

      if (['M', 'L', 'Z', 'H', 'V', 'C', 'S', 'Q', 'T', 'A'].includes(command)) {
        parts.push(item)
        i++

        // Handle different command types
        if (command === 'Z') {
          // Z has no parameters
          continue
        } else if (command === 'M' || command === 'L') {
          // M and L take x, y coordinates
          if (i + 1 < data.length) {
            const xValue = data[i]
            const yValue = data[i + 1]

            // ガード節：undefined チェック
            if (xValue === undefined || yValue === undefined) {
              console.warn(`Invalid coordinates for ${command} command`)
              i += 2
              continue
            }

            const x = evaluateExpression(xValue, vars) + offsetX
            const y = evaluateExpression(yValue, vars) + offsetY
            parts.push(String(x))
            parts.push(String(y))
            i += 2
          }
        } else {
          // For other commands, just evaluate the next value
          if (i < data.length) {
            const value = data[i]
            if (value === undefined) {
              i++
              continue
            }

            const val = evaluateExpression(value, vars)
            parts.push(String(val))
            i++
          }
        }
      } else {
        // Not a command, treat as expression
        const val = evaluateExpression(item, vars)
        parts.push(String(val))
        i++
      }
    } else {
      // Numeric value
      parts.push(String(item))
      i++
    }
  }

  return parts.join(' ')
}
</script>
