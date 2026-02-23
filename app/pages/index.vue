<script setup lang="ts">
import {axisBottom, axisLeft, max, min, scaleBand, scaleUtc, select} from "d3";
import {type Director, directorsMockData} from "../mocks/directors.mock";

const data = directorsMockData.sort((a:Director, b:Director) => new Date(a.from).getTime() - new Date(b.from).getTime())

const graphContainer = ref<HTMLElement | null>(null);

let svgContainer: any

const width = 1000
const height = 400

/* Рассчет отступов */
const calculateMarginLeft = (): number => {
  return Math.max(...data.map(d => d.name.length)) * 7
}

const margin = {top: 30, right: 30, bottom: 30, left: calculateMarginLeft()}
const offset = 10
/* !Рассчет отступов */

/* Создание осей */
const minDate = min(data, d => new Date(d.from)) ?? new Date(0)
const maxDate = max(data, d => d.to ? new Date(d.to) : new Date()) ?? new Date()

const xScale = scaleUtc()
    .domain([minDate, maxDate])
    .range([0, width - margin.left - margin.right])

const yScale = scaleBand()
    .domain(data.map(d => d.name))
    .range([0, height - margin.bottom - margin.top])
    .padding(0)
/* !Создание осей */

const renderGraph = () => {
  // Добавление оси X
  svgContainer.append('g')
      .attr('transform', `translate(${margin.left + offset},${height - margin.bottom})`)
      .call(axisBottom(xScale))

  // Добавление оси Y
  svgContainer.append('g')
      .attr('transform', `translate(${margin.left},0)`)
      .call(axisLeft(yScale))

  // Добавление данных
  svgContainer.selectAll('.bar')
      .data(data)
      .join('rect')
      .attr("x", (d: Director) => xScale(new Date(d.from)) + margin.left + offset)
      .attr('y', (d: Director) => yScale(d.name))
      .attr("height", yScale.bandwidth())
      .attr("width", (d: Director) => xScale(!!d.to ? new Date(d.to) : new Date()) - xScale(new Date(d.from)))
      .attr("fill", '#be97fc')
}

console.log()


onMounted(() => {
  svgContainer = select(graphContainer.value)
      .append('svg').attr('width', width)
      .attr('height', height)

  renderGraph()
})

</script>

<template>
  <div ref="graphContainer"/>
</template>

<style scoped>

</style>