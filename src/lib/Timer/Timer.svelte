<script>
  let timer;
  // let view = {
  //   min: "00",
  //   sec: "00",
  //   msec: 0,
  // };

  let msec = 0,
    sec = 0,
    min = 0;

  export const start = () => {
    let mseconds = 0;
    let minutes = 0;
    let seconds = 0;

    timer = setInterval(() => {
      mseconds++;
      msec = mseconds % 10;

      if (msec == 0) {
        seconds++;
        sec = seconds % 60;

        if (sec == 0) {
          minutes++;
          min = minutes % 60;
        }
      }
    }, 100);
  };

  export const stop = () => {
    clearInterval(timer);
  };

  export const reset = () => {
    msec = 0;
    sec = 0;
    min = 0;
  };
</script>

<slot {min} {sec} {msec} />
