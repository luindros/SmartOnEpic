<script lang="ts">
  import { Router, Route, navigate } from "svelte-routing";
  import SignIn from './SingIn.svelte'; // Ensure the path is correct
  import PatientInfo from './PatientInfo.svelte'; // Import the PatientInfo component

  export let url = "";

  function handleLogoClick() {
      navigate("/");
  }

  // Function to get access token and patient ID from localStorage
  function getAuthData() {
    const accessToken = localStorage.getItem('access_token') || ""; // Provide a default empty string
    const patientId = localStorage.getItem('patient');
    return { accessToken, patientId }; // Return the object directly
  }
</script>

<Router {url}>
  <header>
      <nav>
          <!-- Add navigation links if needed -->
      </nav>
  </header>

  <main>
      <div class="content-container">
          <Route path="/" component={SignIn} />
          <Route path="/patient-info" let:params>
              {#if getAuthData().accessToken && getAuthData().patientId}
                  <PatientInfo 
                      accessToken={getAuthData().accessToken} 
                      patientId={getAuthData().patientId} 
                  />
              {:else}
                  <p>Please sign in to view patient information.</p>
              {/if}
          </Route>
      </div>
  </main>
</Router>

<style>
  header {
      display: flex;
      flex-direction: column;
      align-items: center;
  }
  main {
      padding: 20px;
  }
</style>