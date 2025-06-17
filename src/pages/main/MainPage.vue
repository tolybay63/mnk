<template>
  <div class="q-ma-md row">
    <q-input
      v-if="isDate"
      dense
      v-model="dte"
      :model-value="dte"
      type="date"
      outlined
      @update:model-value="dteChange"
    >
      <q-tooltip> Алғашқы күн </q-tooltip>
    </q-input>
    <q-btn
      :disabled="file"
      v-if="isGenerate"
      dense
      icon="auto_graph"
      no-caps
      class="q-ml-lg"
      style="height: 30px"
      color="purple"
      @click="fnFillTab()"
    >
      <q-tooltip> Деректерді кездейсоқ толтыру </q-tooltip>
    </q-btn>

    <q-btn v-if="!fromFile" class="q-ml-lg" style="height: 30px" dense icon="download" no-caps color="purple">
      <q-menu>
        <q-list dense style="min-width: 100px">
          <q-item clickable v-close-popup>
            <q-item-section @click="fnLoad(1)">1-Нұсқа</q-item-section>
          </q-item>
          <q-separator />
          <q-item clickable v-close-popup>
            <q-item-section @click="fnLoad(2)">2-Нұсқа</q-item-section>
          </q-item>
          <q-separator />
          <q-item clickable v-close-popup>
            <q-item-section @click="fnLoad(3)">3-Нұсқа</q-item-section>
          </q-item>
        </q-list>
      </q-menu>
      <q-tooltip anchor="top middle" self="top middle"> Берілген деректерді жүктеу... </q-tooltip>
    </q-btn>

    <q-btn
      dense
      icon="manage_search"
      no-caps
      class="q-ml-lg"
      style="height: 30px"
      color="purple"
      @click="fnPredict()"
      :disable="!isDone"
    >
      <q-tooltip> Болжау... </q-tooltip>
    </q-btn>

    <!--                            -->
<!--

    <div class="col-4">
      <q-input
        input-class="q-ml-lg primary"
        autofocus
        dense
        type="file"
        model-value="file"
        :clearable="true"
        @update:model-value="updFile"
        @clear="clrFile"
        accept=".csv"
      >
      </q-input>
    </div>

-->

    <q-uploader
      v-if="fromFile"
      ref="uploader"
      label="Файлды жүктеу"
      color="purple"
      square
      flat
      bordered
      class="q-ml-lg"
      style="max-width: 300px"
      v-model="file"
      max-files="1"
      :factory="updFile"
      @removed="clrFile"

      auto-upload


    />

  </div>
  <hr />

  <q-splitter
    v-model="splitterModel"
    :model-value="splitterModel"
    :limits="[0, 100]"
    before-class="overflow-hidden q-mr-sm"
    after-class="overflow-hidden q-ml-sm"
    separator-class="bg-red"
    class="bg-amber-1"
    style="height: calc(100vh - 200px); width: 100%"
  >
    <template v-slot:before>
      <q-table
        style="height: calc(100vh - 200px); width: 100%"
        color="primary"
        card-class="bg-amber-1"
        row-key="dte"
        dense
        :columns="cols"
        :rows="rows"
        :wrap-cells="true"
        :table-colspan="4"
        table-header-class="text-bold text-white bg-blue-grey-13"
        separator="cell"
        :loading="loading"
        :rows-per-page-options="[0]"
      >
      </q-table>
      <q-inner-loading :showing="loading"></q-inner-loading>
    </template>

    <template v-slot:after>
      <div class="bg-green-1 scroll">
        <div class="bg-blue-grey-13 text-white text-center" style="font-size: 1.2em; height: 32px">
          График
        </div>

        <q-card v-if="rows.length > 0">
          <q-card-section>
            <div class="text-h6 text-blue text-weight-bolder fit">
              <div class="row">
                <q-list
                  class="text-weight-regular row"
                  style="font-size: medium"
                  v-for="(si, index) in ser"
                  :key="index"
                >
                  <q-icon size="sm" name="square" :color="si.clr" />
                  <div :style="fnColor(index)">{{ si.label }}</div>
                </q-list>
              </div>
            </div>
          </q-card-section>
          <q-card-section class="q-pa-none echarts" style="height: 600px">
            <IEcharts :option="options" :resizable="true" />
          </q-card-section>
          <q-card-section></q-card-section>
        </q-card>
      </div>
    </template>
  </q-splitter>
