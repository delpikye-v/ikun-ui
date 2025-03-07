<script lang="ts">
	import { getPrefixCls } from '@ikun-ui/utils';
	import { clsx } from 'clsx';
	import type { KCarouselArrowProps } from './types';
	import { KIcon } from '@ikun-ui/icon';
	import { createEventDispatcher } from 'svelte';
	import { fly } from 'svelte/transition';
	export let defaultPageIndex: KCarouselArrowProps['defaultPageIndex'] = 0;
	export let count: KCarouselArrowProps['count'] = 0;
	export let show: KCarouselArrowProps['show'] = false;
	export let loop: KCarouselArrowProps['loop'] = true;
	export let cls: KCarouselArrowProps['cls'] = undefined;
	export let attrs: KCarouselArrowProps['attrs'] = {};

	$: pageIndex = defaultPageIndex;

	export const setPage = (current: number) => {
		if (current < 0 || current >= count) {
			console.warn('[KCarousel]: The target index of the jump exceeds the allowed range.');
			return;
		}
		jumpPage(current);
	};

	const dispatch = createEventDispatcher();
	const jumpPage = (page: number) => {
		let resolveIndex = page;
		let type = 'normal';
		if (loop) {
			if (resolveIndex < 0) {
				resolveIndex = count - 1;
				type = 'fte';
			} else if (resolveIndex >= count) {
				resolveIndex = 0;
				type = 'etf';
			}
		} else {
			resolveIndex = resolveIndex < 0 ? 0 : resolveIndex >= count ? count - 1 : resolveIndex;
		}
		dispatch('change', {
			index: resolveIndex,
			type
		});
	};

	export const gotoPrev = () => jumpPage(pageIndex - 1);

	export const gotoNext = () => jumpPage(pageIndex + 1);

	const prefixCls = getPrefixCls('carousel-arrow');
	$: cnames = clsx(prefixCls, cls);
	$: iconCls = clsx(`${prefixCls}-icon`, `${prefixCls}-icon--dark`);
	$: prevCls = clsx(`${iconCls}`, `${prefixCls}-prev`);
	$: nextCls = clsx(`${iconCls}`, `${prefixCls}-next`);
</script>

<div class={cnames} {...$$restProps} {...attrs}>
	<slot name="prev" {gotoPrev}>
		{#if show}
			<div
				role="button"
				out:fly={{ duration: 500, x: -10 }}
				in:fly={{ duration: 500, x: -10 }}
				class={prevCls}

				on:click={gotoPrev}
			>
				<KIcon icon="i-carbon-chevron-left" width="16px" height="16px" />
			</div>
		{/if}
	</slot>
	<slot name="next" {gotoNext}>
		{#if show}
			<div
				role="button"
				out:fly={{ duration: 500, x: 10 }}
				in:fly={{ duration: 500, x: 10 }}
				class={nextCls}

				on:click={gotoNext}
			>
				<KIcon icon="i-carbon-chevron-right" width="16px" height="16px" />
			</div>
		{/if}
	</slot>
</div>
