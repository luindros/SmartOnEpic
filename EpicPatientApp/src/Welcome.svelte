<script lang="ts">
    export let patientData: any; // Prop to receive patient data

    // Function to extract the EPIC identifier
    function getEpicIdentifier() {
        if (patientData && patientData.identifier) {
            const epicIdentifier = patientData.identifier.find(id => id.type?.text === "EPIC");
            return epicIdentifier ? epicIdentifier.value : 'N/A';
        }
        return 'N/A';
    }
</script>

<style>
    .welcome-container {
        max-width: 600px;
        margin: 0 auto;
        padding: 20px;
        border-radius: 8px;
        background-color: #f9f9f9;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        text-align: center; /* Center text for a welcoming feel */
    }
    h1 {
        color: #007bff;
        margin-bottom: 20px;
    }
    .info {
        background: white;
        border-radius: 8px;
        padding: 20px;
        margin: 10px 0;
        box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1);
        font-size: 1.2em;
    }
</style>

<div class="welcome-container">
    <h1>Welcome!</h1>
    {#if patientData}
        <div class="info">
            <p><strong>Name:</strong> {patientData.name[0]?.text || 'N/A'}</p>
            <p><strong>Gender:</strong> {patientData.gender || 'N/A'}</p>
            <p><strong>Date of Birth:</strong> {patientData.birthDate || 'N/A'}</p>
            <p><strong>EPIC Identifier:</strong> {getEpicIdentifier()}</p> <!-- Display EPIC Identifier -->
        </div>
    {:else}
        <p>No patient information available.</p>
    {/if}
</div>