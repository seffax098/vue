<template>
  <div class="palette-page">

    <!-- ПАНЕЛЬ УПРАВЛЕНИЯ -->
    <section class="controls">
      <!-- Базовые настройки -->
      <div class="control-group">
        <label>Количество цветов:</label>
        <select v-model.number="colorCount" class="select-input">
          <option v-for="option in colorCountOptions" :key="option" :value="option">
            {{ option }}
          </option>
        </select>
      </div>

      <div class="control-group">
        <label>Формат отображения:</label>
        <div class="format-toggle">
          <button
            class="format-button"
            :class="{ active: colorFormat === 'hex' }"
            @click="colorFormat = 'hex'"
          >
            HEX
          </button>
          <button
            class="format-button"
            :class="{ active: colorFormat === 'rgb' }"
            @click="colorFormat = 'rgb'"
          >
            RGB
          </button>
        </div>
      </div>

      <div class="control-group">
        <label>Фон превью:</label>
        <button class="toggle-bg" @click="isDarkPreview = !isDarkPreview">
          {{ isDarkPreview ? 'Тёмный фон' : 'Светлый фон' }}
        </button>
      </div>

      <!-- Продвинутая генерация -->
      <div class="control-group">
        <label>Базовый цвет:</label>
        <div class="base-color-row">
          <input type="color" v-model="baseColor" />
          <span class="base-color-text">{{ baseColor.toUpperCase() }}</span>
        </div>
      </div>

      <div class="control-group">
        <label>Тип палитры:</label>
        <select v-model="paletteType" class="select-input">
          <option value="random">Случайная</option>
          <option value="analogous">Аналогичная</option>
          <option value="monochrome">Монохромная</option>
          <option value="triad">Триада</option>
          <option value="complementary">Комплементарная</option>
        </select>
      </div>

      <div class="control-group">
        <label>Настроение:</label>
        <select v-model="mood" class="select-input">
          <option value="none">Без настроения</option>
          <option value="calm">Спокойная</option>
          <option value="energetic">Энергичная</option>
          <option value="professional">Профессиональная</option>
        </select>
      </div>

      <div class="control-group">
        <button class="generate-button" @click="generatePalette">
          🎲 Сгенерировать палитру
        </button>
      </div>
    </section>

    <!-- ОСНОВНАЯ ПАЛИТРА -->
    <section class="palette">
      <div
        v-for="(color, index) in colors"
        :key="color.id"
        class="color-card"
        :style="{ backgroundColor: color.hex }"
        @click="copyColor(color)"
      >
        <div class="color-header">
          <span class="color-index">#{{ index + 1 }}</span>

          <button
            class="lock-button"
            :class="{ locked: color.locked }"
            @click.stop="toggleLock(color)"
          >
            <span v-if="color.locked">🔒</span>
            <span v-else>🔓</span>
          </button>
        </div>

        <div class="color-value">
          {{ getColorText(color.hex) }}
        </div>

        <div class="color-hint">
          Клик для копирования
        </div>
      </div>
    </section>

    <!-- Уведомление о копировании -->
    <transition name="fade">
      <div v-if="copyMessage" class="copy-toast">
        {{ copyMessage }}
      </div>
    </transition>

    <!-- ПРЕВЬЮ ИНТЕРФЕЙСА -->
    <section
      class="preview"
      :class="{ dark: isDarkPreview }"
    >
      <h3>Превью интерфейса</h3>
      <p class="preview-text">
        Смотрите, как палитра выглядит в UI-компонентах.
      </p>

      <div class="preview-row">
        <button
          class="preview-button"
          :style="{ backgroundColor: colors[0]?.hex || '#3b82f6' }"
        >
          Основная кнопка
        </button>

        <button
          class="preview-button outline"
          :style="{
            color: colors[0]?.hex || '#3b82f6',
            borderColor: colors[0]?.hex || '#3b82f6'
          }"
        >
          Вторичная
        </button>
      </div>

      <div class="preview-card" :style="{ backgroundColor: colors[1]?.hex || '#e5e7eb' }">
        <h4 :style="{ color: getTextColorForBackground(colors[1]?.hex) }">
          Карточка контента
        </h4>
        <p :style="{ color: getTextColorForBackground(colors[1]?.hex, true) }">
          Текст для проверки контраста.
        </p>
      </div>

      <div class="preview-footer" :style="{ borderTopColor: colors[2]?.hex || '#9ca3af' }">
        <span :style="{ color: colors[2]?.hex || '#6b7280' }">
          Акцентный цвет интерфейса
        </span>
      </div>
    </section>

    <!-- ДОСТУПНОСТЬ / CONTRAST -->
    <section class="accessibility">
      <h3>Доступность и контраст (WCAG)</h3>

      <div v-if="contrastChecks.length === 0" class="empty-state">
        Нет достаточных данных для анализа.
      </div>

      <div v-else class="contrast-list">
        <div
          class="contrast-item"
          v-for="check in contrastChecks"
          :key="check.id"
        >
          <div class="contrast-label">
            {{ check.label }}
          </div>
          <div class="contrast-values">
            <span>Контраст: {{ check.ratio.toFixed(2) }} : 1</span>
            <span
              class="badge"
              :class="{
                passAAA: check.level === 'AAA',
                passAA: check.level === 'AA',
                fail: check.level === 'недостаточно'
              }"
            >
              {{ check.level }}
            </span>
          </div>
        </div>
      </div>
    </section>

    <!-- БИБЛИОТЕКА ПАЛИТР -->
    <section class="library">
      <div class="library-header">
        <h3>Библиотека палитр</h3>
        <button class="save-button" @click="saveCurrentPalette">
          💾 Сохранить текущую палитру
        </button>
      </div>

      <div class="library-controls">
        <input
          v-model="librarySearch"
          class="text-input"
          placeholder="Поиск по названию или тегам"
        />
        <label class="fav-filter">
          <input type="checkbox" v-model="showOnlyFavorites" />
          Только избранные
        </label>
      </div>

      <div v-if="filteredPalettes.length === 0" class="empty-state">
        Сохранённых палитр пока нет или ничего не найдено.
      </div>

      <div v-else class="library-list">
        <div
          v-for="palette in filteredPalettes"
          :key="palette.id"
          class="library-item"
        >
          <div class="library-info">
            <div class="library-title-row">
              <h4>{{ palette.name }}</h4>
              <button
                class="fav-button"
                :class="{ active: palette.favorite }"
                @click="toggleFavorite(palette)"
              >
                ★
              </button>
            </div>

            <div class="library-tags" v-if="palette.tags && palette.tags.length">
              <span
                class="tag"
                v-for="tag in palette.tags"
                :key="tag"
              >
                {{ tag }}
              </span>
            </div>

            <div class="library-swatches">
              <span
                v-for="(c, idx) in palette.colors"
                :key="idx"
                class="swatch"
                :style="{ backgroundColor: c.hex }"
              ></span>
            </div>
          </div>

          <div class="library-actions">
            <button class="small-button" @click="loadPalette(palette)">
              Загрузить
            </button>
            <button class="small-button danger" @click="deletePalette(palette.id)">
              Удалить
            </button>
          </div>
        </div>
      </div>
    </section>

    <!-- ЭКСПОРТ -->
    <section class="export-section">
      <h3>Экспорт палитры</h3>

      <div class="export-tabs">
        <button
          class="export-tab"
          :class="{ active: exportFormat === 'css' }"
          @click="exportFormat = 'css'"
        >
          CSS variables
        </button>
        <button
          class="export-tab"
          :class="{ active: exportFormat === 'scss' }"
          @click="exportFormat = 'scss'"
        >
          SCSS
        </button>
        <button
          class="export-tab"
          :class="{ active: exportFormat === 'tailwind' }"
          @click="exportFormat = 'tailwind'"
        >
          Tailwind config
        </button>
      </div>

      <textarea
        class="export-area"
        readonly
        :value="exportCode"
      ></textarea>

      <button class="copy-export" @click="copyExport">
        📋 Скопировать код
      </button>
    </section>
  </div>
