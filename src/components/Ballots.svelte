<script>
    export let id = '';
    export let ballots = [];

    let width = 0;
    let expandedBallots = {};

    // LIBS
    import { onMount } from 'svelte';


    // FUNCTIONS
    function addCommasToNumber(number) {
        return Number(number || 0).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    }

    function votePct(votes, total) {
        return total > 0 ? (Number(votes || 0) / total) * 100 : 0;
    }

    function toggleFullText(refid) {
        const isOpen = !expandedBallots[refid];
        expandedBallots = isOpen ? { [refid]: true } : {};
    }

    function init() {
        console.log('BALLOT INIT!')

    }

    // LIGHTS! CAMERA! ACTION!
    onMount(init);
</script>

<div class="ballot-header" bind:clientWidth={width}>

</div>

{#each ballots as ballot}
    {@const totalVotes = Number(ballot.votes_for || 0) + Number(ballot.votes_against || 0)}
    <div class="ballot">
        <h2>{ballot.summary}</h2>
        {#if ballot.passed !== undefined}
            {#if ballot.passed !== null}
                <p class="result" class:passed={ballot.passed}>
                    {ballot.passed ? 'Passed' : 'Did not pass'}
                </p>
            {/if}
            <ul class="vote-totals">
                <li>In favour: {addCommasToNumber(ballot.votes_for)} ({votePct(ballot.votes_for, totalVotes).toFixed(1)}%)</li>
                <li>Against: {addCommasToNumber(ballot.votes_against)} ({votePct(ballot.votes_against, totalVotes).toFixed(1)}%)</li>
            </ul>
        {/if}
        <h3 class="accordion-heading">
            <button
                type="button"
                class="accordion-trigger"
                id={`ballot-toggle-${ballot.refid}`}
                aria-expanded={!!expandedBallots[ballot.refid]}
                aria-controls={`ballot-panel-${ballot.refid}`}
                on:click={() => toggleFullText(ballot.refid)}
            >
                <span class="accordion-trigger-label">{expandedBallots[ballot.refid] ? 'Hide full text' : 'Show full text'}</span>
                <span class="accordion-caret" class:open={expandedBallots[ballot.refid]} aria-hidden="true">&#9662;</span>
            </button>
        </h3>
        <div
            id={`ballot-panel-${ballot.refid}`}
            role="region"
            aria-labelledby={`ballot-toggle-${ballot.refid}`}
            class="full-text"
            class:open={expandedBallots[ballot.refid]}
        >
            <div class="full-text-inner">
                <p class="question">{ballot.question}</p>
            </div>
        </div>
    </div>

{/each}

<style>
    .ballot {
        margin-bottom: 2rem;
    }
    .ballot > h2 {
        font-size: 1.25rem;
        padding-bottom: 7px;
    }
    :global(#app p.result) {
        font-family: BentonSansCond-Bold, sans-serif;
    }
    .vote-totals li {
        padding: 3px 0;
    }

    .result.passed {
        color: green;
    }

    .ballot .question {
        white-space: pre-line;
    }

    .accordion-trigger {
        align-items: center;
        background: transparent;
        border: none;
        border-bottom: 1px solid var(--grey04);
        cursor: pointer;
        display: flex;
        font-family: BentonSansCond-Bold, sans-serif;
        justify-content: space-between;
        margin-top: 5px;
        min-height: 35px;
        padding: 5px 0 0 0;
        transition: background-color 0.15s ease, color 0.15s ease;
        width: 100%;
    }

    .accordion-trigger:hover {
        /* background: var(--grey02, #e8e8e8); */
        /* color: #00436f; */
    }

    .accordion-trigger:focus-visible {
        /* outline: 2px solid #0062a3; */
        /* outline-offset: 2px; */
    }

    .accordion-caret {
        display: inline-block;
        transform: rotate(180deg);
        transition: transform 0.3s ease;
    }

    .accordion-caret.open {
        transform: rotate(0deg);
    }

    .full-text {
        display: grid;
        grid-template-rows: 0fr;
        transition: grid-template-rows 0.25s ease;
    }

    .full-text.open {
        grid-template-rows: 1fr;
    }

    .full-text-inner {
        /* border: 1px solid var(--grey03, #ccc); */
        border-top: 0;
        border-radius: 0 0 4px 4px;
        overflow: hidden;
        padding: 0 1rem;
    }

    .full-text.open .full-text-inner {
        padding: 0.75rem 1rem;
    }

</style>
