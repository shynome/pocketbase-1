<script>
    import { parseSelected, toString } from "./ExcelPicker.svelte";
    export let show = () => {};
    export let value;
    /**@type {any[]}*/
    export let data;
    $: sample = (function (data, value) {
        if (!value) {
            return [];
        }
        let selcted = parseSelected(value);
        if (selcted.length === 0) {
            return [];
        }
        return data.map((row) => {
            return toString(value, row);
        });
    })(data, value);
</script>

<div>
    <slot />
    <div class="sample">
        <button type="button" class="btn btn-transparent btn-block btn-sm" on:click={show}>
            <i class="ri-table-line" />
            <span class="text">Open table picker</span>
        </button>
        {#if sample.length > 0}
            <div class="list">
                <div class="list-item"><div class="content">sample data</div></div>
                {#each sample as item}
                    <div class="list-item">
                        <div class="content">{item}</div>
                    </div>
                {/each}
            </div>
        {/if}
    </div>
</div>

<style>
    button {
        background-color: var(--baseAlt1Color);
        border-top: 1px solid var(--baseAlt2Color);
        padding: 10px;
    }
    .sample {
        position: relative;
    }
    .sample:hover .list {
        visibility: visible;
        /* margin-top: 5px; */
        z-index: 9;
        box-shadow: 0px 0px 4px 1px var(--dangerAltColor);
        border: 1px solid var(--baseAlt2Color);
    }
    .list {
        position: absolute;
        background-color: var(--baseAlt1Color);
        bottom: 0;
        left: 0;
        visibility: hidden;
    }
    .list-item {
        margin-bottom: 0;
    }
    .content {
        margin-bottom: 0;
    }
    div {
        margin-bottom: var(--baseSpacing);
    }
    div :global(.form-field) {
        margin-bottom: 0;
    }
</style>
