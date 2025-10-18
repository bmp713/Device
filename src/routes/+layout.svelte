<script>
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';
	
	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<!-- Load Montserrat, Inter, Oswald and other chosen fonts directly in the document head to ensure availability -->
	<link href="https://fonts.googleapis.com/css2?family=Anton&family=Montserrat:wght@300;400;500;600;700&family=Inter:wght@300;400;500;600;700&family=Poppins:wght@300;400;500;600;700&family=Roboto:wght@300;400;500;700&family=Lato:wght@300;400;700&family=Oswald:wght@300;400;500;600&display=swap" rel="stylesheet">
	<title>Heart Rate</title>
</svelte:head>

{@render children?.()}
