<script>
	import { cn } from "$lib/utils.js";
	let {
		ref = $bindable(null),
		class: className = undefined,
		variant = "legend",
		children = undefined,
		...restProps
	} = $props();
</script>

<legend
	bind:this={ref}
	data-slot="field-legend"
	data-variant={variant}
	class={cn(
		"mb-3 font-medium",
		"data-[variant=legend]:text-base",
		"data-[variant=label]:text-sm",
		className
	)}
	{...restProps}
>
	{@render children?.()}
</legend>