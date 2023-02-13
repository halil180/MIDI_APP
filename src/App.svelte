<script>
  import { onMount } from "svelte";


  let data;
  let deviceName;

  async function handleMIDIInput() {
    if (!navigator.requestMIDIAccess) {
      return console.error("Web MIDI API is not supported in this browser");
    }

    const midi = await navigator.requestMIDIAccess();

    midi.onstatechange = (event) => {
      const port = event.port;
      console.log(`${port.type} port [${port.manufacturer} ${port.name}] ${port.state}`);

      if (port.state === "connected") {
        deviceName = `${port.manufacturer} ${port.name}`;
      } else {
        deviceName = null;
      }
    };

    const inputs = midi.inputs.values();

    for (const input of inputs) {
      input.onmidimessage = (event) => {
        data = event.data;
        const command = data[0]
        const note = data[1]
        const velocity = data[2]
        // console.log(data);
        console.log('command', command,'note', note,'velocity', velocity)
        switch (command) {
  case 144:
    if (velocity > 0) {
      //note is on
      noteOn(note, velocity);
    } else {
      //note is off
      noteOff(note);
    }
    break;
  case 146:
    if (velocity > 0) {
      //note is on
      noteOn(note, velocity);
    }
    break;
  case 130:
    //note is off
    noteOff(note);
    break;
  case 128:
    //note is off
    noteOff(note);
    break;
  default:
    break;
}

      };
    }
  }
 function noteOn(note,velocity){
  console.log(note,velocity );

  }
  function noteOff(note){
  console.log(note );

  }
  onMount(async () => {
    await handleMIDIInput();
  });
</script>

{#if deviceName}
  <p>Connected to: {deviceName}</p>
{:else}
  <p>Please connect a MIDI device</p>
{/if}
