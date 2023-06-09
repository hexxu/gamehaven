<script setup lang="ts">
import { ref, reactive } from 'vue';
import { VideoGame } from '../types/types';
import {
	insertVideoGame,
	updateVideoGameByName,
	deleteVideoGameByName,
	fetchFilteredVideogames,
} from '~~/api/videogameAPI';

const selectedVideoGame = reactive({
	name: '',
	release_date: '',
	genre: '',
	synopsis: '',
	rating: '',
	sales: 0,
	developer_name: '',
	start_date: 0,
	end_date: 0,
	franchise_name: '',
});

const selectedGame = ref('' as string);

const videoGames = ref([] as VideoGame[]);
const formVisible = ref(false);
const formMode = ref('add');
const ratingFilter = ref('');

async function addVideoGame() {
	const newVideoGame = await insertVideoGame(selectedVideoGame as VideoGame);
	if (newVideoGame) {
		alert(`successfully added ${newVideoGame.name}`);
		videoGames.value.unshift(newVideoGame);
	}
}
async function deleteVideoGame(game_name: string) {
	const { success, message } = await deleteVideoGameByName(game_name);
	if (success) {
		alert(`successfully deleted ${game_name}!`);
		videoGames.value = videoGames.value.filter(vg => vg.name !== game_name);
	} else {
		alert(`Error: ${message}`);
	}
}
async function editVideoGame(game_name: string) {
	const partialVideoGame: Partial<VideoGame> = {};
	for (const key in selectedVideoGame) {
		if (selectedVideoGame[key as keyof object] !== null) {
			partialVideoGame[key as keyof VideoGame] =
				selectedVideoGame[key as keyof object];
		}
	}

	const updatedVideoGame = await updateVideoGameByName(
		game_name,
		partialVideoGame
	);
	if (updatedVideoGame) {
		const index = videoGames.value.findIndex(vg => vg.name === game_name);
		if (index !== -1) {
			videoGames.value[index] = updatedVideoGame;
		}
		alert(`Updated ${updatedVideoGame.name}`);
	}
}

async function submitForm() {
	if (formMode.value === 'add') {
		await addVideoGame();
	} else {
		await editVideoGame(selectedGame.value);
	}
	formVisible.value = false;
}

function showEditForm(videoGame: VideoGame) {
	formMode.value = 'edit';
	formVisible.value = true;
	selectedGame.value = videoGame.name;
	Object.assign(selectedVideoGame, videoGame);
}

async function cancelEdit() {
	formVisible.value = false;
}

const selectedColumns = ref<string[]>([]);

const columns = ref([
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
]);

const fetched = ref(false);

async function fetchFilteredColumns() {
	if (selectedColumns.value.length > 0) {
		videoGames.value = await fetchFilteredVideogames(
			selectedColumns.value,
			ratingFilter.value
		);
	} else {
		videoGames.value = [];
	}
	fetched.value = true;
}

function selectAllColumns() {
	selectedColumns.value = [...columns.value];
}

function unselectAllColumns() {
	selectedColumns.value = [];
	videoGames.value = [];
}
</script>

<template>
	<div class="container">
		<h1>Video Games</h1>
		<div class="fetch-row">
			<div class="column-select">
				<label v-for="column in columns" :key="column">
					{{ column }}
					<input
						type="checkbox"
						:value="column"
						v-model="selectedColumns"
						ref="checkboxes"
					/>
				</label>
				<div v-if="selectedColumns.includes('rating')" class="rating-filter">
					<input
						v-if="selectedColumns.includes('rating')"
						type="text"
						id="ratingFilter"
						v-model="ratingFilter"
						placeholder="Filter by rating"
					/>
				</div>
			</div>
			<div class="fetch-buttons">
				<button @click="selectAllColumns">Select All</button>
				<button @click="unselectAllColumns">Unselect All</button>
				<button @click="fetchFilteredColumns">Fetch Video Games</button>
			</div>
		</div>
		<div class="add-button">
			<button
				@click="
					formVisible = !formVisible;
					formMode = 'add';
				"
				v-if="!formVisible"
			>
				Add Video Game
			</button>

			<button
				v-if="formVisible"
				type="button"
				@click="cancelEdit"
				class="cancel-button"
			>
				Cancel
			</button>
		</div>
		<form @submit.prevent="submitForm" v-if="formVisible">
			<div class="formWrapper">
				<div class="leftHalf">
					<div class="inputField">
						<label for="name">Name:</label>
						<input
							type="text"
							id="name"
							v-model="selectedVideoGame.name"
							required
						/>
					</div>
					<div class="inputField">
						<label for="release_date">Release Date:</label>
						<input
							type="date"
							id="release_date"
							v-model="selectedVideoGame.release_date"
						/>
					</div>
					<div class="inputField">
						<label for="genre">Genre:</label>
						<input type="text" id="genre" v-model="selectedVideoGame.genre" />
					</div>
					<div class="inputField">
						<label for="synopsis">Synopsis:</label>
						<textarea
							id="synopsis"
							v-model="selectedVideoGame.synopsis"
						></textarea>
					</div>
					<div class="inputField">
						<label for="rating">Rating:</label>
						<input type="text" id="rating" v-model="selectedVideoGame.rating" />
					</div>
				</div>
				<div class="rightHalf">
					<div class="inputField">
						<label for="sales">Sales:</label>
						<input type="number" id="sales" v-model="selectedVideoGame.sales" />
					</div>
					<div class="inputField">
						<label for="developer_name">Developer Name:</label>
						<input
							type="text"
							id="developer_name"
							v-model="selectedVideoGame.developer_name"
						/>
					</div>
					<div class="inputField">
						<label for="developer_name">Start date:</label>
						<input
							type="text"
							id="start_date"
							v-model="selectedVideoGame.start_date"
						/>
					</div>
					<div class="inputField">
						<label for="developer_name">End date:</label>
						<input
							type="text"
							id="end_date"
							v-model="selectedVideoGame.end_date"
						/>
					</div>
					<div class="inputField">
						<label for="franchise_name">Franchise Name:</label>
						<input
							type="text"
							id="franchise_name"
							v-model="selectedVideoGame.franchise_name"
						/>
					</div>
				</div>
			</div>
			<button type="submit">
				{{ formMode === 'add' ? 'Add' : 'Submit' }}
			</button>
		</form>

		<table>
			<thead>
				<tr>
					<th v-for="column in selectedColumns" :key="column">
						{{ column }}
					</th>
					<th v-if="selectedColumns.includes('name')">Edit</th>
					<th v-if="selectedColumns.includes('name')">Delete</th>
				</tr>
			</thead>
			<tbody>
				<tr v-for="videoGame in videoGames" :key="videoGame.name">
					<template v-for="column in selectedColumns" :key="column">
						<td>{{ videoGame[column as keyof VideoGame] }}</td>
					</template>
					<td v-if="selectedColumns.includes('name')">
						<button @click="showEditForm(videoGame)">Edit</button>
					</td>
					<td v-if="selectedColumns.includes('name')">
						<button @click="deleteVideoGame(videoGame.name)">Delete</button>
					</td>
				</tr>
			</tbody>
		</table>
	</div>
