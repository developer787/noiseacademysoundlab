<template>
  <div>
    <h1>{{ msg }}</h1>
    <canvas width="300" height="150" style="border:1px solid #BBB;" v-canvas></canvas>
    <div class="DrumMachine">
      <div class="Pad" @click="kick"></div>
      <div class="Pad"></div>
      <div class="Pad"></div>
      <div class="Pad"></div>
    </div>

    <button @click="playOscillator">Play</button>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {
      ctx: window.AudioContext ? new AudioContext() : new webkitAudioContext(),
      playing: false,
      oscillator: null,
      filter: null,
      distortion: null,
      gainNode: null
    };
  },
  mounted() {
    this.gainNode = this.ctx.createGain();
    this.distortion = this.ctx.createWaveShaper();
    this.filter = this.ctx.createBiquadFilter();
    this.analyserNode = this.ctx.createAnalyser();
    this.oscillator = this.ctx.createOscillator();
    this.oscillator.start();
  },
  methods: {
    playOscillator() {
      console.log("Playing " + this.playing);

      this.distortion.curve = this.distortionCurve(100);
      this.distortion.oversample = "4x";

      this.filter.type = "lowpass";
      this.filter.frequency.value = 100;

      this.oscillator.type = "sine";
      this.oscillator.frequency.setValueAtTime(110, this.ctx.currentTime); // value in hertz

      this.oscillator.connect(this.filter);
      this.filter.connect(this.distortion);
      this.distortion.connect(this.gainNode);

      this.gainNode.connect(this.ctx.destination);
      if (this.playing) {
        this.playing = !this.playing;
        this.gainNode.gain.setValueAtTime(0, this.ctx.currentTime);
      } else {
        this.playing = !this.playing;
        this.gainNode.gain.setValueAtTime(0.7, this.ctx.currentTime);
      }
    },
    kick() {
      console.log("Kick");
    },
    snare() {
      console.log("Snare");
    },
    distortionCurve(amount) {
  var k = typeof amount === "number" ? amount : 50,
    n_samples = 44100,
    curve = new Float32Array(n_samples),
    deg = Math.PI / 180,
    i = 0,
    x;
  for (; i < n_samples; ++i) {
    x = (i * 2) / n_samples - 1;
    curve[i] = ((3 + k) * x * 20 * deg) / (Math.PI + k * Math.abs(x));
  }
  return curve;
}
  },
  directives: {
    canvas: (canvasElement, binding, vnode) => {
      // Get canvas context
      
      var ctx = canvasElement.getContext("2d");
      // Clear the canvas
      ctx.clearRect(0, 0, 300, 150);
      // Insert stuff into canvas
      ctx.fillStyle = "black";
      ctx.font = "20px Georgia";
      ctx.fillText("test", 10, 50);
    

    }
  }
};

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.DrumMachine {
  display: flex;
  justify-content: center;
  .Pad {
    height: 100px;
    width: 100px;
    background: grey;
    margin: 10px;
    border-radius: 5px;
  }
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
