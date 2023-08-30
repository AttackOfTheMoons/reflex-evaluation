<script lang="ts">
  export let isActive: boolean;
  export let label: string;
</script>

<div class:active={isActive} class="box">
  <span>{label}</span>
</div>

<style>
  .box {
    width: 100px;
    height: 100px;
    background-color: lightgray;
    margin: 10px;
    display: inline-block;
    text-align: center;
    vertical-align: middle;
    line-height: 100px;
    font-size: 18px;
    transition: background-color 0.3s;
    color: black;
  }

  .active {
    background-color: yellow;
  }
</style>
