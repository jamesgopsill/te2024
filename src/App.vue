<script setup lang="ts">
import Plotly, { Data, Layout } from "plotly.js-dist-min"
import { onMounted, ref } from "vue"
import { ukData } from "./uk"

const printTime = ref<number>(2)
const nMachines = ref<number>(150_000)
let y1: number[] = []
let y2: number[] = []
let x: Date[] = []
const layout: Layout = {
	autosize: true,
	xaxis: {
		title: {
			text: "Date",
		},
	},
	yaxis: {
		title: {
			text: "Tests",
		},
	},
}

const calculate = async () => {
	const maxPrintCapacity = Math.floor((nMachines.value * 24) / printTime.value)
	const capacityUsed: number[] = []
	for (const _ of y1) {
		capacityUsed.push(0)
	}

	for (let i = 0; i < y1.length; i++) {
		let testsRequired = y1[i] as number
		for (let j = i + 1; j < capacityUsed.length; j++) {
			const cu = capacityUsed[j] as number
			// if we have reached max capacity then
			// go back to the day before
			if (cu == maxPrintCapacity) {
				continue
			}
			// check if there is enough print time in the day
			const capacityAvailable = maxPrintCapacity - cu
			if (testsRequired > capacityAvailable) {
				capacityUsed[j] = maxPrintCapacity
				testsRequired -= capacityAvailable
				continue
			}
			//@ts-ignore
			capacityUsed[j] += testsRequired
			break
		}
	}
	y2 = capacityUsed
	plot()
}

onMounted(async () => {
	let d: Date = new Date()
	for (const datapoint of ukData.data) {
		y1.push(datapoint.newVirusTestsByPublishDate || 0)
		d = new Date(datapoint.date)
		x.push(d)
	}
	for (let i = 0; i < 150; i++) {
		let n = d.getTime() - 86400000 * i
		const d1 = new Date(n)
		x.push(d1)
		y1.push(0)
	}
	plot()
	calculate()
})

const plot = async () => {
	const data: Data[] = [
		{
			x: x,
			y: y1,
			name: "Recorded Tests",
		},
		{
			x: x,
			y: y2,
			name: "Manufactured Tests",
		},
	]
	await Plotly.newPlot("plt", data, layout)
}
</script>
<template>
	<div class="container">
		<h2 class="mt-4 mb-5 text-center">
			Distributed Additive Manufacturing: A Social Change in Manufacturing.
		</h2>
		<div class="row justify-content-md-center">
			<div class="col-7 mb-5">
				<p>James Gopsill</p>
				<h5>Abstract</h5>
				<p>
					COVID was an unprecedented event requiring an extreme global response
					to stem the rate of infection. There was a drastic change in product
					demand with many nations requesting Personal Protection Equipment
					(PPE) and Lateral Flow Devices (LFDs). The change in demand coupled
					with lockdown measures exposed the fragility and unresponsiveness of
					global supply chains resulting in nations competing with one another
					for supply. Supply was often delayed and insufficient in quantity.
					Accounts reported it took nearly six months for supply to stabilise.
				</p>
				<p>
					In contrast, Additive Manufacturing (AM) and the ‘Maker’ community
					thrived in designing and producing products to support society.
					Through a reflective study analysing COVID test data, this paper shows
					that the UK's distributed AM capacity (an estimated 168,000 AM
					machines distributed across homes, educational settings, offices, and
					industrial facilities) could have provided the nation with the devices
					it needed with day-ahead demand forecasting.
				</p>
				<p>
					Government funds would have supported UK AM, reduced the carbon
					footprint of shipping LFD tests from other nations, manufactured only
					what was required (25% of PPE was not used) and prevented an estimated
					25 swimming pools worth of single use plastic waste. The paper
					discusses the Social Change and platform required to realise
					distributed AM as a global supply chain that can support nations in
					being more resilient, responsive, and sustainable.
				</p>
				<p>Paper link coming soon</p>
				<div class="card">
					<div class="card-header">
						UK Additive Manufacturing manufacturing tests response model.
					</div>
					<div class="card-body">
						<label>Number of Machines</label>
						<input
							class="form-control mb-3"
							type="number"
							v-model="nMachines"
						/>
						<label>Print Time</label>
						<input
							class="form-control mb-3"
							type="number"
							v-model="printTime"
						/>
						<button class="btn btn-outline-primary" @click="calculate">
							Calculate
						</button>
					</div>
				</div>
				<div id="plt"></div>
			</div>
		</div>
	</div>
</template>
