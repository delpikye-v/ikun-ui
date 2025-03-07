<script lang="ts">
	import { fly } from 'svelte/transition';
	import { KIcon } from '@ikun-ui/icon';
	import { createEventDispatcher, getContext, tick } from 'svelte';
	import { clsx } from 'clsx';
	import { getPrefixCls, collapseWrapperKey } from '@ikun-ui/utils';
	import type { KCollapseProps } from './types';

	export let title: KCollapseProps['title'] = '';
	export let content: KCollapseProps['content'] = '';
	export let attrs: KCollapseProps['attrs'] = {};
	export let cls: KCollapseProps['cls'] = undefined;
	export let show: KCollapseProps['show'] = false;
	export let showClose: KCollapseProps['showClose'] = true;
	export let uid: KCollapseProps['uid'] = '';

	const dispatch = createEventDispatcher();
	let showInner = show;

	const showContent = () => {
		const showCur = !showInner;
		if (showCur) {
			showInner = !showInner;
			setAnimateOpen(() => {
				dispatch('change', showInner);
			});
			if (collapseContext) {
				collapseContext.closeCollapse(uid);
			}
		} else {
			setAnimateClose(() => {
				showInner = !showInner;
				dispatch('change', showInner);
			});
		}
	};

	const closeCollapse = () => {
		setAnimateClose(() => {
			showInner = false;
		});
	};

	const collapseContext = getContext<{
		setCollapseMap: (key: string, cb: () => void) => void;
		closeCollapse: (key: string) => void;
	}>(collapseWrapperKey);
	if (collapseContext && uid) {
		collapseContext.setCollapseMap(uid, closeCollapse);
	}

	$: if (show) {
		showInner = true;
		setAnimateOpen();
	} else {
		setAnimateClose(() => {
			showInner = false;
		});
	}

	let contentRef: HTMLElement | null = null;
	let opacity = 0;
	let contentCls = '';
	const setAnimateOpen = async (cb?: () => void) => {
		await tick();
		if (contentRef) {
			const h = contentRef.clientHeight;
			contentRef.style.height = '0';
			contentRef.style.transition = 'height 0.3s linear';
			contentCls = cnamesContent;
			await tick();
			setTimeout(() => {
				contentRef && (contentRef.style.height = `${h}px`);
				opacity = 1;

				cb && cb();
			}, 0);
		}
	};

	const setAnimateClose = (cb: () => void) => {
		if (contentRef) {
			contentRef && (contentRef.style.height = '0');
			opacity = 0;
			contentCls = '';
			setTimeout(() => {
				cb && cb();
			}, 300);
		}
	};

	// class
	const prefixCls = getPrefixCls('collapse');
	$: clsInner = clsx(
		{
			[`${prefixCls}--base`]: !collapseContext,
			[`${prefixCls}--base--wrapper`]: collapseContext
		},
		cls
	);
	$: cnames = clsx(`${prefixCls}--title`, `${prefixCls}--title__dark`, {
		[`${prefixCls}--title__active`]: showInner && collapseContext
	});

	$: cnamesLine = clsx(`${prefixCls}--line`);

	$: cnamesContent = clsx(`${prefixCls}--content`);

	$: cnamesTitleIcon = clsx({
		'i-carbon-chevron-right': true,
		['rotate-90']: showInner,
		[`${prefixCls}--title__active`]: showInner && collapseContext
	});
</script>

<div class={clsInner} {...attrs}>
	<div class={cnames} on:click={showContent} >
		<slot name="title">
			{title}
		</slot>
		<slot name="closeIcon">
			{#if showClose}
				<KIcon icon={cnamesTitleIcon} />
			{/if}
		</slot>
	</div>
	{#if showInner}
		<div class={contentCls} bind:this={contentRef} style:opacity in:fly={{ y: -30, duration: 300 }}>
			<div class={cnamesLine} />
			<slot name="content">
				{content}
			</slot>
		</div>
	{/if}
</div>
