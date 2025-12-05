<!--
  - This file is part of LiquidBounce (https://github.com/CCBlueX/LiquidBounce)
  -
  - Copyright (c) 2015 - 2025 CCBlueX
  -
  - LiquidBounce is free software: you can redistribute it and/or modify
  - it under the terms of the GNU General Public License as published by
  - the Free Software Foundation, either version 3 of the License, or
  - (at your option) any later version.
  -
  - LiquidBounce is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU General Public License for more details.
  -
  - You should have received a copy of the GNU General Public License
  - along with LiquidBounce. If not, see <https://www.gnu.org/licenses/>.
  -->

<script lang="ts">
    import {onDestroy, onMount} from "svelte";
    import type { Module, InputBind } from "../../../integration/types";
    import { getModules, getPrintableKeyName } from "../../../integration/rest";
    import { listen } from "../../../integration/ws";
    import { convertToSpacedString, spaceSeperatedNames } from "../../../theme/theme_config";
    import { fly } from "svelte/transition";
    import { flip } from "svelte/animate";
    import {accentColorStore} from "../../../theme/accentColorStore";
    import {hexToRgbString} from "../../../integration/util";
    import {getClickGuiColor} from "../../../integration/persistent_storage";

    let modulesWithBinds: Array<{
        module: Module;
        keyName: string;
        displayName: string;
    }> = [];

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

    async function updateModulesWithBinds() {
        try {
            const modules = await getModules();

            const filtered = modules.filter(m => {
                return m.keyBind &&
                    m.keyBind.boundKey &&
                    m.keyBind.boundKey !== "key.keyboard.unknown";
            });

            const modulesWithKeys = await Promise.all(
                filtered.map(async (module) => {
                    try {
                        const printableKey = await getPrintableKeyName(module.keyBind.boundKey);
                        const displayName = $spaceSeperatedNames ? convertToSpacedString(module.name) : module.name;

                        return {
                            module,
                            keyName: printableKey.localized,
                            displayName
                        };
                    }

                    catch (error) {
                        const displayName = $spaceSeperatedNames ? convertToSpacedString(module.name) : module.name;

                        let fallbackKey = module.keyBind.boundKey;
                        if (fallbackKey.startsWith("key.keyboard.")) {
                            fallbackKey = fallbackKey.replace("key.keyboard.", "").toUpperCase();
                        } else if (fallbackKey.startsWith("key.mouse.")) {
                            fallbackKey = fallbackKey.replace("key.mouse.", "Mouse ").toUpperCase();
                        }

                        return {
                            module,
                            keyName: fallbackKey,
                            displayName
                        };
                    }
                })
            );

            modulesWithBinds = modulesWithKeys;
        }

        catch (error) {
            modulesWithBinds = [];
        }
    }

    spaceSeperatedNames.subscribe(updateModulesWithBinds);
    onMount(updateModulesWithBinds);
    listen("moduleToggle", updateModulesWithBinds);
    listen("valueChanged", async (event) => {
        if (event?.value?.name === "Bind") {
            await updateModulesWithBinds();
        }
    });
</script>

<div class="keybinds-panel" transition:fly={{ y: -10, duration: 250 }}>
    <div class="header">
        <span class="title">KeyBinds</span>
        <img class="icon" src="img/hud/keybinds/icon-keybinds.svg" alt="keybinds" />
    </div>
    <div class="entries">
        {#each modulesWithBinds as { module, keyName, displayName } (module.name)}
            <div
                    class="row"
                    class:enabled={module.enabled}
                    animate:flip={{ duration: 250 }}
            >
                <span class="module-name">{displayName}</span>
                <span class="key-tile-btn" class:muted={!module.enabled}>{keyName}</span>
            </div>
        {:else}
            <div class="no-binds">No active keybinds</div>
        {/each}
    </div>
</div>

<style lang="scss">
  @use "../../../colors.scss" as *;

  .keybinds-panel {
    font-family: MyCustomFont;
    width: max-content;
    border-radius: 7.5px;
    overflow: hidden;
    font-size: 14px;
    min-width: 160px;
    max-width: 220px;
    background: rgba($keybinds-base-color, 0.35);
    border: 1px solid rgba(120, 130, 150, 0.15);
    box-shadow: 0 6px 14px rgba($keybinds-base-color, 0.5);
    animation: fadeIn 0.3s ease-out;
  }

  .header {
    background: rgba($keybinds-base-color, 0.55);
    padding: 8px 12px;
    display: flex;
    justify-content: space-between;
    align-items: center;

    .title {
      color: rgba(var(--accent-color), 1);
      text-shadow: 0 0 10px rgba(var(--accent-color), 0.75);
      font-weight: 700;
      font-size: 15px;
      letter-spacing: 0.4px;
    }

    .icon {
      width: 18px;
      height: 18px;
      opacity: 1;
    }
  }

  .entries {
    background: rgba($keybinds-base-color, 0.25);
    padding: 8px 10px;

    .no-binds {
      color: $keybinds-text-dimmed-color;
      font-size: 13px;
      font-weight: 600;
      text-align: center;
      opacity: 0.8;
      padding: 6px;
    }
  }

  .row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 6px;
    gap: 8px;
    min-width: 0;
    transition: all 0.25s ease;

    &:last-child {
      margin-bottom: 0;
    }

    &.enabled {
      .module-name {
        color: #fff;
        font-weight: 600;
        text-shadow: 0 0 8px rgba(var(--accent-color), 0.7);
      }
    }

    .module-name {
      color: $keybinds-text-dimmed-color;
      font-size: 14px;
      flex: 1;
      min-width: 0;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      transition: color 0.25s ease;
    }

    .key-tile-btn {
      font-size: 12px;
      color: white;
      background: linear-gradient(145deg, rgba(var(--accent-color), 0.8), rgba(var(--accent-color), 0.55));
      border: 1px solid rgba(var(--accent-color), 0.4);
      border-radius: 6px;
      padding: 3px 7px;
      font-weight: 700;
      min-width: 32px;
      text-align: center;
      transition: all 0.25s ease;
      box-shadow: 0 0 10px rgba(var(--accent-color), 0.35);

      &.muted {
        background: rgba($keybinds-base-color, 0.4);
        border: 1px solid rgba(150, 150, 150, 0.15);
        box-shadow: none;
        color: #bbb;
      }
    }
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }
</style>
