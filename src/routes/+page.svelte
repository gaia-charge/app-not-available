<script>
	import {
		Heading,
		P,
		Search,
		Button,
		Dropdown,
		DropdownItem,
		Card,
		Footer,
		FooterCopyright,
		FooterIcon
	} from 'flowbite-svelte';
	import {
		ChevronDownOutline,
		GithubSolid,
		InstagramSolid,
		LinkedinSolid
	} from 'flowbite-svelte-icons';
	import appStoreRegions from '$lib/regions.js';

	let searchTerm;
	let selectCategory = 'europe';
	let dropdownOpen = false;
	let searchResults = {};
	let isLoading = false;
	let loadingStates = {};
	let errorMessage = '';

	// Function to extract App ID from Apple App Store URL or return the input if it's already a number
	const extractAppId = (input) => {
		if (!input) return null;
		
		// Check if input is already a numeric ID
		if (/^\d+$/.test(input.trim())) {
			return input.trim();
		}
		
		// Regex to extract ID from Apple App Store URLs
		const urlRegex = /\/id(\d+)(?:\?|$)/;
		const match = input.match(urlRegex);
		
		return match ? match[1] : null;
	};

	const doSearch = async () => {
		if (!searchTerm) return;

		const appId = extractAppId(searchTerm);
		const time = Date.now();
		if (!appId) {
			errorMessage = 'Please enter a valid Apple App Store URL or numeric App ID';
			return;
		}

		errorMessage = ''; // Clear any previous error message
		isLoading = true;
		searchResults = {};
		loadingStates = {}; // Reset loading states

		const promises = appStoreRegions[selectCategory].map(async (region) => {
			loadingStates[region.code] = true; // Set loading state for this region
			try {
				const response = await fetch(
					`https://itunes.apple.com/lookup?id=${appId}&country=${region.code}&rand=${time}`
				);
				const data = await response.json();
				searchResults[region.code] = data.resultCount > 0;
			} catch (error) {
				console.error(`Error checking ${region.code}:`, error);
				searchResults[region.code] = false;
			}
			loadingStates[region.code] = false; // Clear loading state for this region
		});

		await Promise.all(promises);
		isLoading = false;
	};

	const clearErrorMessage = () => {
		if (errorMessage) {
			errorMessage = '';
		}
	};

	const formatCamelCase = (text) => {
		return (
			text
				.replace(/([A-Z])/g, ' $1')
				.trim()
				.charAt(0)
				.toUpperCase() +
			text
				.replace(/([A-Z])/g, ' $1')
				.trim()
				.slice(1)
		);
	};
</script>

<svelte:head>
	<title>App Not Available?</title>
</svelte:head>

<div class="mx-auto flex min-h-screen w-3/4 flex-col text-center 2xl:w-1/2">
	<div class="mt-8 flex-grow">
		<Heading tag="h1" class="mb-4" customSize="text-4xl font-extrabold md:text-5xl lg:text-6xl"
			>🌐 App Not Available?</Heading
		>
		<P class="mb-8 text-center text-lg dark:text-gray-400 lg:text-xl">
			Find in which App Store regions the app is available.
		</P>

		<div class="relative">
			{#if errorMessage}
				<div class="absolute -top-16 left-1/2 z-10 w-full max-w-xl -translate-x-1/2 transform rounded-lg bg-red-100 p-4 text-red-700 shadow-lg dark:bg-red-900 dark:text-red-300">
					{errorMessage}
				</div>
			{/if}

			<form
			id="search-form"
			on:submit|preventDefault={doSearch}
			class="mx-auto flex max-w-xl flex-col gap-2 sm:flex-row"
		>
			<div class="relative w-full sm:w-auto">
				<Button
					color="dark"
					size="lg"
					class="w-full whitespace-nowrap rounded-none sm:rounded-e-none sm:border-e-0"
				>
					{formatCamelCase(selectCategory)}
					<ChevronDownOutline class="ms-2.5 h-2.5 w-2.5" />
				</Button>
				<Dropdown bind:open={dropdownOpen} classContainer="w-full sm:w-40">
					{#each Object.keys(appStoreRegions) as region}
						<DropdownItem
							on:click={() => {
								selectCategory = region;
								dropdownOpen = false;
							}}
							class={selectCategory === region ? 'underline' : ''}
						>
							{formatCamelCase(region)}
						</DropdownItem>
					{/each}
				</Dropdown>
			</div>
			<div class="flex w-full flex-col gap-2 sm:flex-row">
				<Search
					bind:value={searchTerm}
					on:input={clearErrorMessage}
					placeholder="Enter App Store URL or numeric App ID"
					class="w-full rounded-none sm:rounded-s-none"
				/>
				<Button type="submit" color="dark" class="w-full rounded-none sm:w-auto">Search</Button>
			</div>
		</form>
		</div>

		<div class="mb-8 mt-8 grid grid-cols-1 gap-8">
			<div>
				<h2 class="mb-4 text-2xl font-bold">
					{formatCamelCase(selectCategory)}
				</h2>
				<div class="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
					{#each appStoreRegions[selectCategory] as region}
						<Card padding="sm">
							<div class="flex items-center space-x-2">
								<span class="text-2xl">{region.emoji}</span>
								<span class="flex w-full items-center justify-between text-xl">
									{#if searchTerm && Object.hasOwn(searchResults, region.code)}
										{#if loadingStates[region.code]}
											<span>{region.name}</span>
											<svg class="h-5 w-5 animate-spin" viewBox="0 0 24 24">
												<circle
													class="opacity-25"
													cx="12"
													cy="12"
													r="10"
													stroke="currentColor"
													stroke-width="4"
													fill="none"
												/>
												<path
													class="opacity-75"
													fill="currentColor"
													d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
												/>
											</svg>
										{:else if searchResults[region.code]}
											<a
												href="https://apps.apple.com/{region.code}/app/id{extractAppId(searchTerm)}"
												target="_blank"
												class="text-blue-600 hover:underline"
											>
												{region.name}
											</a>
											<span>✅</span>
										{:else}
											<span>{region.name}</span>
											<span>❌</span>
										{/if}
									{:else}
										<span>{region.name}</span>
										<span>❔</span>
									{/if}
								</span>
							</div>
						</Card>
					{/each}
				</div>
			</div>
		</div>
	</div>

	<Footer>
		<div class="mb-8 sm:flex sm:items-center sm:justify-between">
			<FooterCopyright href="https://gaiacharge.com/" by="Gaia Charge" year={2025} />
			<div class="mt-4 flex space-x-6 sm:mt-0 sm:justify-center rtl:space-x-reverse">
				<FooterIcon href="https://www.instagram.com/gaia.charge/">
					<InstagramSolid
						class="h-5 w-5 text-gray-500 hover:text-gray-900 dark:text-gray-500 dark:hover:text-white"
					/>
				</FooterIcon>
				<FooterIcon href="https://www.linkedin.com/in/gaia-charge/">
					<LinkedinSolid
						class="h-5 w-5 text-gray-500 hover:text-gray-900 dark:text-gray-500 dark:hover:text-white"
					/>
				</FooterIcon>
				<FooterIcon href="https://github.com/gaia-charge">
					<GithubSolid
						class="h-5 w-5 text-gray-500 hover:text-gray-900 dark:text-gray-500 dark:hover:text-white"
					/>
				</FooterIcon>
			</div>
		</div>
	</Footer>
</div>
