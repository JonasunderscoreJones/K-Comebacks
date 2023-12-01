<script>
	import { onMount } from "svelte";

	let releases = [];
	let searchArtist = "";
	let searchTitle = "";
	let selectedRelease = "";
	let startDate = null;
	let endDate = null;
	let filteredReleases = [];
	let pageSize = 50;
	let currentPage = 1;
	let releaseTypes = [];
	let totalPages = 1;
	let pages = [];
	let last_update = "Loading...";
	let loading = "Loading...";

	function sortByColumn(column) {
		// Sort releases based on the selected column
		releases.sort((a, b) => {
			if (column === "date") {
				return new Date(b.date) - new Date(a.date);
			} else {
				return a[column].localeCompare(b[column]);
			}
		});
		filterReleases(); // Reapply filtering after sorting
	}

	function filterReleases() {
		filteredReleases = releases.filter((release) => {
			const artistMatch = release.artist
				.toLowerCase()
				.includes(searchArtist.toLowerCase());
			const titleMatch = release.title
				.toLowerCase()
				.includes(searchTitle.toLowerCase());
			const releaseTypeMatch =
				selectedRelease === "" ||
				(release.type === null
					? "Unknown"
					: release.type) === selectedRelease ||
					release.type.includes(selectedRelease);
			const dateMatch =
				(!startDate || new Date(release.date) >= startDate) &&
				(!endDate || new Date(release.date) <= endDate);
			return artistMatch && titleMatch && releaseTypeMatch && dateMatch;
		});

		// Implement pagination
		const startIndex = (currentPage - 1) * pageSize;
		const endIndex = startIndex + pageSize;
		//const startIndex = 0;
		//const endIndex = filteredReleases.length;
		filteredReleases = filteredReleases.slice(startIndex, endIndex);

		// Update pagination buttons
		updatePagination();
	}

	function updatePagination() {
		totalPages = Math.ceil(releases.length / pageSize);
		console.log("totalPages: " + totalPages);
		pages = Array.from({ length: totalPages }, (_, index) => index + 1);
	}

	function goToPage(page) {
		currentPage = page;
		filterReleases();
	}

	onMount(async () => {
    try {
      const response = await fetch('https://cdn.jonasjones.dev/api/kcomebacks/rkpop_data.json');
	  //const response = await fetch('/rkpop_data.json')
      if (response.ok) {
        releases = await response.json();

		last_update = releases[0].last_update;
		releases.shift();

        // Extract unique release types from the releases array
        releaseTypes = Array.from(new Set(releases.map(release => {
          return release.type === null ? 'Unknown' : release.type;
        })));

		// releaseTypes should be an array of all elements of all the release.type lists of all releases
		releaseTypes = releaseTypes.flat();

        // Sort releases by date from most recent to least recent
        releases.sort((a, b) => new Date(b.date) - new Date(a.date));

        filterReleases(); // Initial filtering based on default values

		loading = "";
      } else {
        console.error('Failed to fetch data:', response.status);
      }
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  });
</script>

<div class="container">
	<div class="title-banner">
		<h1 class="title">K-Comebacks</h1>
	</div>
	
	<div class="filter-bar">
		<div class="filter-input">
			<label for="artist">Artist:</label>
			<input
				type="text"
				id="artist"
				bind:value={searchArtist}
				on:input={filterReleases}
			/>
		</div>
	
		<div class="filter-input">
			<label for="title">Title:</label>
			<input
				type="text"
				id="title"
				bind:value={searchTitle}
				on:input={filterReleases}
			/>
		</div>
	
		<div class="filter-input">
			<label for="release">Release Type:</label>
			<select
				id="release"
				bind:value={selectedRelease}
				on:change={filterReleases}
			>
				<option value="">All</option>
				{#each releaseTypes as releaseType}
					<option value={releaseType}>{releaseType}</option>
				{/each}
			</select>
		</div>
	
		<div class="filter-input">
			<label for="start-date">Start Date:</label>
			<input
				type="date"
				id="start-date"
				bind:value={startDate}
				on:input={filterReleases}
			/>
		</div>
	
		<div class="filter-input">
			<label for="end-date">End Date:</label>
			<input
				type="date"
				id="end-date"
				bind:value={endDate}
				on:input={filterReleases}
			/>
		</div>
	</div>
	
	<div class="pagination-bar filter-bar">
		<div class="filter-input">
			<label for="page-size">Entries per Page:</label>
			<select id="page-size" bind:value={pageSize} on:change={filterReleases}>
				<option value="5">5</option>
				<option value="10">10</option>
				<option value="15">15</option>
				<option value="25">25</option>
				<option value="50">50</option>
				<option value="100">100</option>
				<option value="250">250</option>
			</select>
		</div>
		<div>
			{#if totalPages > 1}
				<button on:click={() => goToPage(1)}>First</button>
				<button on:click={() => goToPage(currentPage - 1)}>Previous</button>
				{#if currentPage > 3}
					<button on:click={() => goToPage(currentPage - 3)}>{currentPage - 3}</button>
				{/if}
				{#if currentPage > 2}
					<button on:click={() => goToPage(currentPage - 2)}>{currentPage - 2}</button>
				{/if}
				{#if currentPage > 1}
					<button on:click={() => goToPage(currentPage - 1)}>{currentPage - 1}</button>
				{/if}
				<button class="active">{currentPage}</button>
				{#if currentPage < totalPages}
					<button on:click={() => goToPage(currentPage + 1)}>{currentPage + 1}</button>
				{/if}
				{#if currentPage < totalPages - 1}
					<button on:click={() => goToPage(currentPage + 2)}>{currentPage + 2}</button>
				{/if}
				{#if currentPage < totalPages - 2}
					<button on:click={() => goToPage(currentPage + 3)}>{currentPage + 3}</button>
				{/if}
				<button on:click={() => goToPage(currentPage + 1)}>Next</button>
				<button on:click={() => goToPage(totalPages)}>Last</button>
			{/if}
		</div>
	</div>
	<p>Click on the column title to filter by them</p>
	<p>Last updated: {last_update}</p>
	<table>
		<thead>
			<tr>
				<th>Nr.</th>
				<th on:click={() => sortByColumn("date")}>Date</th>
				<th on:click={() => sortByColumn("artist")}>Artist</th>
				<th on:click={() => sortByColumn("title")}>Title</th>
				<th on:click={() => sortByColumn("type")}>Release Type</th>
			</tr>
		</thead>
		<tbody>
			{#each filteredReleases as release}
				<tr>
					<!--calculate the release number-->
					<td>{releases.length - releases.indexOf(release)}</td>
					<td>{release.date}</td>
					<td>{release.artist}</td>
					<td>{release.title}</td>
					<td
						>{release.type === null
							? "Unknown"
							: release.type}</td
					>
				</tr>
			{/each}
		</tbody>
	</table>
	<h1>{loading}</h1>
</div>

<style>
	@import "../styles/global.css";
	.title {
		color: var(--color-primary);
	}

	table {
		width: 100%;
		border-collapse: collapse;
		margin-top: 20px;
	}

	th,
	td {
		border: 2px solid var(--background-color);
		padding: 10px;
		text-align: left;
	}

	th {
		background-color: var(--background-color);
		cursor: pointer;
	}

	.filter-bar {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 10px;
	}

	.filter-input {
		margin-right: 10px;
	}

	.title-banner {
		display: flex;
		justify-content: center;
		align-items: center;
		margin-bottom: 10px;
		
		height: 200px;
		font-size: 50px;
	}
</style>
