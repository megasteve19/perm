<template>
	<main class="grid min-h-screen grid-cols-2 divide-x-4">
		<section class="flex flex-col max-h-screen gap-4 p-4 overflow-auto">
			<!-- <div
				v-for="group in groups"
				class="p-6 bg-slate-100 rounded-xl"
			>
				
			</div> -->

			<input
				v-model="attributesInput"
				type="text"
				class="p-2 border-2 rounded-lg"
			>

			<ConditionGroup
				v-for="(group, index) in groups"
				:key="index"
				:depth="1"
				:attributes="attributes"
				@change="change"
			/>
		</section>
		<section class="p-4">
			<pre><code class="language-json">{{ json }}</code></pre>
		</section>
	</main>
</template>

<script setup>
import ConditionGroup from './components/ConditionGroup.vue';
import { computed, ref } from 'vue';

const groups = ref([[]]);
const attributesInput = ref('name,surname,age,marital_status,birth_day');
const attributes = computed(() => attributesInput.value.split(',').map(attribute => attribute.trim()));
const json = ref({});

const change = (obj) => {
	json.value = JSON.stringify(obj.items, null, 2);
}; 
</script>
