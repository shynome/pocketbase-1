<script>
    import { onMount } from "svelte";
    export let table = "";

    /**@type {number[]}*/
    export let store = [];
    /**
     * @param i {number}
     */
    function select(i) {
        let nstore = Array.from(store);
        let index = nstore.indexOf(i);
        if (index === -1) {
            nstore.push(i);
        } else {
            nstore.splice(index, 1);
        }
        store = nstore;
    }

    /**@type {HTMLDivElement}*/
    let root;
    /**@type {HTMLElement[]}*/
    let tds = [];
    function apply_changes() {
        /**
         * @param td {HTMLElement}
         * @param i {number}
         */
        function setSelectedClass(td, i) {
            let selectedBy = store.indexOf(i);

            if (selectedBy !== -1) {
                td.classList.add("selected");
            } else {
                td.classList.remove("selected");
            }
        }
        Array.prototype.forEach.call(tds, setSelectedClass);
    }

    onMount(() => {
        function setTds() {
            /**@ts-ignore */
            tds = root.querySelectorAll("tr:first-child td");
        }
        setTds();
        apply_changes();
        function activeOnly(i = 0) {
            let t = tds[i];
            for (let td of tds) {
                if (td.classList.contains("selected")) {
                    continue;
                }
                td.classList.remove("hover");
                if (t === td) {
                    td.classList.add("hover");
                }
            }
        }
        /**
         * @param t {HTMLElement}
         */
        function indexOf(t) {
            return Array.prototype.indexOf.call(/**@type {HTMLElement}*/ (t.parentElement).childNodes, t);
        }
        /**@type {any}*/
        let last = null;
        /**
         * @param e {any}
         */
        function handleMouse(e) {
            /**@type {HTMLElement}*/
            let t = e.target;
            if (t == last) {
                return;
            }
            last = t;
            if (t.tagName != "TD") {
                return;
            }
            let i = indexOf(t);
            activeOnly(i);
        }
        root.addEventListener("mousemove", handleMouse);
        root.addEventListener("mouseleave", () => activeOnly(-1));
        root.addEventListener("mouseout", () => activeOnly(-1));
        root.addEventListener("click", (e) => {
            /**@type {HTMLElement}*/
            let t = /**@type {any}*/ (e.target);
            if (t.tagName != "TD") {
                return;
            }
            let i = indexOf(t);
            select(i);
            apply_changes();
        });
    });
</script>

<div bind:this={root} class="excel-table">
    {@html table}
</div>

<style>
    .excel-table {
        white-space: nowrap;
    }
    .excel-table :global(table) {
        position: relative;
    }
    .excel-table :global(table tr:first-child > td::before) {
        user-select: none;
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        height: 50000px;
        opacity: 0.6;
        z-index: 1;
        border-right: 1px solid #fff;
        background-color: var(--dangerColor);
        visibility: hidden;
    }
    .excel-table :global(table tr:first-child > td.hover::before),
    .excel-table :global(table tr:first-child > td.selected::before) {
        visibility: visible;
    }
    .excel-table :global(table tr td) {
        cursor: pointer;
    }
</style>
