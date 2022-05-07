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
       * @typedef {Object} basicChartDataConfiguration
       * @property {string} name
       * @property {number[]} data
       */
      /**
       * @typedef {Object} basicChartTypeConfiguration
       * @property {string} type
       */
      /**
       * @typedef {Object} basicChartConfiguration
       * @property {basicChartDataConfiguration[]} series
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
      subcategoryChart: {
        series: [{
          name: 'series1',
          data: [31, 40, 28, 51, 42, 109, 100]
        }, {
          name: 'series2',
          data: [11, 32, 45, 32, 34, 52, 41]
        }],
        chart: {
          type: 'area',
          dataLabels: {
            enabled: false
          },
          stroke: {
            curve: 'smooth'
          },
          xaxis: {
            categories: ['2018-09-19',
              '2018-09-20',
              '2018-09-21',
              '2018-09-22',
              '2018-09-23',
              '2018-09-24',
              '2018-09-25']
          }
          // tooltip: {
          //   x: {
          //     format: 'dd/MM/yy HH:mm'
          //   }
          // }
        }
      },
      // subcategory table info
      subcategoryTable: {
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
        data: [
          {
            period: '20-03-22',
            sales: 2222222,
            products: 3144
          },
          {
            period: '28-03-22',
            sales: 225322,
            products: 3144
          },
          {
            period: '25-03-22',
            sales: 23222,
            products: 3144
          }
        ]
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
    async initBasicData () {
      const basicDataArray = []
      const response = await fetch(`http://127.0.0.1:8080/basic-statistics?ref=${this.url}&start=${this.start}&end=${this.end}`)
      let responseData = await response.json()
      const initialResponseDataLength = responseData.length
      responseData = responseData.filter(el => el.averagePrice !== 'null')
      this.basicBannerFlag = initialResponseDataLength !== responseData.length
      responseData.forEach(el => {
        el.period = date.extractDate(el.period, 'DD MM YYYY HH:mm:ss ZZ')
        let day = el.period.getDate()
        let month = el.period.getMonth() + 1
        day = (day < 10 ? '0' : '') + day.toString()
        month = (month < 10 ? '0' : '') + month.toString()
        el.period = day + '.' + month + '.' + el.period.getFullYear().toString()
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
