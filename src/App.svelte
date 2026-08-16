<script>
    // COMPONENTS
    import { onMount } from "svelte";
    import City from "$components/City.svelte";
    import Candidates from "$components/Candidates.svelte";
    import Select from "svelte-select"; // https://github.com/rob-balfre/svelte-select

    // DATA
    import { menuItems } from "$data/menu-items";
    const dataUrl = 'https://raw.githubusercontent.com/vs-postmedia/civic-info-bc-scraper/refs/heads/master/data/data-2022.json';

    // TEST CODE
    let currentURL = 0;
    const dataURLs = [
        'https://raw.githubusercontent.com/vs-postmedia/civic-info-bc-scraper/refs/heads/master/data/data-2022.json',
        'https://raw.githubusercontent.com/vs-postmedia/civic-info-bc-scraper/refs/heads/master/data/data-2022-test.json'
    ]

    // VARIABLES
    let value = null;
    let data = [];
    let filteredData = [];
    // let jurisdiction = 'city';
    let selectedValue = '139';
    let location = {};
    let mayors = [];
    let councillors = [];
    const refreshInterval = 1; // in minutes
    const defaultSelectValue = menuItems.find(item => String(item.id) === selectedValue)?.id ?? menuItems[0]?.id ?? '';

    $: if (value && data.length) {
        updateData(value);
    }

    async function fetchData(url) {
        const resp = await fetch(url);
        const rawData = await resp.text();

        // return JSON.parse(rawData);
        data = JSON.parse(rawData);

        // set select menu
        updateSelectMenu();
    }

    function splitData(currentFilteredData) {
        // reset vars if no location selected (prob don't need this)
        if (!currentFilteredData) {
            location = {};
            mayors = [];
            councillors = [];
            return;
        }

        // get top-level vars from data
        const { ballots_cast, councillors_to_elect, logo, name, population, registered_voters, estimated_registered_voters } = currentFilteredData;

        location = {
            name,
            ballots_cast,
            councillors_to_elect,
            logo,
            population,
            estimated_registered_voters,
            registered_voters
        };

        // prep candidate data
        processCandidates(currentFilteredData);
    }

    function processCandidates(currentFilteredData) {
        const candidates = currentFilteredData?.candidates || [];
        const totalVotes = Number(location.ballots_cast || 0);

        // separate out mayor candidates & calculate vote %
        const mayorCandidates = candidates.filter(d => d.running_for == 'MAYOR');
        mayors = mayorCandidates.map(d => ({
            ...d,
            total_votes: totalVotes,
            votes_pct: totalVotes > 0 ? (Number(d.votes_for || 0) / totalVotes) * 100 : 0
        }));
        
        // same for councillors
        const councillorCandidates = candidates.filter(d => d.running_for == 'COUNCILLOR');
        councillors = councillorCandidates.map(d => ({
            ...d,
            total_votes: totalVotes,
            votes_pct: totalVotes > 0 ? (Number(d.votes_for || 0) / totalVotes) * 100 : 0
        }));
    }

    function updateData(selectedValue) {
        console.log('UPDATE DATA');
        console.log(selectedValue)

        const selectedKey = typeof selectedValue === 'string'
            ? selectedValue
            : selectedValue?.id ?? selectedValue?.value;
        if (!selectedKey) return;

        const match = data.find(d =>
            String(d.id) === String(selectedKey)
        );

        if (!match) {
            filteredData = [];
            location = {};
            mayors = [];
            councillors = [];
            return;
        }

        filteredData = [match];
        splitData(match);
    }

    function updateSelectMenu() {
        if (!value && defaultSelectValue) {
            value = menuItems.find(item =>
                String(item.value) === String(defaultSelectValue) ||
                String(item.id) === String(defaultSelectValue)
            ) ?? null;
        }
    }

    async function init() {
        await fetchData(dataUrl);

        // get city from URL params
        const urlParams = new URLSearchParams(window.location.search);
        if (urlParams.has('name')) {
            const urlName = urlParams.get('name').toLowerCase();
            value = menuItems.find(item =>
                item.value === urlName ||
                String(item.id) === urlName ||
                item.label.toLowerCase().replace(/\s+/g, '-') === urlName
            ) ?? null;
        } else {
            value = null;
        }

        // set select menu
        updateSelectMenu();
    }

    onMount(() => {
        init();

        const refreshData = setInterval(() => {
            if (currentURL === 0) {
                currentURL = 1;
            } else {
                currentURL = 0
            }
            console.log(currentURL)
            fetchData(dataURLs[currentURL]);
        }, refreshInterval * 60 * 1000);

        return () => clearInterval(refreshData);
    });
</script>


<header>
    <h1>2026 local election results for</h1>
    <!-- <p class="subhead">Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p> -->
</header>

<main>
    <!-- <City
        location={location}
    /> -->

    <Select items={menuItems}
        itemId="id"
        bind:value
        change={updateData}
        placeholder="Pick a city..."
		showChevron="true"
		listOpen={false}
    />
    <p class="select-header">Choose a city  <span>⤴️</span></p>

    <p class="timestamp">Last updated: XXX</p>

    {#key value?.value || 'default'}
        <Candidates
            data={mayors}
            role="Mayor"
        />
    {/key}
    
    {#key value?.value || 'default'}
        <Candidates
            data={councillors}
            role="Council"
        />
    {/key}
</main>

<footer>
    <p class="note">NOTE: tk.</p>
    <p class="source">Source:  <a href="https://www.civicinfo.bc.ca/election-results" target="_blank">Civic Info B.C.</a></p>
</footer>
  
<style>
    @import '$css/normalize.css';
    @import '$css/fonts.css';
    @import '$css/colors.css';
    @import '$css/app.css';

	header > h1 {
		text-align: center;
	}
	header .subhead {
		margin: 0 auto;
		max-width: 525px;
		text-align: center;
	}

    /* COMBOBOX SELECTOR */
    :global(p.select-header, p.timestamp) {
        color: var(--grey03) !important;
        font-family: 'BentonSansCond-RegItalic', italic !important;
        font-size: 1rem;
        margin: 0 auto 2vh 0;
        text-align: center;
    }
     :global(p.select-header > span) {
        filter: greyscale(1);
        font-size: 0.85rem;
     }
  	:global(.svelte-select) {
        border: none !important;
		margin: 0 auto !important;
        width: auto !important;
        /* max-width: 250px; */
  	}
    :global(.svelte-select .value-container) {
        justify-content: center;
        text-align: center;
    }
    :global(.svelte-select .selected-item) {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 100%;
        text-align: center;
        padding-right: 0;

        color: var(--blue01) !important;
        font-family: 'Shift-BoldItalic', serif;
        font-size: 2rem !important;
		/* font-family: 'BentonSansCond-Regular', sans; */
    }
    :global(.svelte-select .indicators) {
        position: absolute !important;
        right: 0 !important;
    }

    :global(
		/* .svelte-select .selected-item, */
		.svelte-select .item,
		.svelte-select input
	) {

		font-family: 'BentonSansCond-Regular', sans;
	}
</style>
