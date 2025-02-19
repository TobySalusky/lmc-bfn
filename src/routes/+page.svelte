<script lang="ts">
	import LineGraph from "../LineGraph.svelte";
	// @ts-ignore
	import Slider from '@bulatdashiev/svelte-slider';

	const graph_years = [
		1900,
		1910,
		1920,
		1930,
		1940,
		1950,
		1959,
		1964,
		1969,
		1974,
		1978,
		1982,
		1987,
		1992,
		1997,
		2002,
		2007,
		2012,
		2017,
		2022,
	];
	let value = [0, 100];

	const graph_values = [
		13.01,
		14.03,
		14.34,
		14.02,
		11.17,
		10.39,
		7.35,
		5.83,
		3.20,
		1.97,
		1.65,
		1.48,
		1.10,
		0.98,
		0.97,
		1.34,
		1.39,
		1.58,
		1.55,
		1.48,
	];

	const timeline_data = [
		{
			year: 1933,
			name: 'Agricultural Adjustment Act',
			text: 'New Deal policies aim to address low crop prices by reducing farmland acreage, but lead to the displacement of many Black tenant farmers.',
		},
		{
			year: 1965,
			name: 'USDA Discrimination Report',
			text: 'The US Commission on Civil Rights confirms that the USDA discriminates against Black farmers in loans and conservation payments.',
		},
		{
			year: 1968,
			name: 'Continued Discrimination in Loans',
			text: 'A followup report finds ongoing bias in USDA loan distribution, worsening economic struggles for Black farmers.',
		},
		{
			year: 1970,
			name: 'Lack of Civil Rights Oversight',
			text: 'The Commission on Civil Rights reports persistent discrimination in USDA programs, with no civil rights staff in field offices.',
		},
		{
			year: 1981,
			name: 'Black Farmers and Poverty',
			text: 'A USDA study finds Black and minority farmers are disproportionately affected by poverty and unable to access credit.',
		},
		{
			year: 1983,
			name: 'USDA Civil Rights Office Dismantled',
			text: 'The Reagan administration shuts down the USDA Office of Civil Rights, halting investigations into discrimination.',
		},
		{
			year: 1990,
			name: 'House Report on USDA Discrimination',
			text: 'A House Committee investigation reveals widespread racial bias in USDA loan programs, denying funding for Black farmers.',
		},
		{
			year: 1994,
			name: 'Legal Recognition of Discrimination',
			text: 'The US Assistant Attorney General issues a memo about the USDA\'s authority to award monetary relief to Black farmers who have suffered from discriminatory practices, opening the door for restitution and corrective measures.',
		},
		{
			year: 1995,
			name: 'GAO Report on USDA Inequality',
			text: 'The US General Accounting Office finds the USDA fails to address racial discimination and lacks minority representation on county committees.',
		},
		{
			year: 1997,
			name: 'USDA Civil Rights Action Team Report',
			text: 'The USDA Civil Rights Action Team publishes a report about the enduring nature of discrimination within the department.',
		},
		{
			year: 1999,
			name: 'Pigford v. Glickman Settlement',
			text: 'The USDA compensates $1.03 billion to Black farmers affected by loan discrimination, but only a fraction of claims are accepted.',
		},
		{
			year: 2001,
			name: 'Wave of USDA Discrimination Complaints',
			text: 'The USDA receives over 14,000 discrimination complaints from 2001 to 2008, but only finds merit in one.',
		},
		{
			year: 2010,
			name: 'Pigford Additional Settlement',
			text: 'Congress approves a $1.25 billion settlement to compensate black farmers originally excluded from the Pigford case.',
		},
		{
			year: 2019,
			name: 'USDA Found Distorting Data',
			text: 'An investigation finds USDA has been inflating statistics on black farmers and exaggerating their record on civil rights.  ',
		},
		{
			year: 2021,
			name: 'American Rescue Plan',
			text: 'The American Rescue Plan Act $1.01 billion to assist socially disadvantaged farmers, ranchers, and forest landowners.',
		},
		{
			year: 2022,
			name: 'Inflation Reduction Act',
			text: 'The Inflation Reduction Act provides over $2 billion for farmers who have experienced discrimination.',
		},
	];
	let timeline_point = timeline_data[0];

	function set_event_i(i: number) {
		timeline_point = timeline_data[i];
		value[0] = Math.round((timeline_point.year - 1933) / (2022-1933) * 100);
		set_active_graph_year(timeline_point.year);
	}

	function has_prev_event() {
		const i = timeline_data.findIndex(({ year }) => year == timeline_point.year);
		return i > 0;
	}

	function prev_event() {
		const i = timeline_data.findIndex(({ year }) => year == timeline_point.year);
		if (i === -1) { return; }

		set_event_i(i - 1);
	}

	function has_next_event() {
		const i = timeline_data.findIndex(({ year }) => year == timeline_point.year);
		return i != -1 && i < timeline_data.length - 1;
	}

	function next_event() {
		const i = timeline_data.findIndex(({ year }) => year == timeline_point.year);
		if (i === -1) { return; }

		set_event_i(i + 1);
	}

	const graph_data = graph_years.map((year, i) => ({
		year, value: graph_values[i]
	}))

	let active_graph_year : number = 1910;
	let active_graph_data = graph_data.slice(0, 0);
	function set_active_graph_year(new_year : number) {
		active_graph_year = new_year;
		const timeline_upto = timeline_data.findLastIndex(({ year }) => year <= new_year);
		if (timeline_data[timeline_upto] != null) {
			timeline_point = timeline_data[timeline_upto];
		}

		let upto = graph_data.findLastIndex(({ year }) => year <= new_year);
		if (upto === -1) { upto = 0; }

		active_graph_data = graph_data.slice(0, upto + 1);
		const last_good = graph_data[upto];

		// interpolated final data point!
		if (upto !== graph_data.length - 1 && last_good.year !== new_year) {
			const next = graph_data[upto + 1];
			const deltaValue = next.value - last_good.value;
			const lt = (new_year - last_good.year) / (next.year - last_good.year);

			active_graph_data.push({
				year: new_year,
				value: last_good.value + (deltaValue * lt)
			});
		}
		active_graph_data = active_graph_data; // for svelte to detect change
	}
	set_active_graph_year(1933);

	type Point = { x: number; y: number };
	// function p_sub(p1: Point, p2: Point): Point { return { x: p1.x - p2.x, y: p1.y - p2.y }; }
	function p(x: number, y: number): Point {
		return { x, y };
	}
	function p_div(p1: Point, p2: Point): Point {
		return { x: p1.x / p2.x, y: p1.y / p2.y };
	}
	function p_mult(p1: Point, p2: Point): Point {
		return { x: p1.x * p2.x, y: p1.y * p2.y };
	}

	function p_print(p: Point) {
		console.log(`(x = ${p.x}, y = ${p.y})`);
	}

	// the coordinates of the top left and bottom right corner of the real google maps map GA map that I screen-shotted
	const tl_coords: Point = { x: 35.265865, y: -85.8572 };
	const br_coords: Point = { x: 30.1677, y: -80.815449 };
	const coords_size: Point = { x: tl_coords.x - br_coords.x, y: br_coords.y - tl_coords.y };

	let farm_open: string | undefined = undefined;

	function coords_to_01(coords: Point) {
		const coords_01: Point = p_div(
			{ x: coords.x - br_coords.x, y: coords.y - tl_coords.y },
			coords_size
		);
		coords_01.x = 1 - coords_01.x;

		return coords_01;
	}
	function wp(x: number, y: number): Point {
		return coords_to_01(p(x, y));
	} // world position

	// JAMES'S NEW FARM CONTENT

	type Farm = {
		name: string;
		county: string;
		coords: Point;
		about: string;
		history: string;
		present: string;
		founded: string;
		acres: number;
		image: string;
		source: string;
	};

	const farms: Farm[] = [
		{
			name: 'Rountree',
			county: 'Brooks',
			coords: wp(30.779374150801996, -83.56338783664418),
			about:
				'Founded in 1891, the John and Emma Jane Rountree Farm is a historic African American family farm that has remained in the Rountree family for over a century.',
			history:
				'John Willis Rountree purchased 40 acres in 1891 and expanded it to 84 acres by 1917. He built a family home, barns, and a barbershop for his son while growing peaches, vegetables, pecans, and tobacco. The farm also had a timber reserve for additional income.',
			present:
				'The farm remains under the care of Rountree’s grandchildren and cousins, preserving its rich cultural heritage and symbolizing African American land ownership and historical preservation.',
			founded: '1891',
			acres: 84,
			image: 'images/farms/rountree.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2001, February). John & Emma Jane Rountree Farm: A Centennial Family Farm (Vol. I, No. 2). Jeanne Cyriaque, Editor'
		},
		{
			name: 'Morgan Family Farm',
			county: 'Sumter',
			coords: wp(32.084812679614345, -84.2382799279373),
			about:
				'The Morgan Family Farm is a historically significant African American farm that has been in operation for over a century. Established in 1886 by Nathan Morgan Jr., the farm has remained in the Morgan family for generations.',
			history:
				'Nathan Morgan Jr. (1849–1917), a former slave, purchased 202 acres from A. Windsor in 1886. Milton Morgan later took over, farming cotton, corn, peanuts, and vegetables using mules. Carranza Morgan modernized the farm, adding 6 outbuildings and introducing modern farming techniques.',
			present:
				'The Morgan Family Farm remains a powerful symbol of African American agricultural heritage and perseverance. Its continued operation and historical recognition highlight the importance of preserving African American farming history for future generations.',
			founded: '1886',
			acres: 202,
			image: 'images/farms/morgan.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2000, December). 2000 Georgia Centennial Farm Awards Honor Reverend James Fowler Farm (Vol. I, No. 1). Jeanne Cyriaque, Editor'
		},
		{
			name: 'Lewis Clark',
			county: 'Thomas',
			coords: wp(30.79091102503036, -83.78904414311567),
			about:
				'The Lewis Clark Farm is a historic African American family farm, established in 1875 through the land purchase of Lewis Clark.',
			history:
				'Lewis Clark purchased 50 acres, cultivating cotton, corn, sweet potatoes, sugar cane, and garden vegetables. In 1899, he passed the farm to his daughter, Lenary Clark Allen Williams, who farmed the land until her passing in 1987.',
			present:
				"Though the farm is currently leased and lacks its original structures, it remains a symbol of the Clark family's agricultural legacy and the importance of African American land ownership in Georgia history.",
			founded: '1875',
			acres: 50,
			image: 'images/farms/clark.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2000, December). 2000 Georgia Centennial Farm Awards Honor Reverend James Fowler Farm (Vol. I, No. 1). Jeanne Cyriaque, Editor'
		},
		{
			name: 'Fowler Farm',
			county: 'Worth',
			coords: wp(31.517270005350806, -83.83701212695539),
			about:
				'Founded in 1888 by Fowler, who was born into slavery, this 202-acre farm has remained in the family for generations.',
			history:
				'Fowler’s son inherited the farm and later passed it to his daughter. The farm survived periods of drought and has primarily grown cotton, soybeans, peanuts, and wheat.',
			present:
				'The farm continues to thrive, representing African American agricultural heritage and perseverance.',
			founded: '1888',
			acres: 202,
			image: 'images/farms/fowler.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2000, December). 2000 Georgia Centennial Farm Awards Honor Reverend James Fowler Farm (Vol. I, No. 1). Jeanne Cyriaque, Editor.'
		},
		{
			name: 'Hubert Farm',
			county: 'Hancock',
			coords: wp(33.26606220510343, -82.97665337841785),
			about:
				"Zach Hubert owns a 165-acre farm, which he operates with his wife, Camilla Hillman. Together, they have 12 children who contribute to the farm's success.",
			history:
				'Zach initially rented his first farm with his brother, Moses, before eventually purchasing a 165-acre farm alongside his brothers. Born into slavery, Zach learned to read and write, a skill that would help him build a prosperous future.',
			present:
				"The Hubert children have become vital figures in their community, serving as school principals, college presidents, and founders of community advancement organizations. The farm remains a testament to their family's resilience and contributions.",
			founded: 'N/A',
			acres: 165,
			image: 'images/farms/hubert.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2002, November). The Zach Hubert Farm: A Centennial Family Farm (Vol. III, No. 3). Jeanne Cyriaque, Editor.'
		},
		{
			name: 'Charleston-Allen Farm',
			county: 'Morgan',
			coords: wp(33.55373745990568, -83.43136121356132),
			about:
				'The Charleston-Allen farm received the Georgia Centennial Family Farm Award at the Georgia National Fair in October 2010.',
			history:
				'The farm’s history dates back to 1890 when Anna Charleston, who was born enslaved just 30 years earlier, was deeded nearly 300 acres of land for her oldest daughter, Alameda. Alameda’s father was one of the wealthiest white landowners in Morgan County.',
			present:
				'Currently, the farm spans 200 acres, with 100 acres dedicated to pine and natural hardwood tree production. The remaining land is used for cotton, corn, cattle, and pigs. The farm is bisected by Interstate 20 and remains linked to the Garfield Hall farm through Odessa Allen Hall, the granddaughter of Anna Charleston.',
			founded: '1890',
			acres: 200,
			image: 'images/farms/charleston.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2011, April). Connecting To Land And Community: Two African American Centennial Family Farms (Vol. XI, No. 1). Jeanne Cyriaque, African American Programs Coordinator.'
		},
		{
			name: 'Garfield Hall Farm',
			county: 'Bulloch',
			coords: wp(32.44372966318054, -81.78066506446379),
			about:
				'The Garfield Hall farm was honored with the Georgia Centennial Family Farm Award at the Georgia National Fair in October 2010.',
			history:
				'The farm originally produced cotton, corn, tobacco, and peanuts while raising livestock such as chickens, turkeys, hogs, and rabbits.',
			present:
				'Today, the Hall Farm dedicates 40 acres to tree production and retains a total of 115 acres. Many Hall family members have served as teachers and students at the Willow Hill Elementary School in Portal, Georgia, and the farm remains closely tied to this historic institution.',
			founded: 'N/A',
			acres: 115,
			image: 'images/farms/hall.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2011, April). Connecting To Land And Community: Two African American Centennial Family Farms (Vol. XI, No. 1). Jeanne Cyriaque, African American Programs Coordinator.'
		},
		{
			name: 'Gilliard Farm',
			county: 'Glynn',
			coords: wp(31.16376054153567, -81.62389074555118),
			image: 'images/farms/gilliard.png',
			about:
				'In October 2012, the Gilliard Farm became one of the first organic farms to receive a Centennial Family Farm Award.',
			history:
				'The Gilliard Farm was founded in 1874 by Jupiter Gilliard, who was born enslaved. Located in Brookman, a rural community in Glynn County, Georgia, the farm has 15 acres dedicated to agriculture, producing vegetables, fruits, chickens, and hogs.',
			present:
				'Althea and Matthew Raiford, sixth-generation descendants of the Gilliard family, currently maintain 50 acres of the original 457-acre purchase. Althea, a disabled African American Navy veteran, became the first recipient of a California grant for aspiring farmers. The farm operates a Community Supported Agriculture (CSA) program, offering fresh produce and recipes. The Raifords are also active members of the Southeastern African American Farmers Organic Network, supporting small organic farmers.',
			founded: '1874',
			acres: 50,
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2012, December). Growing food from Reconstruction to organics: Gilliard Farms, an African American Centennial Family Farm (Vol. XI, No. 1). Jeanne Cyriaque, African American Programs Coordinator.'
		},
		{
			name: 'Williams',
			county: 'Thomas',
			coords: wp(30.91402190426384, -83.85165546143168),
			about:
				'The Williams Farm was established in 1883 when Charles Cockrell purchased 245 acres and began growing cotton, corn, sugarcane, and potatoes.',
			history:
				'Though Charles Cockrell passed away nine years after founding the farm, his descendants carried on his legacy. Kenneth Williams, one of his descendants, expanded the farm, which now bears a name that combines his and his wife Octavia’s names.',
			present:
				'Today, Bryan Williams, Kenneth’s great-grandson, manages the farm. It remains a significant producer of cotton, peanuts, corn, and soybeans. The farm received the Centennial Family Farm Award in 2013 and continues to uphold family traditions, serving as a gathering place for Kenneth’s many descendants.',
			founded: '1883',
			acres: 245,
			image: 'images/farms/williams.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2013, December). Farming in Southwest Georgia Since 1883: The Kentavia Williams Farm (Vol. XI, No. 4). Jeanne Cyriaque, African American Programs Coordinator.'
		},
		{
			name: 'Cooper',
			county: 'Burke',
			coords: wp(32.975456566583894, -81.75939468755824),
			about:
				'The Cooper Family Farm, located near Sardis in Burke County, received the Centennial Family Farm Award in 2014, making it the first in Burke County and the 12th African American Centennial Farm in Georgia.',
			history:
				'Frank Cooper Sr. acquired 73 acres in 1885 while also serving as the minister at First McCanaan Baptist Church, a historic African American institution.',
			present:
				'Over the past century, the farm expanded to 299 acres, with 169 acres still dedicated to crops. As one of Georgia’s largest African American Centennial Farms, the Cooper Family Farm continues to preserve African American landownership and agricultural heritage.',
			founded: '1885',
			acres: 295.7,
			image: 'images/farms/cooper.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2014, December). Cooper Family Farm: A Centennial Family Farm (Vol. XII, No. 2). Jeanne Cyriaque, African American Programs Coordinator.'
		},
		{
			name: 'Stephens',
			county: 'Dougherty',
			coords: wp(31.51458649234062, -84.00240727678029),
			about:
				'The Stephens Family Farm, located in Albany, Dougherty County, has played an essential role in Albany’s history and African American heritage.',
			history:
				'The farm traces its origins to 1865 when Titus Stephens acquired the original 100 acres. He is believed to have obtained the land from his former enslaver, setting a foundation for his family’s future. Cotton has been the farm’s primary crop for generations.',
			present:
				'Now owned by the fifth generation of the Stephens family, the farm remains a symbol of African American perseverance. Despite facing Reconstruction, Jim Crow laws, economic struggles, and the Civil Rights Movement, the Stephens family has maintained ownership. The farm was recognized by the Albany Herald in 2013 and received the Centennial Farm Award in 2014.',
			founded: '1865',
			acres: 100,
			image: 'images/farms/stephens.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2014, December). Stephen Family Farm: A Centennial Family Farm (Vol. XII, No. 2). Jeanne Cyriaque, African American Programs Coordinator.'
		},
		{
			name: 'Thompson',
			county: 'Augusta-Richmond',
			coords: wp(33.47800464351064, -82.06263032165144),
			about:
				'Thompson Farm has been operating for over 100 years in downtown Augusta. Established in the 1870s, the farm has grown to more than 2,000 acres, producing soybeans, corn, wheat, oats, rye, and millet.',
			history:
				'The farm began as a dray line business, a mule-powered transportation company. JoAnn Thompson purchased the first 660 acres in 1918, making history as an African American businesswoman acquiring prime land before women could vote. Charles and Harold Thompson continue their grandparents’ vision, transforming the business from a transporter of commodities to a producer. The farm supplies corn to Amy Brawler Farms and Columbia Farms for chicken feed in Richmond County.',
			present:
				'Thompson Farm is the first and only farm in Augusta to receive the Georgia Centennial Farm Family Award. It is recognized by the Augusta-Richmond County Extension Service as the largest commercial commodity farm in the city and county. The Thompson family continues to uphold a legacy of hard work, perseverance, and agricultural excellence.',
			founded: '1870s',
			acres: 2000,
			image: 'images/farms/thompson.png',
			source:
				'WJBF. (2020, February 6). Thompson Farms: Faith, family and farming. https://www.wjbf.com/csra-news/thompson-farms-faith-family-and-farming/'
		},
		{
			name: 'Toomer',
			county: 'Houston',
			coords: wp(32.4540073170362, -83.72931569977366),
			about:
				'The Dave Toomer Farm has been family-owned for over 100 years. Established in 1900 by Dave Toomer, an African American farmer, the estate originally spanned 200 acres in Houston County.',
			history:
				'Early Ownership: Dave Toomer and his wife Dollie fully paid for the land by 1911. In 1930, their son Lewis Toomer inherited the farm and raised eight children while continuing agricultural operations. Fred Toomer, one of Lewis’s children, devoted his life to farming, working until the age of 84. After Fred Toomer’s passing in 1993, his son Kahlid Toomer inherited the farm. Kahlid was one of 12 children raised on the farm, all of whom completed high school, with four attending college.',
			present:
				'The farm, once known for rice production, now spans 75 acres, cultivating fruits, vegetables, soybeans, and peanuts. Historic grapevines, fig, pecan, and walnut trees, some over 100 years old, continue to bear fruit, preserving the farm’s agricultural legacy.',
			founded: '1900',
			acres: 75,
			image: 'images/farms/toomer.png',
			source:
				'Georgia Department of Natural Resources, Historic Preservation Division. (2002, March). Centennial Farm Awards Program Honors African American Farm (Vol. II, No. 2). Jeanne Cyriaque, African American Programs Coordinator'
		}
	];

	const fake_600_coords: Point[] = (
		JSON.parse(
			'[{"x":0.07749646341905764,"y":0.06375752818825377},{"x":0.09374659944140157,"y":0.06762162080572369},{"x":0.23187275563132503,"y":0.09563629228238064},{"x":0.18196162356269724,"y":0.06858764396009116},{"x":0.20633682759621313,"y":0.09853436174548309},{"x":0.25044433965686097,"y":0.06568957449698873},{"x":0.21794406761217308,"y":0.08597606073870583},{"x":0.30848053973666073,"y":0.07534980604066353},{"x":0.28410533570314483,"y":0.0975683385911156},{"x":0.3421415357829446,"y":0.0724517365775611},{"x":0.3316950197685807,"y":0.09950038489985057},{"x":0.4083028038739164,"y":0.07824787550376598},{"x":0.39089194384997644,"y":0.1081945932891579},{"x":0.45705321194094817,"y":0.08114594496686843},{"x":0.512767964017556,"y":0.06762162080572369},{"x":0.501160724001596,"y":0.08790810704744079},{"x":0.4616961079473321,"y":0.1081945932891579},{"x":0.3932133918531684,"y":0.11109266275226035},{"x":0.30151619572708477,"y":0.10433050067168798},{"x":0.23767637563930502,"y":0.11785482483283272},{"x":0.17035438354673726,"y":0.09853436174548309},{"x":0.12392542348289745,"y":0.12558301006777256},{"x":0.09374659944140157,"y":0.12268494060467011},{"x":0.07865718742065363,"y":0.14297142684638722},{"x":0.0856215314302296,"y":0.1555297278531645},{"x":0.1366933875004534,"y":0.128481079530875},{"x":0.19937248358663717,"y":0.1236509637590376},{"x":0.20633682759621313,"y":0.12075289429593515},{"x":0.1645507635387573,"y":0.1584277973162669},{"x":0.22955130762813306,"y":0.11978687114156768},{"x":0.2133011716057891,"y":0.14683551946385714},{"x":0.21794406761217308,"y":0.11785482483283272},{"x":0.17615800355471725,"y":0.10433050067168798},{"x":0.251605063658457,"y":0.14876756577259212},{"x":0.3038376437302768,"y":0.11495675536973027},{"x":0.2678551996808009,"y":0.154563704698797},{"x":0.3293735717653887,"y":0.11882084798720019},{"x":0.32473067575900466,"y":0.14586949630948967},{"x":0.3676774638180565,"y":0.1236509637590376},{"x":0.39089194384997644,"y":0.10433050067168798},{"x":0.35142732779571256,"y":0.154563704698797},{"x":0.30848053973666073,"y":0.10722857013479042},{"x":0.36535601581486454,"y":0.11978687114156768},{"x":0.38044542783561247,"y":0.15746177416189944},{"x":0.4233922158946643,"y":0.13717528792018233},{"x":0.42919583590264426,"y":0.10433050067168798},{"x":0.487232035982444,"y":0.1333111953027124},{"x":0.45589248793935216,"y":0.14586949630948967},{"x":0.5348217200478799,"y":0.14200540369201975},{"x":0.5383038920526678,"y":0.16712200570557426},{"x":0.5661612680909718,"y":0.17774826040361655},{"x":0.502321448003192,"y":0.20479690872590603},{"x":0.4442852479233922,"y":0.15263165839006204},{"x":0.36303456781167254,"y":0.19030656141039382},{"x":0.3467844317893286,"y":0.21542316342394832},{"x":0.3827668758388045,"y":0.1642239362424718},{"x":0.23883709964090102,"y":0.18257837617545394},{"x":0.22258696361855707,"y":0.15359768154442952},{"x":0.1912474155754652,"y":0.20383088557153856},{"x":0.15178279952120136,"y":0.1912725845647613},{"x":0.12624687148608946,"y":0.14683551946385714},{"x":0.0891037034350176,"y":0.16518995939683928},{"x":0.0856215314302296,"y":0.20189883926280358},{"x":0.17267583154992927,"y":0.16905405201430923},{"x":0.10071094345097754,"y":0.23377760335693046},{"x":0.09722877144618956,"y":0.22315134865888817},{"x":0.15990786753237332,"y":0.1999667929540686},{"x":0.0891037034350176,"y":0.2675884137597923},{"x":0.15990786753237332,"y":0.24053976543750283},{"x":0.14597917951322137,"y":0.25406408959864757},{"x":0.23651565163770902,"y":0.21928725604141824},{"x":0.17499727955312125,"y":0.24440385805497275},{"x":0.2899089557111248,"y":0.2270154412763581},{"x":0.32821284776379267,"y":0.1854764456385564},{"x":0.29919474772389276,"y":0.24440385805497275},{"x":0.3061590917334687,"y":0.18354439932982144},{"x":0.3978562878595524,"y":0.21928725604141824},{"x":0.36187384381007653,"y":0.1854764456385564},{"x":0.3467844317893286,"y":0.2501999969811776},{"x":0.4605353839457362,"y":0.19900076979970113},{"x":0.4524103159345642,"y":0.24053976543750283},{"x":0.515089412020748,"y":0.20672895503464098},{"x":0.501160724001596,"y":0.24536988120934022},{"x":0.5116072400159599,"y":0.19030656141039382},{"x":0.5557147520766078,"y":0.23957374228313535},{"x":0.5673219920925677,"y":0.180646329866719},{"x":0.5928579201276797,"y":0.2115590708064784},{"x":0.5406253400558598,"y":0.16325791308810433},{"x":0.5905364721244877,"y":0.17581621409488157},{"x":0.5847328521165077,"y":0.2820787610753045},{"x":0.6102687801516196,"y":0.22798146443072556},{"x":0.5684827160941638,"y":0.2270154412763581},{"x":0.6358047081867315,"y":0.26372432114232236},{"x":0.6218760201675795,"y":0.29173899261897934},{"x":0.6195545721643876,"y":0.2501999969811776},{"x":0.5661612680909718,"y":0.28497683053840694},{"x":0.4849105879792521,"y":0.2733845526859972},{"x":0.52321448003192,"y":0.3129915020150639},{"x":0.46401755595052413,"y":0.29077296946461184},{"x":0.48026769197286806,"y":0.31395752516943143},{"x":0.41178497587870433,"y":0.2782146684578346},{"x":0.42803511190104826,"y":0.3178216177869013},{"x":0.3699989118212485,"y":0.2888409231558769},{"x":0.3409808117813486,"y":0.3149235483237989},{"x":0.35026660379411656,"y":0.27435057584036465},{"x":0.3073198157350647,"y":0.30139922416265413},{"x":0.3177663317494287,"y":0.2666223906054248},{"x":0.2690159236823969,"y":0.30622933993449153},{"x":0.2783017156951649,"y":0.2608262516792199},{"x":0.23883709964090102,"y":0.30333127047138914},{"x":0.23419420363451704,"y":0.2714525063772622},{"x":0.20981899960100112,"y":0.3100934325519615},{"x":0.19705103558344517,"y":0.2608262516792199},{"x":0.18312234756429321,"y":0.3052633167801241},{"x":0.15758641952918132,"y":0.25696215906175},{"x":0.1378541115020494,"y":0.2994671778539192},{"x":0.12856831948928144,"y":0.24343783490060528},{"x":0.10883601146214951,"y":0.2994671778539192},{"x":0.1378541115020494,"y":0.333277988256781},{"x":0.1912474155754652,"y":0.30333127047138914},{"x":0.14597917951322137,"y":0.3236177567131062},{"x":0.15178279952120136,"y":0.2898069463102444},{"x":0.22374768762015307,"y":0.33231196510241356},{"x":0.15874714353077732,"y":0.28014671476656955},{"x":0.21910479161376908,"y":0.3187876409412688},{"x":0.23651565163770902,"y":0.2898069463102444},{"x":0.25973013166962894,"y":0.3187876409412688},{"x":0.2643730276760129,"y":0.2733845526859972},{"x":0.28294461170154883,"y":0.31395752516943143},{"x":0.2933911277159128,"y":0.25696215906175},{"x":0.30964126373825673,"y":0.30139922416265413},{"x":0.30964126373825673,"y":0.2627582979879549},{"x":0.3456237077877326,"y":0.30719536308885903},{"x":0.36535601581486454,"y":0.2685544369141598},{"x":0.35955239580688453,"y":0.309127409397594},{"x":0.4349994559106243,"y":0.29077296946461184},{"x":0.41294569988030033,"y":0.3149235483237989},{"x":0.487232035982444,"y":0.27435057584036465},{"x":0.49303565599042404,"y":0.30236524731702163},{"x":0.5371431680510719,"y":0.27435057584036465},{"x":0.5824114041133157,"y":0.3052633167801241},{"x":0.5243752040335159,"y":0.30236524731702163},{"x":0.5998222641372556,"y":0.2685544369141598},{"x":0.5905364721244877,"y":0.3100934325519615},{"x":0.6508941202074794,"y":0.30236524731702163},{"x":0.6218760201675795,"y":0.24826795067244267},{"x":0.6810729442489754,"y":0.29656910839081674},{"x":0.6416083281947115,"y":0.3294138956393111},{"x":0.7019659762777032,"y":0.3245837798674737},{"x":0.6172331241611956,"y":0.34100617349172085},{"x":0.7089303202872792,"y":0.34197219664608836},{"x":0.6207152961659835,"y":0.34680231241792575},{"x":0.5313395480430919,"y":0.33617605771988346},{"x":0.5406253400558598,"y":0.30719536308885903},{"x":0.49303565599042404,"y":0.3448702661091908},{"x":0.40598135587072437,"y":0.33617605771988346},{"x":0.5406253400558598,"y":0.34680231241792575},{"x":0.36535601581486454,"y":0.35936061342470305},{"x":0.47910696797127206,"y":0.3728849375858478},{"x":0.35374877579890457,"y":0.36419072919654044},{"x":0.3665167398164605,"y":0.3767490302033177},{"x":0.3038376437302768,"y":0.33617605771988346},{"x":0.28178388769995283,"y":0.3719189144314803},{"x":0.28758750770793284,"y":0.34197219664608836},{"x":0.17847945155790926,"y":0.3757830070489502},{"x":0.20169393158982915,"y":0.33714208087425096},{"x":0.18080089956110124,"y":0.3912393775188299},{"x":0.09490732344299757,"y":0.36805482181401034},{"x":0.1633900395371613,"y":0.41345791006928195},{"x":0.11580035547172549,"y":0.3912393775188299},{"x":0.1657114875403533,"y":0.42601621107605925},{"x":0.17035438354673726,"y":0.3757830070489502},{"x":0.19356886357865719,"y":0.411525863760547},{"x":0.251605063658457,"y":0.3777150533576852},{"x":0.26205157967282094,"y":0.4211860953042218},{"x":0.2945518517175088,"y":0.38061312282078763},{"x":0.3177663317494287,"y":0.3448702661091908},{"x":0.3200877797526207,"y":0.38737528490136},{"x":0.3409808117813486,"y":0.35163242818976315},{"x":0.35142732779571256,"y":0.39413744698193237},{"x":0.40365990786753236,"y":0.36998686812274534},{"x":0.43964235191700823,"y":0.3564625439616006},{"x":0.42803511190104826,"y":0.3912393775188299},{"x":0.4454459719249882,"y":0.34873435872666075},{"x":0.37696325583082446,"y":0.38737528490136},{"x":0.49419637999202004,"y":0.33810810402861846},{"x":0.46401755595052413,"y":0.3844772154382575},{"x":0.4860713119808481,"y":0.35742856711596804},{"x":0.4779462439696761,"y":0.3835111922838901},{"x":0.45589248793935216,"y":0.3497003818810282},{"x":0.5359824440494758,"y":0.3883413080557275},{"x":0.5464289600638398,"y":0.34293821980045586},{"x":0.52321448003192,"y":0.37868107651205263},{"x":0.48839275998404,"y":0.40283165537123966},{"x":0.5650005440893757,"y":0.3951034701362998},{"x":0.5882150241212957,"y":0.35742856711596804},{"x":0.5533933040734158,"y":0.3757830070489502},{"x":0.6102687801516196,"y":0.40283165537123966},{"x":0.6172331241611956,"y":0.4008996090625047},{"x":0.6532155682106714,"y":0.3603266365790705},{"x":0.6125902281548116,"y":0.33037991879367856},{"x":0.6775907722441873,"y":0.3400401503373534},{"x":0.6555370162138634,"y":0.3825451691295226},{"x":0.7008052522761072,"y":0.3844772154382575},{"x":0.6485726722042874,"y":0.4018656322168722},{"x":0.6926801842649353,"y":0.3951034701362998},{"x":0.7124124922920672,"y":0.3767490302033177},{"x":0.7100910442888752,"y":0.35936061342470305},{"x":0.7054481482824912,"y":0.3554965208072331},{"x":0.6775907722441873,"y":0.34100617349172085},{"x":0.6845551162537633,"y":0.32071968725000377},{"x":0.6891980122601473,"y":0.32071968725000377},{"x":0.7158946642968551,"y":0.335210034565516},{"x":0.7437520403351591,"y":0.34873435872666075},{"x":0.764645072363887,"y":0.37385096074021523},{"x":0.7379484203271791,"y":0.411525863760547},{"x":0.6880372882585513,"y":0.4018656322168722},{"x":0.7460734883383511,"y":0.4018656322168722},{"x":0.7716094163734629,"y":0.4008996090625047},{"x":0.7100910442888752,"y":0.41925404899548685},{"x":0.768127244368675,"y":0.41538995637801696},{"x":0.8133954804309188,"y":0.42505018792169175},{"x":0.8099133084261309,"y":0.44147258154593894},{"x":0.761162900359099,"y":0.44630269731777633},{"x":0.7193768363016432,"y":0.4385745120828365},{"x":0.6671442562298233,"y":0.4337443963109991},{"x":0.6532155682106714,"y":0.4607930446332886},{"x":0.6485726722042874,"y":0.46465713725075847},{"x":0.7100910442888752,"y":0.4578949751701861},{"x":0.7170553882984512,"y":0.49073976241868045},{"x":0.760002176357503,"y":0.461759067787656},{"x":0.764645072363887,"y":0.48107953087500566},{"x":0.7948238964053829,"y":0.46948725302259586},{"x":0.8122347564293227,"y":0.48590964664684305},{"x":0.8331277884580507,"y":0.4665891835594934},{"x":0.8470564764772026,"y":0.46465713725075847},{"x":0.8563422684899706,"y":0.4849436234924756},{"x":0.8540208204867786,"y":0.5003999939623552},{"x":0.7901810003989989,"y":0.49170578557304795},{"x":0.7843773803910189,"y":0.5071621560429277},{"x":0.755359280351119,"y":0.48687566980121055},{"x":0.7182161123000471,"y":0.5032980634254577},{"x":0.7588414523559069,"y":0.4723853224856983},{"x":0.7008052522761072,"y":0.5023320402710902},{"x":0.6915194602633392,"y":0.4675552067138609},{"x":0.6404476041931154,"y":0.5003999939623552},{"x":0.6416083281947115,"y":0.47141929933133087},{"x":0.6300010881787514,"y":0.5061961328885601},{"x":0.6288403641771555,"y":0.47141929933133087},{"x":0.5731256121005477,"y":0.49653590134488534},{"x":0.5719648880989517,"y":0.4511328130896137},{"x":0.5557147520766078,"y":0.4733513456400658},{"x":0.5545540280750118,"y":0.48204555402937316},{"x":0.6033044361420437,"y":0.4569289520158186},{"x":0.5417860640574559,"y":0.48784169295557805},{"x":0.5452682360622438,"y":0.4385745120828365},{"x":0.5208930320287279,"y":0.47528339194880076},{"x":0.515089412020748,"y":0.42505018792169175},{"x":0.4744640719648881,"y":0.4675552067138609},{"x":0.46401755595052413,"y":0.42601621107605925},{"x":0.4628568319489281,"y":0.4559629288614511},{"x":0.43964235191700823,"y":0.4182880258411194},{"x":0.4199100438898763,"y":0.4492007667808788},{"x":0.4083028038739164,"y":0.41925404899548685},{"x":0.35258805179730857,"y":0.4665891835594934},{"x":0.4222314918930683,"y":0.4511328130896137},{"x":0.4071420798723204,"y":0.4839776003381081},{"x":0.3850883238419964,"y":0.45209883624398123},{"x":0.3433022597845406,"y":0.463691114096391},{"x":0.30151619572708477,"y":0.48784169295557805},{"x":0.35490949980050057,"y":0.5003999939623552},{"x":0.3839275998404004,"y":0.4839776003381081},{"x":0.33633791577496464,"y":0.46562316040512597},{"x":0.29919474772389276,"y":0.4685212298682284},{"x":0.31196271174144874,"y":0.43567644261973404},{"x":0.2690159236823969,"y":0.4559629288614511},{"x":0.24464071964888098,"y":0.48784169295557805},{"x":0.3073198157350647,"y":0.5032980634254577},{"x":0.26205157967282094,"y":0.49846794765362035},{"x":0.3572309478036926,"y":0.49170578557304795},{"x":0.2643730276760129,"y":0.5168223875866025},{"x":0.23187275563132503,"y":0.4685212298682284},{"x":0.20865827559940514,"y":0.489773739264313},{"x":0.23303347963292104,"y":0.5139243181235},{"x":0.17731872755631325,"y":0.5100602255060301},{"x":0.20401537959302116,"y":0.45982702147892107},{"x":0.1645507635387573,"y":0.48204555402937316},{"x":0.15874714353077732,"y":0.42988030369352914},{"x":0.19937248358663717,"y":0.5013660171167228},{"x":0.19821175958504117,"y":0.5448370590632594},{"x":0.2701766476839929,"y":0.5284146654390123},{"x":0.1947295875802532,"y":0.5670555916137114},{"x":0.2144618956073851,"y":0.5989343557078384},{"x":0.2701766476839929,"y":0.566089568459344},{"x":0.1889259675722732,"y":0.6008664020165734},{"x":0.2690159236823969,"y":0.6153567493320855},{"x":0.23535492763611301,"y":0.5757498000030188},{"x":0.3456237077877326,"y":0.5999003788622058},{"x":0.3572309478036926,"y":0.5631914989962415},{"x":0.33865936377815664,"y":0.6066625409427783},{"x":0.41410642388189634,"y":0.564157522150609},{"x":0.35955239580688453,"y":0.5284146654390123},{"x":0.3816061518372085,"y":0.5728517305399163},{"x":0.4257136638978563,"y":0.5487011516807293},{"x":0.3966955638579564,"y":0.5216525033584398},{"x":0.4245529398962603,"y":0.5718857073855489},{"x":0.41642787188508834,"y":0.5245505728215423},{"x":0.4338387319090283,"y":0.5515992211438318},{"x":0.4698211759585041,"y":0.5081281791972951},{"x":0.487232035982444,"y":0.5342108043652172},{"x":0.45357103993616016,"y":0.564157522150609},{"x":0.5278573760383038,"y":0.5303467117477472},{"x":0.515089412020748,"y":0.5602934295331391},{"x":0.5371431680510719,"y":0.5544972906069342},{"x":0.5673219920925677,"y":0.5245505728215423},{"x":0.5348217200478799,"y":0.5631914989962415},{"x":0.5975008161340636,"y":0.5583613832244042},{"x":0.6218760201675795,"y":0.5284146654390123},{"x":0.5951793681308716,"y":0.5535312674525668},{"x":0.5638398200877798,"y":0.5216525033584398},{"x":0.6659835322282274,"y":0.5699536610768139},{"x":0.6625013602234394,"y":0.5216525033584398},{"x":0.6845551162537633,"y":0.5622254758418741},{"x":0.7205375603032391,"y":0.5313127349021147},{"x":0.7460734883383511,"y":0.5264826191302773},{"x":0.7391091443287751,"y":0.5322787580564822},{"x":0.6857158402553593,"y":0.5718857073855489},{"x":0.742591316333563,"y":0.5602934295331391},{"x":0.7402698683303711,"y":0.5245505728215423},{"x":0.753037832347927,"y":0.566089568459344},{"x":0.7716094163734629,"y":0.5361428506739521},{"x":0.766966520367079,"y":0.5747837768486513},{"x":0.775091588378251,"y":0.5342108043652172},{"x":0.7878595523958069,"y":0.5651235453049765},{"x":0.7333055243207951,"y":0.5815459389292237},{"x":0.7240197323080272,"y":0.5371088738283196},{"x":0.8064311364213428,"y":0.5776818463117538},{"x":0.8203598244404947,"y":0.5863760547010611},{"x":0.8366099604628386,"y":0.5380748969826871},{"x":0.8342885124596467,"y":0.5786478694661212},{"x":0.8807174725234865,"y":0.5622254758418741},{"x":0.8737531285139105,"y":0.5197204570497049},{"x":0.8807174725234865,"y":0.564157522150609},{"x":0.9039319525554065,"y":0.5796138926204888},{"x":0.9201820885777504,"y":0.5612594526875067},{"x":0.8888425405346585,"y":0.6172887956408205},{"x":0.9155391925713664,"y":0.620186865103923},{"x":0.8714316805107185,"y":0.6356432355738026},{"x":0.9271464325873263,"y":0.6453034671174774},{"x":0.9178606405745584,"y":0.620186865103923},{"x":0.9410751206064782,"y":0.6433714208087425},{"x":0.9677717726431861,"y":0.6404733513456401},{"x":0.9306286045921144,"y":0.6607598375873571},{"x":0.9480394646160543,"y":0.6626918838960921},{"x":0.8783960245202945,"y":0.670420069131032},{"x":0.8656280605027385,"y":0.6279150503388629},{"x":0.8366099604628386,"y":0.6472355134262124},{"x":0.8308063404548587,"y":0.5979683325534709},{"x":0.8006275164133628,"y":0.6337111892650676},{"x":0.760002176357503,"y":0.5863760547010611},{"x":0.751877108346331,"y":0.622118911412658},{"x":0.7019659762777032,"y":0.5757498000030188},{"x":0.6938409082665312,"y":0.6008664020165734},{"x":0.6671442562298233,"y":0.5506331979894643},{"x":0.6311618121803475,"y":0.5892741241641636},{"x":0.5847328521165077,"y":0.5593274063787717},{"x":0.6381261561899234,"y":0.6047304946340433},{"x":0.5220537560303239,"y":0.5796138926204888},{"x":0.5278573760383038,"y":0.5409729664457895},{"x":0.5208930320287279,"y":0.590240147318531},{"x":0.48258913997606007,"y":0.5844440083923261},{"x":0.42919583590264426,"y":0.5380748969826871},{"x":0.41178497587870433,"y":0.5718857073855489},{"x":0.3966955638579564,"y":0.5941042399360009},{"x":0.35374877579890457,"y":0.5583613832244042},{"x":0.3444629837861366,"y":0.5776818463117538},{"x":0.3409808117813486,"y":0.564157522150609},{"x":0.3699989118212485,"y":0.5854100315466937},{"x":0.39089194384997644,"y":0.5979683325534709},{"x":0.4338387319090283,"y":0.6211528882582904},{"x":0.3932133918531684,"y":0.6675219996679296},{"x":0.3827668758388045,"y":0.6298470966475977},{"x":0.3154448837462367,"y":0.6607598375873571},{"x":0.31428415974464075,"y":0.6240509577213929},{"x":0.3061590917334687,"y":0.5815459389292237},{"x":0.25624795966484093,"y":0.6211528882582904},{"x":0.28526605970474084,"y":0.5738177536942839},{"x":0.25856940766803294,"y":0.5535312674525668},{"x":0.241158547644093,"y":0.5602934295331391},{"x":0.19821175958504117,"y":0.5728517305399163},{"x":0.20169393158982915,"y":0.592172193627266},{"x":0.252765787660053,"y":0.6027984483253083},{"x":0.18080089956110124,"y":0.6346772124194352},{"x":0.23419420363451704,"y":0.6385413050369051},{"x":0.246962167652073,"y":0.6385413050369051},{"x":0.1877652435706772,"y":0.6501335828893149},{"x":0.22490841162174907,"y":0.6530316523524173},{"x":0.28758750770793284,"y":0.6510996060436823},{"x":0.18428307156588922,"y":0.6646239302048271},{"x":0.30035547172548877,"y":0.6762162080572369},{"x":0.2202655156153651,"y":0.6839443932921767},{"x":0.2806231636983568,"y":0.690706555372749},{"x":0.1622293155355653,"y":0.7051969026882613},{"x":0.22606913562334507,"y":0.7109930416144662},{"x":0.17267583154992927,"y":0.7148571342319361},{"x":0.30151619572708477,"y":0.7138911110775686},{"x":0.31080198773985274,"y":0.6559297218155198},{"x":0.27481954369037687,"y":0.6897405322183816},{"x":0.2655337516776089,"y":0.7264494120843459},{"x":0.3154448837462367,"y":0.7322455510105508},{"x":0.3177663317494287,"y":0.7129250879232012},{"x":0.3328557437701766,"y":0.6800803006747068},{"x":0.3560702238020966,"y":0.6520656291980499},{"x":0.3676774638180565,"y":0.6337111892650676},{"x":0.4001777358627444,"y":0.6327451661107002},{"x":0.39089194384997644,"y":0.6588277912786222},{"x":0.39205266785157245,"y":0.7390077130911231},{"x":0.40133845986434036,"y":0.7351436204736532},{"x":0.4210707678914723,"y":0.696502694298954},{"x":0.4245529398962603,"y":0.6858764396009117},{"x":0.4698211759585041,"y":0.6549636986611523},{"x":0.5,"y":0.6472355134262124},{"x":0.4605353839457362,"y":0.6829783701378093},{"x":0.4628568319489281,"y":0.6849104164465442},{"x":0.49071420798723203,"y":0.7119590647688336},{"x":0.43848162791541223,"y":0.7264494120843459},{"x":0.5824114041133157,"y":0.7129250879232012},{"x":0.5208930320287279,"y":0.6675219996679296},{"x":0.4466066959265842,"y":0.6453034671174774},{"x":0.5499111320686279,"y":0.6829783701378093},{"x":0.5336609960462839,"y":0.7167891805406711},{"x":0.6276796401755596,"y":0.7216192963125085},{"x":0.5940186441292756,"y":0.700366786916424},{"x":0.5754470601037397,"y":0.6800803006747068},{"x":0.5858935761181037,"y":0.6539976755067848},{"x":0.6392868801915195,"y":0.6375752818825376},{"x":0.6868765642569553,"y":0.6375752818825376},{"x":0.6346439841851355,"y":0.6607598375873571},{"x":0.6381261561899234,"y":0.6849104164465442},{"x":0.6857158402553593,"y":0.7119590647688336},{"x":0.6613406362218434,"y":0.7361096436280207},{"x":0.6891980122601473,"y":0.7370756667823882},{"x":0.7066088722840872,"y":0.7177552036950385},{"x":0.7019659762777032,"y":0.6878084859096466},{"x":0.7901810003989989,"y":0.668488022822297},{"x":0.757680728354311,"y":0.698434740607689},{"x":0.757680728354311,"y":0.7090609953057312},{"x":0.7994667924117669,"y":0.6713860922853995},{"x":0.762323624360695,"y":0.7100270184600987},{"x":0.734466248322391,"y":0.7496339677891655},{"x":0.6984838042729152,"y":0.700366786916424},{"x":0.7356269723239871,"y":0.6762162080572369},{"x":0.8041096884181508,"y":0.7129250879232012},{"x":0.8528600964851827,"y":0.7293474815474483},{"x":0.8180383764373028,"y":0.7602602224872077},{"x":0.8598244404947586,"y":0.7312795278561833},{"x":0.9085748485617904,"y":0.7380416899367557},{"x":0.8923247125394465,"y":0.7699204540308826},{"x":0.8992890565490225,"y":0.7438378288629606},{"x":0.8389314084660306,"y":0.7303135047018158},{"x":0.8981283325474264,"y":0.7071289489969963},{"x":0.9120570205665784,"y":0.6994007637620564},{"x":0.8505386484819907,"y":0.6955366711445865},{"x":0.9329500525953063,"y":0.6617258607417247},{"x":0.8540208204867786,"y":0.6752501849028694},{"x":0.8691102325075266,"y":0.7100270184600987},{"x":0.8424135804708187,"y":0.7902069402725996},{"x":0.8749138525155066,"y":0.7805467087289248},{"x":0.8923247125394465,"y":0.7679884077221476},{"x":0.8598244404947586,"y":0.8124254728230518},{"x":0.8493779244803946,"y":0.8288478664472989},{"x":0.8540208204867786,"y":0.8520324221521185},{"x":0.8795567485218905,"y":0.8684548157763656},{"x":0.8725924045123146,"y":0.8800470936287754},{"x":0.8284848924516667,"y":0.8800470936287754},{"x":0.7959846204069788,"y":0.8597606073870583},{"x":0.8331277884580507,"y":0.8433382137628112},{"x":0.8412528564692227,"y":0.7921389865813346},{"x":0.8319670644564547,"y":0.7853768245007623},{"x":0.766966520367079,"y":0.7979351255075395},{"x":0.7797344843846349,"y":0.8414061674540761},{"x":0.7762523123798469,"y":0.8433382137628112},{"x":0.764645072363887,"y":0.8607266305414258},{"x":0.7414305923319671,"y":0.8810131167831429},{"x":0.7437520403351591,"y":0.923518135575312},{"x":0.751877108346331,"y":0.9467026912801316},{"x":0.751877108346331,"y":0.9486347375888666},{"x":0.7089303202872792,"y":0.8819791399375104},{"x":0.749555660343139,"y":0.8626586768501607},{"x":0.7588414523559069,"y":0.8307799127560339},{"x":0.6950016322681273,"y":0.8414061674540761},{"x":0.7483949363415431,"y":0.796969102353172},{"x":0.7170553882984512,"y":0.7863428476551297},{"x":0.7483949363415431,"y":0.7486679446347979},{"x":0.7042874242808952,"y":0.7428718057085931},{"x":0.6300010881787514,"y":0.7563961298697378},{"x":0.6578584642170554,"y":0.796969102353172},{"x":0.6578584642170554,"y":0.796969102353172},{"x":0.5754470601037397,"y":0.7795806855745574},{"x":0.5673219920925677,"y":0.7563961298697378},{"x":0.5882150241212957,"y":0.8104934265143168},{"x":0.6300010881787514,"y":0.8153235422861542},{"x":0.5696434400957597,"y":0.7892409171182322},{"x":0.5174108600239399,"y":0.7641243151046777},{"x":0.46866045195690814,"y":0.7660563614134126},{"x":0.46749972795531214,"y":0.7660563614134126},{"x":0.4094635278755123,"y":0.7641243151046777},{"x":0.3850883238419964,"y":0.7631582919503102},{"x":0.3200877797526207,"y":0.7583281761784728},{"x":0.28410533570314483,"y":0.7631582919503102},{"x":0.25624795966484093,"y":0.7786146624201898},{"x":0.23883709964090102,"y":0.7844108013463947},{"x":0.17847945155790926,"y":0.7805467087289248},{"x":0.17615800355471725,"y":0.7621922687959427},{"x":0.1680329355435453,"y":0.7380416899367557},{"x":0.23187275563132503,"y":0.7544640835610028},{"x":0.1633900395371613,"y":0.7931050097357021},{"x":0.14133628350683738,"y":0.8075953570512143},{"x":0.1947295875802532,"y":0.8288478664472989},{"x":0.17964017555950523,"y":0.8462362832259136},{"x":0.23651565163770902,"y":0.8520324221521185},{"x":0.20749755159780914,"y":0.8742509547025705},{"x":0.25740868366643693,"y":0.8810131167831429},{"x":0.25973013166962894,"y":0.8278818432929315},{"x":0.2550872356632449,"y":0.8520324221521185},{"x":0.37580253182922846,"y":0.8558965147695884},{"x":0.3293735717653887,"y":0.8781150473200404},{"x":0.33865936377815664,"y":0.8964694872530226},{"x":0.4187493198882803,"y":0.8897073251724502},{"x":0.4233922158946643,"y":0.8607266305414258},{"x":0.40133845986434036,"y":0.8887413020180828},{"x":0.49187493198882803,"y":0.8984015335617576},{"x":0.4733033479632921,"y":0.8645907231588957},{"x":0.5557147520766078,"y":0.875216977856938},{"x":0.5243752040335159,"y":0.8868092557093478},{"x":0.5742863361021437,"y":0.8945374409442877},{"x":0.5928579201276797,"y":0.8742509547025705},{"x":0.510446516014364,"y":0.8607266305414258},{"x":0.6265189161739635,"y":0.8510663989977509},{"x":0.5916971961260836,"y":0.8742509547025705},{"x":0.5557147520766078,"y":0.8877752788637152},{"x":0.6346439841851355,"y":0.8984015335617576},{"x":0.6880372882585513,"y":0.8926053946355527},{"x":0.6578584642170554,"y":0.8597606073870583},{"x":0.6044651601436396,"y":0.8501003758433835},{"x":0.6253581921723675,"y":0.8182216117492566},{"x":0.5835721281149117,"y":0.8394741211453413},{"x":0.6102687801516196,"y":0.8133914959774192},{"x":0.5278573760383038,"y":0.8336779822191364},{"x":0.5487504080670318,"y":0.8037312644337444},{"x":0.5,"y":0.8240177506754615},{"x":0.4698211759585041,"y":0.8201536580579916},{"x":0.5197323080271319,"y":0.7786146624201898},{"x":0.43848162791541223,"y":0.8037312644337444},{"x":0.38857049584678444,"y":0.8269158201385639},{"x":0.45589248793935216,"y":0.7815127318832923},{"x":0.3328557437701766,"y":0.8153235422861542},{"x":0.38857049584678444,"y":0.7844108013463947},{"x":0.2933911277159128,"y":0.8066293338968469},{"x":0.32473067575900466,"y":0.768954430876515},{"x":0.2713373716855889,"y":0.8085613802055818},{"x":0.2643730276760129,"y":0.768954430876515},{"x":0.3212485037542167,"y":0.7380416899367557},{"x":0.3456237077877326,"y":0.7737845466483525},{"x":0.3839275998404004,"y":0.7138911110775686},{"x":0.4245529398962603,"y":0.7486679446347979},{"x":0.4733033479632921,"y":0.7824787550376598},{"x":0.4628568319489281,"y":0.7283814583930809},{"x":0.504642896006384,"y":0.7554301067153704},{"x":0.5197323080271319,"y":0.7138911110775686},{"x":0.5336609960462839,"y":0.7641243151046777},{"x":0.6683049802314194,"y":0.8085613802055818},{"x":0.6764300482425913,"y":0.7544640835610028},{"x":0.6474119482026914,"y":0.8075953570512143},{"x":0.6729478762378033,"y":0.8095274033599493},{"x":0.7228590083064311,"y":0.7699204540308826},{"x":0.6799122202473793,"y":0.7341775973192858},{"x":0.5638398200877798,"y":0.7515660140979004},{"x":0.5174108600239399,"y":0.7196872500037735},{"x":0.40249918386593636,"y":0.7370756667823882},{"x":0.40598135587072437,"y":0.6926386016814841},{"x":0.35026660379411656,"y":0.7399737362454907},{"x":0.245801443650477,"y":0.7361096436280207},{"x":0.4187493198882803,"y":0.7177552036950385},{"x":0.42687438789945226,"y":0.6974687174533215},{"x":0.43267800790743227,"y":0.7477019214804305},{"x":0.515089412020748,"y":0.744803852017328},{"x":0.5533933040734158,"y":0.7071289489969963},{"x":0.6358047081867315,"y":0.7351436204736532},{"x":0.6671442562298233,"y":0.7853768245007623},{"x":0.6497333962058834,"y":0.8114594496686842},{"x":0.45821393594254417,"y":0.7882748939638647},{"x":0.6717871522362073,"y":0.6839443932921767},{"x":0.7089303202872792,"y":0.7254833889299784},{"x":0.7158946642968551,"y":0.7853768245007623},{"x":0.7936631724037868,"y":0.7380416899367557},{"x":0.506964344009576,"y":0.7061629258426287},{"x":0.3955348398563604,"y":0.6578617681242547},{"x":0.38624904784359243,"y":0.7167891805406711},{"x":0.5975008161340636,"y":0.7032648563795263}]'
		) as Point[]
	).map((p) => ({
		x: (1 - p.x) * 0.9 + 0.05,
		y: (1 - p.y) * 0.9 + 0.05
	}));

	// stages
	const s = {
		usa: 0,

		ga: 100,
		ga_centenial_award: 103,
		ga_centenial_drop: 105,
		ga_grey: 110,

		farms: 200
	};

	let stage_i = 0;
	let stage = 0;
	function is_last_stage(stage_i: number): boolean {
		return stage_i === Object.entries(s).length - 1;
	}
	function next_stage() {
		if (is_last_stage(stage_i)) {
			stage_i = 0;
			stage = 0;
		} else {
			stage_i++;
			stage = Object.entries(s)[stage_i][1];
		}
	}

	function page_text(stage: number): string {
		switch (stage) {
			case s.usa:
				// return 'Over the last century, the USA has seen a decrease in Black farm operators: from 20% to less than 1%!';
			case s.ga:
				return "Let's explore how this can be seen on the State level: Georgia's, for instance.";
			case s.ga_centenial_award:
				return "The Georgia Centennial Farm Award honours those upholding Georgia's agricultural history by maintaining working farms for more than 100 years.";
			case s.ga_centenial_drop:
				return 'However, of the roughly 600 recipients of the award:';
			case s.ga_grey:
				return 'Only 15 of the awarded farms have been Black-owned.';
			case s.farms:
				return "It's critical that the narrative of the Black Belt farmer does not go forgotten or ignored. Learn more about each of the winners by clicking the individual Pins.";
		}
		return '...';
	}

	function on_key_down(e: KeyboardEvent) {
		// `keyup` is the reverse, it fires whenever the physical key was let.
		// go after being held down

		// Just like our `keydown` handler, we need to update the boolean
		// flags, but in the opposite direction.
		switch (e.key) {
			case 'Escape':
				farm_open = undefined;
		}
	}

	let page_width = 1000;
	let page_height = 1000;

	function value_color(val: number) {
		const lt = (1 - ((val - 0.7) / (13.2 - 0.7)));

		const r = 0 + lt * 100;
		const g = 121 + lt * -121;
		const b = 52 + lt * -52;

		return `rgb(${r}, ${g}, ${b})`;
	}
