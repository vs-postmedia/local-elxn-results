<script>
    // COMPONENTS
    import { onMount } from "svelte";
    import ResultsTable from "$components/ResultsTable.svelte";
    import Select from "svelte-select"; // https://github.com/rob-balfre/svelte-select

    // DATA
    import { menuItems } from "$data/menu-items";
	// import ResultsTable from "./components/ResultsTable.svelte";
    const dataUrl = 'https://raw.githubusercontent.com/vs-postmedia/civic-info-bc-scraper/refs/heads/master/data/data-2026.json';   

    // TEST CODE
    let currentURL = 0;
    const dataURLs = [
        'https://raw.githubusercontent.com/vs-postmedia/civic-info-bc-scraper/refs/heads/master/data/data-2026.json',
        'https://raw.githubusercontent.com/vs-postmedia/civic-info-bc-scraper/refs/heads/master/data/data-2026-test.json'
    ]

    // VARIABLES
    let value = null;
    let data = [];
    let ballotResults = [];
    let filteredData = [];
    let timestamp = 'No updates yet...';
    let selectedValue = '139';

    let location = {};
    let sdLocation = {};
    let parkLocation = {};
    let mayors = [];
    let councillors = [];
    let schoolTrustees = [];
    let eaDirectors = [];
    let parkTrustees = [];
    let councilElectedCount = 0;
    let trusteeElectedCount = 0;
    let parkboardElectedCount = 0
    let eaDirectorsElectedCount = 0;
    
    let activeTab = 'mayor-council';
    const refreshInterval = 1; // in minutes
    const defaultSelectValue = menuItems.find(item => String(item.id) === selectedValue)?.id ?? menuItems[0]?.id ?? '';

    $: if (value && data.length) {
        updateData(value);
    }

    async function fetchData(url) {
        const resp = await fetch(url);
        const rawData = await resp.text();

        // return JSON.parse(rawData);
        const jsonData = JSON.parse(rawData);
        data = jsonData.data;
        timestamp = jsonData.timestamp;

        // set select menu
        updateSelectMenu();
    }

    function getSchoolDistrictArea(currentFilteredData, selectedValue) {
        const schoolDistrictAreas = currentFilteredData.school_district?.school_district_areas || [];
        const selectedLocationLabel = selectedValue?.label?.toLowerCase() || '';

        return schoolDistrictAreas.find(area =>
            selectedLocationLabel.includes(area.name.toLowerCase()) ||
            area.name.toLowerCase().includes(selectedLocationLabel)
        ) || schoolDistrictAreas[0];
    }

    function splitData(currentFilteredData, selectedValue) {
        // reset vars if no location selected (prob don't need this)
        if (!currentFilteredData) {
            location = {};
            mayors = [];
            councillors = [];
            c = [];
            eaDirectors = [];
            sdLocation = {};
            councilElectedCount = 0;
            eaDirectorsElectedCount = 0;
            trusteeElectedCount = 0;
            return;
        }

        // get top-level vars from data
        const { ballots_cast, councillors_to_elect, name, population, registered_voters, estimated_registered_voters } = currentFilteredData;

        location = {
            name,
            ballots_cast,
            councillors_to_elect,
            population,
            estimated_registered_voters,
            registered_voters
        };

        const schoolDistrictArea = getSchoolDistrictArea(currentFilteredData, selectedValue);
        sdLocation = schoolDistrictArea
            ? {
                name: schoolDistrictArea.name,
                councillors_to_elect: schoolDistrictArea.councillors_to_elect,
                trustee: true
            }
            : {};
        
        parkLocation = currentFilteredData.park_board
            ? {
                councillors_to_elect: currentFilteredData.park_board.councillors_to_elect
            }
            : {};

        // prep candidate data
        processCandidates(currentFilteredData, schoolDistrictArea);
    }

    function processCandidates(currentFilteredData, schoolDistrictArea) {
        // console.log('PROCESS CANDIDATES')
        // console.log(currentFilteredData)
        const candidates = currentFilteredData?.candidates || [];
        const schoolboardCandidates = schoolDistrictArea?.candidates || [];
        const parkboardCandidates = currentFilteredData.park_board?.candidates || [];
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
        // count how many councillors were elected
        councilElectedCount = councillorCandidates.filter(d => d.elected === 'YES').length;

        // same for school board trustees
        // const trusteeCandidates = schoolboard.filter(d => d.running_for == 'TRUSTEE');
        schoolTrustees = schoolboardCandidates.map(d => ({
            ...d,
            total_votes: totalVotes,
            votes_pct: totalVotes > 0 ? (Number(d.votes_for || 0) / totalVotes) * 100 : 0
        }));
        // count how many trustees were elected
        trusteeElectedCount = schoolboardCandidates.filter(d => d.elected === 'YES').length;

        // Electoral Area A
        const electoralAreaDirectorCandidates = candidates.filter(d => d.running_for == 'ELECTORAL AREA DIRECTOR');
        eaDirectors = electoralAreaDirectorCandidates.map(d => ({
            ...d,
            total_votes: totalVotes,
            votes_pct: totalVotes > 0 ? (Number(d.votes_for || 0) / totalVotes) * 100 : 0
        }));
        // count how many directors were elected
        eaDirectorsElectedCount = electoralAreaDirectorCandidates.filter(d => d.elected === 'YES').length;

        // Vancouver Park board
        const parkTrusteeCandidates = parkboardCandidates.filter(d => d.running_for == 'COMMISSIONER');
        parkTrustees = parkTrusteeCandidates.map(d => ({
            ...d,
            total_votes: totalVotes,
            votes_pct: totalVotes > 0 ? (Number(d.votes_for || 0) / totalVotes) * 100 : 0
        }));
        // count how many directors were elected
        parkboardElectedCount = parkTrusteeCandidates.filter(d => d.elected === 'YES').length;
    }

    function updateData(selectedValue) {
        // console.log('UPDATE DATA');
        // console.log(selectedValue)

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
            schoolTrustees = [];
            eaDirectors = [];
            sdLocation = {};
            councilElectedCount = 0;
            eaDirectorsElectedCount = 0;
            trusteeElectedCount = 0;
            return;
        }

        filteredData = [match];
        splitData(match, selectedValue);

        console.log("MATCH")
        console.log(match)

        ballotResults = match.ballot_results.filter(d => String(d.id) === String(match.id));
        if (activeTab === 'ballot-initiatives' && ballotResults.length === 0) {
            activeTab = 'mayor-council';
        }
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

            fetchData(dataURLs[currentURL]);
        }, refreshInterval * 60 * 1000);

        return () => clearInterval(refreshData);
    });
