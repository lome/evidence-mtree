<script>
    export let data;
    export let columns;
    export let rows;
    export let metric;

    export let aggregation = 'sum'; // sum, count, avg
    export let showRowTotals = true;
    export let showColTotals = true;

    const TOTAL_KEY = '__TOTAL__';

    $: collapsedRowPaths = new Set();

    function toggleRow(path) {
        const pathKey = JSON.stringify(path);
        if (collapsedRowPaths.has(pathKey)) {
            collapsedRowPaths.delete(pathKey);
        } else {
            collapsedRowPaths.add(pathKey);
        }
        collapsedRowPaths = collapsedRowPaths;
    }

    function arrayCompare(a, b) {
        for (let i = 0; i < Math.min(a.length, b.length); i++) {
            if (a[i] === TOTAL_KEY && b[i] !== TOTAL_KEY) return 1;
            if (a[i] !== TOTAL_KEY && b[i] === TOTAL_KEY) return -1;
            if (a[i] < b[i]) return -1;
            if (a[i] > b[i]) return 1;
        }
        return a.length - b.length;
    }

    $: table = buildTable(data, columns, rows, metric, aggregation, showRowTotals, showColTotals);

    $: visibleRowIndices = table.sortedRowPaths
        .map((_, i) => i)
        .filter(i => {
            const path = table.sortedRowPaths[i];
            // A row is visible if none of its parent paths are collapsed.
            for (let j = 1; j < path.length; j++) {
                const parentPath = path.slice(0, j);
                if (collapsedRowPaths.has(JSON.stringify(parentPath))) {
                    // This row is a child of a collapsed path.
                    // It should be hidden, unless it is the total row for that collapsed path
                    const isTotalRowForParent = path.slice(j).every(p => p === TOTAL_KEY);
                    if (!isTotalRowForParent) {
                        return false;
                    }
                }
            }
            return true;
        });
    
    $: visiblePaths = table.sortedRowPaths.filter((_, i) => visibleRowIndices.includes(i));
    $: visibleBody = table.body.filter((_, i) => visibleRowIndices.includes(i));
    $: visibleRowHeaders = buildRowHeaders(visiblePaths, rows.length);

    function getPathsToUpdate(path, withTotals) {
        if (!withTotals) return [path];
        const paths = [path];
        for (let i = 1; i <= path.length; i++) {
            paths.push([...path.slice(0, path.length - i), ...Array(i).fill(TOTAL_KEY)]);
        }
        return paths;
    }

    function buildTable(data, columns, rows, metric, aggregation, showRowTotals, showColTotals) {
        if (!data?.length || !columns?.length || !rows?.length || !metric) {
            return { colHeaders: [], rowHeaders: [], body: [], sortedRowPaths: [], sortedColPaths: [] };
        }

        const cells = new Map();

        for (const d of data) {
            const metricValue = d[metric] ?? 0;
            const originalRowPath = rows.map(r => d[r]);
            const originalColPath = columns.map(c => d[c]);

            const rowPaths = getPathsToUpdate(originalRowPath, showRowTotals);
            const colPaths = getPathsToUpdate(originalColPath, showColTotals);

            for (const rp of rowPaths) {
                for (const cp of colPaths) {
                    const key = `${JSON.stringify(rp)}|${JSON.stringify(cp)}`;
                    let cell = cells.get(key);
                    if (!cell) {
                        cell = { sum: 0, count: 0 };
                        cells.set(key, cell);
                    }
                    cell.sum += metricValue;
                    cell.count++;
                }
            }
        }

        const rowKeys = new Set();
        const colKeys = new Set();
        for(const key of cells.keys()){
            const [rowKey, colKey] = key.split('|');
            rowKeys.add(rowKey);
            colKeys.add(colKey);
        }

        const sortedRowPaths = [...rowKeys].map(JSON.parse).sort(arrayCompare);
        const sortedColPaths = [...colKeys].map(JSON.parse).sort(arrayCompare);

        const body = sortedRowPaths.map(rowPath => {
            const rowKey = JSON.stringify(rowPath);
            return sortedColPaths.map(colPath => {
                const colKey = JSON.stringify(colPath);
                const cell = cells.get(`${rowKey}|${colKey}`);
                if (!cell) return undefined;
                switch (aggregation) {
                    case 'sum': return cell.sum;
                    case 'count': return cell.count;
                    case 'avg': return cell.count > 0 ? cell.sum / cell.count : 0;
                    default: return cell.sum;
                }
            });
        });

        const colHeaders = buildColHeaders(sortedColPaths, columns.length);
        const rowHeaders = buildRowHeaders(sortedRowPaths, rows.length);

        sortedRowPaths.forEach(rp => {
            for (let j=0; j < rp.length; j++){
                let v = JSON.stringify(rp.slice(0,j));
                collapsedRowPaths.add(v);
            }
        })

        collapsedRowPaths = collapsedRowPaths;
        return { colHeaders, rowHeaders, body, sortedRowPaths, sortedColPaths };
    }

    function buildColHeaders(paths, depth) {
        if (!paths.length) return [];
        const headers = Array.from({ length: depth }, () => []);
        for (let level = 0; level < depth; level++) {
            for (let i = 0; i < paths.length; ) {
                const currentPath = paths[i];
                let span = 1;
                for (let j = i + 1; j < paths.length; j++) {
                    if (arrayCompare(paths[j].slice(0, level + 1), currentPath.slice(0, level + 1)) === 0) {
                        span++;
                    } else {
                        break;
                    }
                }

                let value = currentPath[level];
                const isTotal = value === TOTAL_KEY;
                if (isTotal) {
                    const isGrandTotal = currentPath.every(p => p === TOTAL_KEY);
                    if (isGrandTotal) {
                        value = level === 0 ? '' : '';
                    } else {
                        value = '';
                    }
                }
                headers[level].push({ value, span, isTotal });
                i += span;
            }
        }
        return headers;
    }

    function buildRowHeaders(paths, depth) {
        if (!paths.length) return [];
        const headerGrid = paths.map(() => []);
        for (let level = 0; level < depth; level++) {
            for (let i = 0; i < paths.length; ) {
                const currentPath = paths[i];
                let span = 1;
                for (let j = i + 1; j < paths.length; j++) {
                    if (arrayCompare(paths[j].slice(0, level + 1), currentPath.slice(0, level + 1)) === 0) {
                        span++;
                    } else {
                        break;
                    }
                }

                let value = currentPath[level];
                const isTotal = value === TOTAL_KEY;
                if (isTotal) {
                    const isGrandTotal = currentPath.every(p => p === TOTAL_KEY);
                    if (isGrandTotal) {
                        value = '';
                    } else {
                        value = '';
                    }
                }

                const path = currentPath.slice(0, level + 1);
                const isExpandable = level < (depth-1) && !isTotal;

                headerGrid[i][level] = { value, span, isTotal, path, isExpandable };
                for (let k = 1; k < span; k++) {
                    headerGrid[i + k][level] = { value: null, span: 0, isTotal: false };
                }
                i += span;
            }
        }
        return headerGrid;
    }