</template>

<script>
import { date } from 'quasar'
import IEcharts from 'vue3-echarts-v3'
import UpdatePredict from 'pages/main/UpdatePredict.vue'
import { ref } from 'vue'

export default {
  name: 'MainPage',
  components: { IEcharts },

  data() {
    return {
      file: ref(null),
      //
      isGenerate: false,
      isDate: false,
      fromFile: false,
      a0: 0,
      a1: 0,
      a2: 0,
      a3: 0,
      a4: 0,
      isDone: false,
      numberObservations: 30,
      loading: false,
      splitterModel: 30,
      dte: date.formatDate(Date.now(), 'YYYY-MM-DD'),
      cols: [],
      rows: [],
      ser: [],
      options: {
        tooltip: {
          trigger: 'axis',
          showContent: true,
        },

        xAxis: {
          type: 'category',
          data: [],
        },
        yAxis: {},

        series: [
          {
            data: [],
            type: 'line',
            label: {
              show: true,
              position: 'top',
            },
          },
        ],
      },
      //
    }
  },

  methods: {
    clrFile() {
      console.log('clrFile')
      this.file = ref(null)
      this.rows = []
    },

    fnLoaded(data) {
      this.rows = []
      data = data.replaceAll(',', '.')
      let rows = data.split('\r')
      //console.log(rows.length)
      for (let i = 0; i < rows.length - 1; i++) {
        if (!rows[i+1].includes(";")) continue
        let cols = rows[i + 1].split(';')
        this.rows.push({
          dte: i + 1,
          fact: Number(cols[0]),
          rate1: Number(cols[1]),
          rate2: Number(cols[2]),
          rate3: Number(cols[3]),
          rate4: Number(cols[4]),
          clc: 0,
        })
      }
      //
      this.calcKoefModel()
      //
      this.showGraphics()
    },

    updFile(val) {
      if (val !== null) {
        this.file = val[0]
        let reader = new FileReader()
        reader.readAsText(this.file, 'windows-1251')

        setTimeout(() => {
          this.fnLoaded(reader.result)
        }, 200)
      }
    },

    //

    fnKoef() {
      let mat = []
      //1
      let a0 = 54,
        a1 = 0,
        a2 = 0,
        a3 = 0,
        a4 = 0,
        b = 0
      this.rows.forEach((r) => {
        a1 = a1 + r.rate1
        a2 = a2 + r.rate2
        a3 = a3 + r.rate3
        a4 = a4 + r.rate4
        b = b + r.fact
      })
      mat.push([a0, a1, a2, a3, a4, b])
      //2
      a0 = 0
      a1 = 0
      a2 = 0
      a3 = 0
      a4 = 0
      b = 0
      this.rows.forEach((r) => {
        a0 = a0 + r.rate1
        a1 = a1 + r.rate1 * r.rate1
        a2 = a2 + r.rate1 * r.rate2
        a3 = a3 + r.rate1 * r.rate3
        a4 = a4 + r.rate1 * r.rate4
        b = b + r.fact * r.rate1
      })
      mat.push([a0, a1, a2, a3, a4, b])
      //3
      a0 = 0
      a1 = 0
      a2 = 0
      a3 = 0
      a4 = 0
      b = 0
      this.rows.forEach((r) => {
        a0 = a0 + r.rate2
        a1 = a1 + r.rate1 * r.rate2
        a2 = a2 + r.rate2 * r.rate2
        a3 = a3 + r.rate2 * r.rate3
        a4 = a4 + r.rate2 * r.rate4
        b = b + r.fact * r.rate2
      })
      mat.push([a0, a1, a2, a3, a4, b])
      //4
      a0 = 0
      a1 = 0
      a2 = 0
      a3 = 0
      a4 = 0
      b = 0
      this.rows.forEach((r) => {
        a0 = a0 + r.rate3
        a1 = a1 + r.rate1 * r.rate3
        a2 = a2 + r.rate2 * r.rate3
        a3 = a3 + r.rate3 * r.rate3
        a4 = a4 + r.rate3 * r.rate4
        b = b + r.fact * r.rate3
      })
      mat.push([a0, a1, a2, a3, a4, b])
      //5
      a0 = 0
      a1 = 0
      a2 = 0
      a3 = 0
      a4 = 0
      b = 0
      this.rows.forEach((r) => {
        a0 = a0 + r.rate4
        a1 = a1 + r.rate1 * r.rate4
        a2 = a2 + r.rate2 * r.rate4
        a3 = a3 + r.rate3 * r.rate4
        a4 = a4 + r.rate4 * r.rate4
        b = b + r.fact * r.rate4
      })
      mat.push([a0, a1, a2, a3, a4, b])
      return mat
    },

    gauss(a) {
      const n = a.length
      const m = a[0].length
      const cpy = []
      for (let i = 0; i < n; i++) {
        let t = []
        for (let j = 0; j < m; j++) {
          t.push(a[i][j])
        }
        cpy.push(t)
      }

      for (let k = 0; k < n; k++) {
        for (let i = 0; i < m; i++) cpy[k][i] = cpy[k][i] / a[k][k]
        for (let i = k + 1; i < n; i++) {
          let kfc = cpy[i][k] / cpy[k][k]
          for (let j = 0; j < m; j++) cpy[i][j] = cpy[i][j] - cpy[k][j] * kfc
        }
        for (let i = 0; i < n; i++) for (let j = 0; j < m; j++) a[i][j] = cpy[i][j]
      }

      for (let k = n - 1; k > -1; k--) {
        for (let i = n; i > -1; i--) cpy[k][i] = cpy[k][i] / a[k][k]
        for (let i = k - 1; i > -1; i--) {
          let kfc = cpy[i][k] / cpy[k][k]
          for (let j = n; j > -1; j--) cpy[i][j] = cpy[i][j] - cpy[k][j] * kfc
        }
      }

      let solition = []
      for (let i = 0; i < n; i++) solition.push(cpy[i][n])
      return solition
    },

    calcKoefModel: function () {
      //todo

      let mat = this.fnKoef()
      // [a0,a1,a2,a3,a4, b]
      let sol = this.gauss(mat)

      this.a0 = sol[0]
      this.a1 = sol[1]
      this.a2 = sol[2]
      this.a3 = sol[3]
      this.a4 = sol[4]

      this.rows.forEach((r) => {
        r.clc =
          this.a0 + this.a1 * r.rate1 + this.a2 * r.rate2 + this.a3 * r.rate3 + this.a4 * r.rate4
        r.clc = r.clc.toFixed(2)
      })
    },

    fnPredict() {
      this.$q
        .dialog({
          component: UpdatePredict,
          componentProps: {
            a0: this.a0.toFixed(3),
            a1: this.a1.toFixed(3),
            a2: this.a2.toFixed(3),
            a3: this.a3.toFixed(3),
            a4: this.a4.toFixed(3),
          },
        })
        .onOk(() => {})
    },

    getRandomInt(min, max) {
      min = Math.ceil(min)
      max = Math.floor(max)
      return Math.floor(Math.random() * (max - min + 1)) + min
    },

    getRandomFloat(min, max) {
      return Number((Math.random() * (max - min) + min).toFixed(2))
    },

    fnFillTab() {
      try {
        this.loading = true
        this.isDone = false
        this.rows = []
        //this.$refs.uploader
        this.clrFile()

        let dt = this.dte.substring(0, 8) + '01'
        for (let i = 0; i < this.numberObservations; i++) {
          let d = date.addToDate(dt, { month: i })
          let ind = i + 1
          if (this.isDate) ind = date.formatDate(d, 'DD.MM.YYYY')
          this.rows.push({
            dte: ind,
            fact: this.getRandomFloat(16, 22),
            rate1: this.getRandomFloat(12, 37),
            rate2: this.getRandomFloat(44, 131),
            rate3: this.getRandomInt(8, 42),
            rate4: this.getRandomFloat(3, 9),
            clc: 0,
          })
        }
        //
        this.calcKoefModel()
        //
        this.showGraphics()
      } finally {
        this.loading = false
      }
    },

    showGraphics() {
      console.info('rows', this.rows)
      this.ser = []
      this.options.xAxis.data = []
      this.options.series = []
      this.ser.push({ label: 'Берілген', clr: 'green', data: [] })
      this.ser.push({ label: 'Есептелген', clr: 'red', data: [] })

      this.rows.forEach((r) => {
        this.ser[0].data.push(r.fact)
        this.ser[1].data.push(r.clc)
        this.options.xAxis.data.push(r.dte)
      })

      setTimeout(() => {
        for (let j = 0; j < 2; j++) {
          this.options.series.push({
            type: 'line',
            data: this.ser[j].data,
            label: {
              show: true,
              position: 'top',
            },
            color: this.ser[j].clr,
          })
        }
        this.isDone = true
      }, 100)
    },

    dteChange(v) {
      this.dte = v
    },

    fnColor(index) {
      return `color: ${this.ser[index].clr}`
    },

    fnLoad: function (ind) {
      let yFact, yRate1, yRate2, yRate3, yRate4

      let data
      if (ind === 1) data = 'data1.js'
      else if (ind === 2) data = 'data2.js'
      else if (ind === 3) data = 'data3.js'

      import(/* @vite-ignore */ `../../assets/${data}`).then((f) => {
        yFact = f['yFact']
        yRate1 = f['yRate1']
        yRate2 = f['yRate2']
        yRate3 = f['yRate3']
        yRate4 = f['yRate4']

        try {
          this.loading = true
          this.isDone = false
          this.rows = []
          this.ser = []
          this.options.xAxis.data = []
          this.options.series = []

          this.ser.push({ label: 'Факт', clr: 'green', data: [] })
          this.ser.push({ label: 'Расчет', clr: 'red', data: [] })
          //
          let n = Math.min(yFact.length, yRate1.length, yRate2.length, yRate3.length, yRate4.length)

          if (n === 0) {
            //this.$q.notify({ message: 'Деректер жоқ', icon: 'error', position: 'top-right' })
            return
          }

          let dt = this.dte.substring(0, 8) + '01'

          for (let i = 0; i < n; i++) {
            let d = date.addToDate(dt, { month: i })
            let ind = i + 1
            if (this.isDate) ind = date.formatDate(d, 'DD.MM.YYYY')

            this.options.xAxis.data.push(ind)

            this.rows.push({
              dte: ind,
              fact: yFact[i],
              rate1: yRate1[i],
              rate2: yRate2[i],
              rate3: yRate3[i],
              rate4: yRate4[i],
              clc: 0,
            })
          }
          //
          this.calcKoefModel()
          //
          this.showGraphics()
        } finally {
          this.loading = false
        }
      })
    },
  },

  created() {
    this.isDate = process.env.DATE_SHOW
    this.isGenerate = process.env.GENERATE_SHOW
    this.fromFile = process.env.FROM_FILE

    console.log('isDate', this.isDate)
    console.log('isGenerate', this.isGenerate)

    if (this.isDate) this.options.xAxis.axisLabel = { rotate: 90 }

    this.cols = [
      {
        name: 'dte',
        label: this.isDate ? 'Дата' : '№=',
        field: 'dte',
        align: 'left',
        classes: 'bg-blue-grey-1',
        headerStyle: 'font-size: 1.2em; width: 10%',
      },
      {
        name: 'fact',
        label: 'Егін түсімділігі, ц/га',
        field: 'fact',
        align: 'right',
        classes: 'bg-green-2',
        headerStyle: 'font-size: 1.2em; width: 15%',
      },
      {
        name: 'rate1',
        label: 'Өндіріс шарттары, 1 га/тенге',
        field: 'rate1',
        align: 'right',
        classes: 'bg-blue-grey-1',
        headerStyle: 'font-size: 1.2em; width: 15%',
      },
      {
        name: 'rate2',
        label: 'Еңбек шығыны, 1 га адам/сағ',
        field: 'rate2',
        align: 'right',
        classes: 'bg-blue-grey-1',
        headerStyle: 'font-size: 1.2em; width: 15%',
      },
      {
        name: 'rate3',
        label: 'Тыңайтқыш, 1 га/кг',
        field: 'rate3',
        align: 'right',
        classes: 'bg-blue-grey-1',
        headerStyle: 'font-size: 1.2em; width: 15%',
      },
      {
        name: 'rate4',
        label: 'Комбайндар,  1000 га/дана',
        field: 'rate4',
        align: 'right',
        classes: 'bg-blue-grey-1',
        headerStyle: 'font-size: 1.2em; width: 15%',
      },
      {
        name: 'clc',
        label: 'Есептелген түсімділік, ц/га',
        field: 'clc',
        align: 'right',
        classes: 'bg-red-2',
        headerStyle: 'font-size: 1.2em; width: 15%',
      },
    ]
  },
}
</script>

<style scoped>

.feedback__file {
  display: none;
}

</style>
