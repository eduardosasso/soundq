<script>
  import inRange from "lodash-es/inRange";

  // get reference of the canvas elemen from the DOM
  let canvas;
  $: if (canvas) start();

  const bars = 6;
  const barGap = 40;
  const barWidth = 100;
  const barColor = "#fff";

  const frequencySize = 256;
  const maxAmplitude = 255;

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
    // canvasContext.fillStyle = barColor;
    // canvasContext.scale(window.devicePixelRatio, window.devicePixelRatio);
    // canvasContext.clearRect(0, 0, canvas.width, canvas.height);

    // discard frequencies with no amplitude
    const frequencies = dataArray.filter((freq) => freq > 0);
    const frequenciesPerBar = frequencies.length / bars;

    let barPosXx = 0;
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
        // barPosXx = barPosX(bar);

        // get the amplitude of the frequency times number to make it bigger
        // barHeightY = frequencies[index] * 3;

        stereoClassic(bar, frequencies[index]);

        break;
      }

      // const x = barPosXx;
      // const y = canvas.height - barHeightY;
      // const width = barWidth;
      // const height = barHeightY;

      // canvasContext.fillStyle = barColor;
      // canvasContext.fillRect(x, y, width, height);
    });
  };

  const stereoClassic = (soundBar = null, amplitude = null) => {
    const canvasContext = canvas.getContext("2d");

    const levelHeight = 20;
    const levelGap = 15;
    const soundLevels = Math.floor((canvas.height * 0.3) / levelHeight);
    const filledLevels = Math.round((amplitude * soundLevels) / maxAmplitude);

    const yStart = canvas.height / 2; // half of the canvas height
    const width = barWidth;
    const height = levelHeight;

    for (let bar = 0; bar < bars; bar++) {
      if (soundBar !== null && soundBar !== bar) continue;

      const x = barPosX(bar);

      for (let level = 0; level < soundLevels; level++) {
        const y = yStart - (levelHeight + levelGap) * level;

        canvasContext.fillStyle = "#141414";

        if (filledLevels > 0 && level <= filledLevels) {
          const ampRange = (level / soundLevels) * 100;

          const color =
            hiAmpColor(ampRange) ||
            midAmpColor(ampRange) ||
            lowAmpColor(ampRange);

          canvasContext.fillStyle = color;
        }

        canvasContext.fillRect(x, y, width, height);
      }
    }
  };

  const hiAmpColor = (value) => {
    return inRange(value, 81, 100) ? "#FF4136" : null;
  };

  const midAmpColor = (value) => {
    // return inRange(value, 56, 80) ? "#FFDC00" : null;
    return inRange(value, 56, 80) ? "#01FF70" : null;
  };

  const lowAmpColor = (value) => {
    return inRange(value, 0, 55) ? "#01FF70" : null;
  };

  // find side space in the canvas to center the bars on the screen
  const barLeftRightGap = () => {
    return (canvas.width - (barWidth * bars + barGap * (bars - 1))) / 2;
  };

  const barPosX = (bar) => {
    return barLeftRightGap() + (barWidth + barGap) * bar;
  };
</script>

<!-- TODO height vh in tailwind, detect landscape change height, also affect level height maybe-->
<main class="bg-black">
  <div class="min-h-screen flex items-center justify-center flex-col">
    <canvas bind:this={canvas} style="height: 50vh;" class="w-full " />
  </div>
</main>
