<template>
	<view class="container">
		<swiper class="video-swiper" :easing-function="easingFunction" vertical current="1" :duration="duration"
		 @animationfinish="animationfinish">
			<!-- curQueue 循环会导致video重新插入，objectFit 不可变更 -->
			<swiper-item v-for="(v,i) in curQueue" :key="v.id">
				<video :id="'video_' + i" class="video_item" :loop="loop" enable-play-gesture enable-progress-gesture
				 :show-center-play-btn="false" :controls="false" :src="v.url" :data-id="v.id" :object-fit="item.objectFit || 'cover'"
				 :data-index="i">
				</video>
			</swiper-item>
		</swiper>
	</view>
</template>

<script>
	export default {
		props: {
			duration: {
				type: Number,
				value: 500
			},
			easingFunction: {
				type: String,
				value: 'default'
			},
			loop: {
				type: Boolean,
				value: true
			},
			videoList: {
				type: Array,
				value: [],
				observer: function observer() {
					var newVal = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : [];
					console.log(newVal);
					this._videoListChanged(newVal);
				}
			}
		},
		data() {
			return {
				nextQueue: [],
				prevQueue: [],
				curQueue: [],
				circular: false,
				_last: 1,
				_change: -1,
				_invalidUp: 0,
				_invalidDown: 0,
				_videoContexts: []
			}
		},
		watch: {
			videoList() {
				this._videoContexts = [
					wx.createVideoContext('video_0', this),
					wx.createVideoContext('video_1', this),
					wx.createVideoContext('video_2', this),
					wx.createVideoContext('video_3', this),
					wx.createVideoContext('video_4', this),
				];
				this._videoListChanged();
			}
		},
		methods: {
			_videoListChanged() {
				var _this = this;

				this.videoList.forEach(function(item) {
					_this.nextQueue.push(item);
				});
				if (_this.curQueue.length === 0) {
					this.curQueue = _this.nextQueue.splice(0, 5);
					_this.playCurrent(1);
				}
			},
			animationfinish(e) {
				var _last = this._last,
					_change = this._change,
					curQueue = this.curQueue,
					prevQueue = this.prevQueue,
					nextQueue = this.nextQueue;

				var current = e.detail.current;
				var diff = current - _last;
				if (diff === 0) return;
				this._last = current;
				this.playCurrent(current);
				this.$emit('change', {
					activeId: curQueue[current].id
				});
				var direction = diff === 1 || diff === -2 ? 'up' : 'down';
				if (direction === 'up') {
					if (this._invalidDown === 0) {
						var change = (_change + 1) % 3;
						var add = nextQueue.shift();
						var remove = curQueue[change];
						if (add) {
							prevQueue.push(remove);
							curQueue[change] = add;
							this._change = change;
						} else {
							this._invalidUp += 1;
						}
					} else {
						this._invalidDown -= 1;
					}
				}
				if (direction === 'down') {
					if (this._invalidUp === 0) {
						var _change2 = _change;
						var _remove = curQueue[_change2];
						var _add = prevQueue.pop();
						if (_add) {
							curQueue[_change2] = _add;
							nextQueue.unshift(_remove);
							this._change = (_change2 - 1 + 3) % 3;
						} else {
							this._invalidDown += 1;
						}
					} else {
						this._invalidUp -= 1;
					}
				}
				var circular = true;
				if (nextQueue.length === 0 && current !== 0) {
					circular = false;
				}
				if (prevQueue.length === 0 && current !== 2) {
					circular = false;
				}
				this.curQueue = curQueue;
				this.circular = circular;
			},
			playCurrent(current) {
				if (this._videoContexts)
					this._videoContexts.forEach(function(ctx, index) {
						index !== current ? ctx.pause() : ctx.play();
					});
			},
			onPlay(e) {
				this.trigger(e, 'play');
			},
			onPause(e) {
				this.trigger(e, 'pause');
			},
			onEnded(e) {
				this.trigger(e, 'ended');
			},
			onError(e) {
				this.trigger(e, 'error');
			},
			onTimeUpdate(e) {
				this.trigger(e, 'timeupdate');
			},
			onWaiting(e) {
				this.trigger(e, 'wait');
			},
			onProgress(e) {
				this.trigger(e, 'progress');
			},
			onLoadedMetaData(e) {
				this.trigger(e, 'loadedmetadata');
			},
			trigger(e, type) {
				var ext = arguments.length > 2 && arguments[2] !== undefined ? arguments[2] : {};

				var detail = e.detail;
				var activeId = e.target.dataset.id;
				this.$emit(type, Object.assign(Object.assign(Object.assign({}, detail), {
					activeId: activeId
				}), ext));
			}
		}
	}
</script>

<style>
	.container {
		width: 100%;
		height: 100%
	}

	.video-swiper {
		width: 100%;
		height: 100%
	}

	.video_item {
		height: 100%;
		width: 100%
	}
</style>
