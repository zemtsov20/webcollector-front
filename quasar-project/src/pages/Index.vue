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
          placeholder="Введите ссылку на категорию или товар"
          label="Ссылка на категорию или товар"
          outlined
          v-model="url"
          class="col-6"
          v-on:keydown.enter="getUrl"
          error-message="Введите данные"
        />
        <q-btn
          color="primary"
          label="Найти"
          class="col-2 q-ml-md"
          @click="getUrl"
          :loading="loadingState"
        />
      </div>
      <div
        v-if="!loadingState&&resultsExists"
      >
        <div
          class="col-9 q-gutter-y-sm"
        >
          <q-tabs
            v-model="tab"
            narrow-indicator
            class="text-primary"
            dense
            align="justify"
          >
            <q-tab
              name="basic"
              icon="dashboard"
              label="Простая"
            />
            <q-tab
              name="subcategory"
              icon="article"
              label="Подкатегории"
            />
            <q-tab
              name="sellers"
              icon="person"
              label="Продавцы"
            />
          </q-tabs>
        </div>
        <div
          v-show="tab==='basic'"
        >
          <chart-and-table
            :chart="basicChart"
            :table="basicTable"
          />
        </div>
        <div
          v-show="tab==='subcategory'"
        >
          <chart-and-table
            :chart="subcategoryChart"
            :table="subcategoryTable"
          />
        </div>
        <div
          v-show="tab==='sellers'"
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
export default {
  components: {
    ChartAndTable
  },
  name: 'PageIndex',
  data () {
    return {
      loadingState: false,
      resultsExists: false,
      // basic chart info
      basicChart: {
        series: [
          {
            name: 'test',
            data: [1, 2, 13, 4, 5, 7, 11, 32]
          }
        ],
        chart: {
          type: 'line'
        }
      },
      // basic table info
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
          { name: 'sales', align: 'center', label: 'Продажи', field: 'sales', sortable: true },
          { name: 'products', label: 'Товары', field: 'products', sortable: true }
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
          { name: 'sales', align: 'center', label: 'Продажи', field: 'sales', sortable: true },
          { name: 'products', label: 'Товары', field: 'products', sortable: true }
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
          { name: 'sales', align: 'center', label: 'Продажи', field: 'sales', sortable: true },
          { name: 'products', label: 'Товары', field: 'products', sortable: true }
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
      url: '',
      tab: 'basic'
    }
  },
  methods: {
    async getUrl () {
      // fetch('http://127.0.0.1:8080/basic?name=testName&period=20-05-22')
      //   .then((response) => {
      //     return response.json()
      //   })
      //   .catch((data) => {
      //     console.log(data)
      //   })
      this.loadingState = true
      await new Promise((resolve, reject) => setTimeout(resolve, 1000))
      this.resultsExists = true
      this.loadingState = false
    },
    onclick (link) {
      this.url = link
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
