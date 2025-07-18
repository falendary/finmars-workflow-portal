<template>
	<div class="workflow-detail-page">
		<!-- Right side: Workflow Details -->
		<div class="workflow-detail-section">


			<div class="pa-4 workflow-content-holder">

				<div class="v-data-table workflow-side">

					<NuxtLink :to="useGetNuxtLink(`/workflow`, $route.params)">
						<FmIcon icon="mdi-view-list" title="Launch"/>
						Back
					</NuxtLink>

					<h3>Workflow</h3>

					<div class="v-data-table__wrapper">
						<table>
							<tbody>
							<tr>
								<td><strong>ID</strong></td>
								<td>{{ workflow.id }}</td>
							</tr>
							<tr>
								<td><strong>User Code</strong></td>
								<td>{{ workflow.user_code }}</td>
							</tr>
							<tr>
								<td><strong>Workflow Status</strong></td>
								<td>
									<div class="chip" :style="{ backgroundColor: getColor(workflow.status) }">
										{{ workflow.status }}
									</div>
								</td>
							</tr>
							<tr v-if="workflow.created_at">
								<td><strong>Created</strong></td>
								<td v-bind:title="workflow.created_at">{{ formatDate(workflow.created_at) }}</td>
							</tr>
							<tr v-if="workflow.modified_at">
								<td><strong>Updated</strong></td>
								<td v-bind:title="workflow.modified_at">{{ formatDate(workflow.modified_at) }}</td>
							</tr>
							<tr v-if="workflow.finished_at">
								<td><strong>Finished</strong></td>
								<td v-bind:title="workflow.finished_at">{{ formatDate(workflow.finished_at) }}</td>
							</tr>
							</tbody>
						</table>
					</div>


					<h3>Workflow Tasks</h3>

					<div v-for="task in workflow.tasks" :key="task.id">

						<div @click="selectedTask = task" :class="{ active: selectedTask === task }" class="task-button"
							 type="secondary">
							[{{ task.id }}] {{ task.name }} &nbsp;
							<div class="chip" :style="{ backgroundColor: getColor(task.status) }">
								{{ task.status }}
							</div>
						</div>
					</div>

				</div>

				<div style="padding: 0 12px;" class="workflow-task-side">

					<div class="button-group">
						<FmButton @click="refresh" type="secondary">
							<FmIcon icon="mdi-refresh" title="Refresh"/>
							Refresh
						</FmButton>
						<FmButton @click="confirmRelaunch" type="secondary">
							<FmIcon icon="mdi-play" title="Relaunch"/>
							Relaunch
						</FmButton>
						<FmButton @click="cancelWorkflow"
								  v-if="workflow.status === 'init' || workflow.status === 'progress'" type="secondary">
							<FmIcon icon="mdi-cancel" title="Terminate"/>
							Cancel
						</FmButton>
						<FmButton @click="payloadDialog = true" type="secondary">
							<FmIcon icon="mdi-list-box-outline" title="Payload"/>
							Payload
						</FmButton>

						<FmButton @click="jsonDialog = true" type="secondary">
							<FmIcon icon="mdi-code-json" title="Json"/>
							JSON
						</FmButton>


					</div>

					<div v-if="selectedTask" class="pa-4" style="padding-left: 16px;  box-sizing: border-box">

						<h3>Task</h3>

						<div class="v-data-table__wrapper">
							<table>
								<tbody>
								<tr>
									<td><strong>Celery Task ID</strong></td>
									<td>{{ selectedTask.celery_task_id }}</td>
								</tr>

								<tr v-if="selectedTask.celery_task_id">

									<td><strong>Debug</strong></td>
									<td>

										<a target="_blank" :href="getFlowerTaskUrl()" class="pa-6">
											<FmButton type="secondary">
												<FmIcon icon="mdi-flower"/>
												View in Flower
											</FmButton>
										</a>

									</td>

								</tr>

								<tr>
									<td><strong>ID</strong></td>
									<td>{{ selectedTask.id }}</td>
								</tr>
								<tr>
									<td><strong>User Code</strong></td>
									<td>{{ selectedTask.name }}</td>
								</tr>
								<tr>
									<td><strong>Task Status</strong></td>
									<td>
										<div class="chip" :style="{ backgroundColor: getColor(selectedTask.status) }">
											{{ selectedTask.status }}
										</div>
									</td>
								</tr>
								<tr>
									<td><strong>Created</strong></td>
									<td v-bind:title="selectedTask.created_at">
										{{ formatDate(selectedTask.created_at) }}
									</td>
								</tr>
								<tr>
									<td><strong>Updated</strong></td>
									<td v-bind:title="selectedTask.modified_at">{{
											formatDate(selectedTask.modified_at)
										}}
									</td>
								</tr>
								<tr v-if="selectedTask.finished_at">
									<td><strong>Finished</strong></td>
									<td v-bind:title="selectedTask.finished_at">{{
											formatDate(selectedTask.finished_at)
										}}
									</td>
								</tr>

								<tr>
									<td><strong>Workflow ID</strong></td>
									<td>{{ selectedTask.workflow }}</td>
								</tr>

								<tr>
									<td><strong>Worker</strong></td>
									<td>{{ selectedTask.worker_name }}</td>
								</tr>


								</tbody>
							</table>
						</div>


						<div v-if="selectedTask.log">
							<h2 class="text-center" style="margin-bottom: 8px">Log</h2>
							<code style="max-height: 300px">{{ selectedTask.log }}</code>
						</div>

						<div v-if="selectedTask.progress">
							<h2 class="text-center" style="margin-bottom: 8px">Progress</h2>
							<code>{{ selectedTask.progress }}</code>
						</div>

						<div class="row mt-4" v-if="!selectedTask.result">
							<div class="col">
								<div class="notice">
									<FmIcon icon="mdi-information"/>
									No task result
								</div>
							</div>
						</div>

						<div v-else>
							<h2 class="text-center" style="margin-bottom: 8px">Result</h2>
							<code v-if="selectedTask.result && selectedTask.status !== 'error'">{{
									selectedTask.result
								}}</code>
							<div v-if="selectedTask.result && selectedTask.status === 'error'">
								<div class="row">
									<div class="col">
										<div class="alert">
											<FmIcon icon="mdi-information"/>

											{{ selectedTask.result.exception }}
										</div>
									</div>
								</div>
								<code>{{ selectedTask.result.traceback }}</code>
							</div>
						</div>
					</div>


				</div>

			</div>


			<BaseModal v-model="payloadDialog" title="Workflow's Payload" width="500">
				<div>
					<pre>{{ workflow.payload }}</pre>
				</div>
				<template #controls="{ cancel }">
					<div class="flex flex-row justify-between">
						<FmButton type="secondary" @click="payloadDialog = false">Cancel</FmButton>
					</div>
				</template>
			</BaseModal>


			<BaseModal v-model="jsonDialog" title="Workflow's Payload" width="500">
				<div>
					<pre>{{ workflow }}</pre>
				</div>


				<template #controls="{ cancel }">
					<div class="flex flex-row justify-between">
						<FmButton type="secondary" @click="jsonDialog = false">Cancel</FmButton>
					</div>
				</template>
			</BaseModal>

		</div>
	</div>
