<script>
  import PocketBase from "pocketbase"
  import { status, username, user, admin, pageResult, verified } from "../store"
  const client = new PocketBase("http://127.0.0.1:8090")
  let state = { email: "", password: "" }
  let pbError

  $: disabled = !state.email || !state.password

  async function userSignup() {
    try {
      //await client.admins.authViaEmail("jean.moirano@gmail.com", "1234512345")
      const user = await client.users.create({
        email: state.email,
        password: state.password,
        passwordConfirm: state.password
      })

      if (user) {
        sendemail()
        state.email = ""
        state.password = ""
      }
    } catch (error) {
      pbError = error.data
      state.email = ""
      state.password = ""
    }
  }

  async function sendemail() {
    await client.users.requestVerification(state.email)
  }

  async function deleteUser() {
    await client.admins.authViaEmail("jean.moirano@gmail.com", "1234512345")
    await client.admins.delete("w8e6wol7z6gvn0c")
  }
  //deleteUser()

  async function userLogin() {
    try {
      const authData = await client.users.authViaEmail(
        state.email,
        state.password
      )
      if (authData) {
        $username = authData?.user.email.split("@")[0].toUpperCase()
        $user = authData
        state.email = ""
        state.password = ""
        $status = true
        pbError = null
        $verified = true
      }
    } catch (error) {
      pbError = error.data
      state.email = ""
      state.password = ""
    }
  }
  async function adminLogin() {
    try {
      const authData = await client.admins.authViaEmail(
        state.email,
        state.password
      )
      if (authData) {
        $username = authData?.admin.email.split("@")[0].toUpperCase()
        $admin = authData
        state.email = ""
        state.password = ""
        $status = true
        pbError = null
        $verified = undefined
      }
    } catch (error) {
      pbError = error.data
      state.email = ""
      state.password = ""
    }
  }

  async function handleLogout() {
    client.authStore.clear()
    $status = false
    $username = ""
    $pageResult = []
  }
</script>

{#if pbError}
  <span class="error"> {pbError.code} <small>{pbError.message}</small> </span>
{/if}

<div class="cont">
  {#if $username === ""}
    <label for="email">Email</label>
    <input type="email" bind:value={state.email} required />
    <label for="password">Password</label>
    <input type="password" bind:value={state.password} required />
    <button on:click={userLogin} {disabled}>User Login</button>
    <button on:click={adminLogin} {disabled}>Admin Login</button>
    <button on:click={userSignup} {disabled}>User Signup</button>
  {/if}
  {#if $status}
    <button on:click={handleLogout}><small>Logout</small></button>
  {/if}
</div>

<style>
  .cont {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    border: 2px solid goldenrod;
    border-radius: 0.5rem;
    padding: 1rem;
    margin-bottom: 1rem;
    margin-top: 1rem;
  }
  .error {
    background-color: goldenrod;
    color: black;
    padding: 0.5rem;
    border-radius: 0.3rem;
  }
</style>
