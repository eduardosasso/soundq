<script>
  import inRange from "lodash-es/inRange";

  // if added to home screen reload page if it was hidden
  // window.addEventListener("visibilitychange", () => {
  //   location.reload();
  // });

  // get reference of the canvas elemen from the DOM
  let canvas;
  $: if (canvas) start();

  const bars = 6;
  const barGap = 40;
  const barWidth = 100;
  const barColor = "#fff";

  const frequencySize = 2048;

  const userMediaParams = {
    audio: {
      echoCancellation: true,
      autoGainControl: false,
      noiseSuppression: true,
      latency: 0,
    },
    video: false,
  };

  const audioCtx = new AudioContext();
  const analyser = audioCtx.createAnalyser();

  analyser.connect(audioCtx.destination);
  analyser.fftSize = frequencySize;

  const start = () => {
    canvas.width = canvas.clientWidth * window.devicePixelRatio;
    canvas.height = canvas.clientHeight * window.devicePixelRatio;

    // listen for the microphone to be ready
    navigator.mediaDevices.getUserMedia(userMediaParams).then((stream) => {
      const source = audioCtx.createMediaStreamSource(stream);
      source.connect(analyser);

      visualiser();
    });
  };

  const visualiser = async () => {
    requestAnimationFrame(visualiser);
    if (audioCtx.state === "suspended") await audioCtx.resume();

    const bufferLength = analyser.frequencyBinCount;
    const dataArray = new Uint8Array(bufferLength); // audio samples half of fftsize

    analyser.getByteFrequencyData(dataArray);

    const canvasContext = canvas.getContext("2d");
    canvasContext.fillStyle = barColor;
    // canvasContext.scale(window.devicePixelRatio, window.devicePixelRatio);
    canvasContext.clearRect(0, 0, canvas.width, canvas.height);

    // discard frequencies with no amplitude
    const frequencies = dataArray.filter((freq) => freq > 0);

    // number of frequencies per bar
    const frequenciesPerBar = frequencies.length / bars;

    // find side space in the canvas to center the bars on the screen
    const barLeftRightGap =
      (canvas.width - (barWidth * bars + barGap * (bars - 1))) / 2;

    let barPosX = 0;
    let barHeightY = 0;

    frequencies.forEach((item, index) => {
      barHeightY = 0;

      // create a bucket like structure
      // each bar contains a bucket of frequencies
      for (let bar = 0; bar < bars; bar++) {
        const begin = bar * frequenciesPerBar;
        const end = frequenciesPerBar * (bar + 1) + 1;

        if (!inRange(index, begin, end)) continue;

        // find the correct x pos for each bar
        barPosX = barLeftRightGap + (barWidth + barGap) * bar;

        // get the amplitude of the frequency times number to make it bigger
        barHeightY = frequencies[index] * 3;

        break;
      }

      const x = barPosX;
      const y = canvas.height - barHeightY;
      const width = barWidth;
      const height = barHeightY;

      canvasContext.fillRect(x, y, width, height);
    });
  };
</script>

<main class="bg-black">
  <div class="min-h-screen flex items-center justify-center flex-col">
    <canvas bind:this={canvas} style="height: 50vh;" class="w-full " />
  </div>
</main>