</template>
<script setup>

import {FmButton, FmIcon} from "@finmars/ui"
import {useGetNuxtLink} from "~/composables/useMeta";

const route = useRoute();
const store = useStore();

const props = defineProps({
	workflow: Object,
});
const emit = defineEmits(['update'])

const workflow = ref(props.workflow);
const selectedTask = ref(props.workflow.tasks?.[0]);
const payloadDialog = ref(false);
const jsonDialog = ref(false);

function getColor(status) {
	return {
		success: 'green',
		error: 'red',
		canceled: '#B71C1C',
		warning: 'orange',
		progress: 'blue',
		init: 'grey'
	}[status];
}

function countTasksByStatus(status) {
	return props.workflow.tasks?.filter((task) => task.status === status)?.length;
}

function getFlowerTaskUrl() {
	return (
		window.location.origin +
		`/${store.realm_code}/workflow/flower/task/` +
		selectedTask.value.celery_task_id
	);
}

async function cancelWorkflow() {
	let isConfirm = await useConfirm({
		title: 'Cancel the workflow',
		text: `Are you sure you want to cancel Workflow?`
	});
	if (!isConfirm) return false;

	await useApi('cancelWorkflow.post', {
		params: {id: route.params.id},
		body: JSON.stringify({})
	});

	await refresh();
}

