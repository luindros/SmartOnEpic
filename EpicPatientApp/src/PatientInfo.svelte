<script lang="ts">
    import axios from 'axios'; // Import Axios
    import { onMount } from 'svelte';
    import { navigate } from 'svelte-routing';

    export let accessToken: string; // Access token passed as a prop
    export let patientId: string; // Patient ID passed as a prop

    let patientData: any = null; // Variable to hold patient data
    let medications: any[] = []; // Variable to hold medications
    let labReports: any[] = []; // Variable to hold lab reports
    let vitalSigns: any[] = []; // Variable to hold vital signs
    let error: string | null = null; // Variable to hold error messages
    let loading: boolean = true; // Loading state

    // Define interface for FHIR entry
    interface FHIREntry {
        resource: {
            resourceType: string;
            medicationReference?: {
                display: string;
            };
        };
    }

    // Function to fetch patient information
    async function fetchPatientInfo() {
        try {
            const response = await axios.get(`https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/R4/Patient/${patientId}`, {
                headers: {
                    'Authorization': `Bearer ${accessToken}`,
                    'Content-Type': 'application/json',
                },
            });
            patientData = response.data; // Store the patient data
        } catch (err) {
            error = 'Failed to fetch patient information. Please try again later.';
            console.error('Error details:', err.response ? err.response.data : err);
        }
    }

    // Function to fetch medications
    async function fetchMedications() {
        try {
            const response = await axios.get(`https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/R4/MedicationRequest?patient=${patientId}`, {
                headers: {
                    'Authorization': `Bearer ${accessToken}`,
                    'Content-Type': 'application/json',
                },
            });
            response.data.entry.forEach((entry: FHIREntry) => {
                if (entry.resource.resourceType === 'MedicationRequest') {
                    medications.push(entry);
                }
            });
            // medications = response.data.entry.filter(entry => entry.resource.resourceType === 'MedicationRequest') || [];
            //medications = response.data.entry || []; // Store the medications

            // Log the medication names to the console
            
        } catch (err) {
            error = 'Failed to fetch medications.';
            console.error('Error fetching medications:', err); // Log the error details
        }
    }

    // Function to fetch lab reports
    async function fetchLabReports() {
        try {
            const response = await axios.get(`https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/R4/Observation/?category=laboratory&patient=${patientId}`, {
                headers: {
                    'Authorization': `Bearer ${accessToken}`,
                    'Content-Type': 'application/json',
                },
            });
            response.data.entry.forEach((entry: FHIREntry) => {
                if (entry.resource.resourceType === 'Observation') {
                    labReports.push(entry);
                }
            });
            // Store the lab reports
        } catch (err) {
            console.error('Error fetching lab reports:', err);
        }
    }

    // Function to fetch vital signs for the patient
    async function fetchVitalSigns() {
        try {
            const response = await axios.get(`https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/R4/Observation?category=vital-signs&patient=${patientId}`, {
                headers: {
                    'Authorization': `Bearer ${accessToken}`, // Set authorization header with access token
                    'Content-Type': 'application/json', // Set content type to JSON
                },
            });

            response.data.entry.forEach((entry: FHIREntry) => {
                if (entry.resource.resourceType === 'Observation') {
                    vitalSigns.push(entry);
                }
            });
            //vitalSigns = response.data.entry || []; // Store the vital signs from the response

            // Extract systolic and diastolic values for blood pressure
            vitalSigns.forEach((entry) => {
                if (entry.resource.code?.text === "Blood Pressure") {
                    const systolic = entry.resource.component.find(comp => comp.code?.text === "Systolic blood pressure");
                    const diastolic = entry.resource.component.find(comp => comp.code?.text === "Diastolic blood pressure");

                    if (systolic && diastolic) {
                        entry.systolicValue = systolic.valueQuantity.value; // Store systolic value
                        entry.systolicUnit = systolic.valueQuantity.unit; // Store systolic unit
                        entry.diastolicValue = diastolic.valueQuantity.value; // Store diastolic value
                        entry.diastolicUnit = diastolic.valueQuantity.unit; // Store diastolic unit
                    }
                }
            });
        } catch (err) {
            console.error('Error fetching vital signs:', err); // Log error details
        }
    }

    // Fetch all data on component mount
    onMount(async () => {
        await Promise.all([
            fetchPatientInfo(),
            fetchMedications(),
            fetchLabReports(),
            fetchVitalSigns()
        ]);
        loading = false; // Set loading to false after all requests complete
    });

    // Function to extract the EPIC identifier
    function getEpicIdentifier() {
        if (patientData && patientData.identifier) {
            const epicIdentifier = patientData.identifier.find(id => id.type?.text === "EPIC");
            return epicIdentifier ? epicIdentifier.value : 'N/A';
        }
        return 'N/A';
    }

    // Function to handle sign out
    function signOut() {
        localStorage.removeItem('access_token'); // Remove access token from localStorage
        localStorage.removeItem('patient'); // Remove patient ID from localStorage
        navigate('/'); // Navigate back to the sign-in page
    }
