<template>
	<video-swiper class="video-swiper" :video-list="videoList" @play="onPlay" @pause="onPause" @timeupdate="onTimeUpdate"
	 @ended="onEnded" @error="onError" @waiting="onWaiting" @progress="onProgress" @loadedmetadata="onLoadedMetaData"></video-swiper>
</template>

<script>
	export default {
		data() {
			return {
				videoList: []
			}
		},
		async onLoad() {
			for (let i = 0; i < 5; i++) {
				this.videoList.push({
					id: i + 1,
					url: 'https://www.nihaowua.com/v/video.php?t=' + new Date().getTime()
				});
			}
		},
		onPlay(e) {},
		onPause(e) {},
		onEnded(e) {},
		onError(e) {},
		onWaiting(e) {},
		onTimeUpdate(e) {},
		onProgress(e) {},
		onLoadedMetaData(e) {
			console.log('LoadedMetaData', e)
		}
	}
</script>

<style>
	page {
		height: 100%;
	}
</style>