async function refresh() {
	emit('update');
}

async function confirmRelaunch() {
	const isConfirm = await useConfirm({
		title: 'Relaunch the workflow',
		text: `The workflow will be relaunched with the same payload. Note that a new workflow will be created, so the current one (created on ) will not be changed and will still be available in your history.`
	});

	if (!isConfirm) return false;

	await useApi('relaunchWorkflow.post', {
		params: {id: route.params.id},
		body: JSON.stringify({})
	});

	await refresh();
}

function formatDate(dateString) {
	const options = {
		year: 'numeric',
		month: 'short',
		day: 'numeric',
		hour: 'numeric',
		minute: 'numeric',
		second: 'numeric'
	};
	return new Date(dateString).toLocaleDateString(undefined, options);
}
</script>

<style scoped>
.workflow-detail-page {
	display: flex;
}

/* Left side for Rete.js Editor */
.workflow-graph-section {
	flex: 0.3;
	background-color: #f5f5f5;
	padding: 15px;
}

/* Right side for Workflow Details */
.workflow-detail-section {
	flex: 1;
	background-color: #fff;
	padding: 20px;
	border-left: 1px solid #ccc;
	overflow-y: auto;
	z-index: 0;
}

h2 {
	margin-top: 0;
	font-size: 1.5rem;
}

h3 {
	margin-top: 1rem;
	font-size: 1.2rem;
}

/* Button Group Styles */
.button-group {
	max-width: 600px;
	display: flex;
	justify-content: space-between;
	margin-bottom: 15px;
}

.chip {
	text-transform: Capitalize;
	align-items: center;
	cursor: default;
	display: inline-flex;
	line-height: 20px;
	max-width: 100%;
	outline: none;
	overflow: hidden;
	padding: 4px 12px 4px;
	position: relative;
	text-decoration: none;
	transition-duration: 0.28s;
	transition-property: box-shadow, opacity;
	transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
	vertical-align: middle;
	white-space: nowrap;
	color: white;
	border-radius: 12px;
}

code {
	white-space: pre-wrap;
	overflow-x: auto;
	padding: 10px;
	display: block !important;
	white-space: pre;
	background-color: #f5f5f5;
	box-shadow: none;
	font-weight: normal;
	font-size: 90%;
	max-width: 50vw;
}

td {
	font-size: 0.875rem;
	height: 48px;
	padding: 0 16px;
	border: 1px solid rgb(255, 219, 208);
	color: rgb(35, 25, 23);
}

.footer {
	display: flex;
	padding: 16px 0;
	justify-content: center;
}

.headline {
	margin-bottom: 8px;
}

.alert {
	display: flex;
	gap: 16px;
	padding: 16px;
	color: #E53935;
	caret-color: #E53935;

	border: thin solid #E53935;
	border-radius: 4px;
	background-color: rgba(229, 57, 53, 0.12);

	margin-bottom: 16px;
}

.notice {
	display: flex;
	gap: 16px;
	padding: 16px;
	color: #000000;
	caret-color: #000000;

	border: thin solid #000000;
	border-radius: 4px;
	background-color: rgba(000, 000, 000, 0.12);

	margin-bottom: 16px;
}

.task-button {
	cursor: pointer;

	&:hover {
		opacity: .8;
	}
}

.task-button.btn.text.active {
	background-color: rgba(143, 76, 54, .6);
	color: #fff;


}

.task-button.btn.text.active:hover {
	background-color: rgba(143, 76, 54, .6);
	opacity: .9;
}

h3 {
	margin-bottom: 16px;
}

.workflow-side {
	border-right: 1px solid #f1f1f1;
	padding-right: 24px;
	flex: 3; /* 3 parts out of 10 */
	max-width: 35vw;
}

.workflow-task-side {
	flex: 7; /* 7 parts out of 10 */
	position: relative; /* establish a new positioning context */
}

.button-group {
	background-color: white; /* cover what’s behind it */
	z-index: 10; /* stay on top of other stuff */
}

.workflow-content-holder {
	display: flex;
}

</style>
