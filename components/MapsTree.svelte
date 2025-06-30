<script>
    import getSortedData from '@evidence-dev/component-utilities/getSortedData';
    import MapsTreeNode from "./MapsTreeNode.svelte";

    export let treeData, columns, title, name;

    $: tree = buildTree(treeData,columns);

    function buildTree(treeData,columns){
        let tree = {
            name: title,
            filter: [],
            children: [],
            input_name: name,
            columns: columns
        };
        treeData.forEach(d => {
            var leaf = tree;
            columns.forEach(l => {
                var key = ''+d[l];
                var match = leaf.children.filter(n => {
                    return n.name == key;
                });
                if (match.length) {
                    leaf = match[0];
                } else {
                    var f = {}; f[l] = key;
                    var newnode = {
                        name: key,
                        filter: leaf.filter.concat([f]),
                        children: [],
                        input_name: name,
                        columns: columns
                    };
                    leaf.children.push(newnode);
                    leaf = newnode;
                }
            });
        });
        //console.log(tree);
        return tree;
    };

</script>


<MapsTreeNode {...tree}/>