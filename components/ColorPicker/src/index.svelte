<script lang="ts">
	import { formKey, getPrefixCls } from '@ikun-ui/utils';
	import type { IKunSize } from '@ikun-ui/utils';
	import { clsx } from 'clsx';
	import type { HsvaColor, KColorPickerProps } from './types';
	import { KPopover } from '@ikun-ui/popover';
	import { toRgbString, toHsvString, toHex8String, toHsv } from './utils';
	import KColorPickerPalette from './palette.svelte';
	import KColorPickerSlider from './slider.svelte';
	import KColorPickerBlock from './block.svelte';
	import KColorPickerFormat from './format.svelte';
	import KColorPickerPreset from './preset.svelte';
	import { createEventDispatcher, getContext } from 'svelte';
	import type { IKunFormInstance } from '@ikun-ui/form';
	import { formItemKey } from '@ikun-ui/utils';
	export let allowClear: KColorPickerProps['allowClear'] = false;
	export let title: KColorPickerProps['title'] = '';
	export let value: KColorPickerProps['value'] = '';
	export let defaultValue: KColorPickerProps['defaultValue'] = '';
	export let format: KColorPickerProps['format'] = 'hex';
	export let disabled: KColorPickerProps['disabled'] = false;
	export let disabledAlpha: KColorPickerProps['disabledAlpha'] = false;
	export let placement: KColorPickerProps['placement'] = 'top';
	export let presets: KColorPickerProps['presets'] = [];
	export let size: KColorPickerProps['size'] = 'md';
	export let showText: KColorPickerProps['showText'] = false;
	export let trigger: KColorPickerProps['trigger'] = 'click';
	export let cls: KColorPickerProps['cls'] = '';
	export let triggerClass: KColorPickerProps['cls'] = '';
	export let attrs: KColorPickerProps['attrs'] = {};

	function formatColor(format: KColorPickerProps['format'], value: KColorPickerProps['value']) {
		if (format === 'rgb') {
			return toRgbString(value);
		}
		if (format === 'hsv') {
			return toHsvString(value);
		}
		return toHex8String(value);
	}

	const dispatch = createEventDispatcher();
	function handleChangeComplete(e: CustomEvent) {
		isDragging = false;
		const res = { ...e.detail, a: aColor.a };
		aColor = res;
		blockColor = res;
		formatterColor = res;
		isClear = false;
		const resolveColor = formatColor(formatValue, blockColor);
		dispatch('changeComplete', resolveColor);
		if (formInstance) {
			formInstance.updateField(field!, resolveColor, !formInstance.__manual_validate);
			value = resolveColor;
		}
	}

	let isDragging = false;
	function handleChange(e: CustomEvent) {
		isDragging = true;
		const res = { ...e.detail, a: aColor.a };
		aColor = res;
		blockColor = res;
		formatterColor = res;
		isClear = false;

		const resolveColor = formatColor(formatValue, blockColor);
		dispatch('change', resolveColor);
		if (formInstance) {
			formInstance.updateField(field!, resolveColor, !formInstance.__manual_validate);
			value = resolveColor;
		}
	}

	function handleHValueInput(e: CustomEvent) {
		// console.log(aColor)
		const res = { ...e.detail, a: aColor.a };
		aColor = { ...aColor, h: res.h, a: res.a };
		// console.log(aColor)
		blockColor = { ...blockColor, h: res.h, a: res.a };
		formatterColor = { ...formatterColor, h: res.h, a: res.a };
		defaultPaletteColor = { ...defaultPaletteColor, h: res.h, a: res.a };
		isClear = false;

		const resolveColor = formatColor(formatValue, blockColor);
		dispatch('change', resolveColor);
		dispatch('changeComplete', resolveColor);
		if (formInstance) {
			formInstance.updateField(field!, resolveColor, !formInstance.__manual_validate);
			value = resolveColor;
		}
	}

	function handleAValueInput(e: CustomEvent) {
		aColor.a = e.detail.a;
		blockColor = e.detail;
		formatterColor = e.detail;
		isClear = false;
		const resolveColor = formatColor(formatValue, blockColor);
		dispatch('changeComplete', resolveColor);
		if (formInstance) {
			formInstance.updateField(field!, resolveColor, !formInstance.__manual_validate);
			value = resolveColor;
		}
	}

	let focus = false;
	function onDisplayChange(e: CustomEvent) {
		focus = e.detail;
		dispatch('openChange', focus);
	}

	let isClear = false;
	function handleClear() {
		aColor.a = 0;
		blockColor = aColor;
		formatterColor = aColor;
		isClear = true;
		dispatch('clear');
		dispatch('change', null);

		if (formInstance) {
			formInstance.updateField(field!, null, !formInstance.__manual_validate);
		}
	}

	let paletteRef: any = null;
	$: formatValue = format;
	function handleFormatInput(e: CustomEvent) {
		const hsv = toHsv(e.detail.value) as HsvaColor;
		formatValue = e.detail.format;
		blockColor = hsv;
		paletteColor = hsv;
		aColor = hsv;
		hColor = hsv;
		defaultPaletteColor = { ...hColor, s: 1, v: 1, a: 1 };
		formatterColor = hsv;
		isClear = false;
		if (paletteRef) {
			paletteRef.setPickerPos(paletteColor);
		}

		const resolveColor = formatColor(formatValue, blockColor);
		dispatch('change', resolveColor);
		if (formInstance) {
			formInstance.updateField(field!, resolveColor, !formInstance.__manual_validate);
			value = resolveColor;
		}
	}

	function handleFormatChange(e: CustomEvent) {
		formatValue = e.detail;
		dispatch('formatChange', formatValue);
	}

	function handlePresetChange(e: CustomEvent) {
		const v = toHsv(e.detail[0]) as HsvaColor;
		const res = { ...v, a: aColor.a };
		defaultPaletteColor = { ...defaultPaletteColor, h: res.h, a: res.a };
		blockColor = { ...res, a: res.a };
		formatterColor = { ...res, a: res.a };
		paletteColor = { ...res, a: res.a };
		presetColor = res;
		if (paletteRef) {
			paletteRef.setPickerPos(paletteColor);
		}
		isClear = false;
		const resolveColor = formatColor(formatValue, blockColor);
		dispatch('change', resolveColor);
		dispatch('changeComplete', resolveColor);
		if (formInstance) {
			formInstance.updateField(field!, resolveColor, !formInstance.__manual_validate);
			value = resolveColor;
		}
	}

	const defaultColor = { h: 0, s: 0, v: 0, a: 0 };
	let hsvValue: HsvaColor = defaultColor;
	$: {
		if (!isDragging) {
			hsvValue = toHsv(value || defaultColor) as HsvaColor;
		}
	}

	let hsvDefaultValue: HsvaColor = defaultColor;
	$: {
		hsvDefaultValue = toHsv(defaultValue);
	}
	$: paletteColor = hsvValue;
	$: defaultPaletteColor = hsvDefaultValue;
	$: hColor = hsvValue;
	$: aColor = hsvValue;
	$: blockColor = hsvValue;
	$: formatterColor = hsvValue;
	$: presetColor = hsvValue;

	/*********************** KForm logic start ************************/
	let disabledFrom = false;
	$: disabledInner = disabledFrom || disabled;
	let sizeFrom: IKunSize | string = '';
	$: sizeInner = (sizeFrom || size) as IKunSize;
	let isErrorForm = false;
	$: isErrorInner = isErrorForm;
	const formContext = getContext(formItemKey) as string;
	const formInstance = getContext(formKey) as IKunFormInstance;
	let field: string | undefined = '';
	// Initialize the KSwitch value based
	// on the form value in the KFormItem context
	async function formUpdateField(init = false) {
		field = formContext.split('&').pop();
		value = formInstance.getValueByPath(
			field,
			init ? formInstance.__default_value : formInstance.__value
		);
		hsvValue = toHsv(value || defaultColor);
		if (!init) {
			hsvDefaultValue = toHsv(defaultValue);
			isClear = false;
		}
	}
	function formPropsChangeCb(props: Record<any, any>) {
		disabledFrom = props.disabled;
		sizeFrom = props.size;
	}

	function fromFieldError(error: boolean) {
		isErrorForm = error;
	}
	// Register event, KForm can set KColorPicker value
	if (formContext && formInstance) {
		formUpdateField(true);
		formPropsChangeCb(formInstance.__dynamicProps);
		formInstance.__itemCompMap[field] = {
			update: formUpdateField,
			type: 'color-picker'
		};
		formInstance.__errorCompEvtMap[field] = fromFieldError;
		formInstance.__propHandleEvtMap.push(formPropsChangeCb);
	}
	/*********************** KForm logic end ************************/

	const prefixCls = getPrefixCls('color-picker');
	const hsbCls = getPrefixCls('color-picker--hsb');
	const hsCls = getPrefixCls('color-picker--hs');
	const headerCls = getPrefixCls('color-picker-header');
	const clearCls = getPrefixCls('color-picker-clear');
	const lineCls = getPrefixCls('color-picker-line');
	$: triggerCls = clsx(
		{
			[`${prefixCls}-trigger--disabled`]: disabledInner
		},
		triggerClass
	);

	$: txtCls = clsx(`${prefixCls}-txt__dark`, {
		[`${prefixCls}-txt`]: showText
	});
	$: titleCls = clsx({
		[`${prefixCls}-title`]: title
	});
	const clearClsx = clsx(clearCls, lineCls);
	const alphaCls = getPrefixCls('color-picker--alpha');
	$: cnames = clsx(prefixCls, cls);

	let popoverRef: any = null;
	/**
	 * @public
	 * api handleOpen
	 */
	export function handleOpen() {
		popoverRef.updateShow(true);
	}

	/**
	 * @public
	 * api handleClose
	 */
	export function handleClose() {
		popoverRef.updateShow(false);
	}
