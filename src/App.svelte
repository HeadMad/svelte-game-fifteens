<script>
  import { connectFirestoreEmulator } from "firebase/firestore";

  import Fifteen from "./lib/Fifteen/Fifteen.svelte";
  import Timer from "./lib/Timer/Timer.svelte";
  let game;
  let timer;
  let level = 3;
  const size = 100 / level;
  function changeLevel(factor) {
    return function () {
      level += factor;
    };
  }
  let src = 'https://i1.sndcdn.com/artworks-000206189640-evm0ik-t500x500.jpg';
  function loadImage() {
    const file = this.files[0];
    const reader = new FileReader();

    
    reader.addEventListener(
      "load",
      function (event) {
        console.log(event);
        // convert image file to base64 string
        src = String(reader.result);
      },
      false
      );
      
      if (file) {
        reader.readAsDataURL(file);
      }
    }

  $: style = `<style>.cell__inner {background-image: url(${src});}</style>`;
</script>

<div>
  <button on:click={changeLevel(1)}>Добавить ячейку</button>
  <button on:click={changeLevel(-1)}>Удалить ячейку</button>
  <button on:click={game.shuffle}>Перемешать</button>
  <br />

  <button on:click={timer.start}>Start</button>
  <button on:click={timer.stop}>Stop</button>
  <Timer bind:this={timer} let:min let:sec let:msec
    >{min < 10 ? "0" + min : min}:{sec < 10 ? "0" + sec : sec}:{msec}</Timer
  >
  <br />
  <input type="file" on:change={loadImage} />
  <br />
</div>

<div class="field">
  <Fifteen
    {level}
    bind:this={game}
    let:index
    let:inPlace
    on:order={() => {
      console.log("Ok!");
    }}
  >
    <div
      class="cell__inner"
      style=" background-position: -{(index % level) * 100}% -{Math.floor(
        index / level
      ) * 100}%"
    >
    </div>
  </Fifteen>
</div>
{@html style}
<style>
  .field {
    width: 90vmin;
    margin: auto;
  }

  .cell__inner {
    position: absolute;
    top: 3px;
    left: 3px;
    right: 3px;
    bottom: 3px;
    border-radius: 5px;
    background-color: #eee;
    cursor: pointer;
    display: flex;
    text-align: center;
    background-size: 90vmin 90vmin;
    
  }
  
  /* .cell__num {
    user-select: none;
    margin: auto;
  } */

  /* :global(.hello) {
    width: 100vmin;
  } */

  /* .cell__inner::before {
    margin: auto;
    counter-increment: cell;
    content: counter(cell);
  } */
</style>
