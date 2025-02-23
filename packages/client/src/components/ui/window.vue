<template>
<transition :name="$store.state.animation ? 'window' : ''" appear @after-leave="$emit('closed')">
	<div v-if="showing" class="ebkgocck">
		<div class="body _shadow _narrow_" @mousedown="onBodyMousedown" @keydown="onKeydown">
			<div class="header" :class="{ mini }" @contextmenu.prevent.stop="onContextmenu">
				<span class="left">
					<button v-for="button in buttonsLeft" v-tooltip="button.title" class="button _button" :class="{ highlighted: button.highlighted }" @click="button.onClick"><i :class="button.icon"></i></button>
				</span>
				<span class="title" @mousedown.prevent="onHeaderMousedown" @touchstart.prevent="onHeaderMousedown">
					<slot name="header"></slot>
				</span>
				<span class="right">
					<button v-for="button in buttonsRight" v-tooltip="button.title" class="button _button" :class="{ highlighted: button.highlighted }" @click="button.onClick"><i :class="button.icon"></i></button>
					<button v-if="closeButton" class="button _button" @click="close()"><i class="fas fa-times"></i></button>
				</span>
			</div>
			<div class="body">
				<slot></slot>
			</div>
		</div>
		<template v-if="canResize">
			<div class="handle top" @mousedown.prevent="onTopHandleMousedown"></div>
			<div class="handle right" @mousedown.prevent="onRightHandleMousedown"></div>
			<div class="handle bottom" @mousedown.prevent="onBottomHandleMousedown"></div>
			<div class="handle left" @mousedown.prevent="onLeftHandleMousedown"></div>
			<div class="handle top-left" @mousedown.prevent="onTopLeftHandleMousedown"></div>
			<div class="handle top-right" @mousedown.prevent="onTopRightHandleMousedown"></div>
			<div class="handle bottom-right" @mousedown.prevent="onBottomRightHandleMousedown"></div>
			<div class="handle bottom-left" @mousedown.prevent="onBottomLeftHandleMousedown"></div>
		</template>
	</div>
</transition>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import contains from '@/scripts/contains';
import * as os from '@/os';

const minHeight = 50;
const minWidth = 250;

function dragListen(fn) {
	window.addEventListener('mousemove', fn);
	window.addEventListener('touchmove', fn);
	window.addEventListener('mouseleave', dragClear.bind(null, fn));
	window.addEventListener('mouseup', dragClear.bind(null, fn));
	window.addEventListener('touchend', dragClear.bind(null, fn));
}

function dragClear(fn) {
	window.removeEventListener('mousemove', fn);
	window.removeEventListener('touchmove', fn);
	window.removeEventListener('mouseleave', dragClear);
	window.removeEventListener('mouseup', dragClear);
	window.removeEventListener('touchend', dragClear);
}

