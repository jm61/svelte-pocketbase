<script>
  import Header from "./lib/components/Header.svelte"
  import PocketBase from "pocketbase"
  import {
    status,
    username,
    user,
    admin,
    pageResult,
    verified
  } from "../src/lib/store"
  const client = new PocketBase("http://127.0.0.1:8090")
  $: users = []

  async function moviesList() {
    $pageResult = await client.records.getFullList("movies", 100)
  }

  async function usersList() {
    users = await client.users.getFullList(50, { sort: "-created" })
    console.log(users)
  }

  async function resetList() {
    $pageResult = []
    users = []
  }
</script>

<Header />

<h2>Welcome to PocketBase {$username}!</h2>
{#if $status}
  {#if $verified}
    <img
      class:noImg={!$status}
      src={`http://127.0.0.1:8090/api/files/systemprofiles0/${$user.user?.profile.id}/${$user.user?.profile.avatar}?thumb=100x100`}
      alt="avatar"
    />
  {:else if !$verified}
    <img
      class:noImg={!$status}
      src={`http://127.0.0.1:8090/_/images/avatars/avatar${$admin.admin.avatar}.svg`}
      alt="avatar"
    />
  {/if}
{/if}

<div>
  {#if $status === true}
    <h3>Login successful</h3>
  {/if}
  {#if $status === false}
    <h3>Login please...</h3>
  {/if}
</div>
<hr style="color: goldenrod;" />

{#if $status === true}
  <button class="btn" on:click={moviesList}>Movies List</button>
  <button class="btn" on:click={usersList}>Users List</button>
  <button class="btn" on:click={resetList}>Reset List</button> <br />

  <!-- <pre>{JSON.stringify(pageResult, null, 2)}</pre> -->
  {#each $pageResult as items, id (id)}
    <div>
      <h3>{items.title}</h3>
      <em>{items.year}</em>
      <q>{items.genre}</q>
    </div>
  {/each}

  {#each users as user, id (id)}
    <div>
      <q>{user.email}</q>
      <em>{user.profile.name}</em>
    </div>
  {/each}
{/if}

<style>
  .btn {
    margin-top: 1rem;
  }
  img {
    border-radius: 50%;
  }
  .noImg {
    display: none;
  }
</style>
