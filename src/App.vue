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
import { ref, computed, watch } from 'vue'

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

const deviceType = ref<'rmpp' | 'rmppm'>('rmpp')

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
            // const x = evaluateExpression(data[i], vars) + offsetX;
            // const y = evaluateExpression(data[i + 1], vars) + offsetY;
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
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Invalid JSON'
      templateData.value = null
    }
  },
  { immediate: true },
)
</script>
