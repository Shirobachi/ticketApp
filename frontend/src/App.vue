<script setup>

	import axios from "axios";
	import { ref, computed } from "vue";
	import { useToast } from "vue-toastification";
	
	// https://github.com/Maronato/vue-toastification#installation
	const toast = useToast();
	const dates = ref()

	// function to add validated ticket
	function addTicket(isLegal) {
		// make get request to backend
		toast.info("Ticket adding...", {position: "bottom-center"});

		let link = "https://n8n.hryszko.dev/webhook/05ff9771-d474-43ab-a310-7550ef96f45c?isLegal=" + isLegal;
		axios.get(link).then(function() {
			// success toast
			// clear t
			toast.clear();
			toast.success("Ticket added!", {position: "bottom-center"});
			refreshBadges();
		}).catch(function() {
			// error toast
			toast.error("Error adding ticket!", {position: "bottom-center"});
		});

		// toast("I'm a toast!");
	}

	// function to get data from API
	function refreshBadges(){
		console.log("refreshing");
		let link = "https://n8n.hryszko.dev/webhook/05ff9771-d474-43ab-a310-7550ef96f45casd";
		let linkDates = "https://n8n.hryszko.dev/webhook/e06a89c6-24e1-4deb-b9e4-a578a63546a2";

		axios.get(link).then(function(r){
			data.value = r.data;
		}).catch(function(){
			toast.error("Error loading data!", {position: "bottom-center"});
		})

		axios.get(linkDates).then(function(r){
			dates.value = r.data;
		}).catch(function(){
			toast.error("Error loading graph!", {position: "bottom-center"});
		})

		toast.clear();
		toast.success("Data refreshed!", {position: "bottom-center"});
	}

	refreshBadges();

	const data = computed(() => {
		let r = {
			"legal": {
				"count": 0,
				"value": 0,
			},
			"illegal": {
				"count": 0,
				"value": 0,
			},
		}

		if(dates.value)
			dates.value.map(item => {
				if(item.json.fields.isLegal){
					r.legal.count += 1;
					r.legal.value += item.json.fields.value;
				} else {
					r.illegal.count += 1;
					r.illegal.value += item.json.fields.value;
				}
			})

		return r;
	})

	const getGraphLink = computed(() => {
		// check if dates is empty
		if(dates.value == undefined)
			return "https://dummyimage.com/200x100/fff/000.png&text=Loading+grapth...";
			
		let data = {
			type: 'bar',
			data: {
				labels: [],
				datasets: [
					{
						label: 'Legal',
						data: [],
						fill: false,
						backgroundColor: "rgba(255, 99, 132, 0.2)",
						borderColor: "rgb(255, 99, 132)",
						borderWidth: 1,
					}, 
					{
						label: 'Illegal',
						data: [],
						fill: false,
						backgroundColor: "rgba(75, 192, 192, 0.2)",
						borderColor: "rgb(75, 192, 192)",
						borderWidth: 1,
					}
				]
			}
		}

		dates.value.forEach(function(d){
			let temp = d.json.fields.date

			// substring to remove time
			temp = temp.substring(0, 10);
			
			// check if temp in data.data.labels
			if(data.data.labels.indexOf(temp) == -1){
				data.data.labels.push(temp);
				data.data.datasets[0].data.push(0);
				data.data.datasets[1].data.push(0);
			}
			
			let index = data.data.labels.indexOf(temp);
			if(d.json.fields.isLegal){
				data.data.datasets[0].data[index] += d.json.fields.value;
			} else {
				data.data.datasets[1].data[index] += d.json.fields.value;
			}

			})

			return "https://quickchart.io/chart?c=" + JSON.stringify(data)
	});

</script>

<template>

	<div>

		{{dates}}

		<div class="flex justify-center mt-6">

			<button @click="addTicket('True')" class="bg-green-100 hover:bg-gray-200 text-gray-800 font-semibold py-2 border border-gray-400 rounded shadow mx-auto">Add validated ticket (1.5PLN)</button>    
			<button v-on:click="addTicket('False')" class="bg-white hover:bg-gray-100 text-gray-800 font-semibold py-2 px-4 border border-gray-400 rounded shadow mx-auto">Add illegal journey!</button>    

		</div>

		<div class="flex justify-center mt-5">

			<div class="max-w-sm rounded overflow-hidden shadow-lg">
				<a target="_blank" :href="getGraphLink">
					<img class="w-full" :src="getGraphLink" alt="Sunset in the mountains">
				</a>
				<div class="px-6 py-4">
					<div class="font-bold text-base text-center mb-2">Suma wydanych pieniędzy na bilet</div>
						
					<div class="pt-4 pb-2 text-center">
						<span class="inline-block bg-red-200 hover:bg-red-300 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-4 mb-2">{{data.legal.count}} tickets!</span>
						<span class="inline-block bg-red-300 hover:bg-red-400 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-4 mb-2">{{data.legal.value}}PLN</span>
					</div>

					<div class="font-bold text-sm text-center mb-2">Suma zaoszczędzonych pieniędzy na biletach</div>
					<div class="pt-4 pb-2 text-center">
						<span class="inline-block bg-green-200 hover:bg-green-300 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-4 mb-2">{{data.illegal.count}} tickets!</span>
						<span class="inline-block bg-green-300 hover:bg-green-400 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-4 mb-2">{{data.illegal.value}}PLN</span>
					</div>
				</div>
			</div>

		</div>

	</div>

</template>