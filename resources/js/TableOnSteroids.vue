<template>
  <div>
    <Table v-model="data" />
    <label class="form-group-label mt-2 mb-1"
      >Paste your CSV into this field:</label
    >
    <textarea class="input-text w-full mb-2" v-model="csv" rows="4"></textarea>
    <button class="btn btn-default mb-2 mr-1" @click="csvToJavascript">
      Process CSV Data
    </button>
    <button class="btn btn-default mb-2 mr-1" @click="handleClearInput">
      Clear Input
    </button>
    <button class="btn btn-default mb-2" @click="handleClearTable">
      Clear Table
    </button>
    <span
      >delimiter:
      <input
        class="input-text inline w-8"
        :disabled="useTab"
        type="text"
        placeholder=","
        v-model="customDelimiter"
      />
      <span class="mr-1">or use tab</span
      ><input type="checkbox" v-model="useTab" />
    </span>
    <div class="text-red" v-if="error">{{ error }}</div>
  </div>
</template>
<script>
import parse from "csv-parse/lib/sync"

const Table = Vue.component("table-fieldtype")

export default {
  mixins: [Fieldtype],
  data() {
    return {
      csv: "",
      data: [],
      error: "",
      customDelimiter: "",
      useTab: false,
    }
  },
  components: {
    Table,
  },
  computed: {
    delimiter() {
      if (this.useTab) {
        return "\t"
      }
      if (this.customDelimiter === "") {
        return ","
      }
      return this.customDelimiter
    },
  },
  watch: {
    // safe guard if value prop changes from outside
    value: {
      deep: true,
      handler(data) {
        this.data = data
      },
    },
    data: {
      deep: true,
      handler(data) {
        this.update(data)
      },
    },
  },
  created() {
    this.data = this.value
  },
  methods: {
    handleClearTable() {
      this.data = []
    },
    handleClearInput() {
      this.csv = ""
    },
    csvToJavascript() {
      // parse entries from pasted csv
      this.error = ""
      try {
        const entries = parse(this.csv, {
          columns: false,
          skip_empty_lines: true,
          delimiter: this.delimiter,
        })
        var result = []
        entries.forEach((entry) => {
          const line = {
            cells: [],
          }
          for (const [key, value] of Object.entries(entry)) {
            line.cells.push(value)
          }
          result.push(line)
        })
        this.data = result
      } catch (e) {
        this.error = e.message
      }
    },
  },
}
</script>
