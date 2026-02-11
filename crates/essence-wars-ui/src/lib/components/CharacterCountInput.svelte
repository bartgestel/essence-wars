<script lang="ts">
    let { currentLength, maxLength }: { currentLength: number; maxLength: number } = $props();

    const progress = $derived(Number(currentLength) / Number(maxLength));

    const isOver75Percent = $derived(progress > 0.75);
    const isOver90Percent = $derived(progress > 0.9);

    const textColorClass = $derived(
        isOver90Percent
            ? "text-red-500"
            : isOver75Percent
                ? "text-yellow-500"
                : "text-gray-500"
    );
</script>

<div>
    <span class="{textColorClass}" style>{currentLength} / {maxLength}</span>
</div>