export default defineComponent({
	provide: {
		inWindow: true,
	},

	props: {
		initialWidth: {
			type: Number,
			required: false,
			default: 400,
		},
		initialHeight: {
			type: Number,
			required: false,
			default: null,
		},
		canResize: {
			type: Boolean,
			required: false,
			default: false,
		},
		closeButton: {
			type: Boolean,
			required: false,
			default: true,
		},
		mini: {
			type: Boolean,
			required: false,
			default: false,
		},
		front: {
			type: Boolean,
			required: false,
			default: false,
		},
		contextmenu: {
			type: Array,
			required: false,
		},
		buttonsLeft: {
			type: Array,
			required: false,
			default: () => [],
		},
		buttonsRight: {
			type: Array,
			required: false,
			default: () => [],
		},
	},

	emits: ['closed'],

	data() {
		return {
			showing: true,
			id: Math.random().toString(), // TODO: UUIDとかにする
		};
	},

	mounted() {
		if (this.initialWidth) this.applyTransformWidth(this.initialWidth);
		if (this.initialHeight) this.applyTransformHeight(this.initialHeight);

		this.applyTransformTop((window.innerHeight / 2) - (this.$el.offsetHeight / 2));
		this.applyTransformLeft((window.innerWidth / 2) - (this.$el.offsetWidth / 2));

		// 他のウィンドウ内のボタンなどを押してこのウィンドウが開かれた場合、親が最前面になろうとするのでそれに隠されないようにする
		this.top();

		window.addEventListener('resize', this.onBrowserResize);
	},

	unmounted() {
		window.removeEventListener('resize', this.onBrowserResize);
	},

	methods: {
		close() {
			this.showing = false;
		},

		onKeydown(evt) {
			if (evt.which === 27) { // Esc
				evt.preventDefault();
				evt.stopPropagation();
				this.close();
			}
		},

		onContextmenu(ev: MouseEvent) {
			if (this.contextmenu) {
				os.contextMenu(this.contextmenu, ev);
			}
		},

		// 最前面へ移動
		top() {
			(this.$el as any).style.zIndex = os.claimZIndex(this.front ? 'middle' : 'low');
		},

		onBodyMousedown() {
			this.top();
		},

		onHeaderMousedown(evt: MouseEvent) {
			// 右クリックはコンテキストメニューを開こうとした可能性が高いため無視
			if (evt.button === 2) return;

			const main = this.$el as any;

			if (!contains(main, document.activeElement)) main.focus();

			const position = main.getBoundingClientRect();

			const clickX = evt.touches && evt.touches.length > 0 ? evt.touches[0].clientX : evt.clientX;
			const clickY = evt.touches && evt.touches.length > 0 ? evt.touches[0].clientY : evt.clientY;
			const moveBaseX = clickX - position.left;
			const moveBaseY = clickY - position.top;
			const browserWidth = window.innerWidth;
			const browserHeight = window.innerHeight;
			const windowWidth = main.offsetWidth;
			const windowHeight = main.offsetHeight;

			// 動かした時
			dragListen(me => {
				const x = me.touches && me.touches.length > 0 ? me.touches[0].clientX : me.clientX;
				const y = me.touches && me.touches.length > 0 ? me.touches[0].clientY : me.clientY;

				let moveLeft = x - moveBaseX;
				let moveTop = y - moveBaseY;

				// 下はみ出し
				if (moveTop + windowHeight > browserHeight) moveTop = browserHeight - windowHeight;

				// 左はみ出し
				if (moveLeft < 0) moveLeft = 0;

				// 上はみ出し
				if (moveTop < 0) moveTop = 0;

				// 右はみ出し
				if (moveLeft + windowWidth > browserWidth) moveLeft = browserWidth - windowWidth;

				this.$el.style.left = moveLeft + 'px';
				this.$el.style.top = moveTop + 'px';
			});
		},

		// 上ハンドル掴み時
		onTopHandleMousedown(evt) {
			const main = this.$el as any;

			const base = evt.clientY;
			const height = parseInt(getComputedStyle(main, '').height, 10);
			const top = parseInt(getComputedStyle(main, '').top, 10);

			// 動かした時
			dragListen(me => {
				const move = me.clientY - base;
				if (top + move > 0) {
					if (height + -move > minHeight) {
						this.applyTransformHeight(height + -move);
						this.applyTransformTop(top + move);
					} else { // 最小の高さより小さくなろうとした時
						this.applyTransformHeight(minHeight);
						this.applyTransformTop(top + (height - minHeight));
					}
				} else { // 上のはみ出し時
					this.applyTransformHeight(top + height);
					this.applyTransformTop(0);
				}
			});
		},

		// 右ハンドル掴み時
		onRightHandleMousedown(evt) {
			const main = this.$el as any;

			const base = evt.clientX;
			const width = parseInt(getComputedStyle(main, '').width, 10);
			const left = parseInt(getComputedStyle(main, '').left, 10);
			const browserWidth = window.innerWidth;

			// 動かした時
			dragListen(me => {
				const move = me.clientX - base;
				if (left + width + move < browserWidth) {
					if (width + move > minWidth) {
						this.applyTransformWidth(width + move);
					} else { // 最小の幅より小さくなろうとした時
						this.applyTransformWidth(minWidth);
					}
				} else { // 右のはみ出し時
					this.applyTransformWidth(browserWidth - left);
				}
			});
		},

		// 下ハンドル掴み時
		onBottomHandleMousedown(evt) {
			const main = this.$el as any;

			const base = evt.clientY;
			const height = parseInt(getComputedStyle(main, '').height, 10);
			const top = parseInt(getComputedStyle(main, '').top, 10);
			const browserHeight = window.innerHeight;

			// 動かした時
			dragListen(me => {
				const move = me.clientY - base;
				if (top + height + move < browserHeight) {
					if (height + move > minHeight) {
						this.applyTransformHeight(height + move);
					} else { // 最小の高さより小さくなろうとした時
						this.applyTransformHeight(minHeight);
					}
				} else { // 下のはみ出し時
					this.applyTransformHeight(browserHeight - top);
				}
			});
		},

		// 左ハンドル掴み時
		onLeftHandleMousedown(evt) {
			const main = this.$el as any;

			const base = evt.clientX;
			const width = parseInt(getComputedStyle(main, '').width, 10);
			const left = parseInt(getComputedStyle(main, '').left, 10);

			// 動かした時
			dragListen(me => {
				const move = me.clientX - base;
				if (left + move > 0) {
					if (width + -move > minWidth) {
						this.applyTransformWidth(width + -move);
						this.applyTransformLeft(left + move);
					} else { // 最小の幅より小さくなろうとした時
						this.applyTransformWidth(minWidth);
						this.applyTransformLeft(left + (width - minWidth));
					}
				} else { // 左のはみ出し時
					this.applyTransformWidth(left + width);
					this.applyTransformLeft(0);
				}
			});
		},

		// 左上ハンドル掴み時
		onTopLeftHandleMousedown(evt) {
			this.onTopHandleMousedown(evt);
			this.onLeftHandleMousedown(evt);
		},

		// 右上ハンドル掴み時
		onTopRightHandleMousedown(evt) {
			this.onTopHandleMousedown(evt);
			this.onRightHandleMousedown(evt);
		},

		// 右下ハンドル掴み時
		onBottomRightHandleMousedown(evt) {
			this.onBottomHandleMousedown(evt);
			this.onRightHandleMousedown(evt);
		},

		// 左下ハンドル掴み時
		onBottomLeftHandleMousedown(evt) {
			this.onBottomHandleMousedown(evt);
			this.onLeftHandleMousedown(evt);
		},

		// 高さを適用
		applyTransformHeight(height) {
			if (height > window.innerHeight) height = window.innerHeight;
			(this.$el as any).style.height = height + 'px';
		},

		// 幅を適用
		applyTransformWidth(width) {
			if (width > window.innerWidth) width = window.innerWidth;
			(this.$el as any).style.width = width + 'px';
		},

		// Y座標を適用
		applyTransformTop(top) {
			(this.$el as any).style.top = top + 'px';
		},

		// X座標を適用
		applyTransformLeft(left) {
			(this.$el as any).style.left = left + 'px';
		},

		onBrowserResize() {
			const main = this.$el as any;
			const position = main.getBoundingClientRect();
			const browserWidth = window.innerWidth;
			const browserHeight = window.innerHeight;
			const windowWidth = main.offsetWidth;
			const windowHeight = main.offsetHeight;
			if (position.left < 0) main.style.left = 0; // 左はみ出し
			if (position.top + windowHeight > browserHeight) main.style.top = browserHeight - windowHeight + 'px'; // 下はみ出し
			if (position.left + windowWidth > browserWidth) main.style.left = browserWidth - windowWidth + 'px'; // 右はみ出し
			if (position.top < 0) main.style.top = 0; // 上はみ出し
		},
	},
});
</script>

