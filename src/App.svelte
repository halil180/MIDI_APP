<script lang="ts">
  import * as Tonal from 'tonal';

  import { onMount } from "svelte";
  window.AudioContext = window.AudioContext || window.webkitAudioContext;
  let ctx;
  let data;
  let deviceName;
  const oscillators = {};

  function midiToFreq(number: number) {
    const a = 440;
    return (a / 32) * 2 ** ((number - 9) / 12);
  }
  async function handleMIDIInput() {
    if (!navigator.requestMIDIAccess) {
      return console.error("Web MIDI API is not supported in this browser");
    }

    const midi = await navigator.requestMIDIAccess();

    midi.onstatechange = (event) => {
      const port = event.port;
      console.log(
        `${port.type} port [${port.manufacturer} ${port.name}] ${port.state}`
      );

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
        const command = data[0];
        const note = data[1];
        const velocity = data[2];
        // console.log(data);
        console.log("command", command, "note", note, "velocity", velocity);
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

  const handleStart = () => {
    ctx = new AudioContext();
    console.log(ctx);
  };

  function noteOn(note, velocity) {
    const osc = ctx.createOscillator();
    oscillators[note.toString()] = osc;
    // console.log(oscillators);
    
    const oscGain = ctx.createGain();
    oscGain.gain.value = 0.02;

    const velocityGainAmount = (1/ 127) * velocity;
    const velocityGain = ctx.createGain();
    velocityGain.gain.value = velocityGainAmount;

    osc.type = "sine";
    osc.frequency.value = midiToFreq(note);

    osc.connect(oscGain);
    oscGain.connect(velocityGain);
    velocityGain.connect(ctx.destination);
    osc.gain = oscGain;
    oscillators[note.toString()] =osc
    osc.start();
    const key = Tonal.Note.fromMidi(note);
  console.log(`Key: ${key}`);
  }

  function noteOff(note) {
    const  osc = oscillators[note.toString()];
    const oscGain = osc.gain;

    oscGain.gain.setValueAtTime(oscGain.gain.value, ctx.currentTime);
    oscGain.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + 0.03); 

    setTimeout(() => {
      osc.stop();
      osc.disconnect();

    }, 10);

   delete oscillators[note.toString()]

    console.log(oscillators);
    
    
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

<button on:click={handleStart}>Start CTX</button>
