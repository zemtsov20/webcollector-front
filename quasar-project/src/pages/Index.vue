<template>
  <q-page
    class="q-px-sm q-pb-sm q-pt-lg"
  >
    <div
      class="full-width column q-gutter-y-sm"
    >
      <div
        class="full-width row justify-center"
      >
        <q-input
          ref="urlField"
          placeholder="Введите ссылку на категорию или товар"
          label="Ссылка на категорию или товар"
          outlined
          v-model="url"
          hide-bottom-space
          class="col-6"
          :rules="[value=>!!value.length || 'Пожалуйста, введите ссылку или выберите ее в меню слева']"
          v-on:keydown.enter="getUrl"
        />
        <q-btn
          color="primary"
          label="Найти"
          class="col-3 q-ml-md"
          @click="getUrl"
          :loading="loadingState"
        />
      </div>
      <div
        class="full-width row justify-center"
      >
        <q-input
          class="q-mr-lg"
          ref="startDateInput"
          label="От"
          readonly
          square
          :rules="[value=>value.length > 0 || 'Пожалуйста, выберите дату']"
          v-model="start"
        >
          <template
            v-slot:append
          >
            <q-icon
              name="event"
              class="cursor-pointer"
            >
              <q-popup-proxy
                ref="qDateProxy"
                anchor="bottom middle"
              >
                <q-date
                  v-model="start"
                  mask="DD MM YYYY"
                  :options="startDateRestriction"
                >
                  <div
                    class="row items-center justify-end"
                  >
                    <q-btn
                      v-close-popup
                      label="Закрыть"
                      color="secondary"
                      flat
                    />
                  </div>
                </q-date>
              </q-popup-proxy>
            </q-icon>
          </template>
        </q-input>
        <q-input
          ref="endDateInput"
          label="До"
          readonly
          square
          :rules="[value=>value.length > 0 || 'Пожалуйста, выберите дату']"
          v-model="end"
        >
          <template
            v-slot:append
          >
            <q-icon
              name="event"
              class="cursor-pointer"
            >
              <q-popup-proxy
                ref="qDateProxy"
                anchor="bottom middle"
              >
                <q-date
                  v-model="end"
                  mask="DD MM YYYY"
                  :options="endDateRestriction"
                >
                  <div
                    class="row items-center justify-end"
                  >
                    <q-btn
                      v-close-popup
                      label="Закрыть"
                      color="secondary"
                      flat
                    />
                  </div>
                </q-date>
              </q-popup-proxy>
            </q-icon>
          </template>
        </q-input>
      </div>
      <div
        v-if="!loadingState&&resultsExists"
        class="column full-width"
      >
        <div
          class="row full-width q-gutter-y-sm justify-center"
        >
          <q-tabs
            v-model="tab"
            narrow-indicator
            class="text-primary col-9"
            dense
            align="justify"
          >
            <q-tab
              name="basic"
              class="col-4"
              icon="dashboard"
              label="Простая"
            />
            <q-tab
              name="subcategory"
              class="col-4"
              icon="article"
              label="Подкатегории"
            />
            <q-tab
              name="sellers"
              class="col-4"
              icon="person"
              label="Бренды"
            />
          </q-tabs>
        </div>
        <div
          v-show="tab === 'basic'"
          class="column full-width q-gutter-y-sm"
        >
          <div
            class="row justify-center"
          >
            <q-banner
              v-if="basicBannerFlag"
              class="bg-primary text-white q-mt-md col-9"
            >
              <template
                v-slot:avatar
              >
                <q-icon
                  name="bi-exclamation-circle"
                  color="white"
                />
              </template>
              Информация по некоторым датам в промежутке недоступна! Вы картон!
            </q-banner>
          </div>
          <chart-and-table
            :chart="basicChart"
            :table="basicTable"
          />
        </div>
        <div
          v-show="tab === 'subcategory'"
        >
          <div
            class="row justify-center"
          >
            <q-banner
              v-if="basicBannerFlag"
              class="bg-primary text-white q-mt-md col-9"
            >
              <template
                v-slot:avatar
              >
                <q-icon
                  name="bi-exclamation-circle"
                  color="white"
                />
              </template>
              Информация по некоторым датам в промежутке недоступна! Вы батон!
            </q-banner>
          </div>
          <chart-and-table
            :chart="subcategoryChart"
            :table="subcategoryTable"
          />
        </div>
        <div
          v-show="tab === 'sellers'"
        >
          <chart-and-table
            :chart="sellersChart"
            :table="sellersTable"
          />
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import ChartAndTable from 'components/ChartAndTable'
import { date } from 'quasar'
export default {
  components: {
    ChartAndTable
  },
  name: 'PageIndex',
  data () {
    return {
      loadingState: false,
      resultsExists: false,
      basicBannerFlag: false,
      /**
       * @typedef {Object} chartDataConfiguration
       * @property {string} name
       * @property {number[]} data
       */
      /**
       * @typedef {Object} basicChartTypeConfiguration
       * @property {string} type
       */
      /**
       * @typedef {Object} basicChartConfiguration
       * @property {chartDataConfiguration[]} series
       * @property {basicChartTypeConfiguration} chart
       */
      /**
       * @type {basicChartConfiguration}
       */
      basicChart: {
        series: [
          {
            name: 'Ср. цена',
            data: null
          }
        ],
        chart: {
          type: 'line'
        }
      },
      /**
       * @typedef {Object} productInformation
       * @property {string} period
       * @property {string} averagePrice
       * @property {string} goodsCount
       */
      /**
       * @typedef {Object} columnConfiguration
       * @property {string} name
       * @property {boolean} [required]
       * @property {string} label
       * @property {string} [align]
       * @property {string} field
       * @property {boolean} [sortable]
       */
      /**
       * @typedef {Object} basicTableConfiguration
       * @property {columnConfiguration[]} columns
       * @property {productInformation[]} data
       */
      /**
       * @type {basicTableConfiguration}
       */
      basicTable: {
        columns: [
          {
            name: 'period',
            required: true,
            label: 'Период',
            align: 'left',
            field: 'period',
            sortable: true
          },
          {
            name: 'sales',
            align: 'center',
            label: 'Средняя стоимость',
            field: 'averagePrice',
            sortable: true
          },
          {
            name: 'products',
            label: 'Среднее кол-во',
            field: 'goodsCount',
            sortable: true
          }
        ],
        data: null
      },
      // subcategory chart info
      /**
       * @typedef {Object} subcategoryChartXAxisConfiguration
       * @property {string[]} categories
       */
      /**
       * @typedef {Object} subcategoryChartConfiguration
       * @property {string} type
       * @property {Object} dataLabels
       * @property {Object} stroke
       * @property {subcategoryChartXAxisConfiguration} xaxis
       */
      /**
       * @typedef {Object} subcategoryChart
       * @property {chartDataConfiguration[]} series
       * @property {subcategoryChartConfiguration} chart
       */
      /**
       * @type {subcategoryChart}
       */
      subcategoryChart: {
        series: null,
        chart: {
          type: 'area',
          dataLabels: {
            enabled: false
          },
          stroke: {
            curve: 'smooth'
          },
          xaxis: {
            categories: null
          }
        }
      },
      /**
       * @typedef {Object} subcategoryInformation
       * @property {string} subcategory
       * @property {string} averagePrice
       * @property {string} averageProductCount
       */
      /**
       * @typedef {Object} subcategoryTableConfiguration
       * @property {columnConfiguration[]} columns
       * @property {subcategoryInformation[]} data
       */
      /**
       * @type {subcategoryTableConfiguration}
       */
      subcategoryTable: {
        columns: [
          {
            name: 'subcategory',
            required: true,
            label: 'Подкатегория',
            align: 'left',
            field: 'subcategory',
            sortable: true
          },
          {
            name: 'averagePrice',
            align: 'center',
            label: 'Средняя цена',
            field: 'averagePrice',
            sortable: true
          },
          {
            name: 'averageProductCount',
            label: 'Среднее количество товаров',
            field: 'averageProductCount',
            sortable: true
          }
        ],
        data: null
      },
      // sellers chart info
      sellersChart: {
        series: [1, 2, 13, 4],
        chart: {
          type: 'donut',
          labels: ['Apple', 'Mango', 'Orange', 'Watermelon']
        }
      },
      // sellers table info
      sellersTable: {
        columns: [
          {
            name: 'period',
            required: true,
            label: 'Период',
            align: 'left',
            field: 'period',
            sortable: true
          },
          {
            name: 'sales',
            align: 'center',
            label: 'Продажи',
            field: 'sales',
            sortable: true
          },
          {
            name: 'products',
            label: 'Товары',
            field: 'products',
            sortable: true
          }
        ],
        data: [1, 2, 3]
      },
      url: '',
      tab: 'basic',
      start: '',
      end: ''
    }
  },
  methods: {
    dateConverter (initDate, format) {
      const dateBuffer = date.extractDate(initDate, format)
      let day = dateBuffer.getDate()
      let month = dateBuffer.getMonth() + 1
      day = (day < 10 ? '0' : '') + day.toString()
      month = (month < 10 ? '0' : '') + month.toString()
      return day + '.' + month + '.' + dateBuffer.getFullYear().toString()
    },
    async initSubcategoryData () {
      const response = await fetch(`http://127.0.0.1:8080/subcategory-statistics?ref=${this.url}&start=${this.start}&end=${this.end}`)
      const responseData = await response.json()
      const arrayForTable = []
      const dateArray = []
      const figureData = []
      for (let i = 0; i < responseData.length; i++) {
        responseData[i].statistics = responseData[i].statistics.map(el => {
          return {
            period: el.period,
            averagePrice: el.averagePrice === 'null' ? null : Number(el.averagePrice),
            goodsCount: el.goodsCount === 'null' ? null : Number(el.goodsCount)
          }
        })
      }
      console.log(responseData)
      responseData[0].statistics.forEach((el, i, a) => {
        dateArray[i] = this.dateConverter(el.period, 'DD MM YYYY HH:mm:ss ZZ')
      })
      for (let i = 0; i < responseData.length; i++) {
        arrayForTable.push({
          subcategory: responseData[i].categoryUrl,
          averagePrice: 0,
          averageProductCount: 0
        })
        figureData.push({
          name: responseData[i].categoryUrl,
          data: []
        })
        for (let j = 0; j < responseData[i].statistics.length; j++) {
          arrayForTable[i].averagePrice += responseData[i].statistics[j].averagePrice !== null ? responseData[i].statistics[j].averagePrice : 0
          arrayForTable[i].averageProductCount += responseData[i].statistics[j].goodsCount !== null ? responseData[i].statistics[j].goodsCount : 0
          figureData[i].data.push(responseData[i].statistics[j].averagePrice)
        }
        arrayForTable[i].averagePrice = Number(arrayForTable[i].averagePrice / responseData[i].statistics.filter(el => !!el.averagePrice).length).toFixed(0)
        arrayForTable[i].averageProductCount = Number(arrayForTable[i].averageProductCount / responseData[i].statistics.filter(el => !!el.goodsCount).length).toFixed(0)
      }
      this.subcategoryTable.data = arrayForTable
      this.subcategoryChart.series = figureData
      this.subcategoryChart.chart.xaxis = { categories: dateArray }
    },
    async initBasicData () {
      const basicDataArray = []
      const response = await fetch(`http://127.0.0.1:8080/basic-statistics?ref=${this.url}&start=${this.start}&end=${this.end}`)
      let responseData = await response.json()
      const initialResponseDataLength = responseData.length
      responseData = responseData.filter(el => el.averagePrice !== 'null')
      this.basicBannerFlag = initialResponseDataLength !== responseData.length
      responseData.forEach(el => {
        el.period = this.dateConverter(el.period, 'DD MM YYYY HH:mm:ss ZZ')
        basicDataArray.push({ x: el.period, y: el.averagePrice })
      })
      this.basicChart.series[0].data = basicDataArray
      this.basicTable.data = responseData
    },
    async getUrl () {
      const correctURLFieldInput = this.$refs.urlField.validate()
      const correctStartDateInput = this.$refs.startDateInput.validate()
      const correctEndDateInput = this.$refs.endDateInput.validate()
      if (correctURLFieldInput && correctEndDateInput && correctStartDateInput) {
        this.loadingState = true
        await this.initBasicData()
        await this.initSubcategoryData()
        this.resultsExists = true
        this.loadingState = false
      }
    },
    startDateRestriction (dateForCheck) {
      if (this.end === '') {
        return true
      }
      const startDate = new Date(0)
      const maxDate = date.extractDate(this.end, 'DD MM YYYY')
      return date.isBetweenDates(dateForCheck, startDate, maxDate, {
        inclusiveFrom: true,
        inclusiveTo: true,
        onlyDate: true
      })
    },
    endDateRestriction (dateForCheck) {
      if (this.start === '') {
        return true
      }
      const startDate = date.extractDate(this.start, 'DD MM YYYY')
      const maxDate = new Date(8640000000000000)
      return date.isBetweenDates(dateForCheck, startDate, maxDate, {
        inclusiveFrom: true,
        inclusiveTo: true,
        onlyDate: true
      })
    }
  },
  computed: {
    currentLinkField () {
      return this.$store.getters['mySiteStore/currentLinkGetter']
    }
  },
  watch: {
    currentLinkField: function () {
      this.url = this.$store.state.mySiteStore.currentLink
    }
  }
}
</script>
