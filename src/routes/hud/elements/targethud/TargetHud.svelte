<script lang="ts">
    import ArmorStatus from "./ArmorStatus.svelte";
    import { listen } from "../../../../integration/ws.js";
    import type {ItemStack, PlayerData, Vec3} from "../../../../integration/types";
    import { REST_BASE } from "../../../../integration/host";
    import { fly } from "svelte/transition";
    import HealthProgress from "./HealthProgress.svelte";
    import type { TargetChangeEvent } from "../../../../integration/events";
    import { accentColorStore } from "../../../../theme/accentColorStore";
    import { hexToRgbString } from "../../../../integration/util";
    import { onDestroy, onMount } from "svelte";
    import { getClickGuiColor } from "../../../../integration/persistent_storage";

    let target: PlayerData | null = null;
    let visible = true;
    let hideTimeout: number;
    let playerData: PlayerData;

    function startHideTimeout() {
        hideTimeout = setTimeout(() => (visible = false), 800);
    }

    listen("targetChange", (data: TargetChangeEvent) => {
        target = data.target;
        visible = true;
        clearTimeout(hideTimeout);
        startHideTimeout();
    });

    listen("clientPlayerData", (event: any) => {
        playerData = event.playerData;
    });

    function getHealthStatus(): { letter: string; color: string } {
        if (playerData && target) {
            const playerHealth = playerData.actualHealth + playerData.absorption;
            const targetHealth = target.actualHealth + target.absorption;
            const letter = playerHealth > targetHealth ? "W" : "L";
            const color = letter === "W" ? "#19ff7d" : "#ff323c";
            return { letter, color };
        }
        return { letter: "", color: "" };
    }

    function getDurability(stack: ItemStack): number {
        if (!stack?.maxDamage) return 100;
        return Math.round(((stack.maxDamage - stack.damage) / stack.maxDamage) * 100);
    }

    startHideTimeout();

    let accentColor = "dodgerblue";
    const unsubscribeAccent = accentColorStore.subscribe((color: string) => {
        accentColor = color;
        document.documentElement.style.setProperty(
            "--accent-color",
            hexToRgbString(accentColor)
        );
    });

    onMount(async () => {
        accentColorStore.set(getClickGuiColor() ?? "#1e90ff");
    });

    onDestroy(() => {
        unsubscribeAccent();
    });
</script>

{#if visible && target != null}
    <div class="targethud" transition:fly={{ y: -12, duration: 250 }}>
        <div class="avatar" style="border-color: {getHealthStatus().color}">
            <img src="{REST_BASE}/api/v1/client/resource/skin?uuid={target.uuid}" alt="avatar" />
        </div>
        <div class="info">
            <div class="name-status">
                <span class="name">{target.username}</span>
                <span class="hp-top">
                    {((target.actualHealth + target.absorption) / (target.maxHealth + target.absorption) * 20).toFixed(1)}
                    <span class="heart">♥</span>
                </span>
            </div>
            <HealthProgress
                    maxHealth={target.maxHealth + target.absorption}
                    health={target.actualHealth + target.absorption}/>
            <div class="stats-line">
                {#if target.mainHandStack}
                    <div class="hand-slot">
                        <img class="hand-icon" src="{REST_BASE}/api/v1/client/resource/itemTexture?id={target.mainHandStack.identifier}" alt={target.mainHandStack.identifier}/>
                        {#if target.mainHandStack?.maxDamage && getDurability(target.mainHandStack) < 100}
                            <div class="durability-bar">
                                <div class="durability-fill" style="width: {getDurability(target.mainHandStack)}%"></div>
                            </div>
                        {:else if target.mainHandStack?.count && target.mainHandStack.count > 1}
                            <div class="stack-count">{target.mainHandStack.count}</div>
                        {/if}
                    </div>
                {/if}
                {#each [...target.armorItems].reverse() as item}
                    {#if item}
                        <div class="armor-slot">
                            <img class="armor-icon" src="{REST_BASE}/api/v1/client/resource/itemTexture?id={item.identifier}" alt={item.identifier}/>
                            {#if item?.maxDamage && getDurability(item) < 100}
                                <div class="durability-bar">
                                    <div class="durability-fill" style="width: {getDurability(item)}%"></div>
                                </div>
                            {/if}
                        </div>
                    {/if}
                {/each}
            </div>
        </div>
    </div>
{/if}


<style lang="scss">
  @use "../../../../colors.scss" as *;

  .targethud {
    display: flex;
    align-items: center;
    gap: 5px;
    width: 280px;
    padding: 8px;
    border-radius: 7.5px;
    background: rgba($targethud-base-color, 0.45);
    border: 1px solid rgba(150, 150, 150, 0.15);
    box-shadow: 0 6px 14px rgba($targethud-base-color, 0.5);
  }

  .avatar {
    height: 50px;
    width: 50px;
    position: relative;
    border-radius: 7.5px;
    overflow: hidden;
    image-rendering: pixelated;
    border: 2px solid transparent;

    img {
      position: absolute;
      scale: 6.25;
      left: 116px;
      top: 116px;
    }
  }

  .info {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .name-status {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 16px;
    font-weight: 700;
    color: white;

    .hp-top {
      font-size: 14px;
      font-weight: 700;
      color: white;
      display: flex;
      align-items: center;
      gap: 2px;
    }

    .heart {
      color: #ff4b4b;
    }
  }

  .stats-line {
    display: flex;
    align-items: center;
    gap: 6px;
    font-size: 13px;
    color: #eaeaea;
  }

  .armor-slot,
  .hand-slot {
    position: relative;
    width: 24px;
    height: 24px;
    background: rgba(255, 255, 255, 0.08);
    border-radius: 5px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .hand-slot {
    .stack-count {
      position: absolute;
      bottom: -2px;
      right: 2px;
      font-size: 12px;
      font-weight: 600;
      color: white;
      text-shadow: 0 0 3px rgba(0, 0, 0, 0.8);
      pointer-events: none;
    }
  }

  .armor-icon,
  .hand-icon {
    width: 22px;
    height: 22px;
    image-rendering: pixelated;
  }

  .durability-bar {
    position: absolute;
    bottom: -5px;
    left: 2px;
    right: 2px;
    height: 4px;
    background: rgba(0, 0, 0, 0.4);
    border-radius: 4px;
    overflow: hidden;
  }

  .durability-fill {
    height: 100%;
    background: rgba(var(--accent-color), 0.9);
    transition: width 0.2s ease;
  }
</style>
