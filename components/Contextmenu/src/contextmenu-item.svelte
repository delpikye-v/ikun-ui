<script lang="ts">
	import { contextmenuKey, getPrefixCls } from '@ikun-ui/utils';
	import { clsx } from 'clsx';
	import type { KContextmenuProps } from './types';
	import { getContext } from 'svelte';

	export let cls: KContextmenuProps['cls'] = undefined;
	export let attrs: KContextmenuProps['attrs'] = {};
	export let disabled: KContextmenuProps['disabled'] = false;
	export let divider: KContextmenuProps['divider'] = false;

	const ctx = getContext(contextmenuKey) as { close: () => void };
	const handleClick = () => {
		if (disabled) return;
		ctx && ctx.close();
	};

	const prefixCls = getPrefixCls('contextmenu-item');
	const btnCls = `${prefixCls}-btn`;
	$: cnames = clsx(
		prefixCls,
		{
			[`${prefixCls}--base`]: !disabled && !divider,
			[`${prefixCls}--base__dark`]: !disabled && !divider,
			[`${prefixCls}__disabled`]: disabled && !divider,
			[`${prefixCls}__disabled__dark`]: disabled && !divider,
			[`${prefixCls}__divider`]: divider
		},
		cls
	);
</script>

<li class={cnames} {...$$restProps} {...attrs}>
	<button on:click={handleClick} class={btnCls}>
		{#if !divider}
			<slot />
		{/if}
	</button>
</li>
