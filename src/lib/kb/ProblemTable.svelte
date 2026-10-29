<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem merged into the ticket. Reports the desk read as a one-day move
  // (desk-notes.json) are not a day/month swap, so they get their own row instead of a footnote.
  let { ticket, issues = {} }: { ticket: number; issues?: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const shifted = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rows = $derived([
    ...groupByCause(merged.filter((r) => !notes.oneDayMove.includes(r.id)), tenantOf).map((g) => ({
      problem: g.cause,
      reports: g.reports.length,
      tenants: g.tenants,
      seen: `${g.firstSeen} → ${g.lastSeen}`,
    })),
    ...(shifted.length
      ? [{
          problem: 'one-day-shift',
          reports: shifted.length,
          tenants: new Set(shifted.map((r) => r.tenant)).size,
          seen: `${shifted.map((r) => r.receivedAt.slice(0, 10)).sort()[0]} → ${shifted.map((r) => r.receivedAt.slice(0, 10)).sort().at(-1)}`,
        }]
      : []),
  ]);
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong> problems
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr></thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr>
        <td><code>{r.problem}</code></td>
        <td>{r.reports}</td>
        <td>{r.tenants}</td>
        <td>{r.seen}</td>
        <td>{#if issues[r.problem]}<a href={issues[r.problem]}>#{issues[r.problem].split('/').at(-1)}</a>{:else}—{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
