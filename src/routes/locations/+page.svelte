<script>
	// @ts-nocheck
	// import writable stores 
	import { writable } from 'svelte/store';

	export let data;

	let categories = data.categories;
	let locations = data.locations;

	// Get session (returned from layout)
	let { session } = data;
	$: ({ session } = data);



	/**
	 * @param id {number}
	 */
	async function filterByCat(id = 0) {
		let endpoint = '/api/locations/';

		if (id != 0) {
			endpoint = `${endpoint}category/${id}`;
		}

		console.log('api endpoint: ', endpoint);

		// Call fetch
		const response = await fetch(endpoint);

		// if resonse code 200 (ok)
		if (response.ok) {
			// get json from resonse
			const json = await response.json();
			locations = json.data;
		}
	}
	

	async function delete_loc(id = 0) {
		if (confirm(`Permanently deleting product with ID= ${id}\n\nAre you sure?`)) {
			// call the store function if user confirms
			const result = await fetch(`/api/locations/${id}`, {
				method: 'DELETE'
			});

			// show the result
			alert('delete result: ', result);
			console.log('delete: ', result);
			
			filterByCat();
		}
	}
</script>

<!-- The HTML content of the page-->

<div class="container">
	<div class="row">
		<!-- Page Header -->
		<h2 class="mt-5">Locations from Supabase</h2>
	</div>
	{#if categories && locations}
		<div class="row">
			<div class="col-sm-2">
				<!-- Page Body Left Column (menu) -->
				<div id="categories" class="list-group">
					<button on:click={() => filterByCat(0)} class="list-group-item list-group-item-action">
						All Locations
					</button>
					{#each categories as cat}
						<button
							on:click={() => filterByCat(Number(cat.id))}
							class="list-group-item list-group-item-action"
						>
							{cat.name}
						</button>
					{/each}
				</div>
			</div>
			<!-- End category col -->

			<div class="col-sm-10">
				<!-- Page Body Right Side (Content goes here) -->
				<div id="locations">
					<table class="table table-striped table-bordered table-hover">
						<thead>
							<tr>
								<th>id</th>
								<th>Name</th>
								<th>Description</th>
								<th>Location (lat, long)</th>
								<th>Shared</th>
								<th>Favourite</th>
							</tr>
						</thead>
						<tbody>
							{#each locations as location}
								<tr>
									<td>{location.id}</td>
									<td>{location.name}</td>
									<td>{location.description}</td>
									<td
										><a
											href="https://www.openstreetmap.org/search?query={location.latitude}%2C{location.longitude}#map=17/{location.latitude}/{location.longitude}"
											target="_blank">{location.latitude}, {location.longitude}</a
										>
									</td>
									<td>{location.shared}</td>
									<td>{location.favourite}</td>
									<!-- check if logged in user is owner-->
									<!-- ? makes user.id optional in case it is null-->
									{#if session?.user.id == location.user_id && location.user_id != null}
									<td><a
											class="btn btn-sm btn-outline-primary"
											href="/add_up_location/{location.id}"
											role="button">
											<span class="bi bi-pencil-square" />
										</a>
									</td>
									<td>
										<button
											on:click={() => delete_loc(location.id)}
											class="btn btn-sm btn-outline-danger">
											<span class="bi bi-trash" />
										</button>
									</td>
									{/if}
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			</div>
			<!-- End locations col -->
		</div>
		<!-- End Row -->
	{:else}
		<p>No data to display</p>
	{/if}
	<p><a href="/add_up_location">Add new location</a></p>
</div>
<!-- End Main Content-->
