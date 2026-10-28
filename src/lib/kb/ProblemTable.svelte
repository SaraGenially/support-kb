<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import split from '#lib/data/tickets-4471.json';
  import { groupByCause, summarize, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const exported = $derived((reports as Report[]).filter((r) => r.mergedInto === ticket));
  const merged = $derived(uniqueById(exported));
  const shifted = $derived(summarize('shift', merged.filter((r) => notes.oneDayMove.includes(r.id))));
  const groups = $derived([...groupByCause(merged, tenantOf), shifted]);
  const filedUnder = $derived(groupByCause(shifted.reports, tenantOf).map((g) => g.cause));
  const ticketOf = (cause: string) => split.tickets.find((t) => t.problem === cause);
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong> problems
</p>
<p class="note">
  The export lists {exported.length} rows for #{ticket}: {exported.length - merged.length} reports come twice and count once.
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      {@const t = ticketOf(g.cause)}
      <tr class:open={!t}>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>
          {#if t}<a href="https://github.com/{split.repo}/issues/{t.issue}">{split.repo}#{t.issue}</a>{:else}no ticket yet{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<p class="note">
  Counted the desk's way: each report under the cause of its tenant and channel. The
  {shifted.reports.length} <code>shift</code> reports are filed under <code>{filedUnder.join(', ')}</code> too, so the
  rows add up to more than {merged.length}.
</p>

<style>
  .sum { font-size: 1.1rem; margin-bottom: 0; }
  .note { color: var(--muted); font-size: 0.85em; margin-top: 0.25rem; }
  .open td { background: var(--warn-bg); }
  .open td:last-child { color: var(--warn); font-weight: 600; }
</style>
