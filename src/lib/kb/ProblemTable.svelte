<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem merged into the ticket. Causes are counted the desk's way (CauseTable:
  // by tenant). Reports the desk read as a one-day move (desk-notes.json) are not a day/month
  // swap, so they also get a row of their own instead of only a footnote.
  let { ticket, issues = {}, status = {} }: { ticket: number; issues?: Record<string, string>; status?: Record<string, string> } =
    $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const shifted = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const days = (rs: Report[]) => rs.map((r) => r.receivedAt.slice(0, 10)).sort();
  const rows = $derived([
    ...groupByCause(merged, tenantOf).map((g) => ({ problem: g.cause, reports: g.reports, tenants: g.tenants })),
    ...(shifted.length
      ? [{ problem: 'one-day-shift', reports: shifted, tenants: new Set(shifted.map((r) => r.tenant)).size }]
      : []),
  ]);
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong> problems
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th><th>status</th></tr></thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr>
        <td><code>{r.problem}</code></td>
        <td>{r.reports.length}</td>
        <td>{r.tenants}</td>
        <td>{days(r.reports)[0]} → {days(r.reports).at(-1)}</td>
        <td>{#if issues[r.problem]}<a href={issues[r.problem]}>#{issues[r.problem].split('/').at(-1)}</a>{:else}—{/if}</td>
        <td>{status[r.problem] ?? '—'}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#if shifted.length}
  <p class="note">The {shifted.length} <code>one-day-shift</code> reports are also counted under their tenant's cause, as the desk counts them.</p>
{/if}

<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
