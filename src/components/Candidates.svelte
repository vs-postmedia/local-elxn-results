<script>
    export let data = [];
    export let role = '';
    export let location = {};
    export let electedCount = 0;

    import { afterUpdate, onMount } from 'svelte';
    import { Table } from '@flowbite-svelte-plugins/datatable';

    let dataTableInstance = null;
    let resizeTimer;
    let lastRowSignature = '';

    $: console.log('lOCATION')
    $: console.log(location)

    $: datatableOptions = {
        searchable: true,
        sortable: false,
        labels: {
            searchLabel: '',
            placeholder: 'Search candidates...'
        },
        // perPage: 10,
        perPageSelect: null,
        paging: false,
        // hoverable: true
        scrollY: 'auto'
    };

    function addCommasToNumber(number) {
        return Number(number || 0).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    }

    function titleCase(str = '') {
        return String(str)
            .trim()
            .toLowerCase()
            .split(/\s+/)
            .filter(Boolean)
            .map(word => word.charAt(0).toUpperCase() + word.slice(1))
            .join(' ');
    }

    function getCandidateName(row = {}) {
        return [row.candidate_first_name || '', titleCase(row.candidate_last_name)]
            .filter(Boolean)
            .join(' ')
            .trim();
    }

    $: candidateRows = (Array.isArray(data)
        ? data.filter(d => d.candidate_first_name || d.candidate_last_name || d.votes_for)
        : Array.isArray(data?.candidates)
            ? data.candidates
            : [])
        .map(candidate => ({
            ...candidate,
            votes_for: Number(candidate.votes_for || 0)
        }))
        .sort((a, b) => b.votes_for - a.votes_for);

    function handleResize() {
        clearTimeout(resizeTimer);
        resizeTimer = setTimeout(() => {
            dataTableInstance?.update?.(true);
        }, 100);
    }

    afterUpdate(() => {
        const rowSignature = JSON.stringify(candidateRows.map(row => ({
            candidate: getCandidateName(row),
            votes_for: Number(row.votes_for || 0),
            votes_pct: Number(row.votes_pct || 0),
            elected: row.elected || ''
        })));

        if (dataTableInstance && rowSignature !== lastRowSignature) {
            lastRowSignature = rowSignature;
            dataTableInstance?.update?.(true);
        }
    });

    onMount(() => {
        window.addEventListener('resize', handleResize);

        return () => {
            window.removeEventListener('resize', handleResize);
            clearTimeout(resizeTimer);
        };
    });
</script>

