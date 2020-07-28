<template>
  <div id="app">
	<div class="container">
		<div class="row">
			<table class="table table-borderless table-hover my-5 racers">
				<thead class="thead-dark">
					<tr>
						<th class="head-info">Место</th>
						<th class="head-info">Имя</th>
						<th class="head-info">Город</th>
						<th class="head-info">Авто</th>
						<th class="sortable head-info" 
							v-for="(n, index) in this.attemptsQuantity" 
							:key="index"
							@click="sortByQual(index)"
							:class="{'text-success':attemptsQuantity[index]}"
						>
							<img src="@/assets/wheel.png" v-if="attemptsQuantity[index]">
							<span v-else>Q{{ index+1 }}</span>
						</th>
						<th class="sortable head-info" @click="sortByTotal">Total</th>
					</tr>
				</thead>
				<tbody>
					<tr v-for="(racer, index) in racers" :key="index">
						<td>{{ index + 1 }}</td>
						<td>{{ racer.name }}</td>
						<td>{{ racer.city }}</td>
						<td>{{ racer.car }}</td>
						<td 
							v-for="(attempt, index) in racer.attempts" 
							:key="index"
							:class="{'text-success bg-dark':attemptsQuantity[index]}"
						>
							{{ attempt }}
						</td>
						<td>{{ racer.points }}</td>
					</tr>
				</tbody>
			</table>
		</div>
	</div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
	name: 'App',
	data() {
		return {
			racers: [],
			attempts: [],
			attemptsQuantity: []
		}
	},
	computed: {},
	methods: {
		sortByQual(n) {
			this.attemptsQuantity.fill(false)
			this.racers.sort((a,z) => z.attempts[n] - a.attempts[n])
			this.attemptsQuantity[n] = true
		},
		sortByTotal() {
			this.racers.sort((a,z) => z.points - a.points)
			this.attemptsQuantity.fill(false)
		}
	},
	mounted() {
		axios
			.get('http://localhost:3000/attempts')
			.then(response => {
				this.attempts = Object.values(response.data.data);
			})
			.catch(e => {
				console.error(e.message);
			})
		axios
			.get('http://localhost:3000/cars')
			.then(response => {
				this.racers = Object
					.entries(response.data.data)
					.map(racer => {
						return {
							number: racer[0],
							...racer[1],
							attempts: this.attempts.filter(a => a.id === racer[1].id)
						}
					})
				this.racers.forEach(racer => {
					const points = []
					racer.attempts.map(attempt => points.push(attempt.result))
					racer.attempts = points
					this.attemptsQuantity.length = points.length
					racer.points = racer.attempts.reduce((acc, val) => acc + val, 0)
				})
				this.attemptsQuantity.fill(false)
				this.racers.sort((a,z) => z.points - a.points)
			})
			.catch(e => {
				console.error(e.message);
			})
		
	}
}
</script>

<style lang="scss">
body {
    background: url('./assets/bg-bottom.png') bottom, url('./assets/bg-top.png') top;
    background-repeat: no-repeat;
    background-attachment: fixed, fixed;
    background-size: 100% auto;
}

#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;

    text-align: center;

    color: #2c3e50;

    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

.racers {
    background-color: rgba(255,255,255, .5);
    box-shadow: 0 0 1.5rem .25rem black;
}

.head-info {
    // color: tomato !important;
}

.sortable {
    cursor: pointer;
}

</style>