</template>

<script>
import { ref, onMounted, watch, computed } from 'vue'

export default {
  name: 'ColorPaletteGenerator',

  setup() {
    const colorCountOptions = [3, 5, 7]
    const colorCount = ref(5)
    const colors = ref([]) // { id, hex, locked }
    const colorFormat = ref('hex') // 'hex' | 'rgb'
    const isDarkPreview = ref(false)

    const baseColor = ref('#FF6B6B')
    const paletteType = ref('random') // random, analogous, monochrome, triad, complementary
    const mood = ref('none') // none, calm, energetic, professional

    const copyMessage = ref('')
    let copyTimeoutId = null

    // Библиотека палитр
    const savedPalettes = ref([]) // {id, name, colors, tags, favorite}
    const librarySearch = ref('')
    const showOnlyFavorites = ref(false)

    const exportFormat = ref('css')

    // --- COLOR UTILS ---

    const randomHexColor = () => {
      const value = Math.floor(Math.random() * 0xffffff)
      return '#' + value.toString(16).padStart(6, '0').toUpperCase()
    }

    const hexToRgbObject = (hex) => {
      let clean = hex.replace('#', '')
      if (clean.length === 3) {
        clean = clean.split('').map(ch => ch + ch).join('')
      }
      const r = parseInt(clean.slice(0, 2), 16)
      const g = parseInt(clean.slice(2, 4), 16)
      const b = parseInt(clean.slice(4, 6), 16)
      return { r, g, b }
    }

    const hexToRgbString = (hex) => {
      const { r, g, b } = hexToRgbObject(hex)
      return `rgb(${r}, ${g}, ${b})`
    }

    const getLuminance = (hex) => {
      const { r, g, b } = hexToRgbObject(hex)
      const norm = [r, g, b].map(v => {
        const c = v / 255
        return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4)
      })
      return 0.2126 * norm[0] + 0.7152 * norm[1] + 0.0722 * norm[2]
    }

    const getTextColorForBackground = (hex, isSecondary = false) => {
      if (!hex) {
        return isSecondary ? '#9CA3AF' : '#111827'
      }
      const L = getLuminance(hex)
      if (L < 0.5) {
        return isSecondary ? '#E5E7EB' : '#F9FAFB'
      } else {
        return isSecondary ? '#4B5563' : '#111827'
      }
    }

    const getContrastRatio = (hex1, hex2) => {
      const L1 = getLuminance(hex1)
      const L2 = getLuminance(hex2)
      const light = Math.max(L1, L2)
      const dark = Math.min(L1, L2)
      return (light + 0.05) / (dark + 0.05)
    }

    const getWcagLevel = (ratio) => {
      if (ratio >= 7) return 'AAA'
      if (ratio >= 4.5) return 'AA'
      return 'недостаточно'
    }

    // --- HSL УТИЛИТЫ ДЛЯ ТИПОВ ПАЛИТР ---

    const hexToHsl = (hex) => {
      const { r, g, b } = hexToRgbObject(hex)
      const rNorm = r / 255
      const gNorm = g / 255
      const bNorm = b / 255
      const max = Math.max(rNorm, gNorm, bNorm)
      const min = Math.min(rNorm, gNorm, bNorm)
      let h, s
      const l = (max + min) / 2

      if (max === min) {
        h = s = 0
      } else {
        const d = max - min
        s = l > 0.5 ? d / (2 - max - min) : d / (max + min)
        switch (max) {
          case rNorm:
            h = (gNorm - bNorm) / d + (gNorm < bNorm ? 6 : 0)
            break
          case gNorm:
            h = (bNorm - rNorm) / d + 2
            break
          default:
            h = (rNorm - gNorm) / d + 4
        }
        h *= 60
      }

      return {
        h: Math.round(h),
        s: Math.round(s * 100),
        l: Math.round(l * 100)
      }
    }

    const hslToHex = (h, s, l) => {
      s /= 100
      l /= 100

      const c = (1 - Math.abs(2 * l - 1)) * s
      const x = c * (1 - Math.abs(((h / 60) % 2) - 1))
      const m = l - c / 2
      let r1, g1, b1

      if (0 <= h && h < 60) {
        r1 = c; g1 = x; b1 = 0
      } else if (60 <= h && h < 120) {
        r1 = x; g1 = c; b1 = 0
      } else if (120 <= h && h < 180) {
        r1 = 0; g1 = c; b1 = x
      } else if (180 <= h && h < 240) {
        r1 = 0; g1 = x; b1 = c
      } else if (240 <= h && h < 300) {
        r1 = x; g1 = 0; b1 = c
      } else {
        r1 = c; g1 = 0; b1 = x
      }

      const r = Math.round((r1 + m) * 255)
      const g = Math.round((g1 + m) * 255)
      const b = Math.round((b1 + m) * 255)

      return (
        '#' +
        r.toString(16).padStart(2, '0') +
        g.toString(16).padStart(2, '0') +
        b.toString(16).padStart(2, '0')
      ).toUpperCase()
    }

    const clamp = (val, min, max) => Math.min(max, Math.max(min, val))

    const generatePaletteByType = (baseHex, count) => {
      const baseHsl = hexToHsl(baseHex)
      const result = []

      if (paletteType.value === 'analogous') {
        const offsets = [-30, 0, 30, -60, 60]
        for (let i = 0; i < count; i++) {
          const off = offsets[i % offsets.length]
          const h = (baseHsl.h + off + 360) % 360
          result.push(hslToHex(h, baseHsl.s, baseHsl.l))
        }
      } else if (paletteType.value === 'monochrome') {
        for (let i = 0; i < count; i++) {
          const step = (i - (count - 1) / 2) * 10
          const l = clamp(baseHsl.l + step, 10, 90)
          result.push(hslToHex(baseHsl.h, baseHsl.s, l))
        }
      } else if (paletteType.value === 'triad') {
        const triads = [0, 120, 240]
        for (let i = 0; i < count; i++) {
          const h = (baseHsl.h + triads[i % triads.length]) % 360
          result.push(hslToHex(h, baseHsl.s, baseHsl.l))
        }
      } else if (paletteType.value === 'complementary') {
        const offsets = [0, 180, 30, 210, -30]
        for (let i = 0; i < count; i++) {
          const off = offsets[i % offsets.length]
          const h = (baseHsl.h + off + 360) % 360
          result.push(hslToHex(h, baseHsl.s, baseHsl.l))
        }
      } else {
        // fallback на случайную
        for (let i = 0; i < count; i++) {
          result.push(randomHexColor())
        }
      }

      return result
    }

    const generatePaletteByMood = (count) => {
      const colors = []

      for (let i = 0; i < count; i++) {
        let h, s, l

        if (mood.value === 'calm') {
          // холодные, пастельные
          h = Math.floor(Math.random() * 181) + 180 // 180–360 (синий/фиолетовый)
          s = Math.floor(Math.random() * 21) + 20   // 20–40
          l = Math.floor(Math.random() * 21) + 70   // 70–90
        } else if (mood.value === 'energetic') {
          // тёплые, насыщенные
          const ranges = [
            [0, 60],
            [300, 360]
          ]
          const range = ranges[Math.floor(Math.random() * ranges.length)]
          h = Math.floor(Math.random() * (range[1] - range[0])) + range[0]
          s = Math.floor(Math.random() * 31) + 70   // 70–100
          l = Math.floor(Math.random() * 31) + 40   // 40–70
        } else if (mood.value === 'professional') {
          // умеренная насыщенность, нейтрально-холодные
          h = Math.floor(Math.random() * 71) + 190  // 190–260
          s = Math.floor(Math.random() * 31) + 30   // 30–60
          l = Math.floor(Math.random() * 31) + 40   // 40–70
        } else {
          return null
        }

        colors.push(hslToHex(h, s, l))
      }

      return colors
    }

    // --- ЛОГИКА ПАЛИТРЫ ---

    const generatePalette = () => {
      const oldColors = colors.value
      const newColors = []

      // Сначала сгенерируем список новых HEX'ов для НЕЗАКРЕПЛЁННЫХ
      let generatedHexList = []

      if (mood.value !== 'none') {
        generatedHexList = generatePaletteByMood(colorCount.value)
      } else if (paletteType.value !== 'random') {
        generatedHexList = generatePaletteByType(baseColor.value, colorCount.value)
      } else {
        // полностью случайная палитра
        generatedHexList = []
        for (let i = 0; i < colorCount.value; i++) {
          generatedHexList.push(randomHexColor())
        }
      }

      let generatedIndex = 0

      for (let i = 0; i < colorCount.value; i++) {
        const existing = oldColors[i]

        if (existing && existing.locked) {
          newColors.push(existing)
        } else {
          const hex = generatedHexList[generatedIndex] || randomHexColor()
          generatedIndex++

          newColors.push({
            id: existing?.id || Date.now() + i,
            hex,
            locked: existing?.locked || false
          })
        }
      }

      colors.value = newColors
      saveToLocalStorage()
    }

    const toggleLock = (color) => {
      color.locked = !color.locked
      saveToLocalStorage()
    }

    const getColorText = (hex) => {
      return colorFormat.value === 'hex' ? hex : hexToRgbString(hex)
    }

    const copyColor = async (color) => {
      const text = getColorText(color.hex)
      try {
        await navigator.clipboard.writeText(text)
        copyMessage.value = `Цвет ${text} скопирован в буфер обмена`
      } catch (e) {
        copyMessage.value = 'Не удалось скопировать цвет'
      }

      if (copyTimeoutId) {
        clearTimeout(copyTimeoutId)
      }
      copyTimeoutId = setTimeout(() => {
        copyMessage.value = ''
      }, 1500)
    }

    // --- LOCALSTORAGE ---

    const STORAGE_KEY = 'vue-color-palette-v1'
    const STORAGE_KEY_LIBRARY = 'vue-color-palette-library-v1'

    const saveToLocalStorage = () => {
      const data = {
        colors: colors.value,
        colorCount: colorCount.value,
        colorFormat: colorFormat.value,
        isDarkPreview: isDarkPreview.value,
        baseColor: baseColor.value,
        paletteType: paletteType.value,
        mood: mood.value
      }
      localStorage.setItem(STORAGE_KEY, JSON.stringify(data))
    }

    const loadFromLocalStorage = () => {
      try {
        const raw = localStorage.getItem(STORAGE_KEY)
        if (raw) {
          const parsed = JSON.parse(raw)

          if (Array.isArray(parsed.colors) && parsed.colors.length > 0) {
            colors.value = parsed.colors
          }

          if (parsed.colorCount) colorCount.value = parsed.colorCount
          if (parsed.colorFormat) colorFormat.value = parsed.colorFormat
          if (typeof parsed.isDarkPreview === 'boolean') {
            isDarkPreview.value = parsed.isDarkPreview
          }
          if (parsed.baseColor) baseColor.value = parsed.baseColor
          if (parsed.paletteType) paletteType.value = parsed.paletteType
          if (parsed.mood) mood.value = parsed.mood
        } else {
          generatePalette()
        }
      } catch (e) {
        generatePalette()
      }

      try {
        const libRaw = localStorage.getItem(STORAGE_KEY_LIBRARY)
        if (libRaw) {
          const libParsed = JSON.parse(libRaw)
          if (Array.isArray(libParsed)) {
            savedPalettes.value = libParsed
          }
        }
      } catch {
        savedPalettes.value = []
      }
    }

    const saveLibraryToLocalStorage = () => {
      localStorage.setItem(STORAGE_KEY_LIBRARY, JSON.stringify(savedPalettes.value))
    }

    // --- ДОСТУПНОСТЬ ---

    const contrastChecks = computed(() => {
      const result = []
      if (colors.value.length === 0) return result

      const primary = colors.value[0].hex
      const secondary = colors.value[1]?.hex
      const bgLight = '#FFFFFF'
      const bgDark = '#111827'

      // с текущим фоном превью
      const currentBg = isDarkPreview.value ? bgDark : bgLight
      const ratioBg = getContrastRatio(primary, currentBg)
      result.push({
        id: 'primary-bg',
        label: `Основной цвет vs фон превью (${isDarkPreview.value ? 'тёмный' : 'светлый'})`,
        ratio: ratioBg,
        level: getWcagLevel(ratioBg)
      })

      // основной vs второй цвет
      if (secondary) {
        const ratio2 = getContrastRatio(primary, secondary)
        result.push({
          id: 'primary-secondary',
          label: 'Основной цвет vs второй цвет',
          ratio: ratio2,
          level: getWcagLevel(ratio2)
        })
      }

      return result
    })

    // --- БИБЛИОТЕКА ПАЛИТР ---

    const saveCurrentPalette = () => {
      if (colors.value.length === 0) return

      const defaultName = `Палитра от ${new Date().toLocaleString('ru-RU')}`
      const name = window.prompt('Название палитры:', defaultName)
      if (!name) return

      const tagsRaw = window.prompt('Теги (через запятую):', 'ui, web, brand')
      let tags = []
      if (tagsRaw) {
        tags = tagsRaw
          .split(',')
          .map(t => t.trim())
          .filter(Boolean)
      }

      const palette = {
        id: Date.now(),
        name: name.trim(),
        colors: colors.value.map(c => ({ hex: c.hex })),
        tags,
        favorite: false
      }

      savedPalettes.value.push(palette)
      saveLibraryToLocalStorage()
      copyMessage.value = 'Палитра сохранена в библиотеку'
      if (copyTimeoutId) clearTimeout(copyTimeoutId)
      copyTimeoutId = setTimeout(() => (copyMessage.value = ''), 1500)
    }

    const filteredPalettes = computed(() => {
      const q = librarySearch.value.toLowerCase()

      return savedPalettes.value.filter(p => {
        if (showOnlyFavorites.value && !p.favorite) return false

        if (!q) return true

        const inName = p.name.toLowerCase().includes(q)
        const inTags = (p.tags || []).some(t => t.toLowerCase().includes(q))
        return inName || inTags
      })
    })

    const loadPalette = (palette) => {
      colors.value = palette.colors.map((c, idx) => ({
        id: Date.now() + idx,
        hex: c.hex,
        locked: false
      }))
      saveToLocalStorage()
    }

    const deletePalette = (id) => {
      savedPalettes.value = savedPalettes.value.filter(p => p.id !== id)
      saveLibraryToLocalStorage()
    }

    const toggleFavorite = (palette) => {
      palette.favorite = !palette.favorite
      saveLibraryToLocalStorage()
    }

    // --- ЭКСПОРТ ---

    const exportCode = computed(() => {
      if (colors.value.length === 0) return ''

      if (exportFormat.value === 'css') {
        const lines = [':root {']
        colors.value.forEach((c, i) => {
          lines.push(`  --color-${i + 1}: ${c.hex};`)
        })
        lines.push('}')
        return lines.join('\n')
      }

      if (exportFormat.value === 'scss') {
        const lines = []
        colors.value.forEach((c, i) => {
          lines.push(`$color-${i + 1}: ${c.hex};`)
        })
        return lines.join('\n')
      }

      // tailwind
      const lines = [
        'module.exports = {',
        '  theme: {',
        '    extend: {',
        '      colors: {'
      ]
      colors.value.forEach((c, i) => {
        lines.push(`        palette${i + 1}: '${c.hex}',`)
      })
      lines.push('      }')
      lines.push('    }')
      lines.push('  }')
      lines.push('}')
      return lines.join('\n')
    })

    const copyExport = async () => {
      try {
        await navigator.clipboard.writeText(exportCode.value)
        copyMessage.value = 'Код палитры скопирован в буфер обмена'
      } catch {
        copyMessage.value = 'Не удалось скопировать код'
      }
      if (copyTimeoutId) clearTimeout(copyTimeoutId)
      copyTimeoutId = setTimeout(() => (copyMessage.value = ''), 1500)
    }

    // --- WATCHERS & LIFECYCLE ---

    watch(colorCount, () => {
      generatePalette()
    })

    watch([colorFormat, isDarkPreview, baseColor, paletteType, mood], () => {
      saveToLocalStorage()
    })

    watch(savedPalettes, () => {
      saveLibraryToLocalStorage()
    }, { deep: true })

    onMounted(() => {
      loadFromLocalStorage()
      if (colors.value.length === 0) {
        generatePalette()
      }
    })

    return {
      colorCountOptions,
      colorCount,
      colors,
      colorFormat,
      isDarkPreview,
      baseColor,
      paletteType,
      mood,

      copyMessage,

      savedPalettes,
      librarySearch,
      showOnlyFavorites,
      filteredPalettes,

      exportFormat,
      exportCode,

      generatePalette,
      toggleLock,
      getColorText,
      copyColor,
      getTextColorForBackground,

      saveCurrentPalette,
      loadPalette,
      deletePalette,
      toggleFavorite,

      contrastChecks,

      copyExport
    }
  }
}
</script>