</script>

<svelte:head>
	<link
		href="https://fonts.googleapis.com/css?family=Source+Sans+Pro"
		rel="stylesheet"
		type="text/css"
	/>
	<link href="https://fonts.googleapis.com/css?family=Gelasio" rel="stylesheet" />
</svelte:head>

<svelte:window on:keydown={on_key_down} />

<div
	style="width: 100%; display: flex; flex-direction: column; align-items: center;"
	bind:clientWidth={page_width}
	bind:clientHeight={page_height}
>
	<!-- TODO: page click (but don't double up with continue hit!) -->
	<!-- on:click={is_last_stage(stage_i) ? undefined : next_stage} -->
	<div class="nav">
		<a href="https://blackfarmersnetwork.com/" style="height: 100%">
			<img id="logo" src="images/bfn_logo.png" alt="" style="height: 100%" />
		</a>
		<h1>A Legacy of Resilience: Black Farmers in America</h1>
		<div></div>
	</div>

	{#if stage < s.ga}
		<!-- <div style="position: relative; display: flex; width: min(80vw, 120.170212766vh);"> -->
		<div class="text-chunk" style="margin: 0; padding-block: 10px; height: auto; width: 80vw;">
			Since 1900, the USA has seen a marked decrease in Black farm operators...
		</div>

		<div style="position: relative; display: flex; width: 50vh;">
			<img id="map" src="images/usa_map.png" alt="" style="width: 100%" />

			<span class="map-percent"  style={`color: ${value_color(active_graph_data[active_graph_data.length - 1].value)}; position: absolute; left: 50%; top: 50%; translate: -50% -50%;`}>{active_graph_data[active_graph_data.length - 1].value.toFixed(1)}%</span>
			<span class="map-year"  style="position: absolute; left: 50%; top: 50%; translate: -50% -50%; transform: translateY(calc(5rem * 0.6))">{Math.floor(active_graph_data[active_graph_data.length - 1].year)}</span>
		</div>

		<div class="text-chunk" style="color: #333333; margin: 0; padding-block: 10px; padding-top: 20px; height: auto; width: 80vw;">
			Explore why on the timeline below:
		</div>

		<div style="width: 80vw; display: flex; flex-direction: column; align-items: center;;">
			<LineGraph data={active_graph_data} width={page_width * 0.8} height={page_height * 0.25} />
			<Slider bind:value on:input={(e: any) => {set_active_graph_year(1933 + (e.detail[0] / 100) * (2022-1933)); console.log(e.detail)} } />

			<div class="timeline-event">
				<div class="timeline-title">
					{timeline_point.name}
				</div>

				<div class="timeline-year">
					{timeline_point.year}
				</div>

				<div class="timeline-text">
					{timeline_point.text}
				</div>
			</div>
			<div class="timeline-after">
				{#if (has_prev_event())}
					<a on:click={prev_event}>Prev Event</a>
				{/if}
				{#if (!has_prev_event())} <div></div> {/if}

				{#if (has_next_event())}
					<a on:click={next_event}>Next Event</a>
				{/if}
				{#if (!has_next_event())} <div></div> {/if}

			</div>
		</div>
	{/if}
	{#if stage >= s.ga}
		<div style="position: relative; display: flex; width: min(80vw, 66.5798045603vh);">
			<img id="map" src="images/ga_map_counties.png" alt="" style="width: 100%" />

			{#if stage >= s.ga_centenial_drop}
				{#each fake_600_coords as coords}
					<img
						src="images/map_pin.png"
						alt="map pin"
						class={{ pin: true, 'bad-pin': true, bad: stage >= s.ga_grey, invis: stage >= s.farms }}
						style={`width: 26px; position: absolute; top: ${coords.x * 100}%; left: ${coords.y * 100}%; transform: translate(-50%, -100%);`}
					/>
				{/each}

				{#each farms as farm}
					<img
						src="images/map_pin.png"
						alt="map pin"
						class={{ pin: true, good: stage >= s.farms }}
						style={`cursor: pointer; width: 26px; position: absolute; top: ${farm.coords.x * 100}%; left: ${farm.coords.y * 100}%; transform: translate(-50%, -100%);`}
						on:click={() => {
							farm_open = farm.name;
						}}
					/>
				{/each}
			{/if}

			{#if stage === s.ga_centenial_award}
				<img
					src="images/badge.png"
					alt="Centennial Award"
					style="width: 20vw; position: fixed; top: 50vh; left: 5vw; translate: 0 -50%;"
				/>
			{/if}
		</div>
	{/if}

	{#if farm_open != null}
		<div
			class="modal-back"
			on:click={() => {
				farm_open = undefined;
			}}
		></div>
		<div class="farm-modal">
			<span
				class="close"
				on:click={() => {
					farm_open = undefined;
				}}>&times;</span
			>
			{#each farms as farm (farm.name)}
				{#if farm_open === farm.name}
					<div class="farm-details">
						<h2>{farm.name}</h2>

						<p><strong>Founded:</strong> {farm.founded || 'N/A'}</p>
						<p><strong>County:</strong> {farm.county || 'N/A'}</p>

						<h3>About the Farm</h3>
						<p>{farm.about || 'No information available.'}</p>

						<h3>History & Legacy</h3>
						<p>{farm.history || 'No historical data provided.'}</p>

						<h3>Present Day</h3>
						<p>{farm.present || 'No present-day information available.'}</p>

						{#if farm.image}
							<img src={farm.image} alt={farm.name} class="farm-image" />
						{:else}
							<p>No image available.</p>
						{/if}

						<h3>Source</h3>
						<p>{farm.source || 'No source provided.'}</p>
					</div>
				{/if}
			{/each}

			<style>
				.farm-details {
					max-width: 600px;
					margin: auto;
					padding: 20px;
					border-radius: 10px;
					background: #f9f9f9;
					box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);

					max-height: 60vh;
					overflow-y: scroll;
				}

				.farm-details h2 {
					color: #2c3e50;
					font-size: 1.8em;
				}

				.farm-details h3 {
					color: #34495e;
					margin-top: 15px;
				}

				.farm-details p {
					font-size: 1em;
					color: #555;
				}

				.farm-image {
					width: 100%;
					max-height: 400px;
					object-fit: cover;
					border-radius: 8px;
					margin-top: 10px;
				}
			</style>

			<!-- <div style="width: 100%; display: flex; align-items: center; justify-content: center;">
				<img src="images/todo_image.png" alt="" style="width: 20vw" />
			</div> -->
		</div>
	{/if}

	{#if (stage >= s.ga)}
		<div class="text-chunk">
			{page_text(stage)}
		</div>
	{/if}

	<button class="continue-btn" on:click={next_stage}>
		{is_last_stage(stage_i) ? 'START OVER' : 'CONTINUE'}
	</button>
</div>


