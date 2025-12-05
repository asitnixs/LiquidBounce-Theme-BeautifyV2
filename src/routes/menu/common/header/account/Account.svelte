<script lang="ts">
    import ToolTip from "../../ToolTip.svelte";
    import {
        directLoginToCrackedAccount,
        getAccounts,
        getSession,
        loginToAccount,
        openScreen,
        randomUsername
    } from "../../../../../integration/rest";
    import {onMount} from "svelte";
    import {listen} from "../../../../../integration/ws";
    import {location} from "svelte-spa-router";
    import {quintOut} from "svelte/easing";
    import {fade, slide} from "svelte/transition";
    import type {Account} from "../../../../../integration/types";
    import Avatar from "./Avatar.svelte";
    import {notification} from "../notification_store";
    import RippleLoader from "../../RippleLoader.svelte";
    import {isLoggingIn} from "../../../altmanager/altmanager_store";

    let username = "";
    let service = "";
    let avatar = "";
    let online = true;

    let expanded = false;
    let accountElement: HTMLElement;
    let headerElement: HTMLElement;

    let searchQuery = "";
    let accounts: Account[] = [];

    $: renderedAccounts = accounts.filter(a => a.username.toLowerCase().includes(searchQuery.toLowerCase()) || searchQuery === "");

    const inAccountManager = $location === "/altmanager";

    async function refreshSession() {
        const session = await getSession();
        username = session.username;
        service = session.service;
        avatar = session.avatar;
        online = session.online;
    }

    async function refreshAccounts() {
        accounts = await getAccounts();
    }

    onMount(async () => {
        await refreshSession();
        await refreshAccounts();
    });

    listen("session", async () => {
        await refreshSession();
    });

    listen("accountManagerRemoval", async () => {
        await refreshAccounts();
    });

    listen("accountManagerAddition", async () => {
        await refreshAccounts();
    });

    function handleWindowClick(e: MouseEvent) {
        if (!accountElement.contains(e.target as Node)) {
            expanded = false;
            searchQuery = "";
        }
    }

    function handleSelectClick(e: MouseEvent) {
        if (!expanded) {
            // Prevent icon buttons from opening quick switcher
            expanded = !(e.target as HTMLElement).classList.contains("icon");
        } else {
            expanded = !headerElement.contains(e.target as Node);
        }

        if (!expanded) {
            searchQuery = "";
        }
    }

    async function login(account: Account) {
        notification.set({
            title: "AltManager",
            message: "Logging in...",
            error: false
        });

        await loginToAccount(account.id);
    }

    async function loginWithRandomUsername() {
        const username = await randomUsername();
        await directLoginToCrackedAccount(username, false);
    }
</script>

