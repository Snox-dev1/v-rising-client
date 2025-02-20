<template>
	<b-card
			bg-variant="dark"
			text-variant="white"
			no-body
	>
		<b-card-header>
			<div class="d-flex justify-content-between w-100 h3 card-title align-items-center">
				<div v-if="serverInfo && serverInfo.serverName">{{ serverInfo.serverName }}</div>
				<div v-else>{{ $t('server.info') }}</div>
				<div>
					<b-icon icon="broadcast" :variant="color"/>
				</div>
			</div>
			<div class="text-muted card-subtitle">
				{{ subtitle }}
			</div>
		</b-card-header>
		<b-list-group flush>
			<b-list-group-item
					v-if="rows.length === 0"
					variant="warning"
			>
				{{ $t('server.empty') }}
			</b-list-group-item>
			<b-list-group-item
					variant="dark"
					v-for="({title, value, icon, iconClick, iconTitle}, index) in rows"
					:key="index"
			>
				<div
						class="d-flex w-100 justify-content-between align-items-center"
				>
					<h6>{{ title }}</h6>
					<div class="text-right">
						{{ value || '-' }}
						<b-button v-b-tooltip v-if="icon" variant="outlined" size="sm" @click="iconClick" :title="iconTitle">
							<b-icon :icon="icon"></b-icon>
						</b-button>
					</div>
				</div>
			</b-list-group-item>
		</b-list-group>
	</b-card>
</template>

<script>
import {mapActions, mapGetters} from "vuex";
import dayjs from "dayjs";
import relativeTime from "dayjs/plugin/relativeTime";
import utc from 'dayjs/plugin/utc';
import {isFunction} from "lodash";
import {validationMixin} from "vuelidate";

dayjs.extend(relativeTime);
dayjs.extend(utc);

export default {
	name: "ServerInfoCard",
	mixins: [validationMixin],
	mounted() {
		this.loadServerInfo().catch(err => console.error('Failed to load server info', err));
	},
	computed: {
		...mapGetters(['serverInfo', 'isAdmin']),
		subtitle() {
			let subtitle = this.$t('server.state.offline');

			if (this.serverInfo.isSaveLoaded) {
				subtitle = this.$t('server.state.playerReady', this.serverInfo);
			} else if (this.serverInfo.serverSetupComplete) {
				subtitle = this.$t('server.state.configReady');
			}
			return subtitle;
		},
		color() {
			let color = 'danger';

			if (this.serverInfo.isSaveLoaded) {
				color = 'success';
			} else if (this.serverInfo.serverSetupComplete) {
				color = 'warning';
			}

			return color;
		},
		rows() {
			return [
				{
					title: this.$t('server.fields.pid'),
					value: this.serverInfo.pid,
					displayed: () => this.isAdmin && this.serverInfo.pid !== null
				},
				{
					title: this.$t('server.fields.time'),
					value: this.serverInfo.time && dayjs(this.serverInfo.time).isValid() ? dayjs(this.serverInfo.time).fromNow() : null,
					displayed: () => this.serverInfo.serverSetupComplete
				},
				{
					title: this.$t('server.fields.processExitCode'),
					value: this.serverInfo.processExitCode,
					displayed: () => this.isAdmin && this.serverInfo.processExitCode !== null
				},
				{
					title: this.$t('server.fields.version'),
					value: this.serverInfo.version,
					displayed: () => this.serverInfo.serverSetupComplete
				},
				{
					title: this.$t('server.fields.steamID'),
					value: this.serverInfo.steamID,
					displayed: () => this.serverInfo.connectedToSteam,
					icon: 'clipboard',
					iconTitle: this.$t('server.events.copySteamId.title'),
					iconClick: () => this.$copyText(this.serverInfo.steamID).then(() => {
						this.$bvToast.toast(this.$t('server.events.copySteamId.success', {steamId: this.serverInfo.steamID}), {
							autoHideDelay: 2000,
							title: this.$t('server.events.copySteamId.title'),
							toaster: 'b-toaster-bottom-center',
							variant: 'success'
						})
					}).catch(err => {
						this.$bvToast.toast(this.$t('server.events.copySteamId.error', {error: err}), {
							title: this.$t('server.events.copySteamId.title'),
							autoHideDelay: 2000,
							toaster: 'b-toaster-bottom-center',
							variant: 'danger'
						})
					})
				},
				{
					title: this.$t('server.fields.appID'),
					value: this.serverInfo.appID,
					displayed: () => this.serverInfo.connectedToSteam
				},
				{
					title: this.$t('server.fields.currentSaveNumber'),
					value: this.serverInfo.currentSaveNumber,
					displayed: () => this.serverInfo.isSaveLoaded,
					icon: 'download',
					iconClick: () => this.$bvModal.show('download-backup-modal'),
					iconTitle: this.$t('server.dialogs.downloadBackup.title')
				},
				{
					title: this.$t('server.fields.loadedSaveGameVersion'),
					value: this.serverInfo.loadedSaveGameVersion,
					displayed: () => this.serverInfo.isSaveLoaded && !this.serverInfo.isSaveVersionIdentical
				},
				{
					title: this.$t('server.fields.currentGameVersion'),
					value: this.serverInfo.currentGameVersion,
					displayed: () => this.serverInfo.isSaveLoaded && !this.serverInfo.isSaveVersionIdentical
				}
			].filter(row => isFunction(row.displayed) ? row.displayed() : true);
		}
	},
	methods: {
		...mapActions([
			'loadServerInfo'
		])
	}
}
</script>

<style scoped>

</style>