</script>


<header>
    <h1>2026 local election results</h1>
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
    <p class="select-header"><span>⬆️</span>  Choose a city  <span>⬆️</span></p>

    <p class="timestamp">Last update: {timestamp}</p>

    <ResultsTable
        value={value}
        mayors={mayors}
        location={location}
        sdLocation={sdLocation}
        parkLocation={parkLocation}
        councillors={councillors}
        schoolTrustees={schoolTrustees}
        eaDirectors={eaDirectors}
        parkTrustees={parkTrustees}
        ballotResults={ballotResults}
        councilElectedCount={councilElectedCount}
        trusteeElectedCount={trusteeElectedCount}
        parkboardElectedCount={parkboardElectedCount}
    />


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
        margin-bottom: 10px;
		text-align: center;
	}

    :global(p.select-header) {
        font-family: 'BentonSansCond-bold' !important;
        font-size: 1.2rem;
        margin: 0 auto 2vh 0;
        text-align: center;
    }
    :global(p.timestamp) {
        color: var(--grey03) !important;
        font-family: 'BentonSansCond-RegItalic', italic !important;
        font-size: 1rem;
        margin: 0 auto 2vh 0;
        text-align: center;
    }
     :global(p.select-header > span) {
        font-size: 0.85rem;
     }

    /* COMBOBOX SELECTOR */
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
        text-decoration: underline;
        padding-right: 0;

        color: var(--blue01) !important;
        font-family: 'Shift-BoldItalic', serif;
        font-size: 2rem !important;
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

    @media (min-width: 600px) {
        :global(.svelte-select .selected-item) {
            font-size: 3rem !important;
        }
    }
</style>
