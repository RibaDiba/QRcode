<script>
    import jsQR from 'jsqr';
    import axios from 'axios';
      
    let videoElement;
    let canvasElement;

    let displayData = '';
    let ableToScan = false;
    let showVideo = false;
    let getData = false;

    let numCount = 2

    function toggle() {
        getData = !getData
    }
  
    async function startCamera() {
    
        showVideo = true;

      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        videoElement.srcObject = stream;
  
        videoElement.onloadedmetadata = () => {
          videoElement.play();
          ableToScan = true;
        };
      } catch (error) {
        console.error('Error accessing camera:', error);
      }
    }
  
    function scanQRCode() {
        if (ableToScan) {
            const canvas = canvasElement.getContext('2d');
  
            const checkForQRCode = () => {
            canvas.drawImage(videoElement, 0, 0, canvasElement.width, canvasElement.height);
  
            const imageData = canvas.getImageData(0, 0, canvasElement.width, canvasElement.height);
            const code = jsQR(imageData.data, imageData.width, imageData.height);
  
             if (code) {
                console.log('QR Code Data:', code.data);
                displayData = code.data;
            }
  
            requestAnimationFrame(checkForQRCode);
            };
  
            checkForQRCode();
        }
    }

    function postQRData() {
      axios.post('https://sheetdb.io/api/v1/efby1mvgfl6qj', {
        Count: numCount+1,
        Data: displayData
      })
    }
  </script>
  
  <style>
    .video {
      width: 100%;
      max-width: 640px;
      border: 4px solid #03ADC5;
      border-radius: 10px;
      margin: 50px;
      margin-bottom: 25px;
    }
    .center {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
    }
    .buttons {
        margin: 50px;
    }
    button {
        margin: 10px;
    }
    .start-camera {
        font-size: 30px;
        padding: 10px;
        background-color: white;
        border-radius: 10px;
        border: 0px;
    }
    .scan-button {
        font-size: 30px;
        padding: 10px;
        background-color: #03ADC5;
        border-radius: 10px;
        border: 0px;
    }
    .get-data {
        font-size: 30px;
        padding: 10px;
        background-color: #BB86FC;
        border-radius: 10px;
        border: 0px;
    }
    .data { 
        color: white;
        padding: 20px;
        margin: 10px;
        border: 3px solid yellow;
        border-radius: 15px;
        font-size: 25px;
        justify-content: center;
        text-align: center;
        font-family: 'Courier New', Courier, monospace;
    }
  </style>
  
  <main class="center">
    <!-- svelte-ignore a11y-media-has-caption -->
    {#if showVideo}
    <video bind:this={videoElement} class="video"></video>
    <canvas bind:this={canvasElement} style="display:none;"></canvas>
    {/if}

    <div class="buttons">
        <button on:click={startCamera} class="start-camera">Start Camera</button>
        <button on:click={scanQRCode} class="scan-button">Scan for QR Code</button>
        {#if displayData != ''}
            <button on:click={toggle} class="get-data">Get Data</button>
        {/if}

        {#if getData}
            <h1 class="data">QR Code Data: {displayData}</h1>
            <button class="start-camera">Send Data!</button>
        {/if}
    </div>

  </main>