<style scoped>
.palette-page {
  max-width: 980px;
  margin: 20px auto 40px;
  padding: 20px;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(15, 23, 42, 0.08);
}

.subtitle {
  margin-top: 4px;
  margin-bottom: 20px;
  color: #6b7280;
}

/* ПАНЕЛЬ УПРАВЛЕНИЯ */
.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  padding: 16px;
  border-radius: 10px;
  background-color: #f9fafb;
  margin-bottom: 20px;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
  min-width: 170px;
}

label {
  font-size: 0.9rem;
  font-weight: 600;
  color: #374151;
}

.select-input {
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid #d1d5db;
  font-size: 0.9rem;
}

.format-toggle {
  display: flex;
  gap: 8px;
}

.format-button {
  padding: 6px 12px;
  border-radius: 999px;
  border: 1px solid #d1d5db;
  background-color: white;
  font-size: 0.85rem;
  cursor: pointer;
}

.format-button.active {
  background-color: #4f46e5;
  color: white;
  border-color: #4f46e5;
}

.toggle-bg {
  padding: 6px 12px;
  border-radius: 999px;
  border: 1px solid #d1d5db;
  background-color: white;
  cursor: pointer;
  font-size: 0.85rem;
}

.generate-button {
  padding: 8px 16px;
  border-radius: 999px;
  border: none;
  background: linear-gradient(135deg, #6366f1, #ec4899);
  color: white;
  font-weight: 600;
  cursor: pointer;
  white-space: nowrap;
}

.base-color-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.base-color-text {
  font-size: 0.85rem;
  color: #4b5563;
}

/* ПАЛИТРА */
.palette {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
  gap: 12px;
  margin-bottom: 24px;
}

.color-card {
  position: relative;
  border-radius: 10px;
  padding: 10px;
  color: white;
  cursor: pointer;
  min-height: 120px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  box-shadow: 0 8px 18px rgba(15, 23, 42, 0.25);
}

.color-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.color-index {
  font-size: 0.85rem;
  opacity: 0.9;
}

.lock-button {
  width: 28px;
  height: 28px;
  border-radius: 999px;
  border: none;
  background-color: rgba(15, 23, 42, 0.35);
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.lock-button.locked {
  background-color: rgba(22, 163, 74, 0.8);
}

.color-value {
  font-weight: 700;
  font-size: 0.95rem;
}

.color-hint {
  font-size: 0.8rem;
  opacity: 0.9;
}

/* TOAST */
.copy-toast {
  position: fixed;
  left: 50%;
  bottom: 24px;
  transform: translateX(-50%);
  background-color: #111827;
  color: white;
  padding: 10px 16px;
  border-radius: 999px;
  font-size: 0.9rem;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.6);
  z-index: 50;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s, transform 0.2s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translate(-50%, 10px);
}

/* PREVIEW */
.preview {
  padding: 18px;
  border-radius: 10px;
  background-color: #f3f4f6;
  border: 1px solid #e5e7eb;
  margin-bottom: 20px;
}

.preview.dark {
  background-color: #0f172a;
  border-color: #1e293b;
}

.preview h3 {
  margin-bottom: 4px;
}

.preview-text {
  font-size: 0.9rem;
  color: #6b7280;
  margin-bottom: 16px;
}

.preview.dark .preview-text {
  color: #9ca3af;
}

.preview-row {
  display: flex;
  gap: 10px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.preview-button {
  padding: 8px 16px;
  border-radius: 999px;
  border: none;
  color: white;
  font-size: 0.9rem;
  cursor: pointer;
}

.preview-button.outline {
  background-color: transparent;
  border-width: 1px;
  border-style: solid;
}

.preview.dark .preview-button.outline {
  background-color: rgba(15, 23, 42, 0.4);
}

.preview-card {
  padding: 14px;
  border-radius: 10px;
  margin-bottom: 12px;
}

.preview-footer {
  padding-top: 10px;
  border-top-width: 2px;
  border-top-style: solid;
}

/* ACCESSIBILITY */
.accessibility {
  margin-top: 10px;
  margin-bottom: 20px;
  padding: 16px;
  background-color: #f9fafb;
  border-radius: 10px;
  border: 1px solid #e5e7eb;
}

.contrast-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: 8px;
}

.contrast-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 10px;
  border-radius: 8px;
  background-color: #ffffff;
  border: 1px solid #e5e7eb;
}

