<script lang="ts">
    import {listen} from "../../../integration/ws";
    import type {ClientPlayerDataEvent} from "../../../integration/events";
    import type {StatusEffect} from "../../../integration/types";
    import {accentColorStore} from "../../../theme/accentColorStore";
    import {hexToRgbString} from "../../../integration/util";
    import {onDestroy, onMount} from "svelte";
    import {getClickGuiColor} from "../../../integration/persistent_storage";

    let effects: StatusEffect[] = [];

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

    listen("clientPlayerData", (event: ClientPlayerDataEvent) => {
        effects = event.playerData.effects;
    });

    function formatTime(duration: number): string {
        if (duration < 0) {
            return "∞";
        }

        const totalSeconds = Math.floor(duration / 20);
        const minutes = Math.floor(totalSeconds / 60);
        const seconds = totalSeconds % 60;

        return `${minutes}:${seconds.toString().padStart(2, "0")}`;
    }

    function convertToRoman(n: number): string {
        switch (n) {
            case 0: return "1";
            case 1: return "2";
            case 2: return "3";
            case 3: return "4";
            case 4: return "5";
            case 5: return "6";
            case 6: return "7";
            case 7: return "8";
            case 8: return "9";
            case 9: return "10";
            default: return "10+";
        }
    }

</script>

<div class="effects">
    <div class="header">
        <span class="title">Active Effects</span>
        <svg class="header-icon" xmlns="http://www.w3.org/2000/svg" height="18" width="18" viewBox="0 0 640 640">
            <path fill="#ffffff" d="M384 64L224 64C206.3 64 192 78.3 192 96C192 113.7 206.3 128 224 128L224 279.5L103.5 490.3C98.6 499 96 508.7 96 518.7C96 550.4 121.6 576 153.3 576L486.7 576C518.3 576 544 550.4 544 518.7C544 508.7 541.4 498.9 536.5 490.3L416 279.5L416 128C433.7 128 448 113.7 448 96C448 78.3 433.7 64 416 64L384 64zM288 279.5L288 128L352 128L352 279.5C352 290.6 354.9 301.6 360.4 311.3L402 384L238 384L279.6 311.3C285.1 301.6 288 290.7 288 279.5z"/>
        </svg>
    </div>
    {#if effects.length > 0}
        {#each effects as e}
            <div class="effect">
                <img
                        class="effect-icon"
                        src={"img/hud/effects/icon-" + e.effect.replace("minecraft:", "") + ".png"}
                        alt={`${e.localizedName} icon`}
                />
                <div class="text">
                    <span class="name" style="color: {'#' + e.color.toString(16)}">
                        {e.localizedName}
                            {#if e.amplifier > 0}
                        {e.amplifier + 1}
                      {/if}
                    </span>
                </div>
                <span class="duration {(e.duration > 0 && e.duration / 20 <= 4) ? 'blinking' : ''}">{formatTime(e.duration)}</span>
            </div>
        {/each}
    {:else}
        <div class="no-effects">No Active Effects</div>
    {/if}
</div>

<style lang="scss">
  @use "../../../colors.scss" as *;

  .effects {
    background: rgba($effects-base-color, 0.35);
    box-shadow: 0 6px 14px rgba($effects-base-color, 0.5);
    border: 1px solid rgba(120, 130, 150, 0.15);
    border-radius: 7.5px;
    font-family: MyCustomFont;
    font-size: 14px;
    font-weight: 600;
    width: max-content;
    min-width: 180px;
    max-width: 250px;
    overflow: hidden;
    animation: fadeIn 0.3s ease-out;
  }

  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: rgba($effects-base-color, 0.55);
    padding: 8px 12px;

    .title {
      font-size: 15px;
      font-weight: 700;
      color: rgba(var(--accent-color), 1);
      text-shadow: 0 0 10px rgba(var(--accent-color), 0.75);
      letter-spacing: 0.4px;
    }

    .header-icon {
      width: 18px;
      height: 18px;
      opacity: 1;
    }
  }

  .effect {
    display: flex;
    align-items: center;
    gap: 6px;
    padding: 6px 10px;
    transition: background 0.25s ease;

    .effect-icon {
      width: 18px;
      height: 18px;
    }

    .text {
      display: flex;
      flex-direction: column;

      .name {
        font-size: 14px;
        font-weight: 600;
        text-shadow: 0 0 8px rgba(0, 0, 0, 0.35);
      }
    }

      .duration {
        margin-left: auto;
        font-size: 12px;
        font-weight: 500;
        color: $effects-duration-color;
        background: rgba($effects-base-color, 0.4);
        border: 1px solid rgba(150, 150, 150, 0.15);
        border-radius: 5px;
        padding: 2px 4px;
        text-shadow: 0 0 6px rgba($effects-duration-color, 0.25);
        min-width: 35px;
        width: 45px;
        text-align: center;
        transition: 0.25s ease;
      }

      .blinking {
        animation: blink 0.4s infinite alternate;
      }

      @keyframes blink {
        from {
            opacity: 1;
            color: #ff4d4d;
            border-color: rgba(255, 77, 77, 0.6);
            background: rgba(255, 77, 77, 0.15);
        }
        to {
            opacity: 0.5;
        }
      }
    }

  .no-effects {
    color: $effects-duration-color;
    font-size: 13px;
    font-weight: 600;
    text-align: center;
    padding: 8px;
    opacity: 0.8;
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }
</style>
