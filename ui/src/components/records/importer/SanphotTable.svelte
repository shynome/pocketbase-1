<script>
    /**@type {import("pocketbase").Collection}*/
    export let collection;
    /**@type {any[][]}*/
    export let data;
    /**@type {import("pocketbase").Record}*/
    export let record;

    import { toString, isTablePlaceholder } from "./ExcelPicker.svelte";
    import { addSuccessToast } from "@/stores/toasts";

    let failed = [];
    let success = [];
    let cursor = 0;
    import ApiClient from "@/utils/ApiClient";

    export async function save() {
        for (; cursor < data.length; cursor++) {
            let post = record.clone();
            for (let field of collection?.schema) {
                if (!isTablePlaceholder(post[field.name])) {
                    continue;
                }
                post[field.name] = toString(post[field.name].slice(6, -1), data[cursor]);
            }
            await Promise.resolve(ApiClient.collection(collection.id).create(post.export()))
                .then(() => {
                    success.push(cursor);
                })
                .catch((err) => {
                    failed.push(cursor);
                    failed = failed;
                });
        }
        addSuccessToast("record import finished");
    }
</script>

<h4>导入失败 {failed.length}</h4>
<table>
    <thead>
        <tr>
            {#each collection?.schema ?? [] as field (field.name)}
                <th>{field.name}</th>
            {/each}
        </tr>
    </thead>
    <tbody>
        {#each failed as cursor}
            <tr>
                {#each collection?.schema ?? [] as field (field.name)}
                    {#if isTablePlaceholder(record[field.name])}
                        <td>{toString(record[field.name], data[cursor]).slice(6, -1)}</td>
                    {:else}
                        <td>{record[field.name] ?? ""}</td>
                    {/if}
                {/each}
            </tr>
        {/each}
    </tbody>
</table>

<h4>已导入 {cursor}/{data.length}</h4>
<table>
    <thead>
        <tr>
            {#each collection?.schema ?? [] as field (field.name)}
                <th>{field.name}</th>
            {/each}
        </tr>
    </thead>
    <tbody>
        {#each data.slice(cursor) as row}
            <tr>
                {#each collection?.schema ?? [] as field (field.name)}
                    {#if isTablePlaceholder(record[field.name])}
                        <td>{toString(record[field.name], row).slice(6, -1)}</td>
                    {:else}
                        <td>{record[field.name] ?? ""}</td>
                    {/if}
                {/each}
            </tr>
        {/each}
    </tbody>
</table>

<style>
    table {
        white-space: nowrap;
    }
</style>