<style lang="scss" scoped>
.window-enter-active, .window-leave-active {
	transition: opacity 0.2s, transform 0.2s !important;
}
.window-enter-from, .window-leave-to {
	pointer-events: none;
	opacity: 0;
	transform: scale(0.9);
}

.ebkgocck {
	position: fixed;
	top: 0;
	left: 0;

	> .body {
		overflow: hidden;
		display: flex;
		flex-direction: column;
		contain: content;
		width: 100%;
		height: 100%;
		border-radius: var(--radius);

		> .header {
			--height: 45px;

			&.mini {
				--height: 38px;
			}

			display: flex;
			position: relative;
			z-index: 1;
			flex-shrink: 0;
			user-select: none;
			height: var(--height);
			background: var(--windowHeader);
			-webkit-backdrop-filter: var(--blur, blur(15px));
			backdrop-filter: var(--blur, blur(15px));
			//border-bottom: solid 1px var(--divider);
			font-size: 95%;
			font-weight: bold;

			> .left, > .right {
				> .button {
					height: var(--height);
					width: var(--height);

					&:hover {
						color: var(--fgHighlighted);
					}

					&.highlighted {
						color: var(--accent);
					}
				}
			}

			> .left {
				margin-right: 16px;
			}

			> .right {
				min-width: 16px;
			}

			> .title {
				flex: 1;
				position: relative;
				line-height: var(--height);
				white-space: nowrap;
				overflow: hidden;
				text-overflow: ellipsis;
				cursor: move;
			}
		}

		> .body {
			flex: 1;
			overflow: auto;
			background: var(--panel);
		}
	}

	> .handle {
		$size: 8px;

		position: absolute;

		&.top {
			top: -($size);
			left: 0;
			width: 100%;
			height: $size;
			cursor: ns-resize;
		}

		&.right {
			top: 0;
			right: -($size);
			width: $size;
			height: 100%;
			cursor: ew-resize;
		}

		&.bottom {
			bottom: -($size);
			left: 0;
			width: 100%;
			height: $size;
			cursor: ns-resize;
		}

		&.left {
			top: 0;
			left: -($size);
			width: $size;
			height: 100%;
			cursor: ew-resize;
		}

		&.top-left {
			top: -($size);
			left: -($size);
			width: $size * 2;
			height: $size * 2;
			cursor: nwse-resize;
		}

		&.top-right {
			top: -($size);
			right: -($size);
			width: $size * 2;
			height: $size * 2;
			cursor: nesw-resize;
		}

		&.bottom-right {
			bottom: -($size);
			right: -($size);
			width: $size * 2;
			height: $size * 2;
			cursor: nwse-resize;
		}

		&.bottom-left {
			bottom: -($size);
			left: -($size);
			width: $size * 2;
			height: $size * 2;
			cursor: nesw-resize;
		}
	}
}
</style>