<svelte:window on:click={handleWindowClick}/>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="account" class:expanded bind:this={accountElement} on:click={handleSelectClick}>
    <div class="header" bind:this={headerElement}>
        {#if $isLoggingIn}
            <div class="avatar" transition:fade={{ duration: 200 }}>
                <RippleLoader size={68} />
            </div>
        {:else}
            <object data={avatar} type="image/png" class="avatar" aria-label="avatar" in:fade={{ duration: 200, delay: 200 }}>
                <img src="img/steve.png" alt=avatar class="avatar">
            </object>
        {/if}
        <div class="username">{username}</div>
        <div class="account-type">
            {#if online}
                <span class="online">{service}</span>
            {:else}
                <span class="offline">{service}</span>
            {/if}
        </div>
        <div class="buttons">
            <button class="icon-button" type="button" on:click={loginWithRandomUsername}>
                <ToolTip text="Random username"/>

                <img class="icon" src="img/menu/account/icon-random.svg" alt="random username">
            </button>
            <button class="icon-button" disabled={inAccountManager} type="button"
                    on:click={() => openScreen("altmanager")}>
                <ToolTip text="Change account"/>

                <img class="icon" src="img/menu/icon-pen.svg" alt="change account">
            </button>
        </div>
    </div>

    {#if expanded}
        <div class="quick-switcher" transition:fade|global={{ duration: 200, easing: quintOut }}>
            <!-- svelte-ignore a11y_autofocus -->
            <input type="text" autofocus class="account-search" placeholder="Search..." bind:value={searchQuery}>

            {#if accounts.length > 0}
                {#if renderedAccounts.length > 0}
                    <div class="account-list">
                        {#each renderedAccounts as a}
                            <div on:click={() => login(a)} class="account-item"
                                 transition:slide|global={{ duration: 200, easing: quintOut }}
                                 class:active={a.username === username}>
                                <Avatar url={a.avatar}/>
                                <div class="username">{a.username}</div>
                                <div class="type">{a.type}</div>
                            </div>
                        {/each}
                    </div>
                {:else}
                    <div class="placeholder">No results</div>
                {/if}
            {:else}
                <div class="placeholder">Account list is empty</div>
            {/if}
        </div>
    {/if}
</div>

<style lang="scss">
  @use "../../../../../colors" as *;

  .account {
    width: 420px;
    position: relative;
    font-family: MyCustomFont;

    &.expanded {
      .header {
        border-radius: 7.5px 7.5px 7.5px 7.5px;
      }
    }
  }

  .header {
    background: rgba($menu-base-color, 0.75);
    border: 1px solid rgba(255, 255, 255, 0.15);
    padding: 15px;
    border-radius: 7.5px;
    display: grid;
    grid-template-areas:
      "a b c"
      "a d c";
    grid-template-columns: max-content 1fr max-content;
    column-gap: 12px;
    cursor: pointer;
    transition: border-radius .25s ease;

    .avatar {
      height: 60px;
      width: 60px;
      border-radius: 50%;
      grid-area: a;
      border: 2px solid rgba(255, 255, 255, 0.15);
      transition: all 0.25s ease;

      &:hover {
        transform: scale(1.05);
        border-color: $accent-color;
        box-shadow: 0 0 15px rgba($accent-color, 0.65);
      }
    }

    .username {
      font-weight: 700;
      font-size: 18px;
      color: $menu-text-color;
      grid-area: b;
      align-self: flex-end;
      text-shadow: 0 0 6px rgba(0, 0, 0, 0.5);
    }

    .account-type {
      font-weight: 600;
      font-size: 17px;
      grid-area: d;
      align-self: flex-start;

      .online {
        color: $menu-account-premium-color;
        font-weight: 700;
        text-shadow: 0 0 6px rgba($menu-account-premium-color, 0.5);
      }

      .offline {
        color: $menu-text-dimmed-color;
      }
    }

    .buttons {
      grid-area: c;
      display: flex;
      column-gap: 15px;
      align-items: center;

      .icon-button {
        background: rgba($accent-color, 0.05);
        border: 1px solid rgba(255, 255, 255, 0.15);
        border-radius: 7.5px;
        padding: 6px;
        cursor: pointer;
        transition: all 0.2s ease;

        img {
          width: 20px;
          height: 20px;
        }

        &:hover {
          background: rgba($accent-color, 0.25);
          border: 1px solid rgba($accent-color, 0.5);
          box-shadow: 0 0 8px rgba($accent-color, 0.5);
        }

        &:disabled {
          opacity: 0.4;
        }
      }
    }
  }

  .quick-switcher {
    position: absolute;
    z-index: 1000;
    width: 100%;
    border-radius: 7.5px 7.5px 7.5px 7.5px;
    background: rgba($menu-base-color, 0.75);
    border: 1px solid rgba(255, 255, 255, 0.15);
    margin-top: 2px;
    box-shadow: 0 8px 14px rgba(0, 0, 0, 0.5);
    overflow: hidden;
    animation: fadeIn 0.25s ease;

    .placeholder {
      font-weight: 500;
      font-size: 18px;
      color: $menu-text-dimmed-color;
      padding: 20px;
      text-align: center;
    }

    .account-search {
      background: rgba($menu-base-color, 0.5);
      border: none;
      color: $menu-text-color;
      padding: 14px 14px 14px 48px;
      width: 100%;
      font-size: 16px;
      background-size: 18px 18px;
      outline: none;
      transition: all 0.2s ease;

      &:focus {
        background: rgba($menu-base-color, 0.55);
        background-image: url("/img/menu/icon-search.svg");
        background-repeat: no-repeat;
        background-position: 12px center;
      }
    }

    .account-list {
      max-height: 340px;
      overflow-y: auto;

      &::-webkit-scrollbar {
        width: 4px;
      }
      &::-webkit-scrollbar-thumb {
        background: rgba($accent-color, 0.4);
        border-radius: 3px;
      }
    }

    .account-item {
      color: $menu-text-dimmed-color;
      font-size: 18px;
      padding: 14px 18px;
      cursor: pointer;
      display: grid;
      grid-template-areas:
        "a b"
        "a c";
      grid-template-columns: max-content 1fr;
      column-gap: 12px;
      align-items: center;
      transition: all 0.25s ease;

      .username {
        grid-area: b;
        font-weight: 600;
        font-size: 18px;
        transition: color 0.25s ease;
      }

      .type {
        grid-area: c;
        font-size: 14px;
        opacity: 1;
      }

      &:hover {
        background: rgba($accent-color, 0.1);
        color: $menu-text-color;
        border: 1px solid rgba($accent-color, 0.5);
      }

      &.active {
        background: rgba($accent-color, 0.15);
        border-left: 3px solid $accent-color;

        .username {
          color: $accent-color;
        }
      }
    }
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(-5px); }
    to { opacity: 1; transform: translateY(0); }
  }
</style>
