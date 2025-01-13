<script>
  import { onMount } from 'svelte';
  
  let {
    backendInfo,
    error,
    loading,
    reservationNumber = 234,
    reservationResponse,
    logs = []
  } = $props();

  function logMessage(message, type = 'info') {
    const timestamp = new Date().toLocaleTimeString();
    logs = [...logs, `[${timestamp}] ${type}: ${message}`];
  }

  async function fetchBackendInfo() {
    try {
      loading = true;
      error = null;
      logMessage('Attempting to fetch backend info...', 'fetch');
      
      const response = await fetch('http://localhost:8080/backend_info')
        .catch(err => {
          logMessage(`CORS Error: ${err.message}`, 'error');
          logMessage('Access to fetch at http://localhost:8080/backend_info from origin http://localhost:5173 has been blocked by CORS policy', 'error');
          throw err;
        });

      if (!response.ok) {
        logMessage(`HTTP Error: ${response.status} ${response.statusText}`, 'error');
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      backendInfo = await response.json();
      logMessage('Backend info fetched successfully.');
    } catch (err) {
      error = err.message;
      logMessage(`Failed to load resource: ${err.message}`, 'error');
    } finally {
      loading = false;
    }
  }

  async function makeReservation() {
    try {
      logMessage('Attempting to make reservation...', 'fetch');
      
      const response = await fetch('http://localhost:8080/reservation', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ number: reservationNumber })
      }).catch(err => {
        logMessage(`CORS Error: ${err.message}`, 'error');
        logMessage('Access to fetch at http://localhost:8080/reservation from origin http://localhost:5173 has been blocked by CORS policy', 'error');
        throw err;
      });
      
      if (!response.ok) {
        logMessage(`HTTP Error: ${response.status} ${response.statusText}`, 'error');
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      reservationResponse = await response.json();
      logMessage('Reservation made successfully.');
    } catch (err) {
      error = err.message;
      logMessage(`Failed to load resource: ${err.message}`, 'error');
    }
  }
</script>

<main class="container">
  <div class="split-layout">
    <div class="left-panel">
      <h2>Backend Information</h2>
      <button onclick={() => fetchBackendInfo()}>Fetch Backend Info</button>
      
      {#if loading}
        <div class="loader">Loading...</div>
      {/if}

      {#if error}
        <div class="error">
          Error: {error}
        </div>
      {/if}

      {#if backendInfo}
        <div class="info">
          <pre>{JSON.stringify(backendInfo, null, 2)}</pre>
        </div>
      {/if}
    </div>

    <div class="right-panel">
      <h2>Reservation</h2>
      <div class="reservation">
        <input type="number" bind:value={reservationNumber} />
        <button onclick={() => makeReservation()}>Fetch Reservation</button>
      </div>

      {#if reservationResponse}
        <div class="response">
          <pre>{JSON.stringify(reservationResponse, null, 2)}</pre>
        </div>
      {/if}
    </div>
  </div>

  <div class="logs">
    <h2>Console Logs</h2>
    <div class="logs-container">
      {#each logs as log}
        <div class="log-entry" class:error={log.includes('error')}>
          <pre>{log}</pre>
        </div>
      {/each}
    </div>
  </div>
</main>

<style>
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  .split-layout {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 2rem;
    margin-bottom: 2rem;
  }

  .left-panel, .right-panel {
    background-color: #f8f9fa;
    padding: 1.5rem;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  h2 {
    margin-top: 0;
    margin-bottom: 1rem;
    color: #333;
  }

  .loader {
    color: #666;
    text-align: center;
    padding: 1rem;
  }

  .error {
    color: #721c24;
    background-color: #f8d7da;
    border: 1px solid #f5c6cb;
    padding: 1rem;
    border-radius: 4px;
    margin: 1rem 0;
  }

  .info, .response {
    background-color: #fff;
    padding: 1rem;
    border-radius: 4px;
    margin: 1rem 0;
    border: 1px solid #dee2e6;
  }

  .logs {
    background-color: #1e1e1e;
    color: #fff;
    padding: 1rem;
    border-radius: 4px;
    margin: 1rem 0;
  }

  .logs-container {
    max-height: 400px;
    overflow-y: auto;
    font-family: monospace;
  }

  .log-entry {
    margin: 0.5rem 0;
    padding: 0.5rem;
    border-radius: 2px;
  }

  .log-entry.error {
    color: #ff5555;
  }

  .reservation {
    display: flex;
    gap: 1rem;
    margin: 1rem 0;
  }

  input {
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    flex: 1;
  }

  button {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
    white-space: nowrap;
  }

  button:hover {
    background-color: #0056b3;
  }

  pre {
    white-space: pre-wrap;
    word-wrap: break-word;
    margin: 0;
  }

  @media (max-width: 768px) {
    .split-layout {
      grid-template-columns: 1fr;
    }
  }
</style> 