<template>
	<div class="container">
		<header class="header">DEMO: 5-12</header>
		<div class="section">
			<!-- 关注此部分, stage需要宽高, 否则初始化高度为0 -->
			<div class="stage" ref="refStage">
				<div class="status-toast" v-if="imageState === 'loading'">
					图片正在加载中
				</div>
				<div class="status-toast" v-if="imageState === 'noimage'">暂无图片</div>
				<transition name="fade">
					<div
						v-if="trippleVisible"
						class="tripple"
						:style="trippleStyle"
					></div>
				</transition>
				<DragImage
					:url="imageUrl"
					:touchParentOffset.sync="touchParentOffset"
					:imageState.sync="imageState"
					:trippleSize="trippleSize"
					:trippleVisible="trippleVisible"
				>
					<template #touchtripe>
						<div class="tripple">tripple</div>
					</template>
					<template #default="{ imageW, imageH, scale }">
						<vue-drag-custom
							v-for="point in points"
							v-if="imageState === 'loaded'"
							:key="point.code"
							:parent="[imageW, imageH]"
							:snap="true"
							:snapToGrid="true"
							:gridX="20"
							:gridY="20"
							:w.sync="point.iconSize"
							:h.sync="point.iconSize"
							:x.sync="point.positionLeft"
							:y.sync="point.positionTop"
							:z.sync="point.zIndex"
							:handles="['tl', 'tr', 'br', 'bl']"
							:scaleRatio="scale"
							:lockAspectRatio="true"
							:parentW="imageW"
							:parentH="imageH"
							:onDrag="onDrag"
						>
							<span class="dot">{{ point.code }}</span>
						</vue-drag-custom>
					</template>
				</DragImage>
			</div>

			<div class="operate">
				<div>
					<div class="hgroup">
						<h3 class="title">Operate</h3>
						<button @click="changeImage">切换图片</button>
						<button @click="addPoint">添加点</button>
						<button @click="clearPoints">清除点</button>
					</div>
					<ul class="desc">
						<li>imageState: 所有点仅在imageState状态未loaded才会显示</li>
						<li>snap: 支持点的对齐</li>
						<li>snapToGrid: 支持网格对齐</li>
						<li>gridX,gridY: 网格大小</li>
						<li>支持放大后点击点记录坐标， 并添加点到当前位置</li>
						<li>支持自定义标记</li>
					</ul>

					<div>
						<div>
							下一个点添加的位置，在点击图片时会记录当前点击的点相对于父节点的offset:
							{{ touchParentOffset }}
						</div>
					</div>
					<div>
						<h3 class="title">Points</h3>
						<div>{{ points }}</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>
<script>
import VueDragCustom from "@/components/vue-drag-custom/vue-draggable-resizable.vue"
import DragImage from "@/components/DragImage.vue"
import { addEvent } from "@/components/vue-drag-custom/utils/dom"

const IMAG_DEFAULT = [
	"https://img1.qunarzz.com/travel/d2/1711/d5/5529e21b0159acb5.jpg_r_720x480x95_f792770b.jpg",
	"https://img.zcool.cn/community/011da35718c86232f8759bff5c517c.jpg@1280w_1l_2o_100sh.jpg",
]

const POINT_DEFAULT = {
	code: "1", // 唯一标识
	positionLeft: 0, // x坐标
	positionTop: 0, // y坐标
	zIndex: 1, // 深度zIndex
	iconSize: 33, // 默认大小
}

export default {
	name: "Home",
	components: {
		VueDragCustom,
		DragImage,
	},
	data() {
		return {
			imageUrl: IMAG_DEFAULT[1],
			imageState: "noimage", // 图片状态 noimage | loading | loaded
			touchParentOffset: [0, 0], // 相对于DragImage父节点的offset坐标
			trippleSize: 34, // 点击显示的红点样式
			trippleVisible: false, // 红点是否显示
			stageOffset: [0, 0], // 当前点击的红点坐标
			points: [],
		}
	},
	computed: {
		trippleStyle() {
			const [x, y] = this.stageOffset
			const offset = this.trippleSize / 2
			return {
				top: y === 0 ? 0 : `${y - offset}px`,
				left: x === 0 ? 0 : `${x - offset}px`,
				width: `${this.trippleSize}px`,
				height: `${this.trippleSize}px`,
			}
		},
	},
	methods: {
		// 点击红点
		addTripple() {
			const refStage = this.$refs.refStage
			addEvent(refStage, "pointerdown", e => {
				this.trippleVisible = true
				this.stageOffset = [e.clientX, e.clientY]
				if (this.trippleInterval) clearInterval(this.trippleInterval)
				this.trippleInterval = setTimeout(() => {
					this.trippleVisible = false
				}, 180)
			})
			addEvent(refStage, "pointermove", e => {
				if (this.trippleInterval) clearInterval(this.trippleInterval)
				this.trippleVisible = false
			})
		},

		// 切换图片
		changeImage() {
			this.imageUrl =
				IMAG_DEFAULT[0] === this.imageUrl ? IMAG_DEFAULT[1] : IMAG_DEFAULT[0]
		},

		// 新增点
		addPoint() {
			this.points.push({
				...POINT_DEFAULT,
				code: this.points.length + 1,
				zIndex: this.points.length + 1,
				positionLeft: this.touchParentOffset[0] || 0,
				positionTop: this.touchParentOffset[1] || 0,
			})
		},

		// 清除点
		clearPoints() {
			this.points = []
		},
		onDrag(x, y) {
			console.log(x, y)
		},
	},
	mounted() {
		this.$nextTick(() => {
			this.addTripple()
		})
	},
}
</script>

<style>
body,
html {
	padding: 0;
	margin: 0;
	background-color: #000;
}

.container {
	display: flex;
	flex-direction: column;
	height: 100vh;
	min-width: 1000px;
	margin: 0 auto;
	overflow: hidden;
	/* overflow: hidden; */
}

.header {
	padding: 20px;
	color: #fff;
	background-color: #333;
}

.section {
	display: flex;
	flex: 1 1 auto;
	width: 100%;
}

.operate {
	width: 400px;
	overflow: hidden;
	word-break: break-all;
	padding: 20px;
	color: #fff;
	background-color: #555;
}

.stage {
	position: relative;
	flex: 1 1 auto;
	height: 100%;
	background-color: #444;
	overflow: hidden;
}

.status-toast {
	position: absolute;
	z-index: 9999;
	left: 50%;
	top: 50%;
	color: #fff;
	padding: 10px;
	background-color: #000;
	transform: translate(-50%, -50%);
}

.dot {
	background-color: red;
	display: flex;
	justify-content: center;
	align-items: center;
	width: 100%;
	height: 100%;
	border-radius: 100%;
}

.title {
	margin: 0;
	padding-bottom: 10px;
}

.hgroup {
	margin-bottom: 10px;
	padding-bottom: 20px;
	border-bottom: 1px solid #ddd;
	word-break: break-all;
}

/* 点击显示当前红点样式 */
.tripple {
	position: fixed;
	border-radius: 100%;
	width: 100%;
	height: 100%;
	background-color: red;
	z-index: 9999;
}

.fade-enter-active,
.fade-leave-active {
	transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
	opacity: 0;
}

.desc {
	list-style: none;
	margin: 0;
	padding: 0;
}

.desc li {
	padding: 10px 0;
	border-bottom: 1px solid #444;
	margin-bottom: 10px;
}
</style>
