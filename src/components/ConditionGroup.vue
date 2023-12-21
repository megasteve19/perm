<template>
	<div class="flex flex-col gap-4 p-6 border-2 bg-slate-100 rounded-xl">
		<template
			v-for="item in items"
			:key="item.id"
		>
			<ConditionGroup
				v-if="item.type === 'group'"
				:depth="depth + 1"
				:id="item.id"
				@remove="id => removeItem(id)"
				:attributes="attributes"
				@change="sendToParent"
			/>

			<div
				v-if="item.type === 'condition'"
				class="flex flex-col gap-4"
			>
				<div class="flex items-center justify-center w-full gap-2 p-4 bg-gray-200 rounded-xl">
					<select
						v-model="item.data.attribute"
						class="w-full h-full p-2 bg-gray-300 rounded-lg"
					>
						<option value="">Select attribute</option>
						<option
							v-for="attribute in attributes"
							:key="attribute"
							:value="attribute"
						>{{ attribute }}</option>
					</select>

					<select
						v-model="item.data.operator"
						class="h-full p-2 bg-gray-300 rounded-lg "
					>
						<option value="">Select operator</option>
						<option value="=">=</option>
						<option value="!=">!=</option>
					</select>

					<input
						v-model="item.data.value"
						type="text"
						class="w-full h-full p-2 bg-gray-300 rounded-lg"
						placeholder="value"
					>
				</div>

				<div class="flex justify-end">
					<button
						class="p-2 text-sm text-white bg-red-500 rounded-lg"
						@click="removeItem(item.id)"
					>Remove</button>
				</div>
			</div>

			<div
				class="flex items-center justify-between w-full p-4 bg-gray-200 rounded-xl"
				v-else-if="item.type === 'and' || item.type === 'or'"
			>
				<span class="font-black">{{ item.type === 'and' ? 'AND' : 'OR' }}</span>

				<button
					class="p-2 text-sm text-white bg-red-500 rounded-lg"
					@click="removeItem(item.id)"
				>Remove</button>
			</div>
		</template>

		<nav class="flex justify-end gap-2">
			<button
				class="p-4 text-white bg-red-500 rounded-xl"
				@click="$emit('remove', id)"
			>Remove</button>

			<button
				v-if="canAddGroup"
				class="p-4 text-white bg-blue-500 rounded-xl"
				@click="addItem('group')"
			>Group</button>

			<button
				v-if="canAddCondition"
				class="p-4 text-white bg-blue-500 rounded-xl"
				@click="addItem('condition')"
			>Condition</button>

			<template v-if="canAddOperator">
				<button
					v-if="depth < 5"
					class="p-4 text-gray-700 bg-gray-200 rounded-xl"
					@click="addItem('and')"
				>And</button>

				<button
					v-if="depth < 5"
					class="p-4 text-gray-700 bg-gray-200 rounded-xl"
					@click="addItem('or')"
				>Or</button>
			</template>
		</nav>
	</div>
</template>

<script setup>
import { cloneVNode, computed, ref, watch } from 'vue';
import { v4 } from 'uuid';
import ConditionGroup from './ConditionGroup.vue';

const emit = defineEmits(['remove', 'change']);

const props = defineProps({
	depth: Number,
	id: String,
	attributes: Array,
});

const items = ref([]);

const addItem = (type) => {
	if (type === 'condition') {
		items.value.push({
			id: v4(),
			type: type,
			data: {
				attribute: '',
				operator: '',
				value: '',
			}
		});
	}
	else {
		items.value.push({
			id: v4(),
			type: type,
		});
	}
};

const removeItem = (id) => {
	items.value = items.value.filter((item) => item.id !== id);
};

const canAddGroup = computed(() => {
	const depth = props.depth < 5;
	const lastItem = ['and', 'or'].includes(items.value[items.value.length - 1]?.type) || !items.value.length;

	return depth && lastItem;
});

const canAddOperator = computed(() => {
	if (!items.value.length) {
		return false;
	}

	const lastItem = items.value[items.value.length - 1];

	return !['and', 'or'].includes(lastItem.type);
});

const canAddCondition = computed(() => {
	return props.attributes.length && (['and', 'or'].includes(items.value[items.value.length - 1]?.type) || !items.value.length);
});

const sendToParent = (childItem) => {
	const merged = JSON.parse(JSON.stringify(items.value));


	emit('change', {
		id: props.id,
		items: merged.map((item) => {
			if (item.id === childItem.id) {
				return {
					id: childItem.id,
					type: childItem.type,
					items: childItem.items,
				};
			}

			return {
				id: item.id,
				type: item.type,
				data: item.data,
			};
		})
	});
};

watch(items, (items) => {
	emit('change', {
		id: props.id,
		type: 'group',
		items: items,
	});
}, { deep: true });
</script>
