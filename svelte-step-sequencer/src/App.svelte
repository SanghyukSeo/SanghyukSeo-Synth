<script>
  import * as Tone from 'tone';
  let bpm = 120;
  let beat = 0;
  let isPlaying = false;

  const synths = [
    new Tone.Synth().toDestination(),
    new Tone.Synth().toDestination(),
    new Tone.Synth().toDestination(),
    new Tone.Synth().toDestination()
];

// Dorian scale notes
const scaleOfNotes = ["C4", "D4", "E4", "F4"];
 
let rows = [
  Array.from({length:16},(_,i) =>  ({note: scaleOfNotes[3], active: false})),
  Array.from({length:16},(_,i) =>  ({note: scaleOfNotes[2], active: false})),
  Array.from({length:16},(_,i) =>  ({note: scaleOfNotes[1], active: false})),
  Array.from({length:16},(_,i) =>  ({note: scaleOfNotes[0], active: false})),
]

let  beatIndicators = Array.from({length:16},(_,i) =>  i );

Tone.Transport.scheduleRepeat(time => {
    rows.forEach((row, index) => {
    let synth = synths[index];
    let note = row[beat];
    if (note.active)  synth.triggerAttackRelease(note.note, "16n", time);
  });
  beat= (beat +1) % 16; 
}, "16n");

const handleNoteClick = (rowIndex, noteIndex) => {
  rows[rowIndex][noteIndex].active = !rows[rowIndex][noteIndex].active;
  rows = [...rows];
};

const handlePlayClick = () => {
  if(!isPlaying) Tone.start();
  Tone.Transport.bpm.value = bpm;
  Tone.Transport.start();
  isPlaying = true;
};

const handleStopClick = () => {
  Tone.Transport.stop();
  isPlaying = false;
};

$: if (isPlaying) {
  Tone.Transport.bpm.value = bpm;
}
    /*
    kick: new Tone.MembraneSynth().toDestination(),
    snare: new Tone.NoiseSynth().toDestination(),
    hihat: new Tone.NoiseSynth().toDestination()
    */
  
</script>

<div class="bpm-controls">
  <label for="bpm">{bpm} BPM</label>
  <input type="range" id="bpm" min="60" max="240" bind:value={bpm} />
 {#if isPlaying}
    <button on:click={handleStopClick}>Stop</button>
  {:else}
    <button on:click={handlePlayClick}>Play</button>
  {/if}
</div>

<div class= "sequencer">
{#each beatIndicators as beatIndicator, bi}
  <div class="beat-indicator {bi === beat -1 ? 'live': '' }"></div>
{/each}

  {#each rows as row, i}
  {#each row as note, j}
      <!-- svelte-ignore a11y_consider_explicit_label -->
      <button 
      on:click={() => handleNoteClick(i, j)}
      class="note {note.active ? 'active' : ' '} {j % 4 === 0 ? 'first-beat-of-the-bar':''}" ></button>
    {/each}
  {/each}
</div>

<style>

  .bpm-controls {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
  }
  .bpm-controls label {
    color:white;
  }

  .beat-indicator {
    width: 100%; 
    height: 5px;
    background: #3A3A3A58;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #fff;
    font-size: 1.5rem;
    margin: 0 auto;
    margin-bottom: 10px;
  }
  .live{
background: rgb(45, 255, 132);
  }
  
  .sequencer {
    display: grid;
    grid-template-columns: repeat(16, 1fr);
    grid-gap: 5px;
    width: 100%;
  }

  .note {
    background: #3A3A3A58;
    width: 50px;
    height: 50px;
    /*border: 1px solid #757575FF;*/
    border-radius: 0px; /* note border*/
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2rem;
  }

  .note.active{
    background: #D9D9D9FF;
    /*border: 1px solid #CACACAFF;*/
    border-radius: 0px;
    color: #CACACAFF;
  }
  
  .first-beat-of-the-bar{
    background: #53535358;
   /* border: 1px solid #A2A2A2FF;*/
  }

</style>