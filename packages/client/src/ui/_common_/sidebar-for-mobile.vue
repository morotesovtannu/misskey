<template>
<div class="kmwsukvl">
	<div class="body">
		<button v-click-anime class="item _button account" @click="openAccountMenu">
			<MkAvatar :user="$i" class="avatar"/><MkAcct class="text" :user="$i"/>
		</button>
		<MkA v-click-anime class="item index" active-class="active" to="/" exact>
			<i class="icon fas fa-home fa-fw"></i><span class="text">{{ $ts.timeline }}</span>
		</MkA>
		<template v-for="item in menu">
			<div v-if="item === '-'" class="divider"></div>
			<component :is="menuDef[item].to ? 'MkA' : 'button'" v-else-if="menuDef[item] && (menuDef[item].show !== false)" v-click-anime class="item _button" :class="[item, { active: menuDef[item].active }]" active-class="active" :to="menuDef[item].to" v-on="menuDef[item].action ? { click: menuDef[item].action } : {}">
				<i class="icon fa-fw" :class="menuDef[item].icon"></i><span class="text">{{ $ts[menuDef[item].title] }}</span>
				<span v-if="menuDef[item].indicated" class="indicator"><i class="icon fas fa-circle"></i></span>
			</component>
		</template>
		<div class="divider"></div>
		<MkA v-if="$i.isAdmin || $i.isModerator" v-click-anime class="item" active-class="active" to="/admin">
			<i class="icon fas fa-door-open fa-fw"></i><span class="text">{{ $ts.controlPanel }}</span>
		</MkA>
		<button v-click-anime class="item _button" @click="more">
			<i class="icon fa fa-ellipsis-h fa-fw"></i><span class="text">{{ $ts.more }}</span>
			<span v-if="otherMenuItemIndicated" class="indicator"><i class="icon fas fa-circle"></i></span>
		</button>
		<MkA v-click-anime class="item" active-class="active" to="/settings">
			<i class="icon fas fa-cog fa-fw"></i><span class="text">{{ $ts.settings }}</span>
		</MkA>
		<button class="item _button post" data-cy-open-post-form @click="post">
			<i class="icon fas fa-pencil-alt fa-fw"></i><span class="text">{{ $ts.note }}</span>
		</button>
	</div>
</div>
</template>

<script lang="ts">
import { computed, defineAsyncComponent, defineComponent, ref, toRef, watch } from 'vue';
import { host } from '@/config';
import { search } from '@/scripts/search';
import * as os from '@/os';
import { menuDef } from '@/menu';
import { openAccountMenu } from '@/account';
import { defaultStore } from '@/store';

export default defineComponent({
	setup(props, context) {
		const menu = toRef(defaultStore.state, 'menu');
		const otherMenuItemIndicated = computed(() => {
			for (const def in menuDef) {
				if (menu.value.includes(def)) continue;
				if (menuDef[def].indicated) return true;
			}
			return false;
		});

		return {
			host: host,
			accounts: [],
			connection: null,
			menu,
			menuDef: menuDef,
			otherMenuItemIndicated,
			post: os.post,
			search,
			openAccountMenu: (ev) => {
				openAccountMenu({
					withExtraOperation: true,
				}, ev);
			},
			more: () => {
				os.popup(defineAsyncComponent(() => import('@/components/launch-pad.vue')), {}, {
				}, 'closed');
			},
		};
	},
});
</script>

<style lang="scss" scoped>
.kmwsukvl {
	$ui-font-size: 1em; // TODO: どこかに集約したい
	$avatar-size: 32px;
	$avatar-margin: 8px;

	> .body {

		> .divider {
			margin: 16px 16px;
			border-top: solid 0.5px var(--divider);
		}

		> .item {
			position: relative;
			display: block;
			padding-left: 24px;
			font-size: $ui-font-size;
			line-height: 2.85rem;
			text-overflow: ellipsis;
			overflow: hidden;
			white-space: nowrap;
			width: 100%;
			text-align: left;
			box-sizing: border-box;
			color: var(--navFg);

			> .icon {
				position: relative;
				width: 32px;
			}

			> .icon,
			> .avatar {
				margin-right: $avatar-margin;
			}

			> .avatar {
				width: $avatar-size;
				height: $avatar-size;
				vertical-align: middle;
			}

			> .indicator {
				position: absolute;
				top: 0;
				left: 20px;
				color: var(--navIndicator);
				font-size: 8px;
				animation: blink 1s infinite;
			}

			> .text {
				position: relative;
				font-size: 0.9em;
			}

			&:hover {
				text-decoration: none;
				color: var(--navHoverFg);
			}

			&.active {
				color: var(--navActive);
			}

			&:hover, &.active {
				&:before {
					content: "";
					display: block;
					width: calc(100% - 24px);
					height: 100%;
					margin: auto;
					position: absolute;
					top: 0;
					left: 0;
					right: 0;
					bottom: 0;
					border-radius: 999px;
					background: var(--accentedBg);
				}
			}

			&:first-child, &:last-child {
				position: sticky;
				z-index: 1;
				padding-top: 8px;
				padding-bottom: 8px;
				background: var(--X14);
				-webkit-backdrop-filter: var(--blur, blur(8px));
				backdrop-filter: var(--blur, blur(8px));
			}

			&:first-child {
				top: 0;

				&:hover, &.active {
					&:before {
						content: none;
					}
				}
			}

			&:last-child {
				bottom: 0;
				color: var(--fgOnAccent);

				&:before {
					content: "";
					display: block;
					width: calc(100% - 20px);
					height: calc(100% - 20px);
					margin: auto;
					position: absolute;
					top: 0;
					left: 0;
					right: 0;
					bottom: 0;
					border-radius: 999px;
					background: linear-gradient(90deg, var(--buttonGradateA), var(--buttonGradateB));
				}
				
				&:hover, &.active {
					&:before {
						background: var(--accentLighten);
					}
				}
			}
		}
	}
}
</style>
