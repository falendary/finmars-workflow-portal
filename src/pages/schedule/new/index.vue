<template>
	<div class="schedule-edit-page">
		<h1>Edit Schedule</h1>

		<!-- Schedule Form -->
		<form  class="schedule-form">
			<table class="schedule-info-table">

				<tr>
					<th>User Code</th>
					<td><input v-model="schedule.user_code" type="text" required /></td>
				</tr>

				<tr>
					<th>Name</th>
					<td><input v-model="schedule.name" type="text" required /></td>
				</tr>

				<tr>
					<th>Workflow User Code</th>
					<td><input v-model="schedule.workflow_user_code" type="text" required /></td>
				</tr>

				<tr>
					<th>Crontab (UTC) Line</th>
					<td><input v-model="schedule.crontab_line" placeholder="54 12 * * *" type="text" required /></td>
				</tr>


				<tr>
					<th>Notes</th>
					<td><input v-model="schedule.notes" type="text" /></td>
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
			<div class="action-buttons" style="justify-content: space-between; display: flex">

				<NuxtLink :to="useGetNuxtLink(`/schedule`, $route.params)">
					<FmIcon icon="mdi-view-list" title="Back"/> Back
				</NuxtLink>

				<FmButton type="primary" class="save-btn" v-on:click="createSchedule()">Create</FmButton>
			</div>
		</form>
	</div>
</template>

<script setup>
import {VAceEditor} from 'vue3-ace-editor';
import 'ace-builds/src-noconflict/mode-json';
import 'ace-builds/src-noconflict/theme-monokai';

import {ref, onMounted} from 'vue';
import {useRoute, useRouter} from 'vue-router';
import {useGetNuxtLink} from "~/composables/useMeta";


let store = useStore();
store.init();
const route = useRoute();
const router = useRouter();

let schedulePayload = ref('{}')

definePageMeta({
	middleware: "auth",
});

let schedule = ref({is_manager: true, enabled: true});

// Save the updated schedule
async function createSchedule() {
	try {

		schedule.value.payload = JSON.parse(schedulePayload.value)

		const response = await useApi('schedule.post', {
			body: schedule.value,
		});

		useNotify({
			type: 'success',
			title: 'Success',
			text: 'Schedule created successfully!'
		});

		const id = response.id;
		router.push(`/${store.realm_code}/${store.space_code}/w/schedule/${id}/`);

	} catch (error) {
		useNotify({
			type: 'error',
			title: 'Error',
			text: 'Failed to create the schedule.'
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

// Fetch the schedule when the page loads
onMounted(() => {

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

.delete-btn {
	background-color: #ff4d4f;
	color: white;
	padding: 10px 20px;
}
</style>
