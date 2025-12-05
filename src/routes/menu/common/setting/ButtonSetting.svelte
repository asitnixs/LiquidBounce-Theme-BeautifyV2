<script lang="ts">
    import {createEventDispatcher} from "svelte";
    import CircleLoader from "../CircleLoader.svelte";

    export let title: string;
    export let disabled = false;
    export let secondary = false;
    export let inset = false;
    export let listenForEnter = false;
    export let loading = false;

    const dispatch = createEventDispatcher();

    function handleKeyDown(e: KeyboardEvent) {
        if (!listenForEnter) {
            return;
        }
        if (e.key === "Enter") {
            dispatch("click");
        }
    }
</script>

<svelte:window on:keydown={handleKeyDown}/>
<button class="button-setting" class:inset type="button" on:click={() => dispatch("click")} {disabled} class:secondary>
    {#if loading}
        <CircleLoader/>
    {/if}
    <span class="title-text">{title}</span>
</button>

<style lang="scss">
  @use "sass:color";
  @use "../../../../colors.scss" as *;

  .button-setting {
    position: relative;
    border: none;
    outline: none;
    background: linear-gradient(145deg, color.adjust($accent-color, $lightness: 10%), $accent-color);
    color: $menu-text-color;
    font-family: MyCustomFont;
    padding: 16px 28px;
    border-radius: 8px;
    font-size: 18px;
    font-weight: 600;
    letter-spacing: 0.5px;
    transition: background 0.25s ease, transform 0.15s ease, box-shadow 0.25s ease, opacity 0.25s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    box-shadow: 0 0 10px rgba($accent-color, 0.5);

    .title-text {
      position: relative;
      z-index: 2;
    }

    &:not([disabled]):hover {
      transform: translateY(-2px);
      background: linear-gradient(145deg, color.adjust($accent-color, $lightness: 15%), color.adjust($accent-color, $lightness: -5%));
      box-shadow: 0 0 15px rgba($accent-color, 0.75);
      cursor: pointer;
    }

    &:not([disabled]):active {
      transform: translateY(1px) scale(0.98);
      box-shadow: 0 0 6px rgba($accent-color, 0.6);
    }

    &.secondary {
      background: rgba($menu-base-color, 0.35);
      color: $menu-text-color;
      box-shadow: none;

      &:not([disabled]):hover {
        background: rgba($menu-base-color, 0.5);
        box-shadow: inset 0 0 8px rgba($accent-color, 0.4);
      }
    }

    &.inset {
      margin: 0 30px;
    }

    &[disabled] {
      opacity: 0.5;
      cursor: not-allowed;
      box-shadow: none;
      transform: none;
    }
  }
</style>
