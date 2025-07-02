<script>
    import MapsTreeNode from "./MapsTreeNode.svelte";
    import { getInputContext } from '@evidence-dev/sdk/utils/svelte';
	import { duckdbSerialize } from '@evidence-dev/sdk/usql';

    export let treeData, columns, title, name;
    export let filters = []

    let allData = {};

    const inputs = getInputContext();

    $: tree = buildTree(treeData,columns);

    $: {
    }

    function refreshData(){
        console.log('Refresh Data!', filters);
        let qdata = {};
        filters.forEach(f => {
            columns.forEach(c => {
                if (f[c]){
                    if (!qdata[c]) qdata[c] = [];
                    qdata[c].push(f[c]);
                }
            });
        });
        columns.forEach(c => {
            if (!qdata[c]){
                qdata[c] = allData[c];
            }
        });
        Object.keys(qdata).forEach(k => {
            qdata[k] = duckdbSerialize(qdata[k] ? qdata[k] : []);
        });
        // Fill Up wih all Data where needed
        console.log("Query values",qdata,allData);
        $inputs[name] = qdata;
    }

    function addFilter(fin){
        filters = filters.filter(f => f._key != fin._key);
        filters.push(fin);
        refreshData();
    }

    function removeFilter(fin){
        filters = filters.filter(f => f._key != fin._key);
        refreshData();
    }

    const generateHash = (string) => {
        let hash = 0;
        for (const char of string) {
            hash = (hash << 5) - hash + char.charCodeAt(0);
            hash |= 0; // Constrain to 32bit integer
        }
        return hash;
    };

    function buildTree(treeData,columns){
        let tree = {
            name: title,
            filter: {},
            children: [],
            addFilter: addFilter,
            removeFilter: removeFilter
        };
        treeData.forEach(d => {
            var leaf = tree;
            columns.forEach(l => {
                var key = ''+d[l];
                if (allData[l]){
                    if (allData[l].filter(v => v == key).length == 0){
                        allData[l].push(key);
                    }
                } else {
                    allData[l] = [key];
                }
                var match = leaf.children.filter(n => {
                    return n.name == key;
                });
                if (match.length) {
                    leaf = match[0];
                } else {
                    var f = Object.assign({}, leaf.filter);
                    f[l] = key;
                    f['_key'] = generateHash(JSON.stringify(f));
                    var newnode = {
                        name: key,
                        filter: f,
                        children: [],
                        addFilter: addFilter,
                        removeFilter: removeFilter
                    };
                    leaf.children.push(newnode);
                    leaf = newnode;
                }
            });
        });
        //console.log(tree);
        refreshData();
        return tree;
    };

</script>


<MapsTreeNode {...tree}/>