<script>
	import '@evidence-dev/tailwind/fonts.css';
	import '../app.css';
	import { EvidenceDefaultLayout } from '@evidence-dev/core-components';
	export let data;
</script>

<EvidenceDefaultLayout {data} hideSidebar=true hideTOC=true hideHeader=true>
	<slot slot="content" />
</EvidenceDefaultLayout>
