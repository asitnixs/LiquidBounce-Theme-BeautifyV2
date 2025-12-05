<script lang="ts">
    import {createEventDispatcher} from "svelte";
    import RippleLoader from "../RippleLoader.svelte";

    export let image: string;
    export let imageText: string | null = null;
    export let imageTextBackgroundColor: string | null = null;
    export let title: string;
    export let favorite = false;

    const dispatch = createEventDispatcher();

    let previewImageLoaded = false;
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="menu-list-item" on:dblclick={() => dispatch("dblclick")}>
    <div class="image">
        {#if !previewImageLoaded}
            <div class="loader">
                <RippleLoader />
            </div>
        {/if}
        <img class="preview" on:load={() => previewImageLoaded = true} src={image} alt="preview">
        <span class="text" class:visible={imageText !== null && imageTextBackgroundColor !== null}
              style="background-color: {imageTextBackgroundColor};">{imageText}</span>
        {#if favorite}
            <img class="favorite-mark" src="img/menu/icon-favorite-mark.svg" alt="fav">
        {/if}
    </div>
    <div class="title">
        <span class="text">{title}</span>
        <span class="tag-slot">
            <slot name="tag"/>
        </span>
    </div>
    <div class="subtitle">
        <slot name="subtitle"/>
    </div>
    <div class="buttons">
        <div class="active">
            <slot name="active-visible"/>
        </div>

        <slot name="always-visible"/>
    </div>
</div>

<style lang="scss">
  @use "../../../../colors.scss" as *;

  .menu-list-item {
    display: grid;
    grid-template-areas:
        "a b c"
        "a d c";
    grid-template-columns: max-content 1fr max-content;
    background: rgba($menu-base-color, 0.5);
    border: 1px solid rgba(255, 255, 255, 0.15);
    padding: 15px 25px;
    column-gap: 15px;
    border-radius: 7.5px;
    transition: background-color .25s ease, transform .25s ease, box-shadow .25s ease;
    align-items: center;
    cursor: grab;
    position: relative;
    font-family: MyCustomFont;
    font-weight: 600;

    &:hover {
      background: rgba($menu-base-color, 0.9);
      box-shadow: 0 6px 16px rgba($accent-color, .5);
      border-color: $accent-color;

      .subtitle {
        color: $menu-text-color;
      }

      .buttons .active {
        opacity: 1;
      }
    }

    &:active {
      transform: scale(.98);
    }
  }

  .image {
    grid-area: a;
    position: relative;

    .loader {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }

    .preview {
      height: 70px;
      width: 70px;
      border-radius: 50%;
      object-fit: cover;
      filter: drop-shadow(0 2px 8px rgba($accent-color, 0.5));
      transition: transform .25s ease;
    }

    .menu-list-item:hover & .preview {
    }

    .favorite-mark {
      position: absolute;
      top: -8px;
      right: -8px;
      height: 24px;
      width: 24px;
      filter: drop-shadow(0 0 6px rgba($accent-color, .7));
    }

    .text {
      position: absolute;
      bottom: -8px;
      right: -8px;
      display: none;
      color: $menu-text-color;
      font-size: 12px;
      padding: 4px 12px;
      border-radius: 7.5px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.2);

      &.visible {
        display: block;
      }
    }
  }

  .title {
    grid-area: b;
    align-self: flex-end;
    display: flex;
    align-items: center;

    .text {
      font-size: 20px;
      color: $menu-text-color;
      font-weight: 600;
      letter-spacing: .5px;
      transition: color .25s ease;
    }

    .tag-slot {
      margin-left: 10px;
      display: inline-flex;
      align-items: center;
      gap: 4px;
    }
  }

  .subtitle {
    grid-area: d;
    font-size: 16px;
    color: $menu-text-dimmed-color;
    transition: color .25s ease;
    align-self: flex-start;
  }

  .buttons {
    grid-area: c;
    display: flex;
    align-items: center;

    .active {
      margin-right: 20px;
      opacity: 0;
      transition: opacity .25s ease;
    }
  }
</style>