.contrast-values {
  display: flex;
  align-items: center;
  gap: 8px;
}

.badge {
  padding: 2px 8px;
  border-radius: 999px;
  font-size: 0.75rem;
}

.badge.passAAA {
  background-color: #16a34a;
  color: white;
}

.badge.passAA {
  background-color: #facc15;
  color: #111827;
}

.badge.fail {
  background-color: #ef4444;
  color: white;
}

/* LIBRARY */
.library {
  margin-top: 10px;
  margin-bottom: 20px;
  padding: 16px;
  border-radius: 10px;
  background-color: #f9fafb;
  border: 1px solid #e5e7eb;
}

.library-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.save-button {
  padding: 6px 12px;
  border-radius: 999px;
  border: none;
  background-color: #22c55e;
  color: white;
  font-size: 0.85rem;
  cursor: pointer;
}

.library-controls {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 12px;
  margin-bottom: 10px;
}

.text-input {
  flex: 1;
  min-width: 180px;
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid #d1d5db;
  font-size: 0.9rem;
}

.fav-filter {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.85rem;
  color: #4b5563;
}

.library-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.library-item {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  padding: 10px;
  background-color: #ffffff;
  border-radius: 8px;
  border: 1px solid #e5e7eb;
}

.library-info {
  flex: 1;
}

