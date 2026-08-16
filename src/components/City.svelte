<script>
    export let location = {};

    let width = 0;

    // LIBS
    import { onMount } from 'svelte';


    // const margin = {
	// 	top: 20,
	// 	right: 20,
	// 	bottom: 20,
	// 	left: 20 
	// };
    
    // $: console.log(location);
    // $: turnout_pct = parseInt(location.ballots_cast) / parseInt(location.registered_voters) * 100;

    // FUNCTIONS
    function init() {
        console.log('CHART INIT!')

    }

    // LIGHTS! CAMERA! ACTION!
    onMount(init);
</script>

<div class="city-header" bind:clientWidth={width}>
    <div id="head">
        <img src={location.logo} alt="municipality logo"/>
        <h2>Local election results</h2>
    </div>
        <!-- <div id="chart">
            <div id="turnout">
                <h2>{turnout_pct.toFixed(1)}%
            </div>
            <div id="turnout-time"></div>
    </div> -->
</div>

<style>
    /* .city-header {
        display: flex;
        justify-content: space-between;
        margin-bottom: 5vh;
    } */
    .city-header {
        margin: 0 auto 5vh auto;
        text-align: center;
     }
    #app .city-header h2 {
        color: var(--grey03);
        font-size: 1.5rem;
    }
</style>