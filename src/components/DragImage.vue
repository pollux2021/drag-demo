<template>
	<div ref="container" :style="wrapStyle">
		<img :style="imageStyle" ref="imageEl" :src="url" />
		<slot :imageW="imageW" :imageH="imageH" :scale="scale"></slot>
	</div>
</template>
<script>
import VueDragCustom from "./vue-drag-custom/vue-draggable-resizable.vue"
import { addEvent } from "./vue-drag-custom/utils/dom"
export default {
	name: "DragImage",
	components: {
		VueDragCustom,
	},
	props: {
		url: {
			type: String,
			default: "",
		},
		imageState: {
			type: String,
			default: "loading",
			validator: val => {
				const s = new Set(["loading", "noimage", "loaded"])
				return s.has(val)
			},
		},
		touchParentOffset: {
			type: Array,
			default: () => [0, 0],
		},
	},
	data() {
		return {
			x: 0,
			y: 0,
			scale: 1,
			minScale: 0.5,
			maxScale: 9,
			imageW: 0,
			imageH: 0,
			points: { x: 0, y: 0 },
			diff: { x: 0, y: 0 },
			lastPointermove: { x: 0, y: 0 },
			isPointerdown: false,
			parentW: 0,
			parentH: 0,
		}
	},
	watch: {
		url(val) {
			this.getImgSize(val)
		},
	},
	computed: {
		wrapStyle() {
			return {
				position: "relative",
				zIndex: 10,
				width: `${this.imageW}px`,
				height: `${this.imageH}px`,
				transform: `translate3d(${this.x}px, ${this.y}px, 0) scale(${this.scale})`,
			}
		},
		imageStyle() {
			return {
				position: "absolute",
				left: 0,
				width: `${this.imageW}px`,
				height: `${this.imageH}px`,
			}
		},
	},
	mounted() {
		this.$nextTick(function () {
			this.getImgSize(this.url)
			this.addImageEvents()
			this.wheelZoom()
			const wrapEl = this.$refs.refWrap
			if (!wrapEl) return
			addEvent(wrapEl, "mousewheel", this.onMouseWheel)
			addEvent(wrapEl, "DOMMouseScroll", this.onMouseWheel)
			addEvent(
				document.documentElement,
				"mousewheel",
				this.preventDefaultScroll,
				{ passive: false }
			)

			addEvent(
				document.documentElement,
				"DOMMouseScroll",
				this.preventDefaultScroll,
				{ passive: false }
			)
		})
	},
	methods: {
		preventDefaultScroll(ev) {
			if (ev.preventDefault) ev.preventDefault() // 阻止默认事件
			return false
		},
		getImgSize(url) {
			if (!url) {
				this.$emit("update:imageState", "noimage")
				return
			}

			const imageEl = document.createElement("img")
			imageEl.src = url
			this.$emit("update:imageState", "loading")
			addEvent(imageEl, "load", () => {
				const naturalWidth = imageEl.naturalWidth
				const naturalHeight = imageEl.naturalHeight
				const parentNode = this.$el.parentNode
				const maxWidth = parentNode.offsetWidth
				const maxHeight = parentNode.offsetHeight
				const imgRatio = naturalWidth / naturalHeight
				const maxRatio = maxWidth / maxHeight
				this.$emit("update:imageState", "loaded")
				this.parentW = maxWidth
				this.parentH = maxHeight
				let width, height
				// 如果图片实际宽高比例 >= 显示宽高比例
				if (imgRatio >= maxRatio) {
					if (naturalWidth > maxWidth) {
						width = maxWidth
						height = (maxWidth / naturalWidth) * naturalHeight
					} else {
						width = naturalWidth
						height = naturalHeight
					}
				} else {
					if (naturalHeight > maxHeight) {
						width = (maxHeight / naturalHeight) * naturalWidth
						height = maxHeight
					} else {
						width = naturalWidth
						height = naturalHeight
					}
				}
				this.x = (maxWidth - width) * 0.5
				this.y = (maxHeight - height) * 0.5
				this.imageW = width
				this.imageH = height
				this.imageInit = true
			})
		},
		addImageEvents() {
			const image = this.$refs.imageEl
			// 绑定 pointerdown
			addEvent(image, "pointerdown", e => {
				this.isPointerdown = true
				image.setPointerCapture(e.pointerId)
				this.points = { x: e.clientX, y: e.clientY }
				this.lastPointermove = { x: e.clientX, y: e.clientY }
				this.$emit("update:touchParentOffset", [e.offsetX, e.offsetY])
			})
			// 绑定 pointermove
			addEvent(image, "pointermove", e => {
				if (this.isPointerdown) {
					const current1 = { x: e.clientX, y: e.clientY }
					this.diff.x = current1.x - this.lastPointermove.x
					this.diff.y = current1.y - this.lastPointermove.y
					this.lastPointermove = { x: current1.x, y: current1.y }
					this.x += this.diff.x
					this.y += this.diff.y
				}
				e.preventDefault()
			})
			// 绑定 pointerup
			addEvent(image, "pointerup", e => {
				if (this.isPointerdown) {
					this.isPointerdown = false
				}
			})
			// 绑定 pointercancel
			addEvent(image, "pointercancel", e => {
				if (this.isPointerdown) {
					this.isPointerdown = false
				}
			})
		},
		zoom(inNum) {
			let ratio = 1.1
			// 缩小
			if (inNum > 0) ratio = 1 / 1.1
			// 限制缩放倍数
			const _scale = this.scale * ratio
			if (_scale > this.maxScale) {
				ratio = this.maxScale / this.scale
				this.scale = this.maxScale
			} else if (_scale < this.minScale) {
				ratio = this.minScale / this.scale
				this.scale = this.minScale
			} else {
				this.scale = _scale
			}
			return ratio
		},
		zoomOut() {},
		wheelZoom() {
			const container = this.$parent.$el
			container.addEventListener("wheel", e => {
				let ratio = this.zoom(e.deltaY)

				// 目标元素是img说明鼠标在img上，以鼠标位置为缩放中心，否则默认以图片中心点为缩放中心
				if (e.target.tagName === "IMG") {
					const origin = {
						x: (ratio - 1) * this.imageW * 0.5,
						y: (ratio - 1) * this.imageH * 0.5,
					}

					// 计算偏移量
					const x = (ratio - 1) * (e.clientX - this.x) - origin.x
					const y = (ratio - 1) * (e.clientY - this.y) - origin.y
					this.x -= x
					this.y -= y
				}
				e.preventDefault()
			})
		},
	},
}
</script>

<style scoped>
.tripple {
	position: absolute;
	left: 0;
	top: 0;
	z-index: 9999;
}
</style>
