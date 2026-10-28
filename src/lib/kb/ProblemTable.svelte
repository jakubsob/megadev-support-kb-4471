<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // The desk's own view (as CauseTable counts it), one ticket per problem.
  // tickets: problem → the url of the ticket that now owns it.
  let { ticket, tickets }: { ticket: number; tickets: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const ticketNo = (url: string) => '#' + url.split('/').pop();
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{groups.length + 1}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if tickets[g.cause]}<a href={tickets[g.cause]}>{ticketNo(tickets[g.cause])}</a>{:else}no ticket yet{/if}</td>
      </tr>
    {/each}
    <tr>
      <td><code>one-day</code></td>
      <td>{oneDay.length}</td>
      <td>{new Set(oneDay.map((r) => r.tenant)).size}</td>
      <td>{oneDay.map((r) => r.receivedAt.slice(0, 10)).sort()[0]} → {oneDay.map((r) => r.receivedAt.slice(0, 10)).sort().at(-1)}</td>
      <td>{#if tickets['one-day']}<a href={tickets['one-day']}>{ticketNo(tickets['one-day'])}</a>{:else}no ticket yet{/if}</td>
    </tr>
  </tbody>
</table>

{#each groups as g (g.cause)}
  {@const moved = readAs(g, notes.oneDayMove)}
  {#if moved}
    <p class="note">{moved} of the <code>{g.cause}</code> reports read as a one-day move: counted under <code>{g.cause}</code> by tenant, as the desk files them, and listed again under <code>one-day</code>.</p>
  {/if}
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
