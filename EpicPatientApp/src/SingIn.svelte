<!-- EpicPatientApp/src/lib/SignIn.svelte -->
<script lang="ts">
    import axios from 'axios'; // Import Axios
    import { navigate } from 'svelte-routing'; // Import navigate for routing

    const clientId = '01230135-f64c-42ce-a251-7c9a2ea0f45a'; // Replace with your actual client ID
    const redirectUri = 'http://localhost:5173'; // Your callback URL
    const authUrl = 'https://fhir.epic.com/interconnect-fhir-oauth/oauth2/authorize';
    const accessTokenUrl = 'https://fhir.epic.com/interconnect-fhir-oauth/oauth2/token';

    function connectToEpic() {
        // Construct the authorization URL
        const url = `${authUrl}?response_type=code&client_id=${clientId}&redirect_uri=${encodeURIComponent(redirectUri)}`;
        window.open(url, '_blank'); // Open the Auth URL in a new tab
    }

    async function handleCallback() {
        const urlParams = new URLSearchParams(window.location.search);
        const code = urlParams.get('code');

        if (code) {
            try {
                const response = await axios.post(accessTokenUrl, new URLSearchParams({
                    grant_type: 'authorization_code',
                    code: code,
                    redirect_uri: redirectUri,
                    client_id: clientId,
                }), {
                    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
                });

                console.log('Access Token:', response.data.access_token); // Log access token
                const { access_token, patient } = response.data;

                console.log('Patient ID:', patient); // Log the patient ID

                if (access_token) {
                    localStorage.setItem('access_token', access_token); // Store access token
                    if (patient) {
                        localStorage.setItem('patient', patient); 
                        console.log('Stored Patient ID:', patient); // Store patient info
                    }
                    // Redirect to PatientInfo component with token and patient ID
                    navigate(`/patient-info?token=${access_token}&patientId=${patient}`);
                }
            } catch (error) {
                console.error('Error during token exchange:', error);
            }
        }
    }

    // Check for authorization code on component mount
    if (window.location.search.includes('code=')) {
        handleCallback();
    }
</script>

<button on:click={connectToEpic}>
    Connect to EPIC Account
</button>

<style>
    button {
      padding: 1em;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background-color: #0056b3;
    }
</style>