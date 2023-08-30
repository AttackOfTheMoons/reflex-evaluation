<script lang="ts">
  export let overallFailCount: number;
</script>

<div class="broken-heart">
  {#each Array(overallFailCount) as _}
    <span>💔</span>
  {/each}
</div>

<style>
  .broken-heart {
    font-size: 24px;
    color: red;
  }
</style>
