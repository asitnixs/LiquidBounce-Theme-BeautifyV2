<script lang="ts">
    import type { ItemStack } from "../../../../integration/types";
    import { listen } from "../../../../integration/ws";
    import type { ClientPlayerInventoryEvent, PlayerInventory } from "../../../../integration/events";
    import ItemStackView from "./ItemStackView.svelte";
    import { onMount } from "svelte";
    import { getPlayerInventory } from "../../../../integration/rest";

    let stacks: ItemStack[] = [];

    function updateStacks(inventory: PlayerInventory) {
        stacks = inventory.enderChest;
    }

    listen("clientPlayerInventory", (data: ClientPlayerInventoryEvent) => {
        updateStacks(data.inventory);
    });

    onMount(async () => {
        const inventory = await getPlayerInventory();
        updateStacks(inventory);
    });
</script>

<div class="inventory">
    <div class="inventory-grid">
        {#each stacks as stack (stack)}
            <div class="slot">
                <ItemStackView {stack}/>
            </div>
        {/each}
    </div>
</div>

<style lang="scss">
  @use "../../../../colors" as *;

  .inventory {
    padding: 8px;
    background: rgba($inventory-base-color, 0.45);
    border-radius: 7.5px;
    box-shadow: 0 4px 14px rgba($inventory-base-color, 0.5);
    width: max-content;
  }

  .inventory-grid {
    display: grid;
    grid-template-columns: repeat(9, 35px);
    grid-auto-rows: 35px;
    gap: 2px;
  }

  .slot {
    width: 35px;
    height: 35px;
    background: rgba($inventory-base-color, 0.35);
    border: 1px solid rgba(255, 255, 255, 0.15);
    border-radius: 5px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: 0.15s ease;
  }
</style>
