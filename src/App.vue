<script setup lang="ts">
import { ref, onMounted } from "vue";
import {
  Renderer,
  Stave,
  StaveNote,
  Formatter,
  TabStave,
  TabNote,
} from "vexflow";
import * as Pitchfinder from "pitchfinder";

const isMicActive = ref(false);
const errorMsg = ref("");
const detectedNote = ref("");
let audioContext: AudioContext | null = null;
let analyser: AnalyserNode | null = null;
let source: MediaStreamAudioSourceNode | null = null;

const noteNames = [
  "C",
  "C#",
  "D",
  "D#",
  "E",
  "F",
  "F#",
  "G",
  "G#",
  "A",
  "A#",
  "B",
];

const standardTuning = [
  { note: "E2", string: 6, fret: 0 },
  { note: "A2", string: 5, fret: 0 },
  { note: "D3", string: 4, fret: 0 },
  { note: "G3", string: 3, fret: 0 },
  { note: "B3", string: 2, fret: 0 },
  { note: "E4", string: 1, fret: 0 },
];

const getNoteName = (frequency: number | null | undefined): string => {
  if (!frequency || isNaN(frequency)) return "";
  const A4 = 440;
  const semitones = 12 * Math.log2(frequency / A4);
  const noteIndex = Math.round(semitones) + 9; // 9 is A
  const octave = 4 + Math.floor(noteIndex / 12);
  const name = noteNames[(noteIndex + 12 * 10) % 12]; // avoid negative
  return `${name}${octave}`;
};

const getNoteKey = (note: string): string => {
  // Converts "A4" to "a/4", "G#5" to "g#/5", etc.
  if (!note) return "";
  const match = note.match(/^([A-G]#?)(\d)$/);
  if (!match) return "";
  return `${match[1].toLowerCase()}/${match[2]}`;
};

type TabPosition = { string: number; fret: number } | null;

const noteToTab = (note: string): TabPosition => {
  if (!note) return null;
  const match = note.match(/^([A-G]#?)(\d)$/);
  if (!match) return null;
  const noteName = match[1];
  const octave = parseInt(match[2], 10);

  // Calculate MIDI number for the note
  const midiNumber = (octave + 1) * 12 + noteNames.indexOf(noteName);

  // Find the best string/fret combination
  let best: TabPosition = null;
  for (const open of standardTuning) {
    const openNoteName = open.note.slice(0, -1);
    const openOctave = parseInt(open.note.slice(-1), 10);
    const openMidi = (openOctave + 1) * 12 + noteNames.indexOf(openNoteName);
    const fret = midiNumber - openMidi;
    if (fret >= 0 && fret <= 20) {
      if (!best || fret < best.fret) {
        best = { string: open.string, fret };
      }
    }
  }
  return best;
};

const drawSheet = (noteKey: string) => {
  if (!noteKey) return;
  const container = document.getElementById("sheet");
  if (!container) return;
  container.innerHTML = "";
  const renderer = new Renderer(
    container as HTMLDivElement,
    Renderer.Backends.SVG
  );
  renderer.resize(400, 200);
  const context = renderer.getContext();
  const stave = new Stave(10, 40, 380);
  stave.addClef("treble").setContext(context).draw();

  const note = new StaveNote({
    keys: [noteKey],
    duration: "q",
  });
  note.setStyle({ fillStyle: "green" });
  const notes = [note];
  Formatter.FormatAndDraw(context, stave, notes);
};

const drawTablature = (note: string) => {
  if (!note) return;
  const tabContainer = document.getElementById("tab");
  if (!tabContainer) return;
  tabContainer.innerHTML = "";
  const renderer = new Renderer(
    tabContainer as HTMLDivElement,
    Renderer.Backends.SVG
  );
  renderer.resize(400, 200);
  const context = renderer.getContext();
  const tabStave = new TabStave(10, 40, 380);
  tabStave.addTabGlyph().setContext(context).draw();

  const tabPos = noteToTab(note);
  if (!tabPos) return; // Don't draw if mapping fails

  const tabNotes = [
    new TabNote({
      positions: [{ str: tabPos.string, fret: tabPos.fret }],
      duration: "q",
    }),
  ];
  Formatter.FormatAndDraw(context, tabStave, tabNotes);
};

onMounted(() => {
  drawSheet("c/4");
  drawTablature("C4");
});

const requestMicAccess = async () => {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    isMicActive.value = true;
    audioContext = new window.AudioContext();
    analyser = audioContext.createAnalyser();
    source = audioContext.createMediaStreamSource(stream);
    source.connect(analyser);

    const detectPitch = Pitchfinder.AMDF();
    const buffer = new Float32Array(analyser.fftSize);

    const listen = () => {
      analyser?.getFloatTimeDomainData(buffer);
      const pitch = detectPitch(buffer);
      const note = getNoteName(pitch);

      detectedNote.value = note;
      const noteKey = getNoteKey(note);
      if (noteKey) {
        drawSheet(noteKey);
        drawTablature(note);
      }
      requestAnimationFrame(listen);
    };
    listen();
  } catch (err) {
    errorMsg.value = "Microphone access denied or unavailable.";
    isMicActive.value = false;
  }
};
</script>

<template>
  <div
    class="flex flex-col items-center justify-center min-h-screen bg-gray-100"
  >
    <div>
      <h2 class="flex justify-center text-3xl mb-4">Guitar to Sheet Note</h2>
    </div>
    <div class="flex flex-row justify-center gap-8 my-8">
      <div id="sheet"></div>
      <div id="tab"></div>
    </div>
    <div class="flex justify-center mt-4">
      <button
        @click="requestMicAccess"
        class="p-10 bg-blue-500 text-white rounded hover:bg-blue-600 focus:outline-none"
      >
        Access Microphone
      </button>
    </div>
    <div v-if="isMicActive" class="flex justify-center mt-2 text-green-600">
      Microphone is active!
    </div>
    <div v-if="errorMsg" class="flex justify-center mt-2 text-red-600">
      {{ errorMsg }}
    </div>
  </div>
</template>

<style scoped>
#sheet {
  width: 400px;
  height: 300px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: center;
  align-items: center;
}
#tab {
  width: 400px;
  height: 300px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
