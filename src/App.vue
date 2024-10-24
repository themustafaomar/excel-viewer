<script setup lang="ts">
import { ref, shallowRef, useTemplateRef } from 'vue'
import { read, utils, WorkBook } from 'xlsx'
import Table from './components/Table.vue'

const url = useTemplateRef('url')
const workbook = shallowRef<WorkBook>()
const sheetNames = shallowRef<WorkBook['SheetNames']>([])
const activeSheetName = shallowRef('')
const sheetData = ref<string[][]>([])

const fetchExcel = async () => {
  const response = await fetch(url.value?.value as string)
  const arrayBuffer = await response.arrayBuffer()
  const data = new Uint8Array(arrayBuffer)

  workbook.value = read(data, {
    type: 'array',
  })
  sheetNames.value = workbook.value.SheetNames

  setSheet(sheetNames.value[0])
}

const setSheet = (sheetId: string) => {
  sheetData.value = utils.sheet_to_json(workbook.value!.Sheets[sheetId], {
    header: 1,
  })
  activeSheetName.value = sheetId
}
</script>

<template>
  <div class="container px-28 mx-auto">
    <div class="flex items-center justify-center mt-10">
      <div class="relative w-[30rem]">
        <input ref="url" type="text" aria-describedby="helper-text-explanation" class="bg-gray-50 border border-gray-300 text-gray-600 dark:text-gray-400 text-sm border-e-0 focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:focus:ring-blue-500 dark:focus:border-blue-500 rounded-s-lg" value="http://localhost:5173/FoodImports.xlsx" />
      </div>
      <button
        @click="fetchExcel"
        class="flex-shrink-0 z-10 inline-flex items-center py-2.5 px-4 text-sm font-medium text-center text-white bg-blue-700 dark:bg-blue-600 border hover:bg-blue-800 dark:hover:bg-blue-700 rounded-e-lg border-blue-700 dark:border-blue-600 hover:border-blue-700 dark:hover:border-blue-700 focus:ring-4 focus:outline-none focus:ring-blue-300 dark:focus:ring-blue-800"
      >
        Preview Excel
      </button>
    </div>

    <div class="flex items-center justify-center flex-wrap mt-6">
      <template v-for="sheetName in sheetNames" :key="sheetName">
        <button
          @click="setSheet(sheetName)"
          class="bg-gray-200 rounded-lg min-w-16 text-center m-1.5 px-3 py-2"
          :class="{ 'bg-blue-600 text-white': sheetName === activeSheetName }"
        >
          {{ sheetName }}
        </button>
      </template>
    </div>

    <Table :data="sheetData" />
  </div>
</template>
