<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // Three problems: the two causes from causes.ts, plus the reports the desk read as a one-day move.
  type Problem = 'parser' | 'hint' | 'oneDay';
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const problemOf = (r: Report): Problem =>
    notes.oneDayMove.includes(r.id) ? 'oneDay' : (causeOf(r, tenantOf(r)) as Problem);
  const rows = $derived(
    (['parser', 'hint', 'oneDay'] as const).map((p) => {
      const rs = merged.filter((r) => problemOf(r) === p);
      return { problem: p, count: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size };
    }),
  );
</script>

<p class="sum"><strong>{merged.length}</strong> reports, <strong>{rows.length}</strong> problems</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th></tr></thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr><td><code>{row.problem}</code></td><td>{row.count}</td><td>{row.tenants}</td></tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
