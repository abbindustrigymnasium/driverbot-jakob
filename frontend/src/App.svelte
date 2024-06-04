<script>
	// Importing content and defining
	import { onMount } from "svelte";
	import mqtt from "mqtt";
  
	let client;
	let isConnected = false;
	let errorMessage = '';
  
	// Connecting and disconnecting to mqtt
	function connectMQTT() {
	  client = mqtt.connect("mqtt://maqiatto.com:8883", {
		username: "jakob.geffen@hitachigymnasiet.se",
		password: "qGyu2H2@yWDQFuc"
	  });
  
	  client.on("connect", () => {
		console.log("Connected to MQTT broker");
		isConnected = true;
		errorMessage = '';
	  });
  
	  client.on("error", (error) => {
		console.error("Connection error: ", error);
		isConnected = false;
		errorMessage = `Connection error: ${error.message}`;
	  });
  
	  client.on("close", () => {
		console.log("Disconnected from MQTT broker");
		isConnected = false;
	  });
	}
  
	function disconnectMQTT() {
	  if (client) {
		client.end();
		isConnected = false;
		console.log("Disconnected from MQTT broker");
		errorMessage = '';
	  }
	}
  
	// Publish if connected, otherwise send error message
	function sendMessage(command) {
	  if (isConnected) {
		client.publish("jakob.geffen@hitachigymnasiet.se/driverbot", command);
	  } else {
		console.error("Not connected to MQTT broker");
		errorMessage = "Not connected to MQTT broker";
	  }
	}
  
	// Allows for the use of keys to control the driverbot
	function handleKeydown(event) {
	  switch (event.key) {
		case 'w':
		  sendMessage("forward");
		  break;
		case 'a':
		  sendMessage("left");
		  break;
		case 's':
		  sendMessage("reverse");
		  break;
		case 'd':
		  sendMessage("right");
		  break;
		case ' ':
		  sendMessage("stop");
		  break;
		case 'c':
		  sendMessage("center");
		  break;
		default:
		  break;
	  }
	}
  
	onMount(() => {
	  window.addEventListener("keydown", handleKeydown);
  
	  return () => {
		window.removeEventListener("keydown", handleKeydown);
	  };
	});
  </script>
  
  <main>
	<h1>Driverbot Control Panel</h1>
	<button on:click={isConnected ? disconnectMQTT : connectMQTT} class="connect-btn">{isConnected ? "Disconnect MQTT" : "Connect MQTT"}</button>
	{#if errorMessage}
	  <p style="color: red;">{errorMessage}</p>
	{/if}
	<div class="button-grid">
	  <button on:click={() => sendMessage("forward")}>Forward</button>
	  <button on:click={() => sendMessage("left")}>Left</button>
	  <button on:click={() => sendMessage("stop")}>Stop</button>
	  <button on:click={() => sendMessage("right")}>Right</button>
	  <button on:click={() => sendMessage("reverse")}>Reverse</button>
	  <button on:click={() => sendMessage("center")}>Center</button>
	</div>
  </main>
  
  <style>
	main {
	  text-align: center;
	  padding: 1em;
	  max-width: 240px;
	  margin: 0 auto;
	}

	.button-grid {
	  display: grid;
	  grid-template-areas:
		"forward forward forward"
		"left center right"
		"stop stop stop"
		"reverse reverse reverse";
	  gap: 10px;
	  justify-items: center;
	}
  
	button {
	  padding: 0.5em 1em;
	  font-size: 1em;
	}
  
	button:nth-child(1) {
	  grid-area: forward;
	}
  
	button:nth-child(2) {
	  grid-area: left;
	}
  
	button:nth-child(3) {
	  grid-area: stop;
	}
  
	button:nth-child(4) {
	  grid-area: right;
	}
  
	button:nth-child(5) {
	  grid-area: reverse;
	}
  
	button:nth-child(6) {
	  grid-area: center;
	}
  
	.connect-btn {
	  position: absolute;
	  top: 10px;
	  left: 10px;
	}
  </style>
  