</script>

<style>
    table {
        border-collapse: collapse;
        width: 100%;
        font-family: sans-serif;
        font-size: 0.9em;
        white-space: nowrap;
    }
    th, td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: left;
    }
    th {
        background-color: #f2f2f2;
        font-weight: bold;
    }
    thead th {
        text-align: center;
        position: sticky;
        top: 0;
        z-index: 2;
    }
    tbody th {
        position: sticky;
        left: 0;
        z-index: 1;
        background-color: #f2f2f2; /* To make sure it covers what's scrolling under it */
    }
    td {
        text-align: right;
    }
    .scrollbox {
        overflow: auto;
        width: 100%;
    }
    .total {
        font-weight: bold;
        background-color: #e8e8e8;
    }
    tbody th button {
        border: none;
        background: none;
        cursor: pointer;
        font-family: monospace;
        margin-right: 4px;
        padding: 0;
        width: 1em;
    }
</style>

<div class="scrollbox pretty-scrollbar">
    {#if table.body.length}
    <table>
        <thead>
            {#each table.colHeaders as headerRow, i}
                <tr>
                    {#if i === 0 && rows.length > 0}
                        <th colspan={rows.length} rowspan={columns.length}></th>
                    {/if}
                    {#each headerRow as cell}
                        {#if cell.span > 0}
                            <th colspan={cell.span} class:total={cell.isTotal}>{cell.value}</th>
                        {/if}
                    {/each}
                </tr>
            {/each}
        </thead>
        <tbody>
            {#each visibleBody as dataRow, i}
                {@const originalIndex = visibleRowIndices[i]}
                <tr>
                    {#each visibleRowHeaders[i] as cell}
                        {#if cell.span > 0}
                            <th rowspan={cell.span} class:total={cell.isTotal}>
                                {#if cell.isExpandable}
                                    <button on:click={() => toggleRow(cell.path)}>
                                        {collapsedRowPaths.has(JSON.stringify(cell.path)) ? '+' : '-'}
                                    </button>
                                {/if}
                                {cell.value}
                            </th>
                        {/if}
                    {/each}
                    {#each dataRow as value, j}
                        <td class:total={table.sortedRowPaths[originalIndex].includes(TOTAL_KEY) || table.sortedColPaths[j].includes(TOTAL_KEY)}>
                            {#if value !== undefined && value !== null}
                                {value.toLocaleString(undefined, { maximumFractionDigits: 2 })}
                            {/if}
                        </td>
                    {/each}
                </tr>
            {/each}
        </tbody>
    </table>
    {:else}
        <p>Please provide data and configuration to render the pivot table.</p>
    {/if}
</div>