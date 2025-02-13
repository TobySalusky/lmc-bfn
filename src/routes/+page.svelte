<script lang="ts">
	type Point = { x: number, y: number };
	// function p_sub(p1: Point, p2: Point): Point { return { x: p1.x - p2.x, y: p1.y - p2.y }; }
	function p(x: number, y: number): Point { return { x, y }; }
	function p_div(p1: Point, p2: Point): Point { return { x: p1.x / p2.x, y: p1.y / p2.y }; }
	function p_mult(p1: Point, p2: Point): Point { return { x: p1.x * p2.x, y: p1.y * p2.y }; }

	function p_print(p: Point) { console.log(`(x = ${p.x}, y = ${p.y})`); }

	// the coordinates of the top left and bottom right corner of the real google maps map GA map that I screen-shotted
	const tl_coords : Point = { x: 35.265865, y: -85.857200 };
	const br_coords : Point = { x: 30.167700, y: -80.815449 };
	const coords_size : Point = { x: tl_coords.x - br_coords.x, y: br_coords.y - tl_coords.y };

	function coords_to_01(coords: Point) {
		const coords_01 : Point = p_div(
			{ x: (coords.x - br_coords.x), y: coords.y - tl_coords.y },
			coords_size
		);
		coords_01.x = 1 - coords_01.x;

		return coords_01;
	}
	function wp(x: number, y: number): Point { return coords_to_01(p(x, y)); } // world position

	p_print(coords_to_01(tl_coords));
	p_print(coords_to_01(br_coords));

	type Farm = {
		name: string,
		coords: Point
	};

	const farms : Farm[] = [
		{
			name: 'Rountre',
			coords: wp(30.779374150801996, -83.56338783664418),
		},
		{
			name: 'Sumter',
			coords: wp(32.084812679614345, -84.2382799279373),
		},
		{
			name: 'Lewis Clark',
			coords: wp(30.79091102503036, -83.78904414311567),
		},
		{
			name: 'Fowler',
			coords: wp(31.517270005350806, -83.83701212695539),
		},
		{
			name: 'Hubert',
			coords: wp(33.26606220510343, -82.97665337841785),
		},
		{
			name: 'Charleston-Allen',
			coords: wp(33.55373745990568, -83.43136121356132
),
		},
		{
			name: 'Garfield Hall',
			coords: wp(32.44372966318054, -81.78066506446379),
		},
		{
			name: 'Gilliard',
			coords: p(31.16376054153567, -81.62389074555118),
		},
		{
			name: 'Williams',
			coords: p(30.91402190426384, -83.85165546143168),
		},
		{
			name: 'Cooper',
			coords: p(32.975456566583894, -81.75939468755824),
		},
		{
			name: 'Stephens',
			coords: p(31.51458649234062, -84.00240727678029),
		},
		{
			name: 'Thompson',
			coords: p(33.47800464351064, -82.06263032165144),
		},
		{
			name: 'Gouch',
			coords: p(33.08128853705585, -82.01175868075279
),
		},
		{
			name: 'Toomer',
			coords: p(32.4540073170362, -83.72931569977366),
		},
		// {
		// 	name: '',
		// 	coords: p(),
		// },
		// {
		// 	name: '',
		// 	coords: p(),
		// },
	];
</script>

<div style="width: 100%; display: flex; flex-direction: column; align-items: center;">
	<h1>Black Farmer's Network</h1>

	<div style="position: relative; display: flex; width: 80vw;">
		<img src="/images/ga_map_counties.png" alt="" />

		{#each farms as farm}
			<img src="/images/map_pin.png" alt="map pin" style={`width: 26px; position: absolute; top: ${farm.coords.x * 100}%; left: ${farm.coords.y * 100}%; transform: translate(-50%, -100%);`}>
		{/each}
	</div>
</div>
