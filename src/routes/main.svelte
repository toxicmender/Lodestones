<!-- src/routes/index.svelte -->
<script>
    import { onMount } from 'svelte';
    import Map from '$lib/components/Map.svelte';
    import { trackPhoneNumber } from '$lib/services/geoService';
    
    /** @type {string} */
    let phoneNumber = '';
    
    /** @type {import('$lib/types').GeoLocation|null} */
    let location = null;
    
    /** @type {boolean} */
    let loading = false;
    
    /** @type {string|null} */
    let errorMessage = null;
    
    /**
     * Handle form submission to track a phone number
     * @param {Event} event
     */
    async function handleSubmit(event) {
      event.preventDefault();
      
      if (!phoneNumber) {
        errorMessage = 'Please enter a phone number';
        return;
      }
      
      loading = true;
      errorMessage = null;
      
      try {
        location = await trackPhoneNumber(phoneNumber);
      } catch (error) {
        errorMessage = error.message || 'Failed to track phone number';
        location = null;
      } finally {
        loading = false;
      }
    }
  </script>
  
  <svelte:head>
    <title>Phone Number Location Tracker</title>
  </svelte:head>
  
  <main>
    <h1>Phone Number Location Tracker</h1>
    
    <form on:submit={handleSubmit}>
      <div>
        <label for="phoneNumber">Enter Phone Number:</label>
        <input 
          id="phoneNumber" 
          type="tel" 
          bind:value={phoneNumber} 
          placeholder="+1234567890"
          required
        />
      </div>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Tracking...' : 'Track Location'}
      </button>
    </form>
    
    {#if errorMessage}
      <div class="error">
        {errorMessage}
      </div>
    {/if}
    
    {#if location}
      <div class="location-info">
        <h2>Location Information</h2>
        <p><strong>Country:</strong> {location.country}</p>
        <p><strong>City:</strong> {location.city}</p>
        <p><strong>Carrier:</strong> {location.carrier}</p>
        <p><strong>Coordinates:</strong> {location.latitude}, {location.longitude}</p>
      </div>
    {/if}
    
    <div class="map-container">
      <Map {location} />
    </div>
  </main>
  
  <style>
    main {
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
    }
    
    form {
      margin-bottom: 20px;
    }
    
    input {
      padding: 8px;
      width: 100%;
      max-width: 300px;
    }
    
    button {
      margin-top: 10px;
      padding: 8px 16px;
      background-color: #4CAF50;
      color: white;
      border: none;
      cursor: pointer;
    }
    
    button:disabled {
      background-color: #cccccc;
    }
    
    .error {
      color: red;
      margin-bottom: 20px;
    }
    
    .location-info {
      margin-bottom: 20px;
    }
    
    .map-container {
      margin-top: 20px;
      border: 1px solid #ccc;
    }
  </style>
  