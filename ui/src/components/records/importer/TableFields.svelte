<script>
    import TextField from "@/components/records/fields/TextField.svelte";
    import NumberField from "@/components/records/fields/NumberField.svelte";
    import BoolField from "@/components/records/fields/BoolField.svelte";
    import EmailField from "@/components/records/fields/EmailField.svelte";
    import UrlField from "@/components/records/fields/UrlField.svelte";
    import DateField from "@/components/records/fields/DateField.svelte";
    import SelectField from "@/components/records/fields/SelectField.svelte";
    import JsonField from "@/components/records/fields/JsonField.svelte";
    import RelationField from "@/components/records/fields/RelationField.svelte";
    import EditorField from "@/components/records/fields/EditorField.svelte";
    import ExcelPicker, { isTablePlaceholder } from "./ExcelPicker.svelte";
    import ExcelPickerWrapper from "./ExcelPickerWrapper.svelte";

    /**@type {import("pocketbase").Collection}*/
    export let collection;
    export let record;
    export let sheet;
    export let data;
    let picker;
</script>

<ExcelPicker bind:record {sheet} bind:this={picker} />
<form action="" on:submit|preventDefault={() => 0}>
    {#each collection?.schema || [] as field (field.name)}
        <ExcelPickerWrapper
            bind:value={record[field.name]}
            data={data.slice(0, 3)}
            show={() => picker?.show(field)}
        >
            {#if field.type === "text" || isTablePlaceholder(record[field.name])}
                <TextField {field} bind:value={record[field.name]} />
            {:else if field.type === "number"}
                <NumberField {field} bind:value={record[field.name]} />
            {:else if field.type === "bool"}
                <BoolField {field} bind:value={record[field.name]} />
            {:else if field.type === "email"}
                <EmailField {field} bind:value={record[field.name]} />
            {:else if field.type === "url"}
                <UrlField {field} bind:value={record[field.name]} />
            {:else if field.type === "editor"}
                <EditorField {field} bind:value={record[field.name]} />
            {:else if field.type === "date"}
                <DateField {field} bind:value={record[field.name]} />
            {:else if field.type === "select"}
                <SelectField {field} bind:value={record[field.name]} />
            {:else if field.type === "json"}
                <JsonField {field} bind:value={record[field.name]} />
            {:else if field.type === "file"}
                <!-- skip -->
            {:else if field.type === "relation"}
                <RelationField {field} bind:value={record[field.name]} />
            {/if}
        </ExcelPickerWrapper>
    {/each}
</form>
