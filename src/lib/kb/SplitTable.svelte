<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));

  // The desk read five parser reports as a one-day move (desk-notes.json). They are a third problem.
  const isMove = (r: Report) => notes.oneDayMove.includes(r.id);
  const span = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return `${days[0]} → ${days[days.length - 1]}`;
  };

  const rows = $derived.by(() => {
    const parser = groups.find((g) => g.cause === 'parser')?.reports ?? [];
    const hint = groups.find((g) => g.cause === 'hint')?.reports ?? [];
    const sets = [
      { problem: 'parser reads mm/dd for day-first tenants', cause: 'parser', rs: parser.filter((r) => !isMove(r)) },
      { problem: 'booking-v3 form hint says MM/DD/YYYY', cause: 'hint', rs: hint },
      { problem: 'a booking moves by one day', cause: 'one-day move', rs: parser.filter(isMove) },
    ];
    return sets.map((s) => ({ ...s, tenants: new Set(s.rs.map((r) => r.tenant)).size }));
  });
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>cause</th><th>reports</th><th>tenants</th><th>seen</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.cause)}
      <tr>
        <td>{row.problem}</td>
        <td><code>{row.cause}</code></td>
        <td>{row.rs.length}</td>
        <td>{row.tenants}</td>
        <td>{span(row.rs)}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
