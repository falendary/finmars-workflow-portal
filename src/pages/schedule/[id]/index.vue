<template>
	<div class="schedule-edit-page">

		<NuxtLink :to="useGetNuxtLink(`/schedule`, $route.params)">
			<FmIcon icon="mdi-view-list" title="Back"/> Back
		</NuxtLink>


		<h1>Edit Schedule</h1>

		<!-- Schedule Form -->
		<form  class="schedule-form">
			<table class="schedule-info-table">
				<tr>
					<th>ID</th>
					<td>{{ schedule.id }}</td>
				</tr>
				<tr>
					<th>User Code</th>
					<td><input v-model="schedule.user_code" type="text" required/></td>
				</tr>

				<tr>
					<th>Name</th>
					<td><input v-model="schedule.name" type="text" required/></td>
				</tr>

				<tr>
					<th>Workflow User Code</th>
					<td><input v-model="schedule.workflow_user_code" type="text" required/></td>
				</tr>

				<tr>
					<th>Crontab (UTC) Line</th>
					<td><input v-model="schedule.crontab_line" placeholder="54 12 * * *" type="text" required/></td>
				</tr>


				<tr>
					<th>Notes</th>
					<td><input v-model="schedule.notes" type="text"/></td>
				</tr>

				<tr>
					<th>Payload</th>
					<td>

						<v-ace-editor
							v-model:value="schedulePayload"
							@init="payloadEditorInit"
							lang="json"
							theme="monokai"
							style="height: 300px;width: 100%;"/>


					</td>
				</tr>


				<tr>
					<th>Created</th>
					<td>{{ formatDate(schedule.created_at) }}</td>
				</tr>
				<tr>
					<th>Modified</th>
					<td>{{ formatDate(schedule.modified_at) }}</td>
				</tr>

				<tr>
					<th>Next Run At</th>
					<td><span :title="'Server Time: ' + schedule.next_run_at">{{ formatDate(schedule.next_run_at) }}</span></td>
				</tr>
				<tr>
					<th>Last Run At</th>
					<td><span :title="'Server Time: ' + schedule.last_run_at">{{ formatDate(schedule.last_run_at) }}</span></td>
				</tr>

				<tr>
					<th>Owner</th>
					<td>{{ schedule.owner_username }}</td>
				</tr>
				<tr>
					<th>Manager</th>
					<td>
						<select v-model="schedule.is_manager">
							<option :value="true">Yes</option>
							<option :value="false">No</option>
						</select>
					</td>
				</tr>
				<tr>
					<th>Enabled</th>
					<td>
						<select v-model="schedule.enabled">
							<option :value="true">Yes</option>
							<option :value="false">No</option>
						</select>
					</td>
				</tr>
			</table>

			<!-- Actions -->
			<div class="action-buttons" style="display: flex; justify-content: space-between">
				<FmButton  @click="deleteSchedule" class="delete-btn" type="secondary">Delete</FmButton>
				<FmButton type="primary" v-on:click="saveSchedule()" class="save-btn">Save</FmButton>
			</div>

		</form>


		<div style="margin-top: 8px;">

		</div>

		<hr style="margin: 24px 0">

		<FmButton @click="runManual"  type="secondary">Run Manually</FmButton>
	</div>
</template>

<script setup>

import {VAceEditor} from 'vue3-ace-editor';
import 'ace-builds/src-noconflict/mode-json';
import 'ace-builds/src-noconflict/theme-monokai';

import {onMounted, ref} from 'vue';
import {useRoute, useRouter} from 'vue-router';
import {useGetNuxtLink} from "~/composables/useMeta";

let schedulePayload = ref('')


const route = useRoute();
const router = useRouter();

let store = useStore();
store.init();
definePageMeta({
	middleware: "auth",
});

let schedule = ref({});

// Fetch the schedule data
async function getSchedule() {
	const response = await useApi('schedule.get', {params: {id: route.params.id}});
	schedule.value = response;

	if (schedule.value.payload) {
		schedulePayload.value = JSON.stringify(schedule.value.payload, null, 4)
	}
}

// Save the updated schedule
async function saveSchedule() {
	try {

		schedule.value.payload = JSON.parse(schedulePayload.value)

		await useApi('schedule.put', {
			body: schedule.value,
			params: {id: route.params.id}
		});
		useNotify({
			type: 'success',
			title: 'Success',
			text: 'Schedule updated successfully!'
		});
	} catch (error) {
		useNotify({
			type: 'error',
			title: 'Error',
			text: 'Failed to update the schedule.'
		});
	}
}

function payloadEditorInit(payloadEditor) {
	payloadEditor.setHighlightActiveLine(false);
	payloadEditor.setShowPrintMargin(false);
	payloadEditor.setFontSize(14)
	payloadEditor.setBehavioursEnabled(true);

	payloadEditor.focus();
	payloadEditor.navigateFileStart();
}


// Delete the schedule
async function deleteSchedule() {
	try {

		let isConfirm = await useConfirm({
			text: `Are you sure you want to delete Schedule?`,
		})
		if (!isConfirm) return false

		await useApi('schedule.delete', {
			params: {id: route.params.id}
		});
		useNotify({
			type: 'success',
			title: 'Success',
			text: 'Schedule deleted successfully!'
		});

		router.push(`/${store.realm_code}/${store.space_code}/w/schedule/`);

	} catch (error) {
		useNotify({
			type: 'error',
			title: 'Error',
			text: 'Failed to delete the schedule.'
		});
	}
}

async function runManual() {

	await useApi('scheduleRunManual.put', {
		params: {id: route.params.id}
	});
	useNotify({
		type: 'success',
		title: 'Success',
		text: 'Schedule is triggered manually!'
	});
}

// Format the date to a more readable format
function formatDate(dateString) {
	const options = {year: 'numeric', month: 'short', day: 'numeric', hour: 'numeric', minute: 'numeric'};
	return new Date(dateString).toLocaleDateString(undefined, options);
}

// Fetch the schedule when the page loads
onMounted(() => {
	getSchedule();
});
</script>

<style scoped lang="postcss">
.schedule-edit-page {
	padding: 20px;
}

h1 {
	font-size: 2rem;
	font-weight: bold;
	margin-bottom: 20px;
}

.schedule-info-table {
	width: 100%;
	border-collapse: collapse;
	margin-bottom: 20px;
}

.schedule-info-table th, .schedule-info-table td {
	padding: 10px;
	border-bottom: 1px solid #ddd;
	text-align: left;
}

.schedule-info-table th {
	background-color: #f5f5f5;
	width: 150px;
	font-weight: bold;
}

input, select {
	width: 100%;
	padding: 8px;
	border: 1px solid #ccc;
	border-radius: 4px;
}

.action-buttons {
	margin-top: 20px;
}

.save-btn {
	background-color: #007bff;
	color: white;
	padding: 10px 20px;
	margin-right: 10px;
}

.btn.delete-btn {
	background-color: #ff4d4f !important;
	color: white !important;
	padding: 10px 20px;
}
</style>
