<script lang="ts">
import { fetchProjections } from '~/api/projectionAPI';

export default {
	data() {
		return {
			selectedTable: '',
			tables: [
				'characters',
				'comment',
				'country',
				'developer',
				'franchise',
				'individualdeveloper',
				'post',
				'posttagsvideogame',
				'roles',
				'studio',
				'users',
				'videogame',
				'videogamehascharacter',
				'voiceactor',
			],
			columns: [] as string[],
			selectedColumns: [],
			fetchedData: [] as Record<string, unknown>[],
			loading: false,
		};
	},
	methods: {
		updateColumns() {
			const typeMap = {
				roles: [
					'user_role',
					'post_permission',
					'remove_post_permission',
					'change_roles_permission',
				],
				users: ['username', 'email', 'user_password', 'user_role'],
				country: ['country', 'country_code'],
				developer: ['name', 'website_link', 'description'],
				franchise: ['name', 'description'],
				studio: ['name', 'year_established', 'country', 'phone_number'],
				individualdeveloper: ['name', 'birthdate'],
				characters: ['name', 'description', 'history'],
				voiceactor: ['actor_id', 'name', 'biography', 'birthdate'],
				videogame: [
					'name',
					'release_date',
					'genre',
					'synopsis',
					'rating',
					'sales',
					'developer_name',
					'start_date',
					'end_date',
					'franchise_name',
				],
				videogamehascharacter: [
					'game_name',
					'character_name',
					'actor_id',
					'character_role',
				],
				post: ['post_id', 'title', 'body', 'views', 'username', 'time_created'],
				comment: ['comment_id', 'body', 'username', 'time_created', 'post_id'],
				posttagsvideogame: ['post_id', 'game_name'],
			};
			this.columns = typeMap[this.selectedTable as keyof typeof typeMap];
			this.selectedColumns = [];
			this.clearData();
		},
		async fetchData() {
			this.loading = true;
			this.fetchedData = await fetchProjections(
				this.selectedTable,
				this.selectedColumns
			);
			this.loading = false;
		},
		clearData() {
			this.fetchedData = [];
		},
	},
};
</script>

<template>
	<div class="projectionContainer">
		<div class="selectionGroup">
			<label for="table-select">Select a table:</label>
			<select id="table-select" v-model="selectedTable" @change="updateColumns">
				<option disabled value="">Please select a table</option>
				<option v-for="table in tables" :key="table" :value="table">
					{{ table }}
				</option>
			</select>
		</div>
		<div v-if="selectedTable" class="column-select">
			<div class="attributes-wrapper">
				<div v-for="column in columns" :key="column">
					<input
						type="checkbox"
						:id="column"
						:value="column"
						v-model="selectedColumns"
					/>
					<label :for="column">{{ column }}</label>
				</div>
				<button
					@click="fetchData"
					:disabled="loading"
					style="border-color: hsla(160, 100%, 37%, 0.5)"
				>
					{{ loading ? 'Loading...' : 'Query' }}
				</button>
			</div>
		</div>

		<div v-if="selectedColumns && !loading" class="tableContainer">
			<h2>Results</h2>
			<table>
				<thead>
					<tr>
						<th v-for="column in selectedColumns" :key="column">
							{{ column }}
						</th>
					</tr>
				</thead>
				<tbody>
					<tr v-for="(row, index) in fetchedData" :key="index">
						<td v-for="column in selectedColumns" :key="column">
							{{ row[column] }}
						</td>
					</tr>
				</tbody>
			</table>
		</div>

		<div v-if="loading">Loading...</div>
	</div>
</template>

<style scoped>
.projectionContainer {
	display: flex;
	flex-direction: column;
	align-items: center;
}

.selectionGroup {
	margin: 1rem 0 0 0;
	left: 0;
}

.selectionGroup label {
	margin-right: 0.5rem;
}

.tableContainer {
	margin: 10px;
}

.column-select {
	display: flex;
	flex-wrap: wrap;
	gap: 0.5rem;
	margin-top: 1rem;
	margin-bottom: 1rem;
	max-width: 70vw;
}

.attributes-wrapper {
	display: flex;
	flex-wrap: wrap;
	gap: 0.5rem;
	margin-bottom: 1rem;
}

.column-select label {
	display: inline-flex;
	align-items: center;
	background-color: #f2f2f2;
	border: 1px solid #ddd;
	border-radius: 4px;
	padding: 5px 10px;
	font-size: 14px;
	cursor: pointer;
	transition: background-color 0.2s;
}

.column-select input[type='checkbox'] {
	margin-right: 5px;
}

.column-select label:hover {
	background-color: #ddd;
}

.column-select input[type='checkbox']:checked + label {
	background-color: #ddd;
	color: #fff;
}

.column-select input[type='checkbox']:checked + label:hover {
	background-color: #ddd;
}

table {
	font-size: x-small;
	width: 100%;
	border-collapse: collapse;
}

thead th {
	background-color: #f2f2f2;
	border: 1px solid #dddddd;
	text-align: left;
	padding: 8px;
}

tbody td {
	border: 1px solid #dddddd;
	text-align: left;
	padding: 8px;
}
</style>