<div class="chart-container">
    <h2>{role}</h2>
    {#if location.trustee}
        <p class="subhead">{location.name}</p>
    {/if}
    {#if location.councillors_to_elect}
        <p class="subtitle">{electedCount} of {location.councillors_to_elect} candidates elected/acclaimed</p>
    {/if}

    {#if candidateRows.length}
        <Table bind:dataTableInstance={dataTableInstance} dataTableOptions={datatableOptions}>
            <thead>
                <tr>
                    <th>Candidate</th>
                    <th>Vote share</th>
                    <th>Votes</th>
                    <!-- <th>Total votes</th> -->
                </tr>
            </thead>
            <tbody>
                {#each candidateRows as candidate}
                    {@const votesPct = Number(candidate.votes_pct || 0)}
                    {@const labelClass = votesPct >= 30 ? 'inside' : 'outside'}
                    <tr class={candidate.elected ? 'YES' : ''}>
                        <td>
                            <div class="candidate-cell">
                                <div class="candidate-name">
                                {#if candidate.acclamation === 'YES' || candidate.elected == 'YES'}
                                    <span class='subtitle'>✅</span>
                                <!-- {:else}
                                    <span class='subtitle'>{candidate.elected === 'YES' ? '✅ elected' : '' }</span> -->
                                {/if}
                                    {candidate.candidate_first_name || ''} {titleCase(candidate.candidate_last_name) || ''}
                                
                                    <span class='subtitle'>{candidate.previous_experience === 'Incumbent' ? ' (Incumbent)' : '' }</span>
                                    <!-- <span class='subtitle'>{candidate.elected === 'YES' && candidate.acclamation !== 'YES' ? ' elected' : '' }</span> -->
                                </div>
                                <div class="party-name">
                                    {candidate.electoral_organization?.electoral_organization_name === 'None' ? '' : candidate.electoral_organization?.electoral_organization_name.replace(' Electors', '').replace(' Association', '') || ''}
                                </div>
                            </div>
                        </td>
                        <td class="votes-cell">
                            {#if candidate.acclamation === 'YES'}
                                <span class='subtitle'>Acclaimed</span>
                            {:else}
                                <div class="bar-track">
                                    <div class="bar-fill" style={`width: ${Math.max(4, votesPct)}%`}>
                                        <div class={`bar-label ${labelClass}`}>
                                            {votesPct ? `${votesPct.toFixed(1)}%` : '0.0%'}
                                        </div>
                                    </div>
                                </div>
                            {/if}
                        </td>
                        <td>{addCommasToNumber(candidate.votes_for) || ''}</td>
                        <!-- <td>{addCommasToNumber(candidate.total_votes || 0)}</td> -->
                    </tr>
                {/each}
            </tbody>
        </Table>
    {/if}
</div>

<style>
    .chart-container {
        margin-bottom: 5vh;
        width: 100%;
    }

    .chart-container > h2 {
        font-size: 2rem;
    }

    .chart-container :global(.datatable-wrapper) {
        width: 100%;
        overflow-x: auto;
        -webkit-overflow-scrolling: touch;
    }

    .chart-container :global(.datatable-container) {
        max-height: 200px;
    }

    .chart-container :global(.datatable-table) {
        width: 100%;
        border-collapse: collapse;
        table-layout: fixed;
    }

    .chart-container :global(.datatable-table th),
    .chart-container :global(.datatable-table td) {
        padding: 0.6rem 0.75rem;
        text-align: left;
    }

    .chart-container :global(.datatable-table thead th:nth-child(1)),
    .chart-container :global(.datatable-table tbody td:nth-child(1)) {
        width: 50%;
    }

    .chart-container :global(.datatable-table thead th:nth-child(2)),
    .chart-container :global(.datatable-table tbody td:nth-child(2)) {
        width: 34%;
    }

    .chart-container :global(.datatable-table thead th:nth-child(3)),
    .chart-container :global(.datatable-table tbody td:nth-child(3)) {
        width: 15%;
        text-align: right;
    }

    .chart-container :global(.datatable-table thead th) {
        font-weight: 600;
        color: #111827;
    }

    tr.YES {
        border-left: 5px solid #0062A3;
    }
    :global(#app p.subtitle),
    .subtitle {
        color: var(--grey03);
        font-family: 'BentonSansCond-RegItalic', italic;
        font-size: 0.85rem;
    }
    :global(#app p.subhead) {
        font-family: 'BentonSansCond-RegItalic', italic !important;
        font-size: 1rem !important;
    }

    .candidate-cell {
        display: flex;
        flex-direction: column;
        gap: 0.15rem;
    }

    .candidate-name {
        font-weight: 600;
        color: #111827;
    }

    .party-name {
        font-size: 0.75rem;
        color: #999a9c;
        margin-top: 2px;
        text-transform: titlecase;
    }

    .bar-track {
        position: relative;
        width: 100%;
        min-width: 120px;
        height: 24px;
        background: #e5e7eb;
        overflow: hidden;

    }

    .bar-fill {
        position: relative;
        height: 100%;
        background: #0062a3;
        overflow: visible;
    }

    .bar-label {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 0.85rem;
        font-weight: 600;
        color: #111827;
        pointer-events: none;
        white-space: nowrap;
    }

    .bar-label.inside {
        right: 0.25rem;
        justify-content: flex-end;
        color: #ffffff;
    }

    .bar-label.outside {
        left: calc(100% + 0.35rem);
        justify-content: flex-start;
        color: #0062a3;
    }
</style>