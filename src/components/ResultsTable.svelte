<script>
    import Ballots from "$components/Ballots.svelte";
    import Candidates from "$components/Candidates.svelte";

    export let value = '';
    export let location = {};
    export let sdLocation = {};
    export let parkLocation = {};
    export let mayors = [];
    export let councillors = [];
    export let schoolTrustees = [];
    export let eaDirectors = [];
    export let parkTrustees = [];
    export let ballotResults = [];
    export let councilElectedCount = 0;
    export let trusteeElectedCount = 0;
    export let parkboardElectedCount = 0;

    let activeTab = 'mayor-council';

    $: if (activeTab === 'ballot-initiatives' && ballotResults.length === 0) {
        activeTab = 'mayor-council';
    }
</script>

{#if eaDirectors.length > 0}
    {#key value?.id || 'default'}
        <Candidates
            data={eaDirectors}
            role="Directors"
        />
    {/key}
{:else}
    <div class="result-tabs" role="tablist" aria-label="Election results">
            <button
                type="button"
                role="tab"
                aria-selected={activeTab === 'mayor-council'}
                aria-controls="mayor-council-panel"
                class:active={activeTab === 'mayor-council'}
                on:click={() => activeTab = 'mayor-council'}
            >Mayor and Council</button>
            <button
                type="button"
                role="tab"
                aria-selected={activeTab === 'school-park-board'}
                aria-controls="school-park-board-panel"
                class:active={activeTab === 'school-park-board'}
                on:click={() => activeTab = 'school-park-board'}
            >School/Park board</button>
            {#if ballotResults.length > 0}
                <button
                    type="button"
                    role="tab"
                    aria-selected={activeTab === 'ballot-initiatives'}
                    aria-controls="ballot-initiatives-panel"
                    class:active={activeTab === 'ballot-initiatives'}
                    on:click={() => activeTab = 'ballot-initiatives'}
                >Ballot initiatives</button>
            {/if}
    </div>

        {#if activeTab === 'mayor-council'}
            <section id="mayor-council-panel" role="tabpanel">
                <!-- key/value block forces Svelte to recreate each table when the selected city changes -->
                {#key value?.id || 'default'}
                    <Candidates
                        data={mayors}
                        role="Mayor"
                    />
                {/key}
                
                {#key value?.id || 'default'}
                    <Candidates
                        data={councillors}
                        electedCount={councilElectedCount}
                        location={location}
                        role="Council"
                    />
                {/key}
            </section>
        {:else if activeTab === 'school-park-board'}
            <section id="school-park-board-panel" role="tabpanel">
                <!-- parkboard -->
                {#if parkTrustees.length > 0}
                    {#key value?.id || 'default'}
                        <Candidates
                            data={parkTrustees}
                            electedCount={parkboardElectedCount}
                            location={parkLocation}
                            role="Park board"
                        />
                    {/key}
                {/if}
                <!-- school board -->
                {#key value?.id || 'default'}
                    <Candidates
                        data={schoolTrustees}
                        electedCount={trusteeElectedCount}
                        location={sdLocation}
                        role="School board"
                    />
                {/key}
            </section>
        {:else if ballotResults.length > 0}
            <section id="ballot-initiatives-panel" role="tabpanel">
                {#key value?.id || 'default'}
                    <Ballots
                        id={value?.id}
                        ballots={ballotResults}
                    />
                {/key}
            </section>
        {/if}
    {/if}

<style>
    .result-tabs {
        display: flex;
        border-bottom: 1px solid var(--grey03);
        margin: 1.5rem 0;
    }

    .result-tabs button {
        background: transparent;
        border: 0;
        border-bottom: 3px solid transparent;
        color: var(--grey03);
        cursor: pointer;
        font-family: BentonSansCond-Bold, sans-serif;
        font-size: 1rem;
        padding: 0.65rem 1rem 0.5rem;
    }

    .result-tabs button.active {
        border-bottom-color: var(--blue01);
        color: var(--blue01);
    }

    .result-tabs button:focus-visible {
        outline: 2px solid var(--blue01);
        outline-offset: -2px;
    }
</style>