</script>

<KPopover
	bind:this={popoverRef}
	{placement}
	disabled={disabledInner}
	{trigger}
	on:change={onDisplayChange}
	arrow={false}
>
	<div slot="triggerEl" class={triggerCls}>
		{#if $$slots.default}
			<slot {blockColor} />
		{:else}
			<KColorPickerBlock
				disabled={disabledInner}
				value={blockColor}
				trigger
				size={sizeInner}
				{focus}
				error={isErrorInner}
				{isClear}
			>
				<div slot="text" class={txtCls} style:display={showText ? 'initial' : 'none'}>
					{#if showText}
						<slot name="text">
							{formatColor(formatValue, blockColor)}
						</slot>
					{/if}
				</div>
			</KColorPickerBlock>
		{/if}
	</div>
	<div slot="contentEl" class={cnames} {...$$restProps} {...attrs}>
		<div class={headerCls}>
			<slot name="title">
				<span class={titleCls}>{title}</span>
			</slot>
			{#if allowClear}
				<div class={clearClsx}  on:click={handleClear}></div>
			{/if}
		</div>
		<div>
			<KColorPickerPalette
				bind:this={paletteRef}
				value={paletteColor}
				defaultValue={defaultPaletteColor}
				on:change={handleChange}
				on:changeComplete={handleChangeComplete}
			></KColorPickerPalette>
			<div class={hsbCls}>
				<div class={hsCls}>
					<KColorPickerSlider max={360} min={0} step={1} on:input={handleHValueInput} value={hColor}
					></KColorPickerSlider>
					{#if !disabledAlpha}
						<KColorPickerSlider
							isAlpha
							on:input={handleAValueInput}
							cls={alphaCls}
							max={1}
							min={0}
							step={0.01}
							value={aColor}
						></KColorPickerSlider>
					{/if}
				</div>
				<KColorPickerBlock value={blockColor} />
			</div>
			<KColorPickerFormat
				value={formatterColor}
				{disabledAlpha}
				on:change={handleFormatInput}
				on:formatChange={handleFormatChange}
				format={formatValue}
			></KColorPickerFormat>
			<slot name="preset" {presetColor} {handlePresetChange} {presets}>
				{#if presets && presets.length}
					<KColorPickerPreset value={presetColor} on:change={handlePresetChange} {presets}
					></KColorPickerPreset>
				{/if}
			</slot>
		</div>
	</div>
</KPopover>