</script>

<style>
    .container {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
        border-radius: 8px;
        background-color: #f9f9f9;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        text-align: left; /* Align text to the left for a clinical feel */
    }
    h1 {
        color: #007bff;
        margin-bottom: 20px;
    }
    h2 {
        color: #333;
        margin-top: 20px;
        border-bottom: 2px solid #007bff;
        padding-bottom: 5px;
    }
    table {
        width: 100%;
        border-collapse: collapse;
        margin: 10px 0;
    }
    th, td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: left;
    }
    th {
        background-color: #007bff;
        color: white;
    }
    tr:nth-child(even) {
        background-color: #f2f2f2;
    }
    .error {
        color: red;
        font-weight: bold;
    }
    .loading {
        font-style: italic;
        color: #666;
    }
    .info {
        margin-bottom: 20px;
    }
    .sign-out-button {
        position: absolute; /* Position the button absolutely */
        top: 20px; /* Distance from the top */
        right: 20px; /* Distance from the right */
        padding: 10px 15px; /* Padding for the button */
        background-color: #dc3545; /* Red background color */
        color: white; /* White text color */
        border: none; /* No border */
        border-radius: 5px; /* Rounded corners */
        cursor: pointer; /* Pointer cursor on hover */
        font-size: 16px; /* Font size */
    }

    .sign-out-button:hover {
        background-color: #c82333; /* Darker red on hover */
    }
</style>

<div class="container">
    <button class="sign-out-button" on:click={signOut}>Sign Out</button>
    
    {#if loading}
        <p class="loading">Loading patient information...</p>
    {:else if error}
        <p class="error">{error}</p>
    {:else if patientData}
        <h1>Patient Information</h1>
        <div class="info">
            <p><strong>Name:</strong> {patientData.name[0]?.text || 'N/A'}</p>
            <p><strong>Gender:</strong> {patientData.gender || 'N/A'}</p>
            <p><strong>Date of Birth:</strong> {patientData.birthDate || 'N/A'}</p>
            <p><strong>EPIC Identifier:</strong> {getEpicIdentifier()}</p>
        </div>

        <!-- Medications Section -->
        <h2>Medications</h2>
        {#if medications.length > 0}
            <table>
                <thead>
                    <tr>
                        <th>Medication</th>
                    </tr>
                </thead>
                <tbody>
                    {#each medications as medication}
                        <tr>
                            <td>{medication.resource.medicationReference?.display || 'N/A'}</td>
                        </tr>
                    {/each}
                </tbody>
            </table>
        {:else}
            <p>No medications found.</p>
        {/if}

        <!-- Lab Reports Section -->
        <h2>Lab Reports</h2>
        {#if labReports.length > 0}
            <table>
                <thead>
                    <tr>
                        <th>Lab Report</th>
                    </tr>
                </thead>
                <tbody>
                    {#each labReports as report}
                        <tr>
                            <td>{report.resource.code?.text || 'N/A'}</td>
                        </tr>
                    {/each}
                </tbody>
            </table>
        {:else}
            <p>No lab reports found.</p>
        {/if}

        <!-- Vital Signs Section -->
        <h2>Vital Signs</h2>
        {#if vitalSigns.length > 0}
            <table>
                <thead>
                    <tr>
                        <th>Vital Sign</th>
                        <th>Value</th>
                    </tr>
                </thead>
                <tbody>
                    {#each vitalSigns as vitalSign}
                        <tr>
                            <td>{vitalSign.resource.code?.text || 'N/A'}</td>
                            <td>
                                {#if vitalSign.resource.code?.text === "Blood Pressure"}
                                    {vitalSign.systolicValue || 'N/A'} {vitalSign.systolicUnit || ''} / 
                                    {vitalSign.diastolicValue || 'N/A'} {vitalSign.diastolicUnit || ''}
                                {:else}
                                    {vitalSign.resource.valueQuantity?.value || 'N/A'} {vitalSign.resource.valueQuantity?.unit || ''}
                                {/if}
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>
        {:else}
            <p>No vital signs found.</p>
        {/if}
    {:else}
        <p>No patient information available.</p>
    {/if}
</div>