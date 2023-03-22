<script context="module">
    /**
     * @param selected {string}
     * @returns {number[]}
     */
    export function parseSelected(selected) {
        if (typeof selected !== "string") {
            return [];
        }
        /**@type {string[]}*/
        let matched = selected.match(/\$\{\d+\}/g) ?? [];
        return matched.map((v) => {
            return Number(v.slice(2, -1));
        });
    }

    /**
     * @param str {string}
     * @param row {any[]}
     * @returns {string}
     */
    export function toString(str, row) {
        let selcted = parseSelected(str);
        let c = 0;
        return str.replace(/\$\{\d+\}/g, () => {
            let index = selcted[c++];
            return row[index] ?? "";
        });
    }

    export function isTablePlaceholder(value) {
        if (typeof value !== "string") {
            return false;
        }
        return value.startsWith("table`");
    }
</script>

<script>
    import OverlayPanel from "@/components/base/OverlayPanel.svelte";
    import { utils } from "xlsx";
    /**@type {import("xlsx").WorkSheet}*/
    export let sheet;
    $: table = utils.sheet_to_html(sheet, { header: "", footer: "" });

    let panel;
    /**@type {import("pocketbase").SchemaField}*/
    let field;
    let store = [];
    /**@type {{import("pocketbase").Record}}*/
    export let record;

    /**
     * @param _field {import("pocketbase").SchemaField}
     */
    export function show(_field) {
        field = _field;
        /**@type {string}*/
        let selected = record[field.name];
        if (isTablePlaceholder(selected)) {
            store = parseSelected(selected);
        } else {
            store = [];
        }
        panel?.show();
    }
    export function hide() {
        panel?.hide();
    }

    function save() {
        let x = store.map((k) => `\${${k}}`).join(" ");
        if (x.length > 0) {
            record[field.name] = `table\`${x}\``;
        } else {
            // record[field.name] = "";
        }
        panel?.hide();
    }

    import ExcelTable from "./ExcelTable.svelte";
</script>

<OverlayPanel popup class="excel-picker-panel" bind:this={panel}>
    <svelte:fragment slot="header">
        <h4>table picker for field {field.name} {store.join(",")}</h4>
    </svelte:fragment>
    <ExcelTable {table} bind:store />
    <svelte:fragment slot="footer">
        <button type="button" class="btn btn-transparent" on:click={() => hide()}>
            <span class="txt">Cancel</span>
        </button>
        <button type="button" class="btn" on:click={() => save()}>
            <span class="txt">Save selection</span>
        </button>
    </svelte:fragment>
</OverlayPanel>

<style>
    :global(.excel-picker-panel.overlay-panel .panel-content) {
        overflow-x: auto;
    }
    :global(.excel-picker-panel.overlay-panel) {
        width: 100%;
    }
</style>
