<script>
  import PocketBase from "pocketbase"
  const client = new PocketBase("http://127.0.0.1:8090")
  let state = { email: "", password: "" }
  let status = false
  $: username = ""
  console.clear()

  async function handleSignup() {
    const user = await client.users.create({
      email: state.email,
      password: state.password,
      passwordConfirm: state.password
    })

    if (user) {
      sendemail()
    }
  }
  async function sendemail() {
    await client.users.requestVerification(state.email)
  }

  async function deleteUser() {
    await client.users.delete("fgsl6fkgduo8jry")
  }

  async function handleLogin() {
    const authData = await client.users.authViaEmail(
      state.email,
      state.password
    )
    if (authData) {
      console.log("success Login")
      console.log(authData.user.email)
      username = authData?.user.email.split("@")[0].toUpperCase()
      state.email = ""
      state.password = ""
      status = true
    }
  }

  async function handleLogout() {
    client.authStore.clear()
    status = false
    username = ""
  }

  async function moviesList() {
    const pageResult = await client.records.getList("movies", 1, 100)
    console.log(pageResult)
  }
</script>

Welcome to PocketBase <span>{username || ""}</span>
<div>
  {#if status == true}
    <h3>Login successful</h3>
  {/if}
  {#if status == false}
    <h3>Login please...</h3>
  {/if}
</div>
<hr />

<div class="cont">
  <label for="email">Email</label>
  <input type="email" bind:value={state.email} />
  <label for="password">Password</label>
  <input type="password" bind:value={state.password} />
  <button on:click={handleSignup}>Signup</button>
  <button on:click={handleLogin}>Login</button>
  <button on:click={handleLogout}>Logout</button>
  <button on:click={moviesList}>Movies List</button>
</div>

<style>
  .cont {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 1rem;
  }
</style>
