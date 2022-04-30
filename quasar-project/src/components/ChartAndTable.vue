<template>
  <div>
    <div
      class="row justify-center"
    >
      <div class="col-9">
        <apexchart
          height="450px"
          :series="chart.series"
          :type="chart.chart.type"
          :options="chart.chart"
        >
        </apexchart>
      </div>
    </div>
    <div
      class="row justify-center"
    >
      <q-table
        class="col-9"
        title="Статистика"
        :data="table.data"
        :columns="table.columns"
        row-key="name"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: 'ChartAndTable',
  props: {
    chart: {
      type: Object,
      require: true
    },
    table: {
      type: Object,
      require: true
    }
  }
}
</script>
