<script>
    import { Record } from "pocketbase";
    import OverlayPanel from "@/components/base/OverlayPanel.svelte";
    import TableFields from "./importer/TableFields.svelte";
    import SanphotTable from "./importer/SanphotTable.svelte";

    export let collection;

    let record = new Record();
    let importPanel;
    let isSaving = false;
    let excel = Promise.reject(1);
    let tt;
    let snapshot;

    /**
     * @param {File} file
     */
    export function show(file) {
        excel = new Promise(async (rl, rj) => {
            const XLSX = await import("xlsx");
            let buf = await file.arrayBuffer();
            const workbook = XLSX.read(buf);
            let sheet = workbook.Sheets[workbook.SheetNames[0]];
            let data = XLSX.utils.sheet_to_json(sheet, { header: 1 });
            rl({ sheet, data });
        });
        return importPanel?.show();
    }

    export function hide() {
        return importPanel?.hide();
    }

    export function save() {
        isSaving = true;
        Promise.resolve()
            .then(async () => {
                await snapshot?.save();
                // hide();
            })
            .finally(() => {
                isSaving = false;
            });
    }

    let startImport = false;
</script>

<OverlayPanel bind:this={importPanel} class="records-import-panel" on:hide on:show>
    <svelte:fragment slot="header">
        <h4><strong>Import Records</strong></h4>
    </svelte:fragment>

    <div class="tabs-content">
        {#await excel}
            <div class="loading">loading</div>
        {:then { sheet, data }}
            {#if startImport}
                <SanphotTable {record} {collection} {data} bind:this={snapshot} />
            {:else}
                <TableFields {record} {collection} {sheet} {data} bind:this={tt} />
            {/if}
        {:catch err}
            Parse Failed
            <label class="btn btn-outline" for="panel-import-excel-input">Upload Excel</label>
            <input
                class="hidden"
                id="panel-import-excel-input"
                type="file"
                accept=".xls,.xlsx,.csv"
                on:click={(e) => (e.currentTarget.value = null)}
                on:change={(e) => show(e.currentTarget.files[0])}
            />
        {/await}
    </div>

    <svelte:fragment slot="footer">
        <button type="button" class="btn btn-transparent" disabled={isSaving} on:click={() => hide()}>
            <span class="txt">Cancel</span>
        </button>

        {#if startImport}
            <button
                type="button"
                class="btn btn-outline"
                disabled={isSaving}
                on:click={() => (startImport = false)}
            >
                <span class="txt">Reset Fields</span>
            </button>
            <button
                type="submit"
                class="btn btn-expanded"
                class:btn-loading={isSaving}
                disabled={isSaving}
                on:click={save}
            >
                <span class="txt">Import Excel</span>
            </button>
        {:else}
            <button
                type="submit"
                class="btn btn-expanded"
                class:btn-loading={isSaving}
                disabled={isSaving}
                on:click={() => (startImport = true)}
            >
                <span class="txt">Fields Set Ok</span>
            </button>
        {/if}
    </svelte:fragment>
</OverlayPanel>

<style>
    :global(.records-import-panel) {
        width: 960px;
    }
    :global(.records-import-panel.overlay-panel .panel-content) {
        overflow-x: auto;
    }
</style>
