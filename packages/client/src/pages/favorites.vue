<template>
<MkStickyContainer>
	<template #header><MkPageHeader/></template>
	<MkSpacer :content-max="800">
		<MkPagination ref="pagingComponent" :pagination="pagination">
			<template #empty>
				<div class="_fullinfo">
					<img src="https://xn--931a.moe/assets/info.jpg" class="_ghost"/>
					<div>{{ $ts.noNotes }}</div>
				</div>
			</template>

			<template #default="{ items }">
				<XList v-slot="{ item }" :items="items" :direction="'down'" :no-gap="false" :ad="false">
					<XNote :key="item.id" :note="item.note" :class="$style.note"/>
				</XList>
			</template>
		</MkPagination>
	</MkSpacer>
</MkStickyContainer>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import MkPagination from '@/components/ui/pagination.vue';
import XNote from '@/components/note.vue';
import XList from '@/components/date-separated-list.vue';
import { i18n } from '@/i18n';
import { definePageMetadata } from '@/scripts/page-metadata';

const pagination = {
	endpoint: 'i/favorites' as const,
	limit: 10,
};

const pagingComponent = ref<InstanceType<typeof MkPagination>>();

definePageMetadata({
	title: i18n.ts.favorites,
	icon: 'fas fa-star',
});
</script>

<style lang="scss" module>
.note {
	background: var(--panel);
	border-radius: var(--radius);
}
</style>