.library-title-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.library-title-row h4 {
  margin: 0;
}

.fav-button {
  width: 28px;
  height: 28px;
  border-radius: 999px;
  border: none;
  background-color: #e5e7eb;
  cursor: pointer;
}

.fav-button.active {
  background-color: #facc15;
}

.library-tags {
  margin-top: 4px;
  margin-bottom: 6px;
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.tag {
  font-size: 0.75rem;
  padding: 2px 6px;
  border-radius: 999px;
  background-color: #e5e7eb;
}

.library-swatches {
  display: flex;
  gap: 4px;
}

.swatch {
  width: 18px;
  height: 18px;
  border-radius: 4px;
}

.library-actions {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.small-button {
  padding: 4px 8px;
  border-radius: 6px;
  border: none;
  background-color: #4f46e5;
  color: white;
  font-size: 0.8rem;
  cursor: pointer;
}

.small-button.danger {
  background-color: #ef4444;
}

/* EXPORT */
.export-section {
  margin-top: 10px;
  padding: 16px;
  border-radius: 10px;
  background-color: #f9fafb;
  border: 1px solid #e5e7eb;
}

.export-tabs {
  display: flex;
  gap: 8px;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

.export-tab {
  padding: 6px 12px;
  border-radius: 999px;
  border: 1px solid #d1d5db;
  background-color: white;
  font-size: 0.85rem;
  cursor: pointer;
}

.export-tab.active {
  background-color: #0f766e;
  color: white;
  border-color: #0f766e;
}

.export-area {
  width: 100%;
  min-height: 140px;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  padding: 8px;
  font-family: Consolas, monospace;
  font-size: 0.85rem;
  margin-bottom: 10px;
}

.copy-export {
  padding: 6px 12px;
  border-radius: 999px;
  border: none;
  background-color: #2563eb;
  color: white;
  font-size: 0.85rem;
  cursor: pointer;
}

/* Общее */
.empty-state {
  text-align: center;
  padding: 12px;
  color: #6b7280;
  font-size: 0.9rem;
}
</style>
