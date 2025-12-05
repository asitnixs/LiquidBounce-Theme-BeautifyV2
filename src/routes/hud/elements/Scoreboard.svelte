<script lang="ts">
    import {listen} from "../../../integration/ws";
    import type {PlayerData, Scoreboard} from "../../../integration/types";
    import TextComponent from "../../menu/common/TextComponent.svelte";
    import type {ClientPlayerDataEvent} from "../../../integration/events";
    import {accentColorStore} from "../../../theme/accentColorStore";
    import {onDestroy, onMount} from "svelte";
    import {hexToRgbString} from "../../../integration/util";
    import {getClickGuiColor} from "../../../integration/persistent_storage";

    export let settings: { [name: string]: any };

    const cSettings = settings as HudScoreboardSettings & {
        numbers?: boolean;
        address?: string;
    };

    $: sections = cSettings?.show?.length ? cSettings.show : ["Header", "Name", "Score"];
    $: showHeader = sections.includes("Header");
    $: showName = sections.includes("Name");
    $: canShowScore = sections.includes("Score");
    $: showNumbers = canShowScore && (cSettings?.numbers ?? false);

    let scoreboard: Scoreboard | null = null;

    let accentColor = "dodgerblue";
    const unsubscribeAccent = accentColorStore.subscribe((color: string) => {
        accentColor = color;
        document.documentElement.style.setProperty('--accent-color', hexToRgbString(accentColor));
    });

    onMount(async () => {
        accentColorStore.set(getClickGuiColor() ?? '#1e90ff');
    });

    onDestroy(() => {
        unsubscribeAccent();
    });

    listen("clientPlayerData", (e: ClientPlayerDataEvent) => {
        const playerData: PlayerData = e.playerData;
        scoreboard = playerData.scoreboard;
    });
</script>

{#if scoreboard}
    <div class="scoreboard">
        {#if scoreboard.header && showHeader}
            <div class="header">
                <TextComponent fontSize={14} allowPreformatting={true} textComponent={scoreboard.header}/>
            </div>
        {/if}
        <div class="entries">
            {#each scoreboard.entries as {name, score}, i}
                <div class="row">
                    {#if i === scoreboard.entries.length - 1 && cSettings?.address}
                        <div class="custom-ip">
                            {cSettings.address}
                        </div>
                    {:else if showName}
                        <TextComponent fontSize={14} allowPreformatting={true} textComponent={name}/>
                    {/if}
                    {#if showNumbers}
                        <div class="score-value">
                            <TextComponent fontSize={14} allowPreformatting={true} textComponent={score}/>
                        </div>
                    {/if}
                </div>
            {/each}
        </div>
    </div>
{/if}

<style lang="scss">
  @use "../../../colors.scss" as *;

  .scoreboard {
    width: max-content;
    border-radius: 7.5px;
    overflow: hidden;
    font-size: 14px;
    font-family: MyCustomFont, sans-serif;
    box-shadow: 0 4px 14px rgba(0, 0, 0, 0.45);
    border: 1px solid rgba($scoreboard-base-color, 0.25);
  }

  .entries {
    background: linear-gradient(to bottom right, rgba($scoreboard-base-color, 0.45), rgba($scoreboard-base-color, 0.35));
    padding: 5px 10px;
  }

  .row {
    display: flex;
    column-gap: 15px;
    justify-content: space-between;
    align-items: center;
    padding: 3px 0;
    transition: all 0.15s ease;
  }

  .header {
    text-align: center;
    background: linear-gradient(145deg, rgba($notifications-base-color, 0.8), rgba($notifications-base-color, 0.6));
    padding: 7px 12px;
    font-weight: bold;
    color: white;
    text-shadow: 0 0 8px rgba(var(--accent-color), 0.9);
  }

  .score-value {
    min-width: 30px;
    text-align: right;
  }

  .custom-ip {
    color: rgba(var(--accent-color), 1);
    text-shadow: 0 0 10px rgba(var(--accent-color), 0.9);
    font-weight: 700;
    display: block;
    text-align: center;
    width: 100%;
    letter-spacing: 0.5px;
  }
</style>