</template>

<style scoped>
.container {
	width: 100%;
	max-width: 1000px;
	margin: 0 auto;
	padding: 1rem;
	box-sizing: border-box;
}

.inputField {
	display: flex;
	flex-direction: row;
	margin: 0.3rem;
}

.inputField label {
	width: 10rem;
}

.fetch-row {
	width: 100%;
	display: flex;
	flex-direction: row;
	align-items: center;
	justify-content: flex-start;
	flex-wrap: wrap;
}

.fetch-buttons {
	display: flex;
	flex-direction: column;
	align-items: flex-start;
	/* gap: 0.5rem; */
}

.column-select {
	display: flex;
	width: 80%;
	flex-wrap: wrap;
	gap: 0.5rem;
	margin-bottom: 1rem;
}

.column-select label {
	cursor: pointer;
}

h1 {
	text-align: center;
	margin-bottom: 1.5rem;
	color: #233333;
}
form {
	display: flex;
	flex-direction: column;
}

.formWrapper {
	display: flex;
	flex-direction: row;
	gap: 3rem;
	font-size: smaller;
}

.leftHalf {
	width: 50%;
	display: flex;
	flex-direction: column;
	height: 100%;
}

.rightHalf {
	width: 50%;
	height: 100%;
}

input,
textarea {
	width: 100%;
	padding: 0.5rem;
	border: 1px solid #ccc;
	border-radius: 4px;
	box-sizing: border-box;
}

.rating-filter input {
	width: 150px;
	border: 1px solid #ccc;
	border-radius: 4px;
	box-sizing: border-box;
	margin-bottom: 7px;
}

button {
	cursor: pointer;
	padding: 0.5rem 1rem;
	background-color: #007bff;
	color: #fff;
	font-weight: bold;
	border: none;
	border-radius: 4px;
	transition: background-color 0.3s;
	margin-bottom: 0.5rem;
}

button:last-child {
	margin-bottom: 0;
}

button:hover {
	background-color: #0056b3;
}

.add-button {
	width: 150px;
}

.cancel-button {
	background-color: #e26262;
	color: #fff;
	width: 138px;
}

.cancel-button:hover {
	background-color: #ff4949;
}

table {
	margin-top: 10px;
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
	border: 0.5px solid #dddddd;
	text-align: left;
	padding: 8px;
}

@media (max-width: 1024px) {
	.fetch-row {
		flex-direction: column;
		margin-bottom: 1rem;
		gap: 0.7rem;
	}
	table {
		margin-left: 1rem;
	}
	.column-select {
		gap: 0;
		margin-bottom: 0;
		margin-left: 1rem;
	}
	.add-button {
		display: flex;
		justify-content: center;
		margin: 1rem auto;
	}
	.fetch-buttons {
		display: flex;
		flex-direction: row;
		align-items: flex-start;
		gap: 0.5rem;
	}
}

@media (max-width: 768px) {
	.container {
		padding-left: 0;
		padding-right: 0;
	}
	table {
		display: block;
		overflow-x: auto;
		white-space: nowrap;
	}
	th,
	td {
		white-space: normal;
	}
}
</